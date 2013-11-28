---
layout: post
title: C、Python、Java通讯
category: blog
description: 经常会遇到各种不同语言写的服务器端，不同语言的处理也各不一样，必须清楚知道不同在哪里才能写出正确的处理程序。
tag: C Python Java Socket 协议
---

每种语言都有不同的粉丝，各有特色，所以也就有了各种各样的服务器端，我们经验会面对多语言协同的情况，所以了解不同语言间如何处理
通讯协议很重要。下面以Java为Server，C、Python做客户端为例看看不同的语言怎么处理。

Java端代码：

    
    import java.io.DataInputStream;
    import java.io.DataOutputStream;
    import java.net.ServerSocket;
    import java.net.Socket;

    public class Server {
        public static void main(String[] args) throws Exception {
            System.out.println("Server started.");
            DataOutputStream out;
            DataInputStream in;
            ServerSocket serverSocket;
            
            serverSocket = new ServerSocket(8007);
            
            Socket clientSocket = serverSocket.accept();
            
            System.out.println("Client jonied.");
            
            in = new DataInputStream(clientSocket.getInputStream());
            out = new DataOutputStream(clientSocket.getOutputStream());
            
            int size = in.readInt();
            
            System.out.println("Recv size: " + size);
            
            byte ctrlFlag = in.readByte();
            
            System.out.println("Recv ctrl flag: " + ctrlFlag);
            
            byte[] packBuff = new byte[size];
            
            in.read(packBuff);

            System.out.println("Recv from c server: " + new String(packBuff));
            
            String repley = "Hello, c client!";
            out.writeInt(repley.getBytes().length);
            out.write(repley.getBytes());
            
            //out.flush();
            
            out.close();
            in.close();
            
            clientSocket.close();
            serverSocket.close();
            
            System.out.println("Finished.");
        }
    }
      
上面是一个简单的Java服务器端代码，代码首先读取一个4位的int代表整个包的大小，这个大小不包括包大小本身和控制位， 然后读取一个控制位(通常协议会这么定义)， 然后根据读取的int长度读取整个包。对应的C结构：

	typedef	struct tHead {
		int length; // 包长度
		char ctrl_flag; // 命令
		char body[128]; // 包体
	} tHead;

完整代码：

	#include <stdio.h>
	#include <stdlib.h>
	#include <string.h>
	#include <sys/socket.h>
	#include <sys/types.h>
	#include <arpa/inet.h>
	#include <netinet/in.h>
	
	typedef	struct tHead {
		int length;
		char ctrl_flag;
		char body[128];
	} tHead;
	
	typedef tHead *phead;
	
	int main (int argc, char const *argv[])
	{
		int client_sockfd;
		int len;
		struct sockaddr_in remote_addr;
		memset(&remote_addr, 0, sizeof(remote_addr));
		remote_addr.sin_family = AF_INET;
		remote_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
		remote_addr.sin_port = htons(8007);
		
		client_sockfd = socket(PF_INET, SOCK_STREAM, 0);
		if(client_sockfd < 0)
		{
			perror("socket");
			return 1;
		}
		
		if(connect(client_sockfd, (struct sockaddr *)&remote_addr, sizeof(struct sockaddr)) < 0)
		{
			perror("connect");
			return 1;
		}
		
		tHead head;
		printf("Connected to server\n");
		int pack_size;
		
		char *hello = "Hello, Java Server!";
		
		int str_len = strlen(hello) + 1;
		
		int str_len_n = htonl(str_len);
		
		head.ctrl_flag = 0x7f;
		
		strncpy(head.body, hello, strlen(hello) + 1);
		
		printf("head body: %s\n", head.body);
		
		head.length = str_len_n;
		
		send(client_sockfd, &head, sizeof(head), 0); 
	
		len = recv(client_sockfd, &pack_size, sizeof(int), 0);
		
		printf("result of recv: %d\n", len);
		
		pack_size = ntohl(pack_size);
		
		printf("Recv package size: %d\n", pack_size);
		
		char *body = (char*)malloc(sizeof(char) * (pack_size));
		
		len = recv(client_sockfd, body, sizeof(char) * (pack_size), 0);
		printf("Recv package: %s\n", body);
		free(body);
		
		return 0;
	}
	
