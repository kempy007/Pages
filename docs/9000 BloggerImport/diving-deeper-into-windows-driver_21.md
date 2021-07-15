---
title: 'Diving deeper into windows - Driver development - Status Update 2'
date: 2015-05-21T12:20:00.001+01:00
draft: false
aliases: [ "/2015/05/diving-deeper-into-windows-driver_21.html" ]
parent: Blogger
---
#### date: 2015-05-21

Having figured out how to deploy the skeleton KMDF project and step through the code, and after learning how to implement the callbacks recommended as per http://www.adlice.com/making-an-antivirus-engine-the-guidelines/  
  
I attempted to grab the file paths of new processes / images etc and begin by generating the sha256 hashes. however I had hit a roadblock in that some of the header files that would make this easy, like windows.h for PBYTES are not allowed in the KMDF framework, but are in the UMDF2.0.  
  
Now I have to go figure out either a method for inter driver communications or if UMDF2.0 is high enough to request callbacks, that can also block new processes when required.  
  
For now, I'll throw in some of what I have changed to get the callbacks working. The two files in the skeleton KMDF project I modified are below, I've surrounded then with comments and regions but you can always use winmerge to highlight the changes I've made.  
  
Driver.c  
  
```
/\*++  
  
Module Name:  
  
    driver.c  
  
Abstract:  
  
    This file contains the driver entry points and callbacks.  
  
Environment:  
  
    Kernel-mode Driver Framework  
  
\--\*/  
  
  
/\* ########################################  
#########   Kempy's Notes  ################  
###########################################  
  
hit F7 on solution which will install driver.  
right click driver project > debug > step into new instance  
request a break.  
kd> lm  
only nt listed so run  
kd> .reload  
wait for ages, rerun lm  
kd > lm  
should list pages of modules.  
then set my break points on code view.  
f5 to continue  
Often have to use debugger commandline to disable breakpoints etc.  
  
  
windows 8.1 kit includes bcrypt.h, option to explore for hashing functions also referred to as CNG cryptography Next Generation.  
  
\*/  
  
#include "driver.h"  
#include "driver.tmh"  
  
  
#ifdef ALLOC\_PRAGMA  
#pragma alloc\_text (INIT, DriverEntry)  
#pragma alloc\_text (PAGE, KMDFDriver3EvtDeviceAdd)  
#pragma alloc\_text (PAGE, KMDFDriver3EvtDriverContextCleanup)  
#endif  
  
  
NTSTATUS  
DriverEntry(  
    \_In\_ PDRIVER\_OBJECT  DriverObject,  
    \_In\_ PUNICODE\_STRING RegistryPath  
    )  
/\*++  
  
Routine Description:  
    DriverEntry initializes the driver and is the first routine called by the  
    system after the driver is loaded. DriverEntry specifies the other entry  
    points in the function driver, such as EvtDevice and DriverUnload.  
  
Parameters Description:  
  
    DriverObject - represents the instance of the function driver that is loaded  
    into memory. DriverEntry must initialize members of DriverObject before it  
    returns to the caller. DriverObject is allocated by the system before the  
    driver is loaded, and it is released by the system after the system unloads  
    the function driver from memory.  
  
    RegistryPath - represents the driver specific path in the Registry.  
    The function driver can use the path to store driver related data between  
    reboots. The path does not store hardware instance specific data.  
  
Return Value:  
  
    STATUS\_SUCCESS if successful,  
    STATUS\_UNSUCCESSFUL otherwise.  
  
\--\*/  
{  
    WDF\_DRIVER\_CONFIG config;  
    NTSTATUS status;  
    WDF\_OBJECT\_ATTRIBUTES attributes;  
  
    //  
    // Initialize WPP Tracing  
    //  
    WPP\_INIT\_TRACING( DriverObject, RegistryPath );  
  
    TraceEvents(TRACE\_LEVEL\_INFORMATION, TRACE\_DRIVER, "%!FUNC! Entry");  
  
    //  
    // Register a cleanup callback so that we can call WPP\_CLEANUP when  
    // the framework driver object is deleted during driver unload.  
    //  
    WDF\_OBJECT\_ATTRIBUTES\_INIT(&attributes);  
    attributes.EvtCleanupCallback = KMDFDriver3EvtDriverContextCleanup;  
  
    WDF\_DRIVER\_CONFIG\_INIT(&config,  
                           KMDFDriver3EvtDeviceAdd  
                           );  
  
    status = WdfDriverCreate(DriverObject,  
                             RegistryPath,  
                             &attributes,  
                             &config,  
                             WDF\_NO\_HANDLE  
                             );  
  
    if (!NT\_SUCCESS(status)) {  
        TraceEvents(TRACE\_LEVEL\_ERROR, TRACE\_DRIVER, "WdfDriverCreate failed %!STATUS!", status);  
        WPP\_CLEANUP(DriverObject);  
        return status;  
    }  
  
#pragma region kempy\_register\_callbacks  
 /\*##############################################  
 ###########  Add my own callbacks here #########  
 \*/  
 status = PsSetCreateProcessNotifyRoutine(CreateProcessNotifyRoutine, FALSE);  
  
 status = PsSetCreateProcessNotifyRoutineEx(CreateProcessNotifyRoutineEx, FALSE); // added  this and driver is getting code 37, access denied.   
 // have created a driver certificate now, which did not help  
 // Added to driver properties 'configuration properties' > 'linker' > 'command line', 'additional options' /INTEGRITYCHECK   
 // integritycheck option fixed driver loading error!  
  
 /\*##### end section ####  
 ########################  
 \*/  
#pragma endregion  
  
    TraceEvents(TRACE\_LEVEL\_INFORMATION, TRACE\_DRIVER, "%!FUNC! Exit");  
  
    return status;  
}  
  
NTSTATUS  
KMDFDriver3EvtDeviceAdd(  
    \_In\_    WDFDRIVER       Driver,  
    \_Inout\_ PWDFDEVICE\_INIT DeviceInit  
    )  
/\*++  
Routine Description:  
  
    EvtDeviceAdd is called by the framework in response to AddDevice  
    call from the PnP manager. We create and initialize a device object to  
    represent a new instance of the device.  
  
Arguments:  
  
    Driver - Handle to a framework driver object created in DriverEntry  
  
    DeviceInit - Pointer to a framework-allocated WDFDEVICE\_INIT structure.  
  
Return Value:  
  
    NTSTATUS  
  
\--\*/  
{  
    NTSTATUS status;  
  
    UNREFERENCED\_PARAMETER(Driver);  
  
    PAGED\_CODE();  
  
    TraceEvents(TRACE\_LEVEL\_INFORMATION, TRACE\_DRIVER, "%!FUNC! Entry");  
  
    status = KMDFDriver3CreateDevice(DeviceInit);  
  
    TraceEvents(TRACE\_LEVEL\_INFORMATION, TRACE\_DRIVER, "%!FUNC! Exit");  
  
    return status;  
}  
  
#pragma region kempy\_callback\_routines  
/\*###########################################################  
#############  drop in callback routines here  ##############  
\*/  
VOID CreateProcessNotifyRoutine(IN HANDLE ParentId, IN HANDLE ProcessId, IN BOOLEAN Create) {  
 PAGED\_CODE();  
  
 DbgPrint("CreateProcessNotifyRoutine called with ParentId = 0x%p, ProcessId = 0x%p, Create = %d\\n",  
  ParentId,  
  ProcessId,  
  Create);  
}  
  
VOID CreateProcessNotifyRoutineEx(IN OUT PEPROCESS Process,  
        IN HANDLE ProcessId,  
        IN OPTIONAL PPS\_CREATE\_NOTIFY\_INFO CreateInfo) {  
   
 PAGED\_CODE();  
  
 UNREFERENCED\_PARAMETER(CreateInfo);  
  
 DbgPrint("CreateProcessNotifyRoutineEx called with Process = 0x%p, ProcessId = 0x%p\\n",  
  Process,  
  ProcessId);  
   
}  
  
  
  
/\*##### end section ####  
########################  
\*/  
#pragma endregion  
  
VOID  
KMDFDriver3EvtDriverContextCleanup(  
    \_In\_ WDFOBJECT DriverObject  
    )  
/\*++  
Routine Description:  
  
    Free all the resources allocated in DriverEntry.  
  
Arguments:  
  
    DriverObject - handle to a WDF Driver object.  
  
Return Value:  
  
    VOID.  
  
\--\*/  
{  
    UNREFERENCED\_PARAMETER(DriverObject);  
  
    PAGED\_CODE ();  
  
#pragma region Kempy\_cleanup\_section  
 /\*####################################################  
 ###########  cleanup my own callbacks here.  #########  
 \*/  
 PsSetCreateProcessNotifyRoutine(CreateProcessNotifyRoutine, TRUE);  
 PsSetCreateProcessNotifyRoutineEx(CreateProcessNotifyRoutineEx, TRUE);  
  
 /\*##### end section ####  
 ########################  
 \*/  
#pragma endregion  
  
  
    TraceEvents(TRACE\_LEVEL\_INFORMATION, TRACE\_DRIVER, "%!FUNC! Entry");  
  
    //  
    // Stop WPP Tracing  
    //  
    WPP\_CLEANUP( WdfDriverWdmGetDriverObject(DriverObject) );  
  
}  
  

```  
Driver.h  
  
