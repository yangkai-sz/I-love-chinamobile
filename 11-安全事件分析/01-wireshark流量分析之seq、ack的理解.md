# tcp握手和连接通信过程
客户端与服务器端发起三次握手（syn--syn ack--ack），然后客户端正式对服务器的应用请求（如访问访问网页），客户端每次会发送seq号、tcp segement len、next seq（seq号+tcp segment len==next seq），还带上服务器上次返回的ack号，客户端每次向服务器发送数据都是带上最新的seq和最新的服务器端返回的ack号。服务器响应客户端时，服务器端会有自己的seq、tcp segment len、next seq，并带上ack号（这个ack号就时客户端最后一次发过来的next seq号，客户端的next seq就是服务器的ack，服务器的next seq成了下一次客户端请求的ack，都是有顺序的，这样才能组包）。
