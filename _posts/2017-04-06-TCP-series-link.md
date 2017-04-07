---
layout: post
title: TCP知识整理-从命令行看三次握手
category: php
keywords: 魔术方法
---

####nc模拟GET请求 
```
nc lumen.weipaitang.com 80<<EOF  
HEAD / HTTP/1.1  
Host:weipaitang.com  
  
EOF  
```

####TCPDUMP抓包
```
sudo tcpdump -i any tcp port 80 -S
```

-X表明当分析和打印时, tcpdump 会打印每个包的头部数据, 同时会以16进制和ASCII码形式打印出每个包的数据(但不包括连接层的头部)。

-S表明打印TCP 数据包的顺序号时, 使用绝对的顺序号, 而不是相对的顺序号.比如我第一次用tcpdump查看tcp的三次握手时发现第三次握手的ack=1,这个ack就是相对的，因为tcpdump只在SYN包中显示绝对顺序号，而非SYN包则显示相对的，为了便于观察，在抓包时都采用来绝对的顺序号
  
  
####抓包数据
```
18:02:03.399608 IP 172.16.30.74.53409 > 172.16.30.38.http: Flags [S], seq 3348550982, win 65535, options [mss 1460,nop,wscale 5,nop,nop,TS val 618940877 ecr 0,sackOK,eol], length 0
18:02:03.399732 IP 172.16.30.38.http > 172.16.30.74.53409: Flags [S.], seq 285815051, ack 3348550983, win 28960, options [mss 1460,sackOK,TS val 38896158 ecr 618940877,nop,wscale 7], length 0
18:02:03.400768 IP 172.16.30.74.53409 > 172.16.30.38.http: Flags [.], ack 285815052, win 4117, options [nop,nop,TS val 618940878 ecr 38896158], length 0
18:02:03.405780 IP 172.16.30.74.53409 > 172.16.30.38.http: Flags [P.], seq 3348550983:3348551000, ack 285815052, win 4117, options [nop,nop,TS val 618940883 ecr 38896158], length 17: HTTP: GET / HTTP/1.0
18:02:03.405863 IP 172.16.30.38.http > 172.16.30.74.53409: Flags [.], ack 3348551000, win 227, options [nop,nop,TS val 38896159 ecr 618940883], length 0
18:02:03.405986 IP 172.16.30.74.53409 > 172.16.30.38.http: Flags [F.], seq 3348551000, ack 285815052, win 4117, options [nop,nop,TS val 618940883 ecr 38896158], length 0
18:02:03.406630 IP 172.16.30.38.http > 172.16.30.74.53409: Flags [P.], seq 285815052:285815286, ack 3348551001, win 227, options [nop,nop,TS val 38896160 ecr 618940883], length 234: HTTP: HTTP/1.1 200 OK
```
```
18:02:03 （时间）  
399608    (ID)  
IP       (协议）  
172.16.30.74.53409 > 172.16.30.38.80 (源IP，端口，目的IP，端口 中间>表示方向)    
seq 3348550982  (IP包序号，相对序号为0)  
win 65535 (数据窗口大小，告诉对方本机接收窗口大小)  
options [mss 1460,nop,wscale 5,nop,nop,TS val 618940877 ecr 0,sackOK,eol]（对应TCP包头中的选项字段）  
MSS: Maxitum Segment Size 最大分段大小，MSS表示TCP传往另一端的最大块数据的长度。当一个连接建立时，连接的双方都要通告各自的MSS。如果一方不接收来自另一方的MSS值，则MSS就定为默认的536字节。  

Flags
S=SYN   发起连接标志   
P=PUSH  传送数据标志  
F=FIN    关闭连接标志  
RST= RESET  异常关闭连接  
.      表示确认包  
```
    

####备注
seq有相对值和绝对值，如果不加-S参数，第三次握手的时候ACK=1(基于第一次握手的seq，其相对值是0)，