---
title: 'NxLog conf for sysmon'
date: 2016-03-31T20:22:00.000+01:00
draft: false
aliases: [ "/2016/03/nxlog-conf-for-sysmon.html" ]
parent: Blogger
---
#### date: 2016-03-31

Just a quick post detailing the conf file I found to work best with sysmon.

\###### start of config file ############  
##   See the nxlog reference manual at  
##   [http://nxlog-ce.sourceforge.net/nxlog-docs/en/nxlog-reference-manual.pdf](http://nxlog-ce.sourceforge.net/nxlog-docs/en/nxlog-reference-manual.pdf)

define ROOT C:\\Program Files (x86)\\nxlog

Moduledir %ROOT%\\modules  
CacheDir %ROOT%\\data  
Pidfile %ROOT%\\data\\nxlog.pid  
SpoolDir %ROOT%\\data  
LogFile %ROOT%\\data\\nxlog.log

<Extension json>  
    Module      xm\_json  
</Extension>

<Input in>  
    Module      im\_msvistalog  
     
    # ref [https://msdn.microsoft.com/en-us/library/aa385231.aspx](https://msdn.microsoft.com/en-us/library/aa385231.aspx)  
    # had to add tag into query tag and change " for '  
    Query   <QueryList>\\  
                <Query Id='0' Path='Microsoft-Windows-Sysmon/Operational'>\\  
                    <Select Path='Microsoft-Windows-Sysmon/Operational'>\*</Select>\\  
                </Query>\\  
            </QueryList>  
#    Exec if ($TargetUserName == ’SYSTEM’) OR ($EventType == ’VERBOSE’) drop(); #incase you want to filter at later date  
    Exec $raw\_event = to\_json(); #keeps event on one line  
</Input>

<Output out>  
    Module      om\_tcp  
    Host        10.10.10.10  
    Port        5514  
</Output>

<Output outf> #useful for testing  
    Module      om\_file  
    file "nxlog\_output"  
</Output>

<Route 1>  
    Path        in => out  
</Route>  
\###### end of config file ############