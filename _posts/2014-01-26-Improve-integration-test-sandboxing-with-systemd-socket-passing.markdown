---
layout: post
title:  "Improve integration test sandboxing with systemd socket passing"
date:   2014-01-26 12:04:21
categories: python integration-testing testing systemd
---

TL;DR version: Improve integration test sandboxing with [systemd socket passing](http://0pointer.de/public/systemd-man/sd_listen_fds.html).  You can allocate a random port for your daemon and don't need to wait for the daemon to start up to run your test.  It's fast, robust, race-free and **doesn't depend on systemd**.

[Julien Danjou](http://julien.danjou.info) recently wrote an excellent blog post on [Database integration testing strategies with Python](http://julien.danjou.info/blog/2014/db-integration-testing-strategies-python).    In it he discusses integration tests which require databases, but the advice is applicable to integration tests which require any external process.  For one of these tests he:

1. Chooses a port to listen on then starts the database server with the port number passed in as configuration.
2. Waits for it to start up by grepping stdout.
3. Runs the test.
4. Tears the database server down.

This is great advice but can be improved upon.  One weakness to this approach is if the port you've selected is already in use your test will fail.  This can be the case if the port you've chosen just happens to be in-use or if you're running multiple tests in parallel.

The solution is to use a random unused port each time you run the test.  UNIX allows you to do this by asking [`bind`](http://linux.die.net/man/2/bind) (Python: [`socket.bind`](http://docs.python.org/2/library/socket.html#socket.socket.bind)) to bind to port 0.  You can then ask [`getsockname()`](http://linux.die.net/man/2/getsockname) (Python: [`socket.getsockname()`](http://docs.python.org/2/library/socket.html#socket.socket.getsockname)) to find out which port was actually used.  But **you can't know which port you're going to bind to before you've bound to it**.  If you choose an unused port at random then later try to bind to it another process may have beaten you to it and it may already be in use.

So one way to do this would be to tell the server you're starting to choose a random port, wait for it to be ready and then find out what port it has chosen.  Maybe this involves grepping through logs or making IPC calls.

I like to use a different technique: open the sockets myself and then pass them to the daemon.  This way I don't need to wait for the daemon to start-up and don't need to inspect it's logs or query it.  I use the [systemd socket passing protocol](http://0pointer.de/public/systemd-man/sd_listen_fds.html) which some daemons support anyway.  This means I open a listening socket, and then start my daemon with the environment variable `LISTEN_FDS=1` to tell it that I am passing a socket to it and that socket is fd #3†.

An example:

    $ # Show that the LISTEN_FDS environment variable is set in the child:
    $ ./sd-popen.py --outfile=env.log env
    LAUNCHED_PORT=32814
    LAUNCHED_PID=18150
    $ grep LISTEN env.log
    LISTEN_PID=18150
    LISTEN_FDS=1

    $ # Show that the socket is open in the spawned process:
    $ ./sd-popen.py sleep 50
    LAUNCHED_PORT=48729
    LAUNCHED_PID=18898
    $ netstat -lp | grep 48729
    tcp        0      0 *:48729                 *:*                     LISTEN      18898/sleep     

`sd-popen` opens a socket, binds it to port 0, spawns the command passed to it with `LISTEN_FDS` and `LISTEN_PID` set and then prints the pid and port to stdout before exiting.

The output from `sd-popen` is compatible with shell so we can write shell scripts like:

    export $(./sd-popen.py my-webserver)
    wget http://localhost:$LISTEN_PORT/foobar.html
    kill ${LAUNCHED_PID}

All without waiting or worries about port clashes.

This is what sd-popen.py looks like:

```python
#!/usr/bin/python
import socket
import sys
import argparse
import subprocess

def main(argv):
    parser = argparse.ArgumentParser()
    parser.add_argument('cmd', help='Command to run')
    parser.add_argument('args', nargs=argparse.REMAINDER,
                        help='Command arguments')
    parser.add_argument('--outfile', default='/dev/null',
                        type=argparse.FileType('w'),
                        help='File to redirect stdout and stderr to')
    args = parser.parse_args(argv[1:])

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('', 0))
    _, port = s.getsockname()
    s.listen(5)

    p = sd_popen([args.cmd] + args.args, [s.fileno()], stdout=args.outfile,
                 stdin=open('/dev/null', 'r'), stderr=subprocess.STDOUT)
    sys.stdout.write('LAUNCHED_PORT=%i\nLAUNCHED_PID=%i\n' % (port, p.pid))
    return 0


def sd_popen(args, sockets, preexec_fn=None, *aargs, **kwargs):
    import os, subprocess
    def remap_ports():
        sockets_ = sockets
        os.environ["LISTEN_PID"] = str(os.getpid())
        os.environ["LISTEN_FDS"] = str(len(sockets_))
        for new_fd in range(3, 1024):
            if len(sockets_) == 0 :
                try:
                    os.close(new_fd)
                except:
                    pass
            else:
                oldfd = sockets_.pop(0)
                if new_fd != oldfd:
                    if new_fd in sockets_:
                        replacement_fd = os.dup(new_fd)
                        sockets_ = [replacement_fd if fd == new_fd else fd
                                    for fd in sockets_]
                    os.dup2(oldfd, new_fd)
        if preexec_fn is not None:
            preexec_fn()
    return subprocess.Popen(args, preexec_fn=remap_ports,
                            close_fds=False, *aargs, **kwargs)

if __name__ == '__main__':
    sys.exit(main(sys.argv))
```

`sd_popen` is essentially an extension to Python's `subprocess.Popen` but allows passing a list of fds to be passed to the client process.  In this example we open a socket in `main()` and pass it to the subprocess before printing the socket and subprocess details.  `sd_popen` is complicated by the `dup2` dance to rearrange the file-descriptor numbers but is itself a fairly generic function for launching programs with the socket passing protocol.

One thing to note: there's nothing non-portable in the above code.  It doesn't depend on systemd and should run fine on any unix system.  It does depend on the daemon having socket passing support but that's [easy to add](http://0pointer.de/blog/projects/socket-activation.html) and there's nothing non-portable about it either.

I've used this technique for writing a [simple test suite](https://github.com/jech/polipo/pull/8) for the [polipo](http://www.pps.univ-paris-diderot.fr/~jch/software/polipo/) caching HTTP proxy.  The tests are written in a combination of [shell](https://github.com/wmanley/polipo/blob/b2db672cc3ceead0d32b38bd1ed536163cc26bd8/test/run-test.sh) and [C](https://github.com/wmanley/polipo/blob/b2db672cc3ceead0d32b38bd1ed536163cc26bd8/test/sd-launch.c).  I also use it in my prototype [http-dbus-bridge](https://github.com/wmanley/http-dbus-bridge/blob/master/test.sh) which is a combination of shell and Python.

At the end of Julien Danjou's blog post he writes:

> To speed up tests run, you could also run the test in parallel. It can be interesting as you'll be able to spread the workload among a lot of different CPUs. However, note that it can require a different database for each test or a locking mechanism to be in place. It's likely that your tests won't be able to work altogether at the same time on only one database.

I say - use the [`LISTEN_FDS`/`LISTEN_PID` socket passing protocol](http://0pointer.de/public/systemd-man/sd_listen_fds.html).

†: fd 3 is the next one after stdin (0), stdout (1) and stderr (2).