Java/Python通讯，Java Server端代码:

	
	import java.io.ByteArrayOutputStream;
	import java.io.DataOutputStream;
	import java.io.InputStream;
	import java.math.BigInteger;
	import java.net.InetSocketAddress;
	import java.net.ServerSocket;
	import java.net.Socket;
	
	class RequestProcessor implements Runnable {
	
		private Socket clientSocket;
	
		public RequestProcessor(Socket clientSocket) {
			this.clientSocket = clientSocket;
		}
	
		public void run() {
			
			System.out.println("Process request: " + ((InetSocketAddress)this.clientSocket.getRemoteSocketAddress()).getHostName());
			
			try {
				DataOutputStream out = new DataOutputStream( clientSocket.getOutputStream() );
				//DataInputStream in = new DataInputStream( clientSocket.getInputStream() );
				
				InputStream in = clientSocket.getInputStream();
				ByteArrayOutputStream baos = new ByteArrayOutputStream();
				
				byte[] buff = new byte[in.available()];
				
				in.read(buff);
				
				System.out.println("Recv package from clinet: " + new String(buff));
				
				String pack1 = "the package 1";
				String pack2 = "the package 2";
				String pack3 = "the package 3";
				BigInteger bi = new BigInteger(pack1.getBytes().length + "");
				
				out.writeInt(pack1.getBytes().length);
				out.write((byte)0);
				out.write(pack1.getBytes());
				
				
				clientSocket.close();
			} catch (Exception ex) {
				ex.printStackTrace();
			}
		}
	
	}
	
	public class Server {
		public static void main(String[] args) throws Exception {
			ServerSocket ss = new ServerSocket(8007);
	
			System.out.println("Server started.");
			while (true) {
				Socket clientSocket = ss.accept();
				RequestProcessor rp = new RequestProcessor(clientSocket);
				Thread requestHandler = new Thread(rp);
				requestHandler.start();
			}
		}
	}

Python客户端代码：

	import random
	import sruct
	from StringIO import StrnIO
	
	from twisted.internet import rector
	from twisted.internet.protocol import Protocol, ClientFatry
	
	class Receiver(Protool):
    
	    __time = 0
    
	    def __init__(slf):
	        self.__buffer = StrinIO()
	        print "Receiver created with random: ", random.randrange(1,100)
    
	    def connectionMade(slf):
	        print "Conneted"
	        self.transport.write("Hi, I want to some daa.")
    
	    def dataReceived(self, dta):
	        print "length of data: ", len(ata)
	        self.__buffer.write(ata)
	       
	    def connectionLost(self, reaon):
	        pack = self.__buffer.getvaue()
	        print "Length of pack: ", len(ack)
	        print "pack: ", pac[:1]
	       
	        pack_s = struct.unpack("!ic13s", pak) #
	        print "Whole package: ", pck_s
	        self.__buffer.clse()
    
	class ReceiverFactory(ClientFactry):
    
	    clientConnectionLost = clientConnectionFailed = lambda self, connector, reason: reactor.sop()
    
	    def startedConnecting(self, connecor):
	        print "Started connect to sever"
    
	    def buildProtocol(self, adr):
	        print "Connected with addr: ",addr
	        return Receier()
	       
	reactor.connectTCP("192.168.1.102", 8007, ReceiverFactoy())
	reactor.run()	

Python与C、Java交互的时候要注意的是Python的类型系统并不像C、Java那样明确，所以， 在处理协议的时候必须要使用struct包来
转换，例如上面的struct.unpack("!ic13s", pak)就是从一个str转换为一个tHead结构。C与Java交互的时候只需要注意类型匹配就可
以了。
