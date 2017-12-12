# laba2_teh_prog
#include "stdafx.h"
#include <iostream>
#include <windows.h>
#include <conio.h>
#include <stdlib.h>
#include <time.h>

using namespace std;

class Semaphore
{
	CRITICAL_SECTION critical_sec;
	int max_count;
	int count;
	HANDLE event;
public:
	Semaphore(int max_count) :max_count(max_count), count(0)
	{
		InitializeCriticalSection(&critical_sec);
		event = CreateEvent(NULL, TRUE, FALSE, NULL);
	}
	~Semaphore()
	{
		DeleteCriticalSection(&critical_sec);
		CloseHandle(event);
	}
	void lock()
	{
		time_t start;
		start = clock();
		EnterCriticalSection(&critical_sec);
		count++;
		if (count <= max_count){
			Sleep(1000);
			cout << "(lock()_sema)critical_sec is locked, total number of threads:" << count << endl;
			LeaveCriticalSection(&critical_sec);
			cout << "Time = " << (clock() - start) / CLOCKS_PER_SEC << endl;
			return;
		}
		cout << "(lock())critical_sec is unlocked, total number of threads:" << count << endl;
		LeaveCriticalSection(&critical_sec);
		WaitForSingleObject(event, INFINITE);
	}
	void unlock(){
		EnterCriticalSection(&critical_sec);
		count--;
		if (count != 0){
			SetEvent(event);
			cout << "(unlock())critical_sec is unlocked, total number of threads:" << count << endl;
		}
		LeaveCriticalSection(&critical_sec);
	}
};

Semaphore *sema;

DWORD WINAPI
foo(PVOID)
{
	sema->lock();
	Sleep(1000);
	sema->unlock();
	return 0;
}

int
main()
{
	sema = new Semaphore(6);
	HANDLE thread[12];

	for (int i = 0; i < 12; i++)
		thread[i] = CreateThread(NULL, 0, foo, NULL, 0, NULL);

	WaitForMultipleObjects(12, thread, TRUE, INFINITE);

	for (int i = 0; i < 12; i++)
		CloseHandle(thread[i]);

	cout << endl << "Done. Press any key to exit." << endl;
	_getch();
	return 0;
}
