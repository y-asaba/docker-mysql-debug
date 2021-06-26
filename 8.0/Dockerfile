FROM ubuntu:focal as builder

ARG MYSQL_VERSION=8.0.25

WORKDIR /root

RUN DEBIAN_FRONTEND=noninteractive && \ 
    ln -fs /usr/share/zoneinfo/Asia/Tokyo /etc/localtime && \
    apt-get update -y && apt install cmake make gcc g++ openssl bison pkg-config -y && \
    DEBIAN_FRONTEND=noninteractive apt-get install build-essential cmake wget curl libncurses5-dev libssl-dev -y -o Dpkg::Options::='--force-confdef' -o Dpkg::Options::='--force-confold' && \
    wget http://ftp.jaist.ac.jp/pub/mysql/Downloads/MySQL-8.0/mysql-boost-${MYSQL_VERSION}.tar.gz && \
    tar zxf mysql-boost-${MYSQL_VERSION}.tar.gz && \
    cd mysql-${MYSQL_VERSION} && \
    CFLAGS=-O0 cmake -DWITH_DEBUG=1 -DFORCE_INSOURCE_BUILD=1 -DWITH_INNODB_EXTRA_DEBUG=1 -DCMAKE_INSTALL_PREFIX=/mysql-debug -DDOWNLOAD_BOOST=1 -DWITH_BOOST=./boost && \
    make -j$(getconf _NPROCESSORS_ONLN) install && \
    cd .. && \
    rm -rf mysql-${MYSQL_VERSION}

WORKDIR /mysql-debug
RUN ./bin/mysqld  --no-defaults --datadir=./data --basedir=./ --initialize-insecure

FROM ubuntu:focal
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install gdb gdbserver -y -o Dpkg::Options::='--force-confdef' -o Dpkg::Options::='--force-confold' && \
    echo '[mysqld]\nskip-host-cache\nskip-name-resolve\nskip-grant-tables' > /etc/my.cnf && \
    rm /etc/sysctl.d/10-ptrace.conf && \
    sysctl -p

COPY --from=builder /mysql-debug /mysql-debug
EXPOSE 3306
EXPOSE 2345
CMD cd /mysql-debug; sudo gdbserver :2345 ./bin/mysqld --user=root --skip-grant-tables --datadir=./data --basedir=./ --user=root
