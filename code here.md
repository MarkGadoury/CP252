#include <iostream>
using namespace std;
#include "UDPLabProtocol.h"
#include <WinSock2.h>
#include <Windows.h>
#include <conio.h>
#include <string.h>
#include <string> 
// Link with ws2_32.lib
#pragma comment(lib, "Ws2_32.lib")

const int BUF_SIZE = 64;

int main()
{
	SOCKET s;
	SOCKADDR_IN addr;

	int iWSAStatus;		// Windows Socket API Status
	WSADATA wsaData;

	
	sockaddr_in SenderAddr;
	int SenderAddrSize = sizeof(SenderAddr);
	iWSAStatus = WSAStartup(MAKEWORD(2, 2), &wsaData);

	// what did we get?
	cout << wsaData.szDescription << endl;

	// address family (TCP, UDP), datagram socket (UDP), 0 for default protocol
	s = socket(AF_INET, SOCK_DGRAM, 0);

	if (s == INVALID_SOCKET)
	{
		// see windows socket errors
		cout << "Error creating socket: " << WSAGetLastError() << endl;
		WSACleanup();
		return 1;
	}

	addr.sin_family = AF_INET;
	addr.sin_port = htons(PORT);
	addr.sin_addr = in4addr_any;

	// associate local address with a socket


	// Setup the TCP listening socket
	int error = bind(s, (struct sockaddr *)&addr, sizeof(addr));


	if (error == SOCKET_ERROR) {
		printf("bind failed with error: %d\n", WSAGetLastError());
		closesocket(s);
		WSACleanup();
		return 1;
	}

	int iResult = 0;
	int *inlen = 0;
	while (!_kbhit())
	{
		
		
		char buf[BUF_SIZE];
		for (int i = 0; i < BUF_SIZE; i++)
		{
			buf[i] = NULL;
		}

		REQUEST* request = new REQUEST();
		
		const int GET_TYPE = 0;
		const int GET_ID = 1;
		const int GET_COUNT = 2;
		const int GET_NUM = 3;
		int timeout = 5000;
		setsockopt(s, SOL_SOCKET, SO_RCVTIMEO, (char *)&timeout, sizeof(timeout));
		cout << "Receiving....." << endl;
		iResult = recvfrom(s, buf, BUF_SIZE, 0, (SOCKADDR *)&SenderAddr, &SenderAddrSize);
		string sa;
		request = (REQUEST*)&buf;
		RESPONSE response;
		int Result = 0;
		
		char str[INET_ADDRSTRLEN];
		inet_ntop(AF_INET, &(SenderAddr.sin_addr), str, INET_ADDRSTRLEN);
		
		switch (request->type)
		{
		case TYPE_ADD:
			for (int i = 0; i < request->count; i++)
			{
				Result += request->numbers[i];
			}
			printf("%s\n", str);
			cout <<  " sent an Add" << endl;
			
			response.type = request->type;
			response.result = Result;
			response.id = request->id;
			iResult = sendto(s, (char*)&response, BUF_SIZE, 0, (SOCKADDR *)&SenderAddr, SenderAddrSize);
			break;

		case TYPE_MULTIPLY:
			Result = 1;
			for (int i = request->count-1; i >= 0; i--)
			{
				Result = Result * request->numbers[i];
			}
			printf("%s\n", str);
			cout << " sent a Multiply" << endl;
			response.type = request->type;
			response.result = Result;
			response.id = request->id;
			iResult = sendto(s, (char*)&response, BUF_SIZE, 0, (SOCKADDR *)&SenderAddr, SenderAddrSize);
			break;

		case TYPE_ERROR:
			
			break;
		default:
			break;

		}
		
		
	}
	closesocket(s);
	WSACleanup();
	

	return 0;
}
