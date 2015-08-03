#sysbench的安装和做性能测试

sysbench是一个模块化的、跨平台、多线程基准测试工具，主要用于评估测试各种不同系统参数下的数据库负载情况。关于这个项目的详细介绍请看：http://sysbench.sourceforge.net。
****
它主要包括以下几种方式的测试：

1. cpu性能
2. 磁盘io性能
3. 调度程序性能
4. 内存分配及传输速度
5. POSIX线程性能
6. 数据库性能(OLTP基准测试)

目前sysbench主要支持 MySQL,pgsql,oracle 这3种数据库。

一、安装
首先，在 http://sourceforge.net/projects/sysbench 下载源码包。
接下来，按照以下步骤安装：

\> tar zxf sysbench-0.4.8.tar.gz <br/>
\> cd sysbench-0.4.8 <br/>
\> ./configure && make && make install <br/>
\> strip /usr/local/bin/sysbench </br>

以上方法适用于 MySQL 安装在标准默认目录下的情况，如果 MySQL 并不是安装在标准目录下的话，那么就需要自己指定 MySQL 的路径了。比如我的 MySQL 喜欢自己安装在 /usr/local/mysql 下，则按照以下方法编译：

\> ./configure --with-mysql-includes=/usr/local/mysql/include --with-mysql-libs=/usr/local/mysql/lib && make && make install

当然了，用上面的参数编译的话，就要确保你的 MySQL lib目录下有对应的 so 文件，如果没有，可以自己下载 devel 或者 share 包来安装。
另外，如果想要让 sysbench 支持 pgsql/oracle 的话，就需要在编译的时候加上参数

--with-pgsql

或者

--with-oracle

这2个参数默认是关闭的，只有 MySQL 是默认支持的。

二、开始测试

编译成功之后，就要开始测试各种性能了，测试的方法官网网站上也提到一些，但涉及到 OLTP 测试的部分却不够准确。在这里我大致提一下：

1、cpu性能测试

sysbench --test=cpu --cpu-max-prime=20000 run
cpu测试主要是进行素数的加法运算，在上面的例子中，指定了最大的素数为 20000，自己可以根据机器cpu的性能来适当调整数值。

2、线程测试

sysbench --test=threads --num-threads=64 --thread-yields=100 --thread-locks=2 run
3、磁盘IO性能测试

sysbench --test=fileio --num-threads=16 --file-total-size=3G --file-test-mode=rndrw prepare
sysbench --test=fileio --num-threads=16 --file-total-size=3G --file-test-mode=rndrw run
sysbench --test=fileio --num-threads=16 --file-total-size=3G --file-test-mode=rndrw cleanup

上述参数指定了最大创建16个线程，创建的文件总大小为3G，文件读写模式为随机读。

4、内存测试

sysbench --test=memory --memory-block-size=8k --memory-total-size=4G run
上述参数指定了本次测试整个过程是在内存中传输 4G 的数据量，每个 block 大小为 8K。

5、OLTP测试

sysbench --test=oltp --mysql-table-engine=myisam --oltp-table-size=1000000 \
--mysql-socket=/tmp/mysql.sock --mysql-user=test --mysql-host=localhost \
--mysql-password=test prepare

上述参数指定了本次测试的表存储引擎类型为 myisam，这里需要注意的是，官方网站上的参数有一处有误，即 --mysql-table-engine，官方网站上写的是 --mysql-table-type，这个应该是没有及时更新导致的。另外，指定了表最大记录数为 1000000，其他参数就很好理解了，主要是指定登录方式。测试 OLTP 时，可以自己先创建数据库 sbtest，或者自己用参数 --mysql-db 来指定其他数据库。--mysql-table-engine 还可以指定为 innodb 等 MySQL 支持的表存储引擎类型。

好了，主要的就是这些了，想要了解更多信息就访问 sysbench 项目的主页吧。

技术相关:
>[ ngnix相关]()

> [MySQL优化]()