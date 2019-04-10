---
layout:     post
title:      socket通信Demo
subtitle:   网络协议
date:       2018-05-23
author:     liangcs71
header-img: img/cats.jpg
catalog: true
tags:
    - network
    - socket
---


#### 网络配置--WSAStartup()

<strong>WSAStartup();</strong> 主要就是进行相应的socket库绑定, 初始化网络环境。

```java
int WSAStartup(    
  WORD wVersionRequested,       //版本号，一般使用2.2版本  
  LPWSADATA lpWSAData  <span style="white-space:pre">     </span>//WSAData地址  
);
```

<ol>
    <li>使用Socket的程序在使用Socket之前必须调用WSAStartup函数。以后应用程序就可以调用所请求的Socket库中的其它Socket函数了，然后绑定找到的Socket库到该应用程序中。该函数执行成功后返回0。</li>
    <li><code>WORD</code></li>
    <li><code>A.length * A[i].length <= 20000</code>
    版本号，MAKEWORD(2,2)表示使用WINSOCK2版本.</li>
    <li><code>WSADATA</code>数据类型：这个结构被用来存储被WSAStartup函数调用后返回的Winsock.dll执行的数据，就是说用来存储系统传回的关于WINSOCK的资料。</li>
</ol>

#### 创建套接字--socket()

<ol>
    <li>SOCKET PASCAL FAR socket(int af, int type, int protocol)
    <code>
	SOCKET socket(    
  		int af,         //IP协议簇  
  		int type,       //套接字类型，TCP应该用SOCK_STREAM  
  		int protocol<span style="white-space:pre">  </span>  //协议  
	);  
    </code>
	</li>
    <li>这个函数建立一个协议族为domain、协议类型为type、协议编号为protocol的套接字文件描述符。如果函数调用成功，会返回一个标识这个套接字的文件描述符，失败的时候返回-1。</li>
    <li>该调用要接收三个参数：af、type、protocol。参数af指定通信发生的区域：AF_UNIX、AF_INET、AF_NS等，而DOS、WINDOWS中仅支持AF_INET，它是网际网区域。因此，地址族与协议族相同。</li>
    <li>(1）一是TCP流式套接字(SOCK_STREAM)提供了一个面向连接、可靠的数据传输服务，数据无差错、无重复地发送，且按发送顺序接收。内设流量控制，避免数据流超限；数据被看作是字节流，无长度限制。文件传送协议（FTP）即使用流式套接字。
	（2）二是数据报式套接字(SOCK_DGRAM)提供了一个无连接服务。数据包以独立包形式被发送，不提供无错保证，数据可能丢失或重复，并且接收顺序混乱。网络文件系统（NFS）使用数据报式套接字。
	（3）三是原始式套接字(SOCK_RAW)该接口允许对较低层协议，如IP、ICMP直接访问。常用于检验新的协议实现或访问现有服务中配置的新设备。</li>
	<li>参数protocol说明该套接字使用的特定协议，如果调用者不希望特别指定使用的协议，则置为0，使用默认的连接模式。protocol用于制定某个协议的特定类型，即type类型中的某个类型。通常某协议中只有一种特定类型，这样protocol参数仅能设置为0；但是有些协议有多种特定的类型，就需要设置这个参数来选择特定的类型。</li>
	<li>根据这三个参数建立一个套接字，并将相应的资源分配给它，同时返回一个整型套接字号。因此，socket()系统调用实际上指定了相关五元组中的“协议”这一元。</li>

</ol>

#### 绑定地址端口--sockaddr_in
<ol>
    <li>创建好套接字后呢，我们需要告诉操作系统需要在哪个地址和端口上进行网络操作，相当于管道通信中绑定到标准输入输出口上。绑定的时候，需要有一个SOCKADDR_IN这个结构体)
    </li>
</ol>

```java
struct sockaddr_in{  
    short              sin_family;//协议簇  
    unsigned short     sin_port;//端口  
    struct   in_addr   sin_addr;//ip地址  
    char               sin_zero[8];//为了设置和SOCKADDR结构等长的补充字节  
}; 
```

其中in_addr结构体用来存放32位IPV4地址，如下

```java
struct in_addr{  
    I_addr_t               s_addr;//32位IPV4地址 
}; 
```

INADDR_ANY 监听0.0.0.0地址 socket只绑定端口让路由表决定传到哪个ip,转换过来就是0.0.0.0，泛指本机的意思，也就是表示本机的所有IP

```java
sockaddr_in addr = { 0 };
addr.sin_family = AF_INET; //指定地址族
addr.sin_addr.s_addr = INADDR_ANY;//IP初始化
addr.sin_port = htons(8090);//端口号初始化
```

