## MySQL Debug Build Docker Image ##

[![Build Status](https://travis-ci.org/y-asaba/docker-mysql-debug.svg?branch=master)](https://travis-ci.org/y-asaba/docker-mysql-debug)

This Dockerfile builds MySQL database server with debugging information.

**Please do not use this image in production envrionment because mysqld is compiled without optimization (-O0 option).**

### How to Build ###

#### MySQL 8.0 ####

```
$ docker build -t mysql-debug-8.0 8.0
```

#### MySQL 5.7 ####

```
$ docker build -t mysql-debug-5.7 5.7
```

#### MySQL 5.6 ####

```
$ docker build -t mysql-debug-5.6 5.6
```

#### Optional

Setting the target Version

```
$ docker build --build-arg MYSQL_VERSION=5.7.21 -t mysql-debug-5.7 5.7
```


### How To Remote Debug on MacOS ###

#### Install GDB which recognizes GNU/Linux ABI. ####
You need to install GDB which recognizes GNU/Linux ABI. You can install GDB with the following step.

```
$ brew tap homebrew/dupes
$ brew install gdb --with-all-target
```

You can confirm supported ABIs with `set osabi` command.

```
$ gdb
(gdb) set osabi
Requires an argument. Valid arguments are auto, default, none, GNU/Linux, Symbian, NetBSD/a.out, NetBSD/ELF, OpenBSD/ELF, WindowsCE, Cygwin, FreeBSD/a.out, FreeBSD/ELF, GNU/Hurd, QNX-Neutrino, Solaris, SVR4, DJGPP, DICOS, Darwin, SDE, AIX, LynxOS178, Newlib, OpenVMS.
(gdb) q
```

#### Docker run ####

```
$ mkdir -p ~/src
$ cd ~/src
$ wget http://ftp.jaist.ac.jp/pub/mysql/Downloads/MySQL-5.7/mysql-boost-5.7.20.tar.gz
$ tar zxf mysql-boost-5.7.20.tar.gz
$ docker run -v ~/src/mysql-5.7.20:/mysql-debug/mysql-5.7.20 -p 3306:3306 -p 2345:2345 -t --cap-add=SYS_PTRACE --security-opt seccomp=unconfined mysql-debug-5.7 /bin/bash -c 'cd /mysql-debug; cp /mysql-debug/bin/mysqld /mysql-debug/mysql-5.7.20; sudo gdbserver :2345 ./bin/mysqld --user=root'
```

#### Remote debug with GDB ####
The following example is to trace ALTER TABLE statement.

1. Load symbols
2. Set source code path to `~/src/mysql-5.7.20`
3. Set breakpoint
4. Attach the mysqld process remotely

```
$ gdb
(gdb) file ~/src/mysql-5.7.20/mysqld
Reading symbols from ~/src/mysql-5.7.20/mysqld...done.
(gdb) set substitute-path /root/mysql-5.7.20 ~/src/mysql-5.7.20
(gdb) b mysql_alter_table
Breakpoint 1 at 0x15d88c8: file /mysql-debug/mysql-5.7.20/sql/sql_table.cc, line 8955.
(gdb) target remote :2345
Remote debugging using :2345
Reading /lib64/ld-linux-x86-64.so.2 from remote target...
...
(gdb) c
Continuing.
...

Thread 3 hit Breakpoint 1, mysql_alter_table (thd=0x7fffb4289b20, 
    new_db=0x7fffb4044c38 "hoge", new_name=0x0, 
    create_info=0x7fffec4b6530, table_list=0x7fffb40446b0, 
    alter_info=0x7fffec4b6480)
    at /mysql-debug/mysql-5.7.20/sql/sql_table.cc:8955
8955	/mysql-debug/mysql-5.7.20/sql/sql_table.cc: No such file or directory.
(gdb) bt
#0  mysql_alter_table (thd=0x7fffb4289b20, new_db=0x7fffb4044c38 "hoge", 
    new_name=0x0, create_info=0x7fffec4b6530, table_list=0x7fffb40446b0, 
    alter_info=0x7fffec4b6480)
    at /mysql-debug/mysql-5.7.20/sql/sql_table.cc:8955
    ...
```

Optional: Connecting Docker container

```
$ docker ps | grep mysql # search of container id
$ docker exec -it ${DOCKER_CONTAINER_ID} bash
```

#### mysql command on MacOS ####
To connect to the MySQL server, you can connect to 127.0.0.1:3306 with the mysql command.

```
$ mysql -h 127.0.0.1 -uroot
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.20-debug Source distribution
...
mysql>
```
