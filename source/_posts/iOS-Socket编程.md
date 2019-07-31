---
title: iOS Socket编程
date: 2019-07-24 20:35:23
tags:
	- iOS	
	- socket
---

> 版权说明，此文是在[源文档]([https://elliotsomething.github.io/2015/08/29/iOS-%E4%B9%8B-Socket%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/](https://elliotsomething.github.io/2015/08/29/iOS-之-Socket学习笔记/))上修改的，如有问题请联系我删除

iOS App 网络请求大部分使用HTTP，也就是请求–响应这种应答式的方式。这种的话如果是比较小的项目还是蛮合适的，能够节省资源。但是比较大的项目的话就比较劣势了，而用socket就比较好，因为项目中网络请求比较多，时不时的需要发请求，socket的响应速度比HTTP要快，用户体验会要好很多。

Socket其实就是tcp连接，当客户端与服务端三次握手之后就一直连着，所以他的响应速度会比HTTP的应答式快。

### C语言 socket

研究了一下C语言的socket，直接上代码，下面演示一个一对一的socket服务端对客户端的demo：

**服务端：**

- 首先是sockaddr_in 这个结构体是用来保存socket地址的，具体参数基本看得懂
- 接下来是创建socket，使用的是socket()这个API创建的，具体参数以后在讲
- 然后是socket绑定地址也就是刚刚创建的sockaddr_in这个结构体，用bind()这个API绑定
- 接下来是监听这个socket了，用listen()这个API，注意监听的socket是只用来监听的，不能用来读写数据
- 监听之后就是新建一个socket用来读写数据了，用accept()这个API
- 最后就是读写数据嘛，分别用send()，recv()者两个API

**代码如下：**

```C
/* C语言 socket */
struct sockaddr_in server_addr;
server_addr.sin_len = sizeof(struct sockaddr_in);
server_addr.sin_family = AF_INET;//Address families AF_INET互联网地址簇 	
server_addr.sin_port = htons(11332);
server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
bzero(&(server_addr.sin_zero),8);
//创建socket 	
int server_socket = socket(AF_INET, SOCK_STREAM, 0);
//SOCK_STREAM 有连接 	
if (server_socket == -1) {
		perror("socket error");
		return 1;
	}
	//绑定socket：将创建的socket绑定到本地的IP地址和端口，此socket是半相关的，只是负责侦听客户端的连接请求，并不能用于和客户端通信 	
 int bind_result = bind(server_socket, (struct sockaddr *)&server_addr, sizeof(server_addr));
if (bind_result == -1) {
		perror("bind error");
		return 1;
}
	//listen侦听 第一个参数是套接字，第二个参数为等待接受的连接的队列的大小，在connect请求过来的时候,完成三次握手后先将连接放到这个队列中，直到被accept处理。如果这个队列满了，且有新的连接的时候，对方可能会收到出错信息。 	
if (listen(server_socket, 5) == -1) {
		perror("listen error");
		return 1;
}
struct sockaddr_in client_address;
socklen_t address_len;
int client_socket = accept(server_socket, (struct sockaddr *)&client_address, &address_len);
	//返回的client_socket为一个全相关的socket，其中包含client的地址和端口信息，通过client_socket可以和客户端进行通信。 	
if (client_socket == -1) {
		perror("accept error");
		return -1;
}
char recv_msg[1024];
char reply_msg[1024];
while (1) {
		bzero(recv_msg, 1024);
		bzero(reply_msg, 1024);
		printf("reply:");
		scanf("%s",reply_msg);
		send(client_socket, reply_msg, 1024, 0);
		long byte_num = recv(client_socket,recv_msg,1024,0);
		recv_msg[byte_num] = '\0';
		printf("client said:%s\n",recv_msg);
}
```

**客户端：**

- 基本步骤和服务端一样，
- 首先保存创建socket地址，不同的是客户端需要写服务端的IP地址
- 然后是创建socket，绑定地址
- 接下来和服务端不同，需要连接服务端，所以用到connect()这个API来连接服务端
- 连接之后就是读写数据了

**代码如下：**

```c
/* C语言 socket */
	struct sockaddr_in server_addr;
	server_addr.sin_len = sizeof(struct sockaddr_in);
	server_addr.sin_family = AF_INET;
	server_addr.sin_port = htons(11332);
	server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
	bzero(&(server_addr.sin_zero),8);
	int server_socket = socket(AF_INET, SOCK_STREAM, 0);
	if (server_socket == -1) {
		perror("socket error");
		return 1;
	}
	char recv_msg[1024];
	char reply_msg[1024];
	if (connect(server_socket, (struct sockaddr *)&server_addr, sizeof(struct sockaddr_in))==0)     {
		//connect 成功之后，其实系统将你创建的socket绑定到一个系统分配的端口上，且其为全相关，包含服务器端的信息，可以用来和服务器端进行通信。 		
    while (1) {
			bzero(recv_msg, 1024);
			bzero(reply_msg, 1024);
			long byte_num = recv(server_socket,recv_msg,1024,0);
			recv_msg[byte_num] = '\0';
			printf("server said:%s\n",recv_msg);

			printf("reply:");
			scanf("%s",reply_msg);
			if (send(server_socket, reply_msg, 1024, 0) == -1) {
				perror("send error");
			}
		}
	}
```

### OC的socket

上面讲的是C语言的socket，下面来讲一下OC的socket，其实很多都是C的，不过是对其进行了一层封装

**首先服务端：**

- 基本步骤和C语言的一样
- 创建socket用的是CFSocketCreate()；基本参数见代码
- 然后是创建保存地址
- 然后是绑定地址，用的是CFSocketSetAddress()这个API
- 然后就是和C语言不同的地方了，OC由于有runloop，不向C语言一样需要自己写一个死循环，所以需要将socket加入runloop中去，不断的监听
- 接下来是回调，这个是在创建socket的时候绑定的回调，同样需要重新创建一个socket来管理数据的读写，不过OC可以直接交给CFSocketNativeHandle这个类去处理，同样的数据流也有CFStream这个类去处理，他的子类可以分别管理读写流

**代码如下：**

```objective-c
-(int)setupSocket
{
	CFSocketContext sockContext = {0, // 结构体的版本，必须为0 		(__bridge void *)(self),
		NULL, // 一个定义在上面指针中的retain的回调， 可以为NULL 		NULL,
		NULL};
	_socket = CFSocketCreate(kCFAllocatorDefault, // 为新对象分配内存，可以为nil 	
                           PF_INET, // 协议族，如果为0或者负数，则默认为PF_INET 				
                           SOCK_STREAM, // 套接字类型，如果协议族为PF_INET,则它会默认为SOCK_STREAM 
                           IPPROTO_TCP, // 套接字协议，如果协议族是PF_INET且协议是0或者负数，它会默认为IPPROTO_TCP 							 
                           kCFSocketAcceptCallBack, // 触发回调函数的socket消息类型，具体见Callback Types 							 
                           SocketConnectionAcceptedCallBack, // 上面情况下触发的回调函数 							 &sockContext // 一个持有CFSocket结构信息的对象，可以为nil 							 
                          );
	if (NULL == _socket) {
		NSLog(@"Cannot create socket!");
		return 0;
	}
	int optval = 1;
	setsockopt(CFSocketGetNative(_socket), SOL_SOCKET, SO_REUSEADDR, // 允许重用本地地址和端口 			   (void *)&optval, sizeof(optval));
	struct sockaddr_in addr4;
	memset(&addr4, 0, sizeof(addr4));
	addr4.sin_len = sizeof(addr4);
	addr4.sin_family = AF_INET;
	addr4.sin_port = htons(8080);
	addr4.sin_addr.s_addr = htonl(INADDR_ANY);
	CFDataRef address = CFDataCreate(kCFAllocatorDefault, (UInt8 *)&addr4, sizeof(addr4));
	if (kCFSocketSuccess != CFSocketSetAddress(_socket, address))
	{
		NSLog(@"Bind to address failed!");
		if (_socket)
			CFRelease(_socket);
		_socket = NULL;
		return 0;
	}
	CFRunLoopRef cfRunLoop = CFRunLoopGetCurrent();
	CFRunLoopSourceRef source = CFSocketCreateRunLoopSource(kCFAllocatorDefault, _socket, 0);
	CFRunLoopAddSource(cfRunLoop, source, kCFRunLoopCommonModes);
	CFRunLoopRun();
	CFRelease(source);
	return 1;
}

//回调 
static void SocketConnectionAcceptedCallBack(CFSocketRef socket,
											 CFSocketCallBackType type,
											 CFDataRef address,
											 const void *data, void *info) {
	if (kCFSocketAcceptCallBack == type)
	{
		// 本地套接字句柄 		
    CFSocketNativeHandle nativeSocketHandle = *(CFSocketNativeHandle *)data;
		uint8_t name[SOCK_MAXADDRLEN];
		socklen_t nameLen = sizeof(name);
		if (0 != getpeername(nativeSocketHandle, (struct sockaddr *)name, &nameLen)) {
			NSLog(@"error");
			exit(1);
		}
		CFReadStreamRef iStream;
		CFWriteStreamRef oStream;
		// 创建一个可读写的socket连接 		
    CFStreamCreatePairWithSocket(kCFAllocatorDefault, nativeSocketHandle, &iStream, &oStream);
		if (iStream && oStream)
		{
			CFStreamClientContext streamContext = {0, info, NULL, NULL};
			if (!CFReadStreamSetClient(iStream, kCFStreamEventHasBytesAvailable,readStream, &streamContext))
			{
				exit(1);
			}

			if (!CFWriteStreamSetClient(oStream, kCFStreamEventCanAcceptBytes, writeStream, &streamContext))
			{
				exit(1);
			}
			CFReadStreamScheduleWithRunLoop(iStream, CFRunLoopGetCurrent(), kCFRunLoopDefaultMode);
			CFWriteStreamScheduleWithRunLoop(oStream, CFRunLoopGetCurrent(), kCFRunLoopDefaultMode);
			CFReadStreamOpen(iStream);
			CFWriteStreamOpen(oStream);
		} else
		{
			close(nativeSocketHandle);
		}
	}
}
// 读取数据 
void readStream(CFReadStreamRef stream, CFStreamEventType eventType, void *clientCallBackInfo)
{
	UInt8 buff[1024*4];

	NSMutableData *recvData = [NSMutableData data];

	while (1) {
		CFIndex count = CFReadStreamRead(stream, buff, 1024*4);
		if (count==0 || count == -1) {

		}else{
			[recvData appendBytes:buff length:count];
			if (!CFReadStreamHasBytesAvailable(stream)) {
				break;
			}
		}
	}
	NSString *recvString = [[NSString alloc]initWithData:recvData encoding:NSUTF8StringEncoding];
	///根据delegate显示到主界面去 	NSString *strMsg = [[NSString alloc]initWithFormat:@"客户端传来消息：%@",recvString];
//	CSocketServer *info = (__bridge CSocketServer*)clientCallBackInfo; //	[info ShowMsgOnMainPage:strMsg]; 	NSLog(@"%@",strMsg);

//	char *str = "你好 Client"; 	NSString *string = @"你好 Client\n";
	NSData *data = [string dataUsingEncoding:NSUTF8StringEncoding];
	NSMutableData *data1 = [NSMutableData dataWithData:data];
//	uint8_t * uin8b = (uint8_t *)str; 	const uint8_t * uin8b = data1.bytes;
	if (outputStream != NULL)
	{

		while (1) {
			if (data1.length>0 && CFWriteStreamCanAcceptBytes(outputStream)) {
				CFIndex writeLength = CFWriteStreamWrite(outputStream, uin8b, data1.length);

				if (writeLength == 0 || writeLength == -1) {

				}else{
					[data1 replaceBytesInRange:NSMakeRange(0, writeLength) withBytes:NULL length:0];
				}
			}else{
				break;
			}

		}
	}
	else {
		NSLog(@"Cannot send data!");
	}

}
void writeStream (CFWriteStreamRef stream, CFStreamEventType eventType, void *clientCallBackInfo)
{
	outputStream = stream;

}
```

**客户端：** 步骤和服务端差不多,直接上代码：

```objective-c
-(void)CreateConnect:(NSString*)strAddress
{
	CFSocketContext sockContext = {0, // 结构体的版本，必须为0 		(__bridge void *)(self),
		NULL, // 一个定义在上面指针中的retain的回调， 可以为NULL 		NULL,
		NULL};
	_socket = CFSocketCreate(kCFAllocatorDefault, // 为新对象分配内存，可以为nil 							 PF_INET, // 协议族，如果为0或者负数，则默认为PF_INET 							 SOCK_STREAM, // 套接字类型，如果协议族为PF_INET,则它会默认为SOCK_STREAM 							 IPPROTO_TCP, // 套接字协议，如果协议族是PF_INET且协议是0或者负数，它会默认为IPPROTO_TCP 							 kCFSocketConnectCallBack, // 触发回调函数的socket消息类型，具体见Callback Types 							 TCPClientConnectCallBack, // 上面情况下触发的回调函数 							 &sockContext // 一个持有CFSocket结构信息的对象，可以为nil 							 );
	if(_socket != NULL)
	{
		struct sockaddr_in addr4;   // IPV4 		memset(&addr4, 0, sizeof(addr4));
		addr4.sin_len = sizeof(addr4);
		addr4.sin_family = AF_INET;
		addr4.sin_port = htons(8080);
		addr4.sin_addr.s_addr = inet_addr([strAddress UTF8String]);  // 把字符串的地址转换为机器可识别的网络地址 
		// 把sockaddr_in结构体中的地址转换为Data 		CFDataRef address = CFDataCreate(kCFAllocatorDefault, (UInt8 *)&addr4, sizeof(addr4));
		CFSocketConnectToAddress(_socket, // 连接的socket 								 address, // CFDataRef类型的包含上面socket的远程地址的对象 								 -1  // 连接超时时间，如果为负，则不尝试连接，而是把连接放在后台进行，如果_socket消息类型为kCFSocketConnectCallBack，将会在连接成功或失败的时候在后台触发回调函数 								 );
		CFRunLoopRef cRunRef = CFRunLoopGetCurrent();    // 获取当前线程的循环 		// 创建一个循环，但并没有真正加如到循环中，需要调用CFRunLoopAddSource 		CFRunLoopSourceRef sourceRef = CFSocketCreateRunLoopSource(kCFAllocatorDefault, _socket, 0);
		CFRunLoopAddSource(cRunRef, // 运行循环 						   sourceRef,  // 增加的运行循环源, 它会被retain一次 						   kCFRunLoopCommonModes  // 增加的运行循环源的模式 						   );
		CFRelease(sourceRef);
		NSLog(@"connect ok");
	}
}
// socket回调函数，同客户端 static void TCPClientConnectCallBack(CFSocketRef socket, CFSocketCallBackType type, CFDataRef address, const void *data, void *info)
{
	if (kCFSocketAcceptCallBack == type)
	{
		// 本地套接字句柄 		CFSocketNativeHandle nativeSocketHandle = *(CFSocketNativeHandle *)data;
		uint8_t name[SOCK_MAXADDRLEN];
		socklen_t nameLen = sizeof(name);
		if (0 != getpeername(nativeSocketHandle, (struct sockaddr *)name, &nameLen)) {
			NSLog(@"error");
			exit(1);
		}
		CFReadStreamRef iStream;
		CFWriteStreamRef oStream;
		// 创建一个可读写的socket连接 		CFStreamCreatePairWithSocket(kCFAllocatorDefault, nativeSocketHandle, &iStream, &oStream);
		if (iStream && oStream)
		{
			CFStreamClientContext streamContext = {0, info, NULL, NULL};
			if (!CFReadStreamSetClient(iStream, kCFStreamEventHasBytesAvailable,readStream, &streamContext))
			{
				exit(1);
			}
			if (!CFWriteStreamSetClient(oStream, kCFStreamEventCanAcceptBytes, writeStream, &streamContext))
			{
				exit(1);
			}
			CFReadStreamScheduleWithRunLoop(iStream, CFRunLoopGetCurrent(), kCFRunLoopDefaultMode);
			CFWriteStreamScheduleWithRunLoop(oStream, CFRunLoopGetCurrent(), kCFRunLoopDefaultMode);
			CFReadStreamOpen(iStream);
			CFWriteStreamOpen(oStream);
		} else
		{
			close(nativeSocketHandle);
		}
	}
}

// 读取数据 void readStream(CFReadStreamRef stream, CFStreamEventType eventType, void *clientCallBackInfo)
{
	UInt8 buff[1024*4];

	NSMutableData *recvData = [NSMutableData data];

	while (1) {
		CFIndex count = CFReadStreamRead(stream, buff, 1024*4);
		if (count==0 || count == -1) {

		}else{
			[recvData appendBytes:buff length:count];
			if (!CFReadStreamHasBytesAvailable(stream)) {
				break;
			}
		}
	}
	NSString *recvString = [[NSString alloc]initWithData:recvData encoding:NSUTF8StringEncoding];
	///根据delegate显示到主界面去 	NSString *strMsg = [[NSString alloc]initWithFormat:@"客户端传来消息：%@",recvString];


	//	CSocketServer *info = (__bridge CSocketServer*)clientCallBackInfo; 	//	[info ShowMsgOnMainPage:strMsg]; 	NSLog(@"%@",strMsg);

	//	char *str = "你好 Client"; 	NSString *string = @"你好 Client\n";
	NSData *data = [string dataUsingEncoding:NSUTF8StringEncoding];


	NSMutableData *data1 = [NSMutableData dataWithData:data];
	//	uint8_t * uin8b = (uint8_t *)str; 	const uint8_t * uin8b = data1.bytes;
	if (outputStream != NULL)
	{

		while (1) {
			if (data1.length>0 && CFWriteStreamCanAcceptBytes(outputStream)) {
				CFIndex writeLength = CFWriteStreamWrite(outputStream, uin8b, data1.length);

				if (writeLength == 0 || writeLength == -1) {

				}else{
					[data1 replaceBytesInRange:NSMakeRange(0, writeLength) withBytes:NULL length:0];

				}
			}else{
				break;
			}

		}
	}
	else {
		NSLog(@"Cannot send data!");
	}

}
void writeStream (CFWriteStreamRef stream, CFStreamEventType eventType, void *clientCallBackInfo)
{
	outputStream = stream;

}
```

### AsycSocket浅析

Socket基本就是这样了，还有一个牛人封装好的一个socket，用起来确实简单多了，不过坏处是有很多不需要的代码，所以需要自己去优化，也直接上代码吧：

**服务端：**

```objective-c
-(void)start{
	asyncSocket = [[AsyncSocket alloc] initWithDelegate:self];
	NSError *err = nil;
	if (![asyncSocket acceptOnPort:8080 error:&err]) {
		return;
	}
	NSLog(@"开始监听");
}

- (void)onSocket:(AsyncSocket *)sock didAcceptNewSocket:(AsyncSocket *)newSocket
{
	NSLog(@"%s", __func__);
	self.clientAsycSocket = newSocket;

}
- (void)onSocket:(AsyncSocket *)sock didConnectToHost:(NSString *)host port:(UInt16)port{
	NSLog(@"%s",__func__);

	[sock readDataWithTimeout:-1 tag:0];

	NSData *aData=[@"Hi there server" dataUsingEncoding:NSUTF8StringEncoding];
	[sock writeData:aData withTimeout:-1 tag:0];
}


//读取数据 -(void)onSocket:(AsyncSocket *)sock didReadData:(NSData *)data withTag:(long)tag
{
	NSString *aStr=[[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
	NSLog(@"aStr==%@",aStr);
}
//是否加密 -(void)onSocketDidSecure:(AsyncSocket *)sock
{
	NSLog(@"onSocket:%p did go a secure line:YES",sock);
}
//遇到错误时关闭连接 -(void)onSocket:(AsyncSocket *)sock willDisconnectWithError:(NSError *)err
{
	NSLog(@"onSocket:%p will disconnect with error:%@",sock,err);
}
//断开连接 -(void)onSocketDidDisconnect:(AsyncSocket *)sock
{
	NSLog(@"onSocketDidDisconnect:%p",sock);
}
- (void )dealloc
{
	NSLog(@"%s", __func__);
}
```

**客户端代码：**

```objective-c
-(void)start{
	asyncSocket = [[AsyncSocket alloc] initWithDelegate:self];
	NSError *err = nil;
	if(![asyncSocket connectToHost:@"127.0.0.1" onPort:8080 error:&err])
	{
		NSLog(@"Error: %@", err);
	}
//	NSData *aData=[@"Hi there" dataUsingEncoding:NSUTF8StringEncoding]; //	[asyncSocket writeData:aData withTimeout:-1 tag:0]; }

//建立连接 -(void)onSocket:(AsyncSocket *)sock didConnectToHost:(NSString *)host port:(UInt16)port
{
	NSLog(@"onScoket:%p did connecte to host:%@ on port:%d",sock,host,port);

//	NSData *aData=[@"Hi there" dataUsingEncoding:NSUTF8StringEncoding]; //	[sock writeData:aData withTimeout:-1 tag:0]; 	[sock readDataWithTimeout:-1 tag:0];
}

//读取数据 -(void)onSocket:(AsyncSocket *)sock didReadData:(NSData *)data withTag:(long)tag
{
	NSString *aStr=[[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
	NSLog(@"aStr==%@",aStr);
	NSData *aData=[@"Hi there" dataUsingEncoding:NSUTF8StringEncoding];
	[sock writeData:aData withTimeout:-1 tag:0];
	[sock readDataWithTimeout:-1 tag:0];
}
//是否加密 -(void)onSocketDidSecure:(AsyncSocket *)sock
{
	NSLog(@"onSocket:%p did go a secure line:YES",sock);
}
//遇到错误时关闭连接 -(void)onSocket:(AsyncSocket *)sock willDisconnectWithError:(NSError *)err
{
	NSLog(@"onSocket:%p will disconnect with error:%@",sock,err);
}
//断开连接 -(void)onSocketDidDisconnect:(AsyncSocket *)sock
{
	NSLog(@"onSocketDidDisconnect:%p",sock);
}
```

好啦，Socket学习笔记到此结束。 [demo在此Github](https://github.com/Elliotsomething/socketDemo)