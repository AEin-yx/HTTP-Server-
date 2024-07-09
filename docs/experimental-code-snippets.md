# Server Functions

1.Create a socket  
2.Bind it to a port(Here Port is fixed and also manual)  
3.Listen to new connection  
4.If a new Connection is present,Bind yourself to same IP,Port and create a new socket and connect to the client so that the previous socket could still be listening to new connections  
5.send and receive data using recv(),send(),sendto(),recvfrom() 6.Once every is done, we close the server by giving EOF(End of File) or closeSocket()  

In terms of functions we can say:  
1.Initialize WSA-WSAStartup()  
2.Create a socket-Socket()  
3.Bind the socket-Bind()  
4.Listen on the socket-listen()  
5.Accept a connection -accept(), connect()  
6.Send and receive data -recv(), send(), recvfrom(),sendto()  
7.Disconnect-closesocket()  

___
Load DLL by using WSAStartup  
We load DLL because all lower level repetitive tasks to manage communication is done by DLL.  

 ```
int WSAStartup(WORD wVersionRequested,LPWSADATA IpWSAData);
```
// LPWSADATA,WORD are windows datatype usually in the form of capital letters  
 // wVersionRequested is the highest version of windows socket specification that the caller can use.  
 // IpWSAData recieves a pointer to LPWSADATA datastructure that recieves details of windows Sockets implementations  
// If it is successful, WSAStartup returns 0  

what does LPWSADATA data structure looks like?  
```
typedef struct WSAData{
	WORD wVersion;
    WORD wHighVersion;
    char szDescription [WSADESCRIPTION_LEN+1];
    char szSystemStatus [WSASYS_STATUS_LEN+1];
    unsigned short iMaxSockets;
    unsigned short iMaxUdpDg;
    char FAR *IpVendorInfo;
}
```

Finding if a DLL was found or not:  
```
WSADATA wsaData;
int wsaerr;
WORD wVersion Requested = MAKEWORD(2, 2);
wsaerr = WSAStartup(wVersionRequested, &wsaData); 
if (wsaerr != 0){
    cout << "The Winsock dll not found!" << endl; 
    return 0;
}
else{
    cout << "The Winsock dll was found!" << endl;
    cout<<"The Status:"<<wsaData.szSystemStatus<<endl;
}
```
---
Socket()  
The socket function creates a socket that is bound to a specific transport service provider.  
```
SOCKET WSAAPI socket(int af, int type, int protocol);
```
af: The address family specification (AF_INET for UDP or TCP).  
type: The type specification for the new socket (SOCK_STREAM for TCP and SOCK_DGRAM for UDP).  
protocol: The protocol to be used (IPPROTO_TCP for TCP).  
*If the output was 0, we have a socket.*  

Also, a socket cleanup is also important. We call this deregistering DLL(Ws2_32.dll) after we are   done with socket use and also why it is done is because we need to prevent conflicts with other applications,resolve issues, maintains system integrity, prevents errors and facilitate uninstallations.
```
SOCKET serverSocket = INVALID_SOCKET;
serverSocket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
if (serverSocket== INVALID_SOCKET){
    cout << "Error at socket(): " << WSAGetLastError() << endl; WSACleanup();
    return 0;
}
else {
    cout << "socket() is OK!" << endl;
}
//...
WSACleanup();
```

Also, a socket is closed in this way:  
Note:Ensure that the socket was opened before  
```
int closesocket(Socket s);
SOCKET serverSocket;
serverSocket = INVALID_SOCKET;
serverSocket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
//... Process socket
closesocket(serverSocket);
```
___
Now we will do binding  
binding associates a local address with a socket  
Binding a socket to a local address means associating a local IP address with a socket. This local   address can be a specific IP address, such as 127.0.0.1, or it can be INADDR_ANY or ::, which means  the socket will listen on all available network interfaces.  
If local address is 127.0.0.1 bind to socket, it will only accept connections from local machine   because 127.0.0.1 is also called loopback address and if the socked is bind to ::, then it will accept   incoming connection from everywhere  
```
int bind(SOCKET s, const struct sockaddr* name, int socklen);
```
s:descriptor identifying an unbound socket  
name: address to assign to the socket from the sockaddr structure  
socklen:length in the bytes of the address structure  