```
/\*++  
  
Module Name:  
  
    driver.h  
  
Abstract:  
  
    This file contains the driver definitions.  
  
Environment:  
  
    Kernel-mode Driver Framework  
  
\--\*/  
  
#define INITGUID  
  
#include   
#include   
  
#include "device.h"  
#include "queue.h"  
#include "trace.h"  
  
//  
// WDFDRIVER Events  
//  
  
DRIVER\_INITIALIZE DriverEntry;  
EVT\_WDF\_DRIVER\_DEVICE\_ADD KMDFDriver3EvtDeviceAdd;  
EVT\_WDF\_OBJECT\_CONTEXT\_CLEANUP KMDFDriver3EvtDriverContextCleanup;  
  
#pragma region kempy\_declare\_functions  
/\*##############################################  
###########  Declare Functions #########  
\*/  
VOID     CreateProcessNotifyRoutine(IN HANDLE ParentId, IN HANDLE ProcessId, IN BOOLEAN Create);  
VOID     CreateProcessNotifyRoutineEx(IN OUT PEPROCESS Process, IN HANDLE ProcessId,  
 IN OPTIONAL PPS\_CREATE\_NOTIFY\_INFO CreateInfo);  
/\*##### end section ####  
########################  
\*/  
#pragma endregion  
  

```