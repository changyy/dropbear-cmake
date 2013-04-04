dropbear-cmake
==============
<pre>
$ git clone https://github.com/changyy/dropbear-cmake.git
$ cd dropbear-cmake
$ mkdir build
$ cd build
$ cmake .. -D_PATH_SSH_PROGRAM=/tmp/your/dbclient
$ make
$ ./bin/scp
Use ssh_program: /tmp/your/dbclient
usage: scp [-1246BCpqrv] [-c cipher] [-F ssh_config] [-i identity_file]
</pre>
