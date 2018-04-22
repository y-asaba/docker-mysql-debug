FROM ubuntu:trusty as builder

ARG MYSQL_VERSION=5.6.40

WORKDIR /root

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install build-essential cmake wget curl libncurses5-dev -y -o Dpkg::Options::='--force-confdef' -o Dpkg::Options::='--force-confold' && \
    wget http://ftp.jaist.ac.jp/pub/mysql/Downloads/MySQL-5.6/mysql-${MYSQL_VERSION}.tar.gz && \
    tar zxf mysql-${MYSQL_VERSION}.tar.gz && \
    cd mysql-${MYSQL_VERSION} && \
    CFLAGS=-O0 cmake -DWITH_PIC=1 -DWITH_DEBUG=1 -DWITH_INNODB_EXTRA_DEBUG=1 -DCMAKE_INSTALL_PREFIX=/mysql-debug && \
    make -j$(getconf _NPROCESSORS_ONLN) install && \
    cd .. && \
    rm -rf mysql-${MYSQL_VERSION}

WORKDIR /mysql-debug
RUN ./scripts/mysql_install_db --force

FROM ubuntu:trusty
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install gdb gdbserver -y -o Dpkg::Options::='--force-confdef' -o Dpkg::Options::='--force-confold' && \
    echo '[mysqld]\nskip-host-cache\nskip-name-resolve\nskip-grant-tables' > /etc/my.cnf && \
    rm /etc/sysctl.d/10-ptrace.conf && \
    sysctl -p

COPY --from=builder /mysql-debug /mysql-debug
EXPOSE 3306
EXPOSE 2345
CMD cd /mysql-debug; sudo gdbserver :2345 ./bin/mysqld --user=root --skip-grant-tables --datadir=./data --basedir=./ --user=root
