---
title: SocketTimeOut引发的问题
date: 2017-06-06 11:41:09
tags:
-   网络
---

---

    最近工作的时候和上游对接广告，出现一些超时的问题。

    总的请求时间= 网络传输时间 + 服务器响应时间
    服务器总的响应时间 可以通过 nginx的request_time 来统计，即nginx处理请求的时间
    网络传输时间可以使用 ping 或者 ab 命令进行验证。

#### 压力测试命令 ab

    Usage: ab [options] [http[s]://]hostname[:port]/path
    Options are:
        -n requests     Number of requests to perform  			//总请求数
        -c concurrency  Number of multiple requests to make	//每次请求的并发数
        -t timelimit    Seconds to max. wait for responses		//最大等待响应时间
        -b windowsize   Size of TCP send/receive buffer, in bytes	//TCP传输接收的缓冲区大小
        -p postfile     File containing data to POST. Remember also to set -T	//POST请求
        -u putfile      File containing data to PUT. Remember also to set -T 	//GET请求
        -T content-type Content-type header for POSTing, eg.	//设置contentType
                        'application/x-www-form-urlencoded'
                        Default is 'text/plain'
        -v verbosity    How much troubleshooting info to print		//详细输出
        -w              Print out results in HTML tables				//打印HTML样式的结果
        -i              Use HEAD instead of GET					//HEAD代替GET
        -x attributes   String to insert as table attributes			//设置html table的属性
        -y attributes   String to insert as tr attributes			//设置html tr属性
        -z attributes   String to insert as td or th attributes		//设置html td th 属性
        -C attribute    Add cookie, eg. 'Apache=1234. (repeatable)	//添加请求的cookie
        -H attribute    Add Arbitrary header line, eg. 'Accept-Encoding: gzip'	//添加请求 header
                        Inserted after all normal header lines. (repeatable)
        -A attribute    Add Basic WWW Authentication, the attributes	//添加身份认证
                        are a colon separated username and password.
        -P attribute    Add Basic Proxy Authentication, the attributes	//添加代理认证
                        are a colon separated username and password.
        -X proxy:port   Proxyserver and port number to use			//使用代理
        -V              Print version number and exit					//ab 版本
        -k              Use HTTP KeepAlive feature					//使用长连接(并发和总请求数相同不会触发长连接)
        -d              Do not show percentiles served table.
        -S              Do not show confidence estimators and warnings.
        -g filename     Output collected data to gnuplot format file.		//输出到指定文件
        -e filename     Output CSV file with percentages served		//输出到CSV文件
        -r              Don't exit on socket receive errors.				//不因为接受错误退出
        -h              Display usage information (this message)
        -Z ciphersuite  Specify SSL/TLS cipher suite (See openssl ciphers)	//指定SSL/TLS认证
        -f protocol     Specify SSL/TLS protocol (SSL3, TLS1, or ALL)		//指定SSL/TLS协议

    总请求1000次,并发10,保持长连接,请求上游的地址。和真实线上的环境差不多。
    aixuw@uy01-07:~$ ab  -n 1000 -c  10 -v -k  http://adapi.xxxx.com:7702/time
    This is ApacheBench, Version 2.3 <$Revision: 655654 $>
    Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
    Licensed to The Apache Software Foundation, http://www.apache.org/

    Benchmarking adapi.xxxx.com (be patient)
    Completed 100 requests
    Completed 200 requests
    Completed 300 requests
    Completed 400 requests
    Completed 500 requests
    Completed 600 requests
    Completed 700 requests
    Completed 800 requests
    Completed 900 requests
    Completed 1000 requests
    Finished 1000 requests


    Server Software:        nginx/1.9.12
    Server Hostname:        adapi.xxxx.com
    Server Port:            7702

    Document Path:          /time
    Document Length:        26 bytes

    Concurrency Level:      10				//并发
    Time taken for tests:   5.967 seconds		//测试总用时
    Complete requests:      1000				//完成的请求
    Failed requests:        0
    Write errors:           0
    Total transferred:      209000 bytes
    HTML transferred:       26000 bytes
    Requests per second:    167.59 [#/sec] (mean)
    Time per request:       59.670 [ms] (mean)	//平均每次请求时间
    Time per request:       5.967 [ms] (mean, across all concurrent requests)  //并发平均每个请求时间
    Transfer rate:          34.21 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:       26   30  31.5     29    1025
    Processing:    26   29   1.8     29      40
    Waiting:       26   29   1.8     29      40
    Total:         53   59  31.6     57    1053

    Percentage of the requests served within a certain time (ms)
      50%     57
      66%     59
      75%     60
      80%     61
      90%     63
      95%     66
      98%     68
      99%     70
     100%   1053 (longest request)

    据上游反应服务器的响应时间也就是nginx处理请求时间在200到250ms以内加上网络传输时间超过了我们设置的250ms超时时间。
     最后我们这边将超时时间改为300ms，超时率下降。


#### 完整的http连接
    客户端C,服务端S
    三次握手
        1.C向S发送TCP的包,带上标志SYN
        2.S接受后开始回复SYN+ACK给C,表示同意连接
        3.C发送ACK给S,完成三次握手
    四次挥手
        1.C经过大约30s表示不会再请求数据,便向S发送FIN的标志
        2.S收到向C发送ACK。   此时连接处于半关闭,S可以向C发送,C无法向S请求数据
        3.S要把连接关闭向C发送FIN
        4.C向S发送ACK,连接完全关闭

参考: [https://zhuanlan.zhihu.com/p/27021102?group_id=849574141450412032](https://zhuanlan.zhihu.com/p/27021102?group_id=849574141450412032)

 ---