#### 指定本地地址--bind()

<ol>
    <li>当一个套接字用socket()创建后，存在一个名字空间(地址族)，但它没有被命名。bind()将套接字地址（包括本地主机地址和本地端口地址）与所创建的套接字号联系起来，即将名字赋予套接字，以指定本地半相关。其调用格式如下：
    <code>
	int bind(    
  		SOCKET s,                          //我们创建的那个socket  
  		const struct sockaddr FAR *name,   //sockaddr结构指针  
  		int namelen                        //sockaddr长度  
	);  
    </code>
	</li>
    <li>参数s是由socket()调用返回的并且未作连接的套接字描述符(套接字号)。参数name 是赋给套接字s的本地地址（名字），其长度可变，结构随通信域的不同而不同。namelen表明了name的长度。如果没有错误发生，bind()返回0。否则返回SOCKET_ERROR。</li>
</ol>

```c++
bind(sock, (sockaddr*)&addr, sizeof(addr));//分配IP和端口
```

#### 监听连接--listen()

<ol>
    <li>当一个套接字用socket()创建后，存在一个名字空间(地址族)，但它没有被命名。bind()将套接字地址（包括本地主机地址和本地端口地址）与所创建的套接字号联系起来，即将名字赋予套接字，以指定本地半相关。其调用格式如下：
    <code>
	int listen(    
  		SOCKET s,      //我们创建的socket  
  		int backlog   //最大连接的队列长度  
	); 
    </code>
	</li>
    <li>参数s标识一个本地已建立、尚未连接的套接字号，服务器愿意从它上面接收请求。backlog表示请求连接队列的最大长度，用于限制排队请求的个数，目前允许的最大值为5。如果没有错误发生，listen()返回0。否则它返回SOCKET_ERROR。</li>
    <li>listen()在执行调用过程中可为没有调用过bind()的套接字s完成所必须的连接，并建立长度为backlog的请求连接队列。
    </li>
    <li>
    它在调用socket()分配一个流套接字，且调用bind()给s赋于一个名字之后调用，而且一定要在accept()之前调用。
    </li>
</ol>

#### 建立套接字连接--accept()

<ol>
    <li>当一个套接字用socket()创建后，存在一个名字空间(地址族)，但它没有被命名。bind()将套接字地址（包括本地主机地址和本地端口地址）与所创建的套接字号联系起来，即将名字赋予套接字，以指定本地半相关。其调用格式如下：
    <code>
	SOCKET accept(    
  		SOCKET s,   //我们监听的那个socket, 本地服务器socket  
  		struct sockaddr FAR *addr,    //我们需要传递一个sockaddr的地址，用于保存客户端的地址  ，空就可以
  		int FAR *addrlen  //sockaddr的长度指针  
	); 
    </code>
	</li>
    <li>参数s为本地套接字描述符，在用做accept()调用的参数前应该先调用过listen()。addr 指向客户方套接字地址结构的指针，用来接收连接实体的地址。addr的确切格式由套接字创建时建立的地址族决定。addrlen 为客户方套接字地址的长度（字节数）。如果没有错误发生，accept()返回一个SOCKET类型的值，表示接收到的套接字的描述符。否则返回值INVALID_SOCKET。</li>
    <li>accept()用于面向连接服务器。参数addr和addrlen存放客户方的地址信息。调用前，参数addr 指向一个初始值为空的地址结构，而addrlen 的初始值为0；调用accept()后，服务器等待从编号为s的套接字上接受客户连接请求，而连接请求是由客户方的connect()调用发出的。当有连接请求到达时，accept()调用将请求连接队列上的第一个客户方套接字地址及长度放入addr 和addrlen，并创建一个与s有相同特性的新套接字号。新的套接字可用于处理服务器并发请求。
	</li>
</ol>

#### 数据传输--send()与recv()

<ol>
    <li>当一个套接字用socket()创建后，存在一个名字空间(地址族)，但它没有被命名。bind()将套接字地址（包括本地主机地址和本地端口地址）与所创建的套接字号联系起来，即将名字赋予套接字，以指定本地半相关。其调用格式如下：
    <code>
	int recv(    
 		SOCKET s,         //客户端的socket  
  		char FAR *buf,    //接收的缓冲区  
 		int len,        //缓冲区的大小  
 		int flags       //标志位，一般为0  
	);  
    </code>
	</li>
    <li>recv()调用用于socket指定的已连接的数据报或流套接字上接收输入数据。参数s 为已连接的套接字描述符。buf指向接收输入数据缓冲区的指针，其长度由len 指定。flags 指定传输控制方式，如是否接收带外数据等。如果没有错误发生，recv()返回总共接收的字节数。如果连接被关闭，返回0。否则它返回SOCKET_ERROR。
 	<code>
	int send(    
  		SOCKET s,             //客户端的socket  
  		const char FAR *buf,  //发送数据的缓冲区  
  		int len,              //缓冲区的大小  
  		int flags             //标志位，一般为0  
	);   
    </code>
    </li>
