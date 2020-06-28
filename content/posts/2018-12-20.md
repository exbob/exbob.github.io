---
title: UDP socket 编程实践
date: 2018-12-20T08:00:00+08:00
draft: false
toc:
comments: true
---


UDP 是面向非连接的，如果通讯双方都在局域网中，服务器端和客户端没有区别，如果是局域网内的主机与公网上的主机通讯，由于 NAT 地址转换协议的作用，必须由局域网内的主机主动向公网的主句发送数据，公网的主机作为服务器端，必须接收客户端的数据后，解析出源 IP 和端口，再反向发送，实现双向通讯。下面是一个简单的 Demo 。

服务器端：

```c
    // server.c
    #include <stdio.h>
    #include <unistd.h>
    #include <sys/types.h>
    #include <sys/socket.h>
    #include <netinet/in.h>
    #include <arpa/inet.h>
    #include <string.h>
    
    #define SERVER_PORT 6789
    #define CPU_PORT 9009
    #define BUFF_LEN 1024
    
    int main(int argc, char* argv[])
    {
        int server_fd, count;
        struct sockaddr_in ser_addr; 
        struct sockaddr_in src_addr;
        socklen_t len;
        char buf[BUFF_LEN];
    
        server_fd = socket(AF_INET, SOCK_DGRAM, 0); //IPV4,UDP
    
        memset(&ser_addr, 0, sizeof(ser_addr));
        ser_addr.sin_family = AF_INET;
        ser_addr.sin_addr.s_addr = htonl(INADDR_ANY); // any ip
        ser_addr.sin_port = htons(SERVER_PORT);  // server port
    
        bind(server_fd, (struct sockaddr*)&ser_addr, sizeof(ser_addr));
    
        while(1)
        {
            // recv
            memset(buf, 0, BUFF_LEN);
            len = sizeof(src_addr);
            count = recvfrom(server_fd, buf, BUFF_LEN, 0, (struct sockaddr*)&src_addr, &len);  //block
            if(count == -1)
            {
                printf("recieve data fail!\n");
                return 0;
            }
            printf("server recv from %s:%d : %s\n",inet_ntoa(src_addr.sin_addr),ntohs(src_addr.sin_port),buf);
    
            // send
            memset(buf, 0, BUFF_LEN);
            sprintf(buf, "Server have recieved %d bytes data!", count);
            printf("server send : %s\n\n",buf);
            // send to src_addr
            sendto(server_fd, buf, BUFF_LEN, 0, (struct sockaddr*)&src_addr, len);
        }
    
        close(server_fd);
        return 0;
    }
```

客户端：

```c:n
    // cpu.c
    #include <stdio.h>
    #include <unistd.h>
    #include <sys/types.h>
    #include <sys/socket.h>
    #include <netinet/in.h>
    #include <arpa/inet.h>
    #include <string.h>
    
    #define SERVER_IP "118.89.16.24"
    #define SERVER_PORT 6789
    #define CPU_PORT 9009
    #define BUFF_LEN 1024
    
    int main(int argc, char* argv[])
    {
        int cpu_fd;
        struct sockaddr_in ser_addr;
        struct sockaddr_in src_addr;
        struct sockaddr_in my_addr;
        socklen_t len;
        char buf[BUFF_LEN] = "cpu msg!";
    
        cpu_fd = socket(AF_INET, SOCK_DGRAM, 0); //IPV4,UDP
    
        memset(&my_addr, 0, sizeof(my_addr));
        my_addr.sin_family = AF_INET;
        my_addr.sin_addr.s_addr = htonl(INADDR_ANY); // any ip
        my_addr.sin_port = htons(CPU_PORT);  // server port
        bind(cpu_fd, (struct sockaddr*)&my_addr, sizeof(my_addr));
    
        memset(&ser_addr, 0, sizeof(ser_addr));
        ser_addr.sin_family = AF_INET;
        ser_addr.sin_addr.s_addr = inet_addr(SERVER_IP); 
        ser_addr.sin_port = htons(SERVER_PORT);
    
        len = sizeof(ser_addr);
        printf("CPU send : %s\n",buf);  
        sendto(cpu_fd, buf, BUFF_LEN, 0, (struct sockaddr*)&ser_addr, len);
    
        memset(buf, 0, BUFF_LEN);
        recvfrom(cpu_fd, buf, BUFF_LEN, 0, (struct sockaddr*)&src_addr, &len);  //recv msg from server
        printf("CPU recv from %s:%d : %s\n",inet_ntoa(src_addr.sin_addr),ntohs(src_addr.sin_port),buf);
    
        close(cpu_fd);
        return 0;
    }
```

