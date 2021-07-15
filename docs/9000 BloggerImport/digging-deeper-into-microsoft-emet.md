---
title: 'Digging deeper into Microsoft EMET protection.'
date: 2015-11-19T17:53:00.000Z
draft: false
aliases: [ "/2015/11/digging-deeper-into-microsoft-emet.html" ]
parent: Blogger
---
#### date: 2015-11-19

Download and extract to %windir%\\system32 listdlls.exe from https://technet.microsoft.com/en-gb/sysinternals/bb896656.aspx  
  
open notepad protected by emet5.5, note the pid from task manager or emet console.  
  
run following command, changing pid for one noted  
c:\\Windows\\System32>Listdlls.exe \[PID\] > z:\\emetProtectedNotepad.txt  
  
remove the emet protection by deleteing notepad from apps list in emet console.  
rerun the following command after closing notepad and restarting it noting the new pid and using in following command.  
c:\\Windows\\System32>Listdlls.exe \[PID\] > z:\\notProtectedNotepad.txt  
  
Open the two files in winmerge highlights the following dlls different.  
  
 C:\\Windows\\system32\\apphelp.dll  334kb  
 C:\\Windows\\AppPatch\\AppPatch64\\EMET64.dll  1053kb  
  
will assume that the following is for 32bit apps;  
  
 C:\\Windows\\AppPatch\\EMET.dll 745kb  
  
Now open visual studio developer command window.  
run;  
C:\\Program Files (x86)\\Microsoft Visual Studio 14.0>dumpbin /export C:\\Windows\\system32\\apphelp.dll > "z:\\Emet Reverse engineering\\apphelpdll.dumpbin.txt"  
  
C:\\Program Files (x86)\\Microsoft Visual Studio 14.0>dumpbin /exports C:\\Windows\\AppPatch\\AppPatch64\\EMET64.dll > "z:\\Emet Reverse engineering\\EMET64.dumpbin.txt"  
  
C:\\Program Files (x86)\\Microsoft Visual Studio 14.0>dumpbin /exports C:\\Windows\\AppPatch\\EMET.dll > "z:\\Emet Reverse engineering\\EMET.dumpbin.txt"  
  
These files list the following;  
  
Microsoft (R) COFF/PE Dumper Version 14.00.23026.0  
Copyright (C) Microsoft Corporation.  All rights reserved.  
  
  
Dump of file C:\\Windows\\system32\\apphelp.dll  
  