</ol>


#### 关闭连接

<ol>
    <li>当传输完数据后，应该调用WSACleanup和closesocket来进行关闭网络环境和套接字。
	</li>
    <li>rWSACleanup函数用来解除与Socket库的绑定 。应用程序在完成对请求的Socket库的使用后，要调用WSACleanup函数来解除与Socket库的绑定并且释放Socket库所占用的系统资源。
    closesocket()关闭套接字s，并释放分配给该套接字的资源；如果s涉及一个打开的TCP连接，则该连接被释放。
 	<code>
	int  WSACleanup (void);  
	int closesocket(    
  		SOCKET s  //要关闭的套接字  
	);  
  	</code>
    </li>
</ol>


#### 示例代码

```java
//非Unix系统
#if defined(_MSC_VER) || defined(__MINGW32__) || defined(WIN32)
#include <WinSock2.h>
#include <WS2tcpip.h>
#include <windows.h>
#define close closesocket

class WinSockInit
{
	WSADATA _wsa;
public:
	WinSockInit()
	{  //分配套接字版本信息2.0，WSADATA变量地址
		WSAStartup(MAKEWORD(2, 0), &_wsa);
	}
	~WinSockInit()
	{
		WSACleanup();//功能是终止Winsock 2 DLL (Ws2_32.dll) 的使用
	}
};
#endif


#include <sys/types.h>
#include <sys/socket.h>
#include <iostream>
#include <string>
#include <stdio.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/shm.h>
using namespace std;

//处理URL
void UrlRouter(int clientSock, string const & url)
{
	string hint;
	if (url == "/")
	{
		cout << url << " 收到信息\n";
		hint = "haha, this is home page!";
		send(clientSock, hint.c_str(), hint.length(), 0);
	}
	else if (url == "/hello")
	{
		cout << url << " 收到信息\n";
		hint = "hello !";
		send(clientSock, hint.c_str(), hint.length(), 0);
	}
	else
	{
		cout << url << " 收到信息\n";
		hint = "no such URL!";
		send(clientSock, hint.c_str(), hint.length(), 0);
	}

}

int main()
{
#if defined(_MSC_VER) || defined(__MINGW32__) || defined(WIN32)
	WinSockInit socklibInit;//如果为Windows系统，进行WSAStartup
#endif

	int sock = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);//建立套接字，失败返回-1
	sockaddr_in addr = { 0 };
	addr.sin_family = AF_INET; //指定地址族
	addr.sin_addr.s_addr = INADDR_ANY;//IP初始化
	addr.sin_port = htons(8090);//端口号初始化

	int rc;
	rc = bind(sock, (sockaddr*)&addr, sizeof(addr));//分配IP和端口
	rc = listen(sock, 0);//设置监听
    printf("绑定套接字成功\n");                
    printf("服务器已经进入监听状态\n");
	//设置客户端
	sockaddr_in clientAddr;
	int clientAddrSize = sizeof(clientAddr);
	int clientSock;
	//接受客户端请求
	while (-1 != (clientSock = accept(sock, (sockaddr*)&clientAddr, (socklen_t*)&clientAddrSize)))
	{
		// 收请求
		string requestStr;
		int bufSize = 4096;
		requestStr.resize(bufSize);
		//接受数据
		recv(clientSock, &requestStr[0], bufSize, 0);

		//取得第一行
		string firstLine = requestStr.substr(0, requestStr.find("\r\n"));
		//取得URL
		firstLine = firstLine.substr(firstLine.find(" ") + 1);//substr，复制函数，参数为起始位置（默认0），复制的字符数目
		string url = firstLine.substr(0, firstLine.find(" "));//find返回找到的第一个匹配字符串的位置，而不管其后是否还有相匹配的字符串。

		//发送响应头
		string response =
			"HTTP/1.1 200 OK\r\n"
			"Content-Type: text/html; charset=gbk\r\n"
			"Connection: close\r\n"
			"\r\n";
		send(clientSock, response.c_str(), response.length(), 0);
		//处理URL
		UrlRouter(clientSock, url);

		close(clientSock);//关闭客户端套接字
	}

	close(sock);//关闭服务器套接字
	return 0;
}

```