1.1 Introduction When writing programs that communicate across a computer network, one must first invent a protocol, an agreement on how those programs will communicate. Before delving into the design details of a protocol, high-level decisions must be made about which program is expected to initiate communication and when responses are expected. For example, a Web server is typically thought of as a long-running program (or daemon) that sends network messages only in response to requests coming in from the network. The other side of the protocol is a Web client, such as a browser, which always initiates communication with the server. This organization into client and server is used by most network-aware applications. Deciding that the client always initiates requests tends to simplify the protocol as well as the programs themselves. Of course, some of the more complex network applications also require asynchronous callback communication, where the server initiates a message to the client. But it is far more common for applications to stick to the basic client/server model shown in Figure 1.1.
Figure 1.1. Network application: client and server.
Clients normally communicate with one server at a time, although using a Web browser as an example, we might communicate with many different Web servers over, say, a 10-minute time period. But from the server's perspective, at any given point in time, it is not unusual for a server to be communicating with multiple clients. We show this in Figure 1.2. Later in this text, we will cover several different ways for a server to handle multiple clients at the same time.
Figure 1.2. Server handling multiple clients at the same time.
The client application and the server application may be thought of as communicating via a network protocol, but actually, multiple layers of network protocols are typically involved. In this text, we focus on the TCP/IP protocol suite, also called the Internet protocol suite. For example, Web clients and servers communicate using the Transmission Control Protocol, or TCP. TCP, in turn, uses the Internet Protocol, or IP, and IP communicates with a datalink layer of some form. If the client and server are on the same Ethernet, we would have the arrangement shown in Figure 1.3.
Figure 1.3. Client and server on the same Ethernet communicating using TCP.
Page 29
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
Even though the client and server communicate using an application protocol, the transport layers communicate using TCP. Note that the actual flow of information between the client and server goes down the protocol stack on one side, across the network, and up the protocol stack on the other side. Also note that the client and server are typically user processes, while the TCP and IP protocols are normally part of the protocol stack within the kernel. We have labeled the four layers on the right side of Figure 1.3.
TCP and IP are not the only protocols that we will discuss. Some clients and servers use the User Datagram Protocol (UDP) instead of TCP, and we will discuss both protocols in more detail in Chapter 2. Furthermore, we have used the term "IP," but the protocol, which has been in use since the early 1980s, is officially called IP version 4 (IPv4). A new version, IP version 6 (IPv6) was developed during the mid-1990s and could potentially replace IPv4 in the years to come. This text covers the development of network applications using both IPv4 and IPv6. Appendix A provides a comparison of IPv4 and IPv6, along with other protocols that we will discuss.
The client and server need not be attached to the same local area network (LAN) as we show in Figure 1.3. For instance, in Figure 1.4, we show the client and server on different LANs, with both LANs connected to a wide area network (WAN) using routers.
Figure 1.4. Client and server on different LANs connected through a WAN.
Routers are the building blocks of WANs. The largest WAN today is the Internet. Many companies build their own WANs and these private WANs may or may not be connected to the Internet.
The remainder of this chapter provides an introduction to the various topics that are covered in detail later in the text. We start with a complete example of a TCP client, albeit a simple one, that demonstrates many of the function calls and concepts that we will encounter throughout the text. This client works with IPv4 only, and we show the changes
Page 30
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
required to work with IPv6. A better solution is to write protocol-independent clients and servers, and we will discuss this in Chapter 11. This chapter also shows a complete TCP server that works with our client.
To simplify all our code, we define our own wrapper functions for most of the system functions that we call throughout the text. We can use these wrapper functions most of the time to check for an error, print an appropriate message, and terminate when an error occurs. We also show the test network, hosts, and routers used for most examples in the text, along with their hostnames, IP addresses, and operating systems.
Most discussions of Unix these days include the term "X," which is the standard that most vendors have adopted. We describe the history of POSIX and how it affects the Application Programming Interfaces (APIs) that we describe in this text, along with the other players in the standards arena.
[ Team LiB ]
Page 31
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
[ Team LiB ]
1.2 A Simple Daytime Client Let's consider a specific example to introduce many of the concepts and terms that we will encounter throughout the book. Figure 1.5 is an implementation of a TCP time-of-day client. This client establishes a TCP connection with a server and the server simply sends back the current time and date in a human-readable format.
Figure 1.5 TCP daytime client.
intro/daytimetcpcli.c
1 #include "unp.h"
2 int
3 main(int argc, char **argv)
4 {
5 int sockfd, n;
6 char recvline[MAXLINE + 1];
7 struct sockaddr_in servaddr;
8 if (argc != 2)
9 err_quit("usage: a.out ");
10 if ( (sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0)
11 err_sys("socket error");
12 bzero(&servaddr, sizeof(servaddr));
13 servaddr.sin_family = AF_INET;
14 servaddr.sin_port = htons(13); /* daytime server */
15 if (inet_pton(AF_INET, argv[1], &servaddr.sin_addr) <= 0)
16 err_quit("inet_pton error for %s", argv[1]);
17 if (connect(sockfd, (SA *) &servaddr, sizeof(servaddr)) < 0)
18 err_sys("connect error");
19 while ( (n = read(sockfd, recvline, MAXLINE)) > 0) {
20 recvline[n] = 0; /* null terminate */
21 if (fputs(recvline, stdout) == EOF)
22 err_sys("fputs error");
23 }
24 if (n < 0)
25 err_sys("read error");
26 exit(0);
27 }
This is the format that we will use for all the source code in the text. Each nonblank line is numbered. The text describing portions of the code notes the starting and ending line numbers in the left margin, as shown shortly. Sometimes a paragraph is preceded by a short, descriptive, bold heading, providing a summary statement of the code being described.
The horizontal rules at the beginning and end of a code fragment specify the source code filename: the file daytimetcpcli.c in the directory intro for this example. Since the source code for all the examples in the text is freely available (see the Preface), this lets you locate the appropriate source file. Compiling, running, and especially modifying these programs while reading this text is an excellent way to learn the concepts of network
Page 32
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
programming.
Throughout the text, we will use indented, parenthetical notes such as this to describe implementation details and historical points.
If we compile the program into the default a.out file and execute it, we will have the following output:
solaris % a.out 206.168.112.96 our input
Mon May 26 20:58:40 2003 the program's output
Whenever we display interactive input and output, we will show our typed input in bold and the computer output like this. Comments are added on the right side in italics. We will always include the name of the system as part of the shell prompt (solaris in this example) to show on which host the command was run. Figure 1.16 shows the systems used to run most of the examples in this book. The hostnames usually describe the operating system (OS) as well.
There are many details to consider in this 27-line program. We mention them briefly here, in case this is your first encounter with a network program, and provide more information on these topics later in the text.
Include our own header 1 We include our own header, unp.h, which we will show in Section D.1. This header includes numerous system headers that are needed by most network programs and defines various constants that we use (e.g., MAXLINE).
Command-line arguments 2 3 This is the definition of the main function along with the command-line arguments. We have written the code in this text assuming an American National Standards Institute (ANSI) C compiler (also referred to as an ISO C compiler).
Create TCP socket 10 11 The socket function creates an Internet (AF_INET) stream (SOCK_STREAM) socket, which is a fancy name for a TCP socket. The function returns a small integer descriptor that we can use to identify the socket in all future function calls (e.g., the calls to connect and read that follow).
The if statement contains a call to the socket function, an assignment of the return value to the variable named sockfd, and then a test of whether this assigned value is less than 0. While we could break this into two C statements,
sockfd = socket(AF_INET, SOCK_STREAM, 0);
if (sockfd < 0)
it is a common C idiom to combine the two lines. The set of parentheses around the function call and assignment is required, given the precedence rules of C (the less-than operator has a higher precedence than assignment). As a matter of coding style, the authors always place a space between the two opening parentheses, as a visual indicator that the left-hand side of the comparison is also an assignment. (This style is copied from the Minix source code [Tanenbaum 1987].) We use this same style in the while statement later in the program.
Page 33
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
We will encounter many different uses of the term "socket." First, the API that we are using is called the sockets API. In the preceding paragraph, we referred to a function named socket that is part of the sockets API. In the preceding paragraph, we also referred to a TCP socket, which is synonymous with a TCP endpoint.
If the call to socket fails, we abort the program by calling our own err_sys function. It prints our error message along with a description of the system error that occurred (e.g., "Protocol not supported" is one possible error from socket) and terminates the process. This function, and a few others of our own that begin with err_, are called throughout the text. We will describe them in Section D.3.
Specify server's IP address and port 12 16 We fill in an Internet socket address structure (a sockaddr_in structure named servaddr) with the server's IP address and port number. We set the entire structure to 0 using bzero, set the address family to AF_INET, set the port number to 13 (which is the well-known port of the daytime server on any TCP/IP host that supports this service, as shown in Figure 2.18), and set the IP address to the value specified as the first command-line argument (argv[1]). The IP address and port number fields in this structure must be in specific formats: We call the library function htons ("host to network short") to convert the binary port number, and we call the library function inet_pton ("presentation to numeric") to convert the ASCII command-line argument (such as 206.62.226.35 when we ran this example) into the proper format.
bzero is not an ANSI C function. It is derived from early Berkeley networking code. Nevertheless, we use it throughout the text, instead of the ANSI C memset function, because bzero is easier to remember (with only two arguments) than memset (with three arguments). Almost every vendor that supports the sockets API also provides bzero, and if not, we provide a macro definition of it in our unp.h header.
Indeed, the author of TCPv3 made the mistake of swapping the second and third arguments to memset in 10 occurrences in the first printing. A C compiler cannot catch this error because both arguments are of the same type. (Actually, the second argument is an int and the third argument is size_t, which is typically an unsigned int, but the values specified, 0 and 16, respectively, are still acceptable for the other type of argument.) The call to memset still worked, but did nothing. The number of bytes to initialize was specified as 0. The programs still worked, because only a few of the socket functions actually require that the final 8 bytes of an Internet socket address structure be set to 0. Nevertheless, it was an error, and one that could be avoided by using bzero, because swapping the two arguments to bzero will always be caught by the C compiler if function prototypes are used.
This may be your first encounter with the inet_pton function. It is new with IPv6 (which we will talk more about in Appendix A). Older code uses the inet_addr function to convert an ASCII dotted-decimal string into the correct format, but this function has numerous limitations that inet_pton corrects. Do not worry if your system does not (yet) support this function; we will provide an implementation of it in Section 3.7.
Establish connection with server 17 18 The connect function, when applied to a TCP socket, establishes a TCP connection with the server specified by the socket address structure pointed to by the second argument. We must also specify the length of the socket address structure as the third argument to connect, and for Internet socket address structures, we always let the compiler calculate the length using C's sizeof operator.
In the unp.h header, we #define SA to be struct sockaddr, that is, a generic socket address structure. Everytime one of the socket functions requires a pointer to a socket
Page 34
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
address structure, that pointer must be cast to a pointer to a generic socket address structure. This is because the socket functions predate the ANSI C standard, so the void * pointer type was not available in the early 1980s when these functions were developed. The problem is that "struct sockaddr" is 15 characters and often causes the source code line to extend past the right edge of the screen (or page, in the case of a book), so we shorten it to SA. We will talk more about generic socket address structures when explaining Figure 3.3.
Read and display server's reply 19 25 We read the server's reply and display the result using the standard I/O fputs function. We must be careful when using TCP because it is a byte-stream protocol with no record boundaries. The server's reply is normally a 26-byte string of the form
Mon May 26 20 : 58 : 40 2003\r\n
where \r is the ASCII carriage return and \n is the ASCII linefeed. With a byte-stream protocol, these 26 bytes can be returned in numerous ways: a single TCP segment containing all 26 bytes of data, in 26 TCP segments each containing 1 byte of data, or any other combination that totals to 26 bytes. Normally, a single segment containing all 26 bytes of data is returned, but with larger data sizes, we cannot assume that the server's reply will be returned by a single read. Therefore, when reading from a TCP socket, we always need to code the read in a loop and terminate the loop when either read returns 0 (i.e., the other end closed the connection) or a value less than 0 (an error).
In this example, the end of the record is being denoted by the server closing the connection. This technique is also used by version 1.0 of the Hypertext Transfer Protocol (HTTP). Other techniques are available. For example, the Simple Mail Transfer Protocol (SMTP) marks the end of a record with the two-byte sequence of an ASCII carriage return followed by an ASCII linefeed. Sun Remote Procedure Call (RPC) and the Domain Name System (DNS) place a binary count containing the record length in front of each record that is sent when using TCP. The important concept here is that TCP itself provides no record markers: If an application wants to delineate the ends of records, it must do so itself and there are a few common ways to accomplish this.
Terminate program 26 exit terminates the program. Unix always closes all open descriptors when a process terminates, so our TCP socket is now closed.
As we mentioned, the text will go into much more detail on all the points we just described.
[ Team LiB ]
Page 35
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
[ Team LiB ]
1.3 Protocol Independence Our program in Figure 1.5 is protocol-dependent on IPv4. We allocate and initialize a sockaddr_in structure, we set the family of this structure to AF_INET, and we specify the first argument to socket as AF_INET.
To modify the program to work under IPv6, we must change the code. Figure 1.6 shows a version that works under IPv6, with the changes highlighted in bold.
Figure 1.6 Version of Figure 1.5 for IPv6.
intro/daytimetcpcliv6.c
1 #include "unp.h"
2 int
3 main(int argc, char **argv)
4 {
5 int sockfd, n;
6 char recvline[MAXLINE + 1];
7 struct sockaddr_in6 servaddr;
8 if (argc != 2)
9 err_quit("usage: a.out ");
10 if ( (sockfd = socket(AF_INET6, SOCK_STREAM, 0)) < 0)
11 err_sys("socket error");
12 bzero(&servaddr, sizeof(servaddr));
13 servaddr.sin6_family = AF_INET6;
14 servaddr.sin6_port = htons(13); /* daytime server */
15 if (inet_pton(AF_INET6, argv[1], &servaddr.sin6_addr) <= 0)
16 err_quit("inet_pton error for %s", argv[1]);
17 if (connect(sockfd, (SA *) &servaddr, sizeof(servaddr)) < 0)
18 err_sys("connect error");
19 while ( (n = read(sockfd, recvline, MAXLINE)) > 0) {
20 recvline[n] = 0; /* null terminate */
21 if (fputs(recvline, stdout) == EOF)
22 err_sys("fputs error");
23 }
24 if (n < 0)
25 err_sys("read error");
26 exit(0);
27 }
Only five lines are changed, but what we now have is another protocol-dependent program; this time, it is dependent on IPv6. It is better to make a program protocol-independent. Figure 11.11 will show a version of this client that is protocol-independent by using the getaddrinfo function (which is called by tcp_connect).
Another deficiency in our programs is that the user must enter the server's IP address as a dotted-decimal number (e.g., 206.168.112.219 for the IPv4 version). Humans work better with names instead of numbers (e.g., www.unpbook.com). In Chapter 11, we will discuss the functions that convert between hostnames and IP addresses, and between service
Page 36
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
names and ports. We purposely put off the discussion of these functions and continue using IP addresses and port numbers so we know exactly what goes into the socket address structures that we must fill in and examine. This also avoids complicating our discussion of network programming with the details of yet another set of functions.
[ Team LiB ]
Page 37
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
[ Team LiB ]
1.4 Error Handling: Wrapper Functions In any real-world program, it is essential to check every function call for an error return. In Figure 1.5, we check for errors from socket, inet_pton, connect, read, and fputs, and when one occurs, we call our own functions, err_quit and err_sys, to print an error message and terminate the program. We find that most of the time, this is what we want to do. Occasionally, we want to do something other than terminate when one of these functions returns an error, as in Figure 5.12, when we must check for an interrupted system call.
Since terminating on an error is the common case, we can shorten our programs by defining a wrapper function that performs the actual function call, tests the return value, and terminates on an error. The convention we use is to capitalize the name of the function, as in
sockfd = Socket(AF_INET, SOCK_STREAM, 0);
Our wrapper function is shown in Figure 1.7.
Figure 1.7 Our wrapper function for the socket function.
lib/wrapsock.c
236 int
237 Socket(int family, int type, int protocol)
238 {
239 int n;
240 if ( (n = socket(family, type, protocol)) < 0)
241 err_sys("socket error");
242 return (n);
243 }
Whenever you encounter a function name in the text that begins with an uppercase letter, that is one of our wrapper functions. It calls a function whose name is the same but begins with the lowercase letter.
When describing the source code that is presented in the text, we always refer to the lowest level function being called (e.g., socket), not the wrapper function (e.g., Socket).
While these wrapper functions might not seem like a big savings, when we discuss threads in Chapter 26, we will find that thread functions do not set the standard Unix errno variable when an error occurs; instead, the errno value is the return value of the function. This means that every time we call one of the pthread_ functions, we must allocate a variable, save the return value in that variable, and then set errno to this value before calling err_sys. To avoid cluttering the code with braces, we can use C's comma operator to combine the assignment into errno and the call of err_sys into a single statement, as in the following:
int n;
if ( (n = pthread_mutex_lock(&ndone_mutex)) != 0)
Page 38
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
errno = n, err_sys("pthread_mutex_lock error");
Alternately, we could define a new error function that takes the system's error number as an argument. But, we can make this piece of code much easier to read as just
Pthread_mutex_lock(&ndone_mutex);
by defining our own wrapper function, as shown in Figure 1.8.
Figure 1.8 Our wrapper function for pthread_mutex_lock.
lib/wrappthread.c
72 void
73 Pthread_mutex_lock(pthread_mutex_t *mptr)
74 {
75 int n;
76 if ( (n = pthread_mutex_lock(mptr)) == 0)
77 return;
78 errno = n;
79 err_sys("pthread_mutex_lock error");
80 }
With careful C coding, we could use macros instead of functions, providing a little run-time efficiency, but these wrapper functions are rarely the performance bottleneck of a program.
Our choice of capitalizing the first character of a function name is a compromise. Many other styles were considered: prefixing the function name with an "e" (as done on p. 182 of [Kernighan and Pike 1984]), appending "_e" to the function name, and so on. Our style seems the least distracting while still providing a visual indication that some other function is really being called.
This technique has the side benefit of checking for errors from functions whose error returns are often ignored: close and listen, for example.
Throughout the rest of this book, we will use these wrapper functions unless we need to check for an explicit error and handle it in some way other than terminating the process. We do not show the source code for all our wrapper functions, but the code is freely available (see the Preface).
Unix errno Value When an error occurs in a Unix function (such as one of the socket functions), the global variable errno is set to a positive value indicating the type of error and the function normally returns 1. Our err_sys function looks at the value of errno and prints the corresponding error message string (e.g., "Connection timed out" if errno equals ETIMEDOUT).
The value of errno is set by a function only if an error occurs. Its value is undefined if the function does not return an error. All of the positive error values are constants with all-uppercase names beginning with "E," and are normally defined in the header. No error has a value of 0.
Storing errno in a global variable does not work with multiple threads that share all global variables. We will talk about solutions to this problem in Chapter 26.
Page 39
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
Throughout the text, we will use phrases such as "the connect function returns ECONNREFUSED" as shorthand to mean that the function returns an error (typically with a return value of 1), with errno set to the specified constant.
[ Team LiB ]
Page 40
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
[ Team LiB ]
1.5 A Simple Daytime Server We can write a simple version of a TCP daytime server, which will work with the client from Section 1.2. We use the wrapper functions that we described in the previous section and show this server in Figure 1.9.
Figure 1.9 TCP daytime server.
intro/daytimetcpsrv.c
1 #include "unp.h".
2 #include
3 int
4 main(int argc, char **argv)
5 {
6 int listenfd, connfd;
7 struct sockaddr_in servaddr;
8 char buff[MAXLINE];
9 time_t ticks;
10 listenfd = Socket(AF_INET, SOCK_STREAM, 0);
11 bzeros(&servaddr, sizeof(servaddr));
12 servaddr.sin_family = AF_INET;
13 servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
14 servaddr.sin_port = htons(13); /* daytime server */
15 Bind(listenfd, (SA *) &servaddr, sizeof(servaddr));
16 Listen(listenfd, LISTENQ);
17 for ( ; ; ) {
18 connfd = Accept(listenfd, (SA *) NULL, NULL);
19 ticks = time(NULL);
20 snprintf(buff, sizeof(buff), "%.24s\r\n", ctime(&ticks));
21 Write(connfd, buff, strlen(buff));
22 Close(connfd);
23 }
24 }
Create a TCP socket 10 The creation of the TCP socket is identical to the client code.
Bind server's well-known port to socket 11 15 The server's well-known port (13 for the daytime service) is bound to the socket by filling in an Internet socket address structure and calling bind. We specify the IP address as INADDR_ANY, which allows the server to accept a client connection on any interface, in case the server host has multiple interfaces. Later we will see how we can restrict the server to accepting a client connection on just a single interface.
Convert socket to listening socket
Page 41
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
16 By calling listen, the socket is converted into a listening socket, on which incoming connections from clients will be accepted by the kernel. These three steps, socket, bind, and listen, are the normal steps for any TCP server to prepare what we call the listening descriptor (listenfd in this example).
The constant LISTENQ is from our unp.h header. It specifies the maximum number of client connections that the kernel will queue for this listening descriptor. We say much more about this queueing in Section 4.5.
Accept client connection, send reply 17 21 Normally, the server process is put to sleep in the call to accept, waiting for a client connection to arrive and be accepted. A TCP connection uses what is called a three-way handshake to establish a connection. When this handshake completes, accept returns, and the return value from the function is a new descriptor (connfd) that is called the connected descriptor. This new descriptor is used for communication with the new client. A new descriptor is returned by accept for each client that connects to our server.
The style used throughout the book for an infinite loop is
for ( ; ; ) {
. . .
}
The current time and date are returned by the library function time, which returns the number of seconds since the Unix Epoch: 00:00:00 January 1, 1970, Coordinated Universal Time (UTC). The next library function, ctime, converts this integer value into a human-readable string such as
Mon May 26 20:58:40 2003
A carriage return and linefeed are appended to the string by snprintf, and the result is written to the client by write.
If you're not already in the habit of using snprintf instead of the older sprintf, now's the time to learn. Calls to sprintf cannot check for overflow of the destination buffer. snprintf, on the other hand, requires that the second argument be the size of the destination buffer, and this buffer will not overflow.
snprintf was a relatively late addition to the ANSI C standard, introduced in the version referred to as ISO C99. Virtually all vendors provide it as part of the standard C library, and many freely available versions are also available. We use snprintf throughout the text, and we recommend using it instead of sprintf in all your programs for reliability.
It is remarkable how many network break-ins have occurred by a hacker sending data to cause a server's call to sprintf to overflow its buffer. Other functions that we should be careful with are gets, strcat, and strcpy, normally calling fgets, strncat, and strncpy instead. Even better are the more recently available functions strlcat and strlcpy, which ensure the result is a properly terminated string. Additional tips on writing secure network programs are found in Chapter 23 of [Garfinkel, Schwartz, and Spafford 2003].
Terminate connection
Page 42
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
22 The server closes its connection with the client by calling close. This initiates the normal TCP connection termination sequence: a FIN is sent in each direction and each FIN is acknowledged by the other end. We will say much more about TCP's three-way handshake and the four TCP packets used to terminate a TCP connection in Section 2.6.
As with the client in the previous section, we have only examined this server briefly, saving all the details for later in the book. Note the following points:
 As with the client, the server is protocol-dependent on IPv4. We will show a protocol-independent version that uses the getaddrinfo function in Figure 11.13.
 Our server handles only one client at a time. If multiple client connections arrive at about the same time, the kernel queues them, up to some limit, and returns them to accept one at a time. This daytime server, which requires calling two library functions, time and ctime, is quite fast. But if the server took more time to service each client (say a few seconds or a minute), we would need some way to overlap the service of one client with another client.
 The server that we show in Figure 1.9 is called an iterative server because it iterates through each client, one at a time. There are numerous techniques for writing a concurrent server, one that handles multiple clients at the same time. The simplest technique for a concurrent server is to call the Unix fork function (Section 4.7), creating one child process for each client. Other techniques are to use threads instead of fork (Section 26.4), or to pre-fork a fixed number of children when the server starts (Section 30.6).
 If we start a server like this from a shell command line, we might want the server to run for a long time, since servers often run for as long as the system is up. This requires that we add code to the server to run correctly as a Unix daemon: a process that can run in the background, unattached to a terminal. We will cover this in Section 13.4.
[ Team LiB ]
Page 43
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
[ Team LiB ]
1.6 Roadmap to Client/Server Examples in the Text Two client/server examples are used predominantly throughout the text to illustrate the various techniques used in network programming:
 A daytime client/server (which we started in Figures 1.5, 1.6, and 1.9)
 An echo client/server (which will start in Chapter 5)
To provide a roadmap for the different topics that are covered in this text, we will summarize the programs that we will develop, and give the starting figure number and page number in which the source code appears. Figure 1.10 lists the versions of the daytime client, two versions of which we have already seen. Figure 1.11 lists the versions of the daytime server. Figure 1.12 lists the versions of the echo client, and Figure 1.13 lists the versions of the echo server.
Figure 1.10. Different versions of the daytime client developed in the text.
Figure 1.11. Different versions of the daytime server developed in the text.
Figure 1.12. Different versions of the echo client developed in the text.
Page 44
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
Figure 1.13. Different versions of the echo server developed in the text.
[ Team LiB ]
Page 45
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
[ Team LiB ]
1.7 OSI Model A common way to describe the layers in a network is to use the International Organization for Standardization (ISO) open systems interconnection (OSI) model for computer communications. This is a seven-layer model, which we show in Figure 1.14, along with the approximate mapping to the Internet protocol suite.
Figure 1.14. Layers in OSI model and Internet protocol suite.
We consider the bottom two layers of the OSI model as the device driver and networking hardware that are supplied with the system. Normally, we need not concern ourselves with these layers other than being aware of some properties of the datalink, such as the 1500-byte Ethernet maximum transfer unit (MTU), which we describe in Section 2.11.
The network layer is handled by the IPv4 and IPv6 protocols, both of which we will describe in Appendix A. The transport layers that we can choose from are TCP and UDP, and we will describe these in Chapter 2. We show a gap between TCP and UDP in Figure 1.14 to indicate that it is possible for an application to bypass the transport layer and use IPv4 or IPv6 directly. This is called a raw socket, and we will talk about this in Chapter 28.
The upper three layers of the OSI model are combined into a single layer called the application. This is the Web client (browser), Telnet client, Web server, FTP server, or whatever application we are using. With the Internet protocols, there is rarely any distinction between the upper three layers of the OSI model.
The sockets programming interfaces described in this book are interfaces from the upper three layers (the "application") into the transport layer. This is the focus of this book: how to write applications using sockets that use either TCP or UDP. We already mentioned raw sockets, and in Chapter 29 we will see that we can even bypass the IP layer completely to read and write our own datalink-layer frames.
Why do sockets provide the interface from the upper three layers of the OSI model into the transport layer? There are two reasons for this design, which we note on the right side of Figure 1.14. First, the upper three layers handle all the details of the application (FTP, Telnet, or HTTP, for example) and know little about the communication details. The lower four layers know little about the application, but handle all the communication details: sending data, waiting for acknowledgments, sequencing data that arrives out of order, calculating and verifying checksums, and so on. The second reason is that the upper three layers often form what is called a user process while the lower four layers are normally provided as part of the operating system (OS) kernel. Unix provides this separation between the user process and the kernel, as do many other contemporary operating systems. Therefore, the interface between layers 4 and 5 is the natural place to build the API.
Page 46
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
[ Team LiB ]
Page 47
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
[ Team LiB ]
1.8 BSD Networking History The sockets API originated with the 4.2BSD system, released in 1983. Figure 1.15 shows the development of the various BSD releases, noting the major TCP/IP developments. A few changes to the sockets API also took place in 1990 with the 4.3BSD Reno release, when the OSI protocols went into the BSD kernel.
Figure 1.15. History of various BSD releases.
The path down the figure from 4.2BSD through 4.4BSD shows the releases from the Computer Systems Research Group (CSRG) at Berkeley, which required the recipient to already have a source code license for Unix. But all the networking code, both the kernel support (such as the TCP/IP and Unix domain protocol stacks and the socket interface), along with the applications (such as the Telnet and FTP clients and servers), were developed independently from the AT&T-derived Unix code. Therefore, starting in 1989, Berkeley provided the first of the BSD networking releases, which contained all the networking code and various other pieces of the BSD system that were not constrained by the Unix source code license requirement. These releases were "publicly available" and eventually became available by anonymous FTP to anyone.
The final releases from Berkeley were 4.4BSD-Lite in 1994 and 4.4BSD-Lite2 in 1995. We note that these two releases were then used as the base for other systems: BSD/OS, FreeBSD, NetBSD, and OpenBSD, most of which are still being actively developed and enhanced. More information on the various BSD releases, and on the history of the various Unix systems in general, can be found in Chapter 01 of [McKusick et al. 1996].
Page 48
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
Many Unix systems started with some version of the BSD networking code, including the sockets API, and we refer to these implementations as Berkeley-derived implementations. Many commercial versions of Unix are based on System V Release 4 (SVR4). Some of these versions have Berkeley-derived networking code (e.g., UnixWare 2.x), while the networking code in other SVR4 systems has been independently derived (e.g., Solaris 2.x). We also note that Linux, a popular, freely available implementation of Unix, does not fit into the Berkeley-derived classification: Its networking code and sockets API were developed from scratch.
[ Team LiB ]
Page 49
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
[ Team LiB ]
1.9 Test Networks and Hosts Figure 1.16 shows the various networks and hosts used in the examples throughout the text. For each host, we show the OS and the type of hardware (since some of the operating systems run on more than one type of hardware). The name within each box is the hostname that appears in the text.
The topology shown in Figure 1.16 is interesting for the sake of our examples, but the machines are largely spread out across the Internet and the physical topology becomes less interesting in practice. Instead, virtual private networks (VPNs) or secure shell (SSH) connections provide connectivity between these machines regardless of where they live physically.
Figure 1.16. Networks and hosts used for most examples in the text.
The notation "/24" indicates the number of consecutive bits starting from the leftmost bit of the address used to identify the network and subnet. Section A.4 will talk about the /n notation used today to designate subnet boundaries.
The real name of the Sun OS is SunOS 5.x and not Solaris 2.x, but everyone refers to it as Solaris, the name given to the sum of the OS and other software bundled with the base OS.
Discovering Network Topology We show the network topology in Figure 1.16 for the hosts used for the examples throughout this text, but you may need to know your own network topology to run the examples and exercises on your own network. Although there are no current Unix standards with regard to network configuration and administration, two basic commands are provided by most Unix systems and can be used to discover some details of a network: netstat and ifconfig. Check the manual (man) pages for these commands on your system to see the details on the information that is output. Also be aware that some vendors place these commands in an administrative directory, such as /sbin or /usr/sbin, instead of the normal /usr/bin, and these directories might not be in your normal shell search path (PATH).
1. netstat -i provides information on the interfaces. We also specify the -n flag to print numeric addresses, instead of trying to find names for the networks. This shows us the interfaces and their names.
2. 3.
Page 50
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
4. 5. linux % netstat -ni
6. Kernel Interface table
7. Iface MTU Met RX-OK RX-ERR RX-DRP RX-OVR TX-OK TX-ERR TX-DRP TX-OVR
Flg
8. eth0 1500 049211085 0 0 040540958 0 0 0
BMRU
9. lo 16436 098613572 0 0 098613572 0 0 0
LRU
10.
The loopback interface is called lo and the Ethernet is called eth0. The next example shows a host with IPv6 support.
freebsd % netstat -ni
Name Mtu Network Address Ipkts Ierrs Opkts Oerrs
Coll
hme0 1500 08:00:20:a7:68:6b 29100435 35 46561488 0
0
hme0 1500 12.106.32/24 12.106.32.254 28746630 - 46617260 -
-
hme0 1500 fe80:1::a00:20ff:fea7:686b/64
fe80:1::a00:20ff:fea7:686b
0 - 0 -
-
hme0 1500 3ffe:b80:1f8d:1::1/64
3ffe:b80:1f8d:1::1 0 - 0 -
-
hme1 1500 08:00:20:a7:68:6b 51092 0 31537 0
0
hme1 1500 fe80:2::a00:20ff:fea7:686b/64
fe80:2::a00:20ff:fea7:686b
0 - 90 -
-
hme1 1500 192.168.42 192.168.42.1 43584 - 24173 -
-
hme1 1500 3ffe:b80:1f8d:2::1/64
3ffe:b80:1f8d:2::1 78 - 8 -
-
lo0 16384 10198 0 10198 0
0
lo0 16384 ::1/128 ::1 10 - 10 -
-
lo0 16384 fe80:6::1/64 fe80:6::1 0 - 0 -
-
lo0 16384 127 127.0.0.1 10167 - 10167 -
-
gif0 1280 6 0 5 0
0
gif0 1280 3ffe:b80:3:9ad1::2/128
3ffe:b80:3:9ad1::2 0 - 0 -
-
gif0 1280 fe80:8::a00:20ff:fea7:686b/64
fe80:8::a00:20ff:fea7:686b
0 - 0 -
-
11. netstat -r shows the routing table, which is another way to determine the
Page 51
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
interfaces. We normally specify the -n flag to print numeric addresses. This also shows the IP address of the default router.
12. 13. 14. 15. freebad % netstat -nr 16. Routing tables 17. 18. Internet: 19. Destination Gateway Flags Refs Use Netif
Expire
20. default 12.106.32.1 UGSc 10 6877 hme0 21. 12.106.32/24 link#1 UC 3 0 hme0 22. 12.106.32.1 00:b0:8e:92:2c:00 UHLW 9 7 hme0
1187
23. 12.106.32.253 08:00:20:b8:f7:e0 UHLW 0 1 hme0 140
24. 12.106.32.254 08:00:20:a7:6e:6b UHLW 0 2 lo0 25. 127.0.0.1 127.0.0.1 UH 1 10167 lo0 26. 192.168.42 link#2 UC 2 0 hme1 27. 192.168.42.1 08:00:20:a7:68:6b UHLW 0 11 lo0 28. 192.168.42.2 00:04:ac:17:bf:38 UHLW 2 24108 hme1
210
29. 30. Internet6: 31. Destination Gateway Flags
Netif Expire
32. ::/96 ::1 UGRSc lo0 =>
33. default 3ffe:b80:3:9ad1::1 UGSc gif0
34. ::1 ::1 UH lo0
35. ::ffff:0.0.0.0/96 ::1 UGRSc lo0
36. 3ffe:b80:3:9adl::1 3ffe:b80:3:9adl::2 UH gif0
37. 3ffe:b80:3:9adl::2 link#8 UHL lo0
38. 3ffe:b80:1f8d::/48 lo0 USc lo0
39. 3ffe:b80:1f8d:1::/64 link#1 UC hme0
40. 3ffe:b80:lf8d:1::1 08:00:20:a7:68:6b UHL lo0
41. 3ffe:b80:lf8d:2::/64 link#2 UC hme1
42. 3ffe:b80:lf8d:2::1 08:00:20:a7:68:6b UHL lo0
43. 3ffe:b80:lf8d:2:204:acff:fe17:bf38 00:04:ac:17:bf:38 UHLW hme1
44. fe80::/10 ::1 UGRSc lo0
45. fe80::%hme0/64 link#1 UC hme0
46. fe80::a00:20ff:fea7:686b%hme0 08:00:20:a7:68:6b UHL lo0
47. fe80::%hme1/64 link#2 UC hme1
48. fe80::a00:20ff:fea7:686b%hme1 08:00:20:a7:68:6b UHL
Page 52
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
lo0
49. fe80::%lo0/64 fe80::1%lo0 Uc lo0
50. fe80::1%lo0 link#6 UHL lo0
51. fe80::%gif0/64 link#8 UC gif0
52. fe80::a00:20ff:fea7:686b%gif0 link#8 UC lo0
53. ff01::/32 ::1 U lo0
54. ff02::/16 ::1 UGRS lo0
55. ff02::%hme0/32 link#1 UC hme0
56. ff02::%hme1/32 link#2 UC hme1
57. ff02::%lo0/32 ::1 UC lo0
58. ff02::%gif0/32 link#8 UC gif0
59.
60. Given the interface names, we execute ifconfig to obtain the details for each interface.
61. 62. 63. 64. linux % ifconfig eth0 65. eth0 Link encap:Ethernet HWaddr 00:C0:9F:06:B0:E1 66. inet addr:206.168.112.96 Bcast:206.168.112.127
Mask:255.255.255.128
67. UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1 68. RX packets:49214397 errors:0 dropped:0 overruns:0 frame:0 69. TX packets:40543799 errors:0 dropped:0 overruns:0 carrier:0 70. collisions:0 txqueuelen:100 71. RX bytes:1098069974 (1047.2 Mb) TX bytes:3360546472 (3204.8
Mb)
72. Interrupt:11 Base address:0x6000 73.
This shows the IP address, subnet mask, and broadcast address. The MULTICAST flag is often an indication that the host supports multicasting. Some implementations provide a -a flag, which prints information on all configured interfaces.
74. One way to find the IP address of many hosts on the local network is to ping the broadcast address (which we found in the previous step).
75. 76. 77. 78. linux % ping -b 206.168.112.127 79. WARNING: pinging broadcast address 80. PING 206.168.112.127 (206.168.112.127) from 206.168.112.96 : 56(84) bytes
of data.
81. 64 bytes from 206.168.112.96: icmp_seq=0 ttl=255 time=241 usec 82. 64 bytes from 206.168.112.40: icmp_seq=0 ttl=255 time=2.566 msec (DUP!) 83. 64 bytes from 206.168.112.118: icmp_seq=0 ttl=255 time=2.973 msec (DUP!) 84. 64 bytes from 206.168.112.14: icmp_seq=0 ttl=255 time=3.089 msec (DUP!) 85. 64 bytes from 206.168.112.126: icmp_seq=0 ttl=255 time=3.200 msec (DUP!) 86. 64 bytes from 206.168.112.71: icmp_seq=0 ttl=255 time=3.311 msec (DUP!) 87. 64 bytes from 206.168.112.31: icmp_seq=0 ttl=64 time=3.541 msec (DUP!)
Page 53
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
88. 64 bytes from 206.168.112.7: icmp_seq=0 ttl=255 time=3.636 msec (DUP!) 89. ... 90.
[ Team LiB ]
Page 54
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
[ Team LiB ]
1.10 Unix Standards At the time of this writing, the most interesting Unix standardization activity was being done by The Austin Common Standards Revision Group (CSRG). Their efforts have produced roughly 4,000 pages of specifications covering over 1,700 programming interfaces [Josey 2002]. These specifications carry both the IEEE POSIX designation as well as The Open Group's Technical Standard designation. The net result is that you'll likely encounter references to the same standard by various names: ISO/IEC 9945:2002, IEEE Std 1003.1-2001, and the Single Unix Specification Version 3, for example. In this text, we will refer to this standard as simply The POSIX Specification, except in sections like this one where we are discussing specifics of various older standards.
The easiest way to acquire a copy of this consolidated standard is to either order it on CD-ROM or access it via the Web (free of charge). The starting point for either of these methods is
http://www.UNIX.org/version3
Background on POSIX POSIX is an acronym for Portable Operating System Interface. POSIX is not a single standard, but a family of standards being developed by the Institute for Electrical and Electronics Engineers, Inc., normally called the IEEE. The POSIX standards have also been adopted as international standards by ISO and the International Electrotechnical Commission (IEC), called ISO/IEC. The POSIX standards have an interesting history, which we cover only briefly here:
 IEEE Std 1003.1 1988 (317 pages) was the first POSIX standard. It specified the C language interface into a Unix-like kernel and covered the following areas: process primitives (fork, exec, signals, and timers), the environment of a process (user IDs and process groups), files and directories (all the I/O functions), terminal I/O, system databases (password file and group file), and the tar and cpio archive formats.
The first POSIX standard was a trial-use version in 1986 known as "IEEE-IX." The name "POSIX" was suggested by Richard Stallman.
 IEEE Std 1003.1 1990 (356 pages) was next, and it was also known as ISO/IEC 9945 1: 1990. Minimal changes were made from the 1988 to the 1990 version. Appended to the title was "Part 1: System Application Program Interface (API) [C Language]," indicating that this standard was the C language API.
 IEEE Std 1003.2 1992 came next in two volumes (about 1,300 pages). Its title contained "Part 2: Shell and Utilities." This part defined the shell (based on the System V Bourne shell) and about 100 utilities (programs normally executed from a shell, from awk and basename to vi and yacc). Throughout this text, we will refer to this standard as POSIX.2.
 IEEE Std 1003.1b 1993 (590 pages) was originally known as IEEE P1003.4. This was an update to the 1003.1 1990 standard to include the real-time extensions developed by the P1003.4 working group. The 1003.1b 1993 standard added the following items to the 1990 standard: file synchronization, asynchronous I/O, semaphores, memory management (mmap and shared memory), execution scheduling, clocks and timers, and message queues.
Page 55
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
 IEEE Std 1003.1, 1996 Edition [IEEE 1996] (743 pages) came next and included 1003.1 1990 (the base API), 1003.1b 1993 (real-time extensions), 1003.1c 1995 (pthreads), and 1003.1i 1995 (technical corrections to 1003.1b). This standard was also called ISO/IEC 9945 1: 1996. Three chapters on threads were added, along with additional sections on thread synchronization (mutexes and condition variables), thread scheduling, and synchronization scheduling. Throughout this text, we will refer to this standard as POSIX.1. This standard also contains a Foreword stating that ISO/IEC 9945 consists of the following parts:
o Part 1: System API (C language)
o Part 2: Shell and utilities
o Part 3: System administration (under development)
Parts 1 and 2 are what we call POSIX.1 and POSIX.2.
Over one-quarter of the 743 pages are an appendix titled "Rationale and Notes." This appendix contains historical information and reasons why certain features were included or omitted. Often, the rationale is as informative as the official standard.
 IEEE Std 1003.1g: Protocol-independent interfaces (PII) became an approved standard in 2000. Until the introduction of The Single Unix Specification Version 3, this POSIX work was the most relevant to the topics covered in this book. This is the networking API standard and it defines two APIs, which it calls Detailed Network Interfaces (DNIs):
o DNI/Socket, based on the 4.4BSD sockets API
o DNI/XTI, based on the X/Open XPG4 specification
Work on this standard started in the late 1980s as the P1003.12 working group (later renamed P1003.1g). Throughout this text, we will refer to this standard as POSIX.1g.
The current status of the various POSIX standards is available from
http://www.pasc.org/standing/sd11.html
Background on The Open Group The Open Group was formed in 1996 by the consolidation of the X/Open Company (founded in 1984) and the Open Software Foundation (OSF, founded in 1988). It is an international consortium of vendors and end-user customers from industry, government, and academia. Here is a brief background on the standards they produced:
 X/Open published the X/Open Portability Guide, Issue 3 (XPG3) in 1989.
 Issue 4 was published in 1992, followed by Issue 4, Version 2 in 1994. This latest version was also known as "Spec 1170," with the magic number 1,170 being the sum of the number of system interfaces (926), the number of headers (70), and the number of commands (174). The latest name for this set of specifications is the "X/Open Single Unix Specification," although it is also called "Unix 95."
 In March 1997, Version 2 of the Single Unix Specification was announced. Products conforming to this specification were called "Unix 98." We will refer to this specification as just "Unix 98" throughout this text. The number of interfaces required by Unix 98 increases from 1,170 to 1,434, although for a workstation this jumps to 3,030, because it includes the Common Desktop Environment (CDE), which in turn requires the X Window System and the Motif user interface. Details are available in [Josey 1997] and at http://www.UNIX.org/version2. The networking
Page 56
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
services that are part of Unix 98 are defined for both the sockets and XTI APIs. This specification is nearly identical to POSIX.1g.
Unfortunately, Unix 98 referred to networking standards as XNS: X/Open Networking Services. The version of this document that defines sockets and XTI for Unix 98 ([Open Group 1997]) is called "XNS Issue 5." In the networking world XNS has always been an abbreviation for the Xerox Network Systems architecture. We will avoid this use of XNS and refer to this X/Open document as just the Unix 98 network API standard.
Unification of Standards The above brief backgrounds on POSIX and The Open Group both continue with The Austin Group's publication of The Single Unix Specification Version 3, as mentioned at the beginning of this section. Getting over 50 companies to agree on a single standard is certainly a landmark in the history of Unix. Most Unix systems today conform to some version of POSIX.1 and POSIX.2; many comply with The Single Unix Specification Version 3.
Historically, most Unix systems show either a Berkeley heritage or a System V heritage, but these differences are slowly disappearing as most vendors adopt the standards. The main differences still existing deal with system administration, one area that no standard currently addresses.
The focus of this book is on The Single Unix Specification Version 3, with our main focus on the sockets API. Whenever possible we will use the standard functions.
Internet Engineering Task Force (IETF) The Internet Engineering Task Force (IETF) is a large, open, international community of network designers, operators, vendors, and researchers concerned with the evolution of the Internet architecture and the smooth operation of the Internet. It is open to any interested individual.
The Internet standards process is documented in RFC 2026 [Bradner 1996]. Internet standards normally deal with protocol issues and not with programming APIs. Nevertheless, two RFCs (RFC 3493 [Gilligan et al. 2003] and RFC 3542 [Stevens et al. 2003]) specify the sockets API for IPv6. These are informational RFCs, not standards, and were produced to speed the deployment of portable applications by the numerous vendors working on early releases of IPv6. Although standards bodies tend to take a long time, many APIs were standardized in The Single Unix Specification Version 3.
[ Team LiB ]
Page 57
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
[ Team LiB ]
1.11 64-Bit Architectures During the mid to late 1990s, the trend began toward 64-bit architectures and 64-bit software. One reason is for larger addressing within a process (i.e., 64-bit pointers), which can address large amounts of memory (more than 232 bytes). The common programming model for existing 32-bit Unix systems is called the ILP32 model, denoting that integers (I), long integers (L), and pointers (P) occupy 32 bits. The model that is becoming most prevalent for 64-bit Unix systems is called the LP64 model, meaning only long integers (L) and pointers (P) require 64 bits. Figure 1.17 compares these two models.
Figure 1.17. Comparison of number of bits to hold various datatypes for the ILP32 and LP64 models.
From a programming perspective, the LP64 model means we cannot assume that a pointer can be stored in an integer. We must also consider the effect of the LP64 model on existing APIs.
ANSI C invented the size_t datatype, which is used, for example, as the argument to malloc (the number of bytes to allocate), and as the third argument to read and write (the number of bytes to read or write). On a 32-bit system, size_t is a 32-bit value, but on a 64-bit system, it must be a 64-bit value, to take advantage of the larger addressing model. This means a 64-bit system will probably contain a typedef of size_t to be an unsigned long. The networking API problem is that some drafts of POSIX.1g specified that function arguments containing the size of a socket address structures have the size_t datatype (e.g., the third argument to bind and connect). Some XTI structures also had members with a datatype of long (e.g., the t_info and t_opthdr structures). If these had been left as is, both would change from 32-bit values to 64-bit values when a Unix system changes from the ILP32 to the LP64 model. In both instances, there is no need for a 64-bit datatype: The length of a socket address structure is a few hundred bytes at most, and the use of long for the XTI structure members was a mistake.
The solution is to use datatypes designed specifically to handle these scenarios. The sockets API uses the socklen_t datatype for lengths of socket address structures, and XTI uses the t_scalar_t and t_uscalar_t datatypes. The reason for not changing these values from 32 bits to 64 bits is to make it easier to provide binary compatibility on the new 64-bit systems for applications compiled under 32-bit systems.
[ Team LiB ]
Page 58
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
[ Team LiB ]
1.12 Summary Figure 1.5 shows a complete, albeit simple, TCP client that fetches the current time and date from a specified server, and Figure 1.9 shows a complete version of the server. These two examples introduce many of the terms and concepts that are expanded on throughout the rest of the book.
Our client was protocol-dependent on IPv4 and we modified it to use IPv6 instead. But this just gave us another protocol-dependent program. In Chapter 11, we will develop some functions to let us write protocol-independent code, which will be important as the Internet starts using IPv6.
Throughout the text, we will use the wrapper functions developed in Section 1.4 to reduce the size of our code, yet still check every function call for an error return. Our wrapper functions all begin with a capital letter.
The Single Unix Specification Version 3, known by several other names and called simply The POSIX Specification by us, is the confluence of two long-running standards efforts, finally drawn together by The Austin Group.
Readers interested in the history of Unix networking should consult [Salus 1994] for a description of Unix history, and [Salus 1995] for the history of TCP/IP and the Internet.
[ Team LiB ]
Page 59
ABC Amber CHM Converter Trial version, http://www.processtext.com/abcchm.html
[ Team LiB ]
Exercises 1.1 Go through the steps at the end of Section 1.9 to discover information
about your network topology.
1.2 Obtain the source code for the examples in this text (see the Preface). Compile and test the TCP daytime client in Figure 1.5. Run the program a few times, specifying a different IP address as the command-line argument each time.
1.3 Modify the first argument to socket in Figure 1.5 to be 9999. Compile and run the program. What happens? Find the errno value corresponding to the error that is printed. How can you find more information on this error?
1.4 Modify Figure 1.5 by placing a counter in the while loop, counting the number of times read returns a value greater than 0. Print the value of the counter before terminating. Compile and run your new client.
1.5 Modify Figure 1.9 as follows: First, change the port number assigned to the sin_port member from 13 to 9999. Next, change the single call to write into a loop that calls write for each byte of the result string. Compile this modified server and start it running in the background. Next, modify the client from the previous exercise (which prints the counter before terminating), changing the port number assigned to the sin_port member from 13 to 9999. Start this client, specifying the IP address of the host on which the modified server is running as the command-line argument. What value is printed as the client's counter? If possible, also try to run the client and server on different hosts.