File Type: DLL  
  
  Section contains the following exports for apphelp.dll  
  
    00000000 characteristics  
    56324A6A time date stamp Thu Oct 29 16:33:46 2015  
        0.00 version  
           1 ordinal base  
         195 number of functions  
         195 number of names  
  
    ordinal hint RVA      name  
  
          1    0 0001C07C AllowPermLayer  
          2    1 00011574 ApphelpCheckExe  
          3    2 0001A3D0 ApphelpCheckIME  
          4    3 0001A8B0 ApphelpCheckInstallShieldPackage  
          5    4 000073C0 ApphelpCheckModule  
          6    5 0001AC4C ApphelpCheckMsiPackage  
          7    6 0001A810 ApphelpCheckRunApp  
          8    7 0000B3F0 ApphelpCheckRunAppEx  
          9    8 0000726C ApphelpCheckShellObject  
         10    9 00001010 ApphelpCreateAppcompatData  
         11    A 0001AF90 ApphelpFixMsiPackage  
         12    B 0001B274 ApphelpFixMsiPackageExe  
         13    C 0001C5B4 ApphelpFreeFileAttributes  
         14    D 0001C5A8 ApphelpGetFileAttributes  
         15    E 0001BB98 ApphelpGetMsiProperties  
         16    F 0001A3F8 ApphelpGetNTVDMInfo  
         17   10 0001C5DC ApphelpGetShimDebugLevel  
         18   11 0001BD00 ApphelpParseModuleData  
         19   12 0001BCD0 ApphelpQueryModuleData  
         20   13 0001BE78 ApphelpQueryModuleDataEx  
         21   14 0001C578 ApphelpShowDialog  
         22   15 0001C1D8 ApphelpUpdateCacheEntry  
         23   16 0001C528 GetPermLayers  
         24   17 00006A18 SE\_DllLoaded  
         25   18 000017C4 SE\_DllUnloaded  
         26   19 0001CF90 SE\_DynamicShim  
         27   1A 0001D020 SE\_GetHookAPIs  
         28   1B 0001D190 SE\_GetMaxShimCount  
         29   1C 0001CDB8 SE\_GetProcAddressIgnoreIncExc  
         30   1D 0001CE10 SE\_GetProcAddressLoad  
         31   1E 0001D1A0 SE\_GetShimCount  
         32   1F 00001820 SE\_InstallAfterInit  
         33   20 00005C7C SE\_InstallBeforeInit  
         34   21 00005978 SE\_IsShimDll  
         35   22 0000980C SE\_LdrEntryRemoved  
         36   23 00001C94 SE\_ProcessDying  
         37   24 0001F958 SdbAddLayerTagRefToQuery  
         38   25 00021AAC SdbApphelpNotify  
         39   26 000219B0 SdbApphelpNotifyEx  
         40   27 000218E0 SdbApphelpNotifyEx2  
         41   28 000237F8 SdbBeginWriteListTag  
         42   29 000270B0 SdbBuildCompatEnvVariables  
         43   2A 00022554 SdbCloseApphelpInformation  
         44   2B 0000F680 SdbCloseDatabase  
         45   2C 000242E8 SdbCloseDatabaseWrite  
         46   2D 00027DCC SdbCloseLocalDatabase  
         47   2E 00024650 SdbCommitIndexes  
         48   2F 00023584 SdbCreateDatabase  
         49   30 00022EEC SdbCreateHelpCenterURL  
         50   31 00028B64 SdbCreateMsiTransformFile  
         51   32 000242F4 SdbDeclareIndex  
         52   33 000253B8 SdbDeletePermLayerKeys  
         53   34 000278E0 SdbDumpSearchPathPartCaches  
         54   35 00023870 SdbEndWriteListTag  
         55   36 0002867C SdbEnumMsiTransforms  
         56   37 00021DD0 SdbEscapeApphelpURL  
         57   38 00028EF4 SdbFindCustomActionForPackage  
         58   39 000290E0 SdbFindFirstDWORDIndexedTag  
         59   3A 00028FD8 SdbFindFirstGUIDIndexedTag  
         60   3B 000284F0 SdbFindFirstMsiPackage  
         61   3C 00028464 SdbFindFirstMsiPackage\_Str  
         62   3D 00029410 SdbFindFirstNamedTag  
         63   3E 0000D1F0 SdbFindFirstStringIndexedTag  
         64   3F 000035E0 SdbFindFirstTag  
         65   40 0000B2C0 SdbFindFirstTagRef  
         66   41 00028E6C SdbFindMsiPackageByID  
         67   42 000291A4 SdbFindNextDWORDIndexedTag  
         68   43 000290A4 SdbFindNextGUIDIndexedTag  
         69   44 00028560 SdbFindNextMsiPackage  
         70   45 00002910 SdbFindNextStringIndexedTag  
         71   46 00003B98 SdbFindNextTag  
         72   47 000071C0 SdbFindNextTagRef  
         73   48 00010C04 SdbFormatAttribute  
         74   49 00027720 SdbFreeDatabaseInformation  
         75   4A 00010A50 SdbFreeFileAttributes  
         76   4B 00007610 SdbFreeFileInfo  
         77   4C 0001E914 SdbFreeFlagInfo  
         78   4D 00029A70 SdbGUIDFromString  
         79   4E 00002CC8 SdbGUIDToString  
         80   4F 000257D4 SdbGetAppCompatDataSize  
         81   50 000257C0 SdbGetAppPatchDir  
         82   51 0000EAA0 SdbGetBinaryTagData  
         83   52 0001E81C SdbGetDatabaseGUID  
         84   53 0000F7B8 SdbGetDatabaseID  
         85   54 00007EE0 SdbGetDatabaseInformation  
         86   55 00027740 SdbGetDatabaseInformationByName  
         87   56 00029EDC SdbGetDatabaseMatch  
         88   57 000275E0 SdbGetDatabaseVersion  
         89   58 0000EE40 SdbGetDllPath  
         90   59 00010708 SdbGetEntryFlags  
         91   5A 00010EE0 SdbGetFileAttributes  
         92   5B 000297B0 SdbGetFileImageType  
         93   5C 000298FC SdbGetFileImageTypeEx  
         94   5D 0000523C SdbGetFileInfo  
         95   5E 00006984 SdbGetFirstChild  
         96   5F 00027910 SdbGetImageType  
         97   60 00006D0C SdbGetIndex  
         98   61 0000E940 SdbGetItemFromItemRef  
         99   62 00026FE8 SdbGetLayerName  
        100   63 0001F550 SdbGetLayerTagRef  
        101   64 0001E7D8 SdbGetLocalPDB  
        102   65 0001F820 SdbGetMatchingExe  
        103   66 00028CD8 SdbGetMsiPackageInformation  
        104   67 0001F4CC SdbGetNamedLayer  
        105   68 00005A50 SdbGetNextChild  
        106   69 0000F33C SdbGetNthUserSdb  
        107   6A 0000F1BC SdbGetPDBFromGUID  
        108   6B 000089F0 SdbGetPermLayerKeys  
        109   6C 00009D1C SdbGetShowDebugInfoOption  
        110   6D 00020268 SdbGetShowDebugInfoOptionValue  
        111   6E 000267B0 SdbGetStandardDatabaseGUID  
        112   6F 00007FB0 SdbGetStringTagPtr  
        113   70 00003020 SdbGetTagDataSize  
        114   71 000094C0 SdbGetTagFromTagID  
        115   72 0002B7B0 SdbGrabMatchingInfo  
        116   73 0002B7DC SdbGrabMatchingInfoEx  
        117   74 0001BC60 SdbInitDatabase  
        118   75 000076B0 SdbInitDatabaseEx  
        119   76 0000FD50 SdbIsNullGUID  
        120   77 00026734 SdbIsStandardDatabase  
        121   78 0001E804 SdbIsTagrefFromLocalDB  
        122   79 0001E7EC SdbIsTagrefFromMainDB  
        123   7A 0002039C SdbLoadString  
        124   7B 00002A30 SdbMakeIndexKeyFromString  
        125   7C 00021FE0 SdbOpenApphelpDetailsDatabase  
        126   7D 00022084 SdbOpenApphelpDetailsDatabaseSP  
        127   7E 00022318 SdbOpenApphelpInformation  
        128   7F 000221C4 SdbOpenApphelpInformationByID  
        129   80 0002049C SdbOpenApphelpResourceFile  
        130   81 000095C8 SdbOpenDatabase  
        131   82 000202B0 SdbOpenDbFromGuid  
        132   83 00027D9C SdbOpenLocalDatabase  
        133   84 00008534 SdbPackAppCompatData  
        134   85 00022824 SdbQueryApphelpInformation  
        135   86 0001EAC4 SdbQueryBlockUpgrade  
        136   87 00001CCC SdbQueryContext  
        137   88 0002A09C SdbQueryData  
        138   89 0002A704 SdbQueryDataEx  
        139   8A 0002A0CC SdbQueryDataExTagID  
        140   8B 0001E8D0 SdbQueryFlagInfo  
        141   8C 0000B1E0 SdbQueryFlagMask  
        142   8D 0002BF08 SdbQueryName  
        143   8E 0001EB60 SdbQueryReinstallUpgrade  
        144   8F 00002090 SdbReadApphelpData  
        145   90 00020B88 SdbReadApphelpDetailsData  
        146   91 00029AAC SdbReadBYTETag  
        147   92 00029B54 SdbReadBYTETagRef  
        148   93 0000F864 SdbReadBinaryTag  
        149   94 0000EA28 SdbReadDWORDTag  
        150   95 00029C20 SdbReadDWORDTagRef  
        151   96 0002A7F0 SdbReadEntryInformation  
        152   97 00028850 SdbReadMsiTransformInfo  
        153   98 00027EB0 SdbReadPatchBits  
        154   99 00010280 SdbReadQWORDTag  
        155   9A 00010230 SdbReadQWORDTagRef  
        156   9B 0000E838 SdbReadStringTag  
        157   9C 0000E7E0 SdbReadStringTagRef  
        158   9D 00002D90 SdbReadWORDTag  
        159   9E 00029BB8 SdbReadWORDTagRef  
        160   9F 00026580 SdbRegisterDatabase  
        161   A0 00025ED8 SdbRegisterDatabaseEx  
        162   A1 00007598 SdbReleaseDatabase  
        163   A2 0001FAE0 SdbReleaseMatchingExe  
        164   A3 00026A94 SdbResolveDatabase  
        165   A4 00022700 SdbSetApphelpDebugParameters  
        166   A5 000256A8 SdbSetEntryFlags  
        167   A6 00027920 SdbSetImageType  
        168   A7 00024DBC SdbSetPermLayerKeys  
        169   A8 00021BD0 SdbShowApphelpDialog  
        170   A9 0000E53C SdbShowApphelpFromQuery  
        171   AA 0002469C SdbStartIndexing  
        172   AB 000246C0 SdbStopIndexing  
        173   AC 0001EFC8 SdbStringDuplicate  
        174   AD 0001F088 SdbStringReplace  
        175   AE 0001F2D4 SdbStringReplaceArray  
        176   AF 0000B350 SdbTagIDToTagRef  
        177   B0 0000A6D0 SdbTagRefToTagID  
        178   B1 00010B58 SdbTagToString  
        179   B2 0000F08C SdbUnpackAppCompatData  
        180   B3 00026590 SdbUnregisterDatabase  
        181   B4 00023A38 SdbWriteBYTETag  
        182   B5 00023BD8 SdbWriteBinaryTag  
        183   B6 00023C14 SdbWriteBinaryTagFromFile  
        184   B7 00023AD0 SdbWriteDWORDTag  
        185   B8 000239F8 SdbWriteNULLTag  
        186   B9 00023B1C SdbWriteQWORDTag  
        187   BA 00023E30 SdbWriteStringRefTag  
        188   BB 00023B68 SdbWriteStringTag  
        189   BC 00023E7C SdbWriteStringTagDirect  
        190   BD 00023A84 SdbWriteWORDTag  
        191   BE 0001C534 SetPermLayerState  
        192   BF 0001C4BC SetPermLayers  
        193   C0 000096CC ShimDbgPrint  
        194   C1 0001C5D0 ShimDumpCache  
        195   C2 0001C5C0 ShimFlushCache  
  
  Summary  
  
        4000 .data  
        3000 .pdata  
       14000 .rdata  
        1000 .reloc  
        9000 .rsrc  
       31000 .text  

  

  

  

  

Microsoft (R) COFF/PE Dumper Version 14.00.23026.0

Copyright (C) Microsoft Corporation.  All rights reserved.

  

  

Dump of file C:\\Windows\\AppPatch\\AppPatch64\\EMET64.dll

  

File Type: DLL

  

  Section contains the following exports for EMET64.dll

  

    00000000 characteristics

    55F897DF time date stamp Tue Sep 15 23:12:47 2015

        0.00 version

           1 ordinal base

           3 number of functions

           3 number of names

  

    ordinal hint RVA      name

  

          1    0 00086AB0 EMETSendCert

          2    1 000481B0 GetHookAPIs

          3    2 00048140 NotifyShims

  

  Summary

  

       20000 .data

       20000 .detourc

       20000 .detourd

       20000 .didat

       20000 .pdata

       80000 .rdata

       20000 .reloc

       20000 .rsrc

       80000 .text

  

  

  

Microsoft (R) COFF/PE Dumper Version 14.00.23026.0

Copyright (C) Microsoft Corporation.  All rights reserved.

  

  

Dump of file C:\\Windows\\AppPatch\\EMET.dll

  

File Type: DLL

  

  Section contains the following exports for EMET.dll

  

    00000000 characteristics

    55F8975B time date stamp Tue Sep 15 23:10:35 2015

        0.00 version

           1 ordinal base

           3 number of functions

           3 number of names

  

    ordinal hint RVA      name

  

          1    0 00060F20 EMETSendCert

          2    1 00028B30 GetHookAPIs

          3    2 00028AE0 NotifyShims

  

  Summary

  

       20000 .data

       20000 .detourc

       20000 .detourd

       20000 .didat

       60000 .rdata

       20000 .reloc

       20000 .rsrc

       60000 .text

  

  

We haven't discovered too much here but one could try including the .dll into a console app to try and reveal more or disassemble it to pseudo c code.

  

Once that's done one might understand the hooks and detours used to ruin an exploits day.