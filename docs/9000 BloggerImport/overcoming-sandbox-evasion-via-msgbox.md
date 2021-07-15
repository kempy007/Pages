---
title: 'Overcoming sandbox evasion via msgbox'
date: 2015-06-22T22:54:00.001+01:00
draft: false
aliases: [ "/2015/06/overcoming-sandbox-evasion-via-msgbox.html" ]
parent: Blogger
---
#### date: 2015-06-22

Well it seems that a simple msgbox is able to stump all sorts of vendors who employ sandboxing.  
So I've decided to try and do something about this, and my first attempt has resulted in a brute force mouse clicker. However I still need to detect when to invoke my BruteClick wrath. I'm sure a Real programmer would find a more elegant approach.  
Anyhow I've no time left so here's the code so far;  
  
  
```
/\* # Brute force clicking app for killing dialog boxes  
   #  
 \*/  
  
  
#include <Windows.h>  
#include <vector>  
#pragma comment (lib, "user32.lib")  
#include <stdio.h>  
#include <stdlib.h>  
#include <time.h>  
#include <conio.h>  
#include <string.h>  
  
  
using namespace System;  
using namespace std;  
   
  
void MouseSetup(INPUT \*buffer, long SCREEN\_WIDTH, long SCREEN\_HEIGHT)  
{  
 buffer->type = INPUT\_MOUSE;  
 buffer->mi.dx = (0 \* (0xFFFF / SCREEN\_WIDTH));  
 buffer->mi.dy = (0 \* (0xFFFF / SCREEN\_HEIGHT));  
 buffer->mi.mouseData = 0;  
 buffer->mi.dwFlags = MOUSEEVENTF\_ABSOLUTE;  
 buffer->mi.time = 0;  
 buffer->mi.dwExtraInfo = 0;  
}  
  
  
void MouseMoveAbsolute(INPUT \*buffer, int x, int y, long SCREEN\_WIDTH, long SCREEN\_HEIGHT)  
{  
 buffer->mi.dx = (x \* (0xFFFF / SCREEN\_WIDTH));  
 buffer->mi.dy = (y \* (0xFFFF / SCREEN\_HEIGHT));  
 buffer->mi.dwFlags = (MOUSEEVENTF\_ABSOLUTE | MOUSEEVENTF\_MOVE);  
  
 SendInput(1, buffer, sizeof(INPUT));  
}  
  
  
void MouseClick(INPUT \*buffer)  
{  
 buffer->mi.dwFlags = (MOUSEEVENTF\_ABSOLUTE | MOUSEEVENTF\_LEFTDOWN);  
 SendInput(1, buffer, sizeof(INPUT));  
  
 Sleep(10);  
  
 buffer->mi.dwFlags = (MOUSEEVENTF\_ABSOLUTE | MOUSEEVENTF\_LEFTUP);  
 SendInput(1, buffer, sizeof(INPUT));  
}  
  
void main()  
{  
 // start by getting the screen resolution  
 long fScreenWidth = GetSystemMetrics(SM\_CXSCREEN);  
 long fScreenHeight = GetSystemMetrics(SM\_CYSCREEN);  
  
 // http://msdn.microsoft.com/en-us/library/ms646260(VS.85).aspx  
 // If MOUSEEVENTF\_ABSOLUTE value is specified, dx and dy contain normalized absolute coordinates between 0 and 65,535.  
 // The event procedure maps these coordinates onto the display surface.  
 // Coordinate (0,0) maps onto the upper-left corner of the display surface, (65535,65535) maps onto the lower-right corner.  
 long startWidth = fScreenWidth / 4;  
 long endWidth = fScreenWidth / 4 \* 3;  
  
 long startHeight = fScreenHeight / 2;  
 long endHeight = fScreenHeight / 4 \* 3;  
  
 INPUT buffer\[1\];  
  
 MouseSetup(buffer,fScreenWidth, fScreenHeight);  
  
 long wX = startWidth;  
 long hX = startHeight;  
  
 // need to hide this window so as not to loose focus.  
 ShowWindow(GetConsoleWindow(), SW\_HIDE);  
  
 while (hX < endHeight)  
 {  
  while (wX < endWidth)  
  {  
   MouseMoveAbsolute(buffer, wX, hX, fScreenWidth, fScreenHeight);  
   MouseClick(buffer);  
   wX = wX + 10;  
  }  
  wX = startWidth;  
  hX = hX + 10;  
 }  
   
  
 //MouseMoveAbsolute(buffer, 700, 450, fScreenWidth, fScreenHeight);  
 //MouseClick(buffer);  
  
 //return 0;  
}
```