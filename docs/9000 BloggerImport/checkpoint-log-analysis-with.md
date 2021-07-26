---
title: 'Checkpoint Log analysis with ElasticSearch & Kibana'
date: 2017-03-14T21:13:00.001Z
draft: false
aliases: [ "/2017/03/checkpoint-log-analysis-with.html" ]
parent: Blogger
---
#### date: 2017-03-14

The first thing to answer is why are the native tools not preferred?

The frustration is created by the daily log rollover, meaning you would have to repeat searches and filtering for every rule, and repeat for each days worth’s of logs. We can use the console to grep through the logs but if rule orders are changed added or removed, and compounded by rules without names I’m left with only the rule guid which yields no results when I try to find it with the following command;

fw log -n ./2017-02\*.log | egrep -E "Date:|{F46A067D-FAFA-469F-BEEC-DD902A516B31}"

Therefore via SmartView Tracker I have opened and exported each days logs.

These exported logs are space delimited, therefore I created another job in Talend Studio to iterate over these and to transform into comma delimited format (See Job definition at end of this section).

I download the windows zip version of ElasticSearch (5.2.1) and Kibana (5.2.1) from [https://www.elastic.co/downloads](https://www.elastic.co/downloads) and extract them into c:\\bin\\

\# Start elastic search service

C:\\bin\\elasticsearch-5.2.1\\bin\\elasticsearch.bat

\# Start kibana service

C:\\bin\\kibana-5.2.1-windows-x86\\bin\\kibana.bat

With the work I did a year ago to allow import from CSV into ElasticSearch, I forked the csv2es code to allow appending to an index instead, as the default behavior wanted to delete and recreate which is no good when I have several files to import. This can be obtained from [https://github.com/kempy007/csv2es](https://github.com/kempy007/csv2es) with only [https://github.com/kempy007/csv2es/blob/master/csv2es.py](https://github.com/kempy007/csv2es/blob/master/csv2es.py) of interest.

In order to get the tools setup properly I installed Python2.7, and because of the proxy I had to open command prompt as admin, and run the following commands with username and password changed to your details;

set HTTP\_PROXY=http://username:password@proxy.local:8080

set HTTPS\_PROXY=http://username:password@proxy.local:8080

I was then able install the official csv2es package plus dependancies with the following command;

C:\\python27\\scripts\\easy\_install-2.7.exe csv2es

I download my custom csv2es.py to c:\\python27\\ and CD here, ensure elastic search is running.

\# to purge an existing index

c:\\Python27>python.exe csv2es.py --index-name cplfiles --delete-index --doc-type none --import-file none

\# to create and load first set of data

c:\\Python27>python.exe csv2es.py --index-name cplfiles --create-index --doc-type none --import-file c:\\Users\\kemp\\Desktop\\MISC\\CPLogs\\CSV\\Feb8.txt.csv

\# to append more data to existing index

c:\\Python27>python.exe csv2es.py --index-name cplfiles --doc-type none --import-file c:\\Users\\kemp\\Desktop\\MISC\\CPLogs\\CSV\\Feb9.txt.csv

Complete the append step on the remaining files.

Ensure the kibana service is running and elastic search is still running. Browse to [http://localhost:5601](http://localhost:5601)

First time you’ll be nagged to set an index pattern, I just use \* and untick ‘Index contains time-based events‘ see ‘Figure 4 - Kibana Index Pattern’

[![image](https://lh3.googleusercontent.com/-RNfJS5hJ_4Q/WMhc2KudbyI/AAAAAAAAAFs/kZ1hHoltD-M/image_thumb%25255B1%25255D.png?imgmax=800 "image")](https://lh3.googleusercontent.com/-KaZLVSzUnOQ/WMhc1kiXsLI/AAAAAAAAAFo/oSeKdRcC0nQ/s1600-h/image%25255B3%25255D.png)

Figure 4 - Kibana Index Pattern

Now on the discover panel, we can 1. Scroll available fields and then 2. Hover over and ‘Add’ until 3. We have ‘timestamp’ ‘Rule’ ‘Protocol’ ‘Source’ ‘Destination’ ‘Action’ ‘Service’ and then you can 4. Search for Rule:x and any other filtering required to perform the analysis of the rulebase, see ‘Figure 5 - Kibana Search/Discover’.

[![image](https://lh3.googleusercontent.com/-pPvu6MgMY7E/WMhc3FffHmI/AAAAAAAAAF0/qX-4WhvIN5E/image_thumb%25255B4%25255D.png?imgmax=800 "image")](https://lh3.googleusercontent.com/-fDOyq98AKy4/WMhc2htWHFI/AAAAAAAAAFw/2VVObPZdMtg/s1600-h/image%25255B8%25255D.png)

Figure 5 - Kibana Search/Discover

Alternatively omit step 4, and save the search for all rules, then build visualizations using barcharts and metrics for Source, Destination and Service, then use these to build a dashboard. Once in the dashboard, apply filters to review each rule.

#### Talend Job - CheckpointLogTransform

Jobs

 

Generated by Talend Open Studio for Data Integration

Project Name

CheckpointLogTransform

GENERATION DATE

22-Feb-2017 11:32:15

AUTHOR

user@talend.com

Talend Open Studio VERSION

6.3.0.20161026\_1219

##### Summary

[**Project Description**](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#ProjectDescription)

[**Description**](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#JobDescription)

[**Preview Picture**](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#JobPreviewPicture)

[**Settings**](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#Job Settings)

[**Context List**](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#Context List)

[**Component List**](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#ComponentList)

[**Components Description**](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#ComponentsDescription)

##### Project Description

**Properties**

**Values**

Name

CheckpointLogTransform

Language

java

Description

 

##### Description

**Properties**

**Values**

Name

LogTransform

Author

user@talend.com

Version

0.1

Purpose

lol

Status

 

Description

lol

Creation

20-Feb-2017 10:47:39

Modification

20-Feb-2017 16:55:07

##### Preview Picture

[![image](https://lh3.googleusercontent.com/-l0-4OxtD8YA/WMhc3-o25WI/AAAAAAAAAF8/oF-8grjd_0k/image_thumb%25255B6%25255D.png?imgmax=800 "image")](https://lh3.googleusercontent.com/-8P1t3O5kPJg/WMhc3SzPIDI/AAAAAAAAAF4/fJPUwbK8SRc/s1600-h/image%25255B12%25255D.png)

##### Settings

**Extra settings**

**Name**

**Value**

COMP\_DEFAULT\_FILE\_DIR

 

Multi thread execution

false

Implicit tContextLoad

false

**Status & Logs**

**Name**

**Value**

Use statistics (tStatCatcher)

false

Use logs (tLogCatcher)

false

Use volumetrics (tFlowMeterCatcher)

false

On Console

false

On Files

false

On Databases

false

Catch components statistics

false

Catch runtime errors

true

Catch user errors

true

Catch user warnings

true

##### Context List

**ContextDefault**

**Name**

**Prompt**

**Need Prompt?**

**Type**

**Value**

**Source**

##### Component List

**Component Name**

**Component Type**

[tFileInputDelimited\_1](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#tFileInputDelimited_1)

tFileInputDelimited

[tFileList\_1](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#tFileList_1)

tFileList

[tFileOutputDelimited\_1](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#tFileOutputDelimited_1)

tFileOutputDelimited

[tMap\_1](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#tMap_1)

tMap

##### Components Description

**Component   tFileInputDelimited**

 

UNIQUE NAME

tFileInputDelimited\_1

INPUT(S)

[tFileList\_1](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#tFileList_1)

LABEL

CheckpointLogFormat

OUTPUT(S)

[tMap\_1](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#tMap_1)

**Component Parameters:**

**Properties**

**Values**

Unique Name

tFileInputDelimited\_1

Component Name

tFileInputDelimited

Version

0.102 (ALPHA)

Family

File/Input

Start

false

Startable

true

SUBTREE\_START

false

END\_OF\_FLOW

false

Activate

true

DUMMY

false

tStatCatcher Statistics

false

Help

org.talend.help.tFileInputDelimited

Update components

true

IREPORT\_PATH

 

JAVA\_LIBRARY\_PATH

C:\\TOS\_DI-20161026\_1219-V6.3.0\\configuration\\lib\\java

Subjob color

 

Title color

 

Property Type

Built-In

File name/Stream

((String)globalMap.get("tFileList\_1\_CURRENT\_FILEPATH"))

CSV options

true

Row Separator

"\\n"

CSV Row Separator

"\\n"

Field Separator

" "

Escape char

"""

Text enclosure

"\\""

Header

1

Footer

0

Limit

 

Skip empty rows

false

Uncompress as zip file

false

Die on error

false

REPOSITORY\_ALLOW\_AUTO\_SWITCH

true

Schema

repository: DELIM:CheckpointLogFormat - CLF-metadata

Schema

repository: DELIM:CheckpointLogFormat - CLF-metadata

!!!TEMP\_DIR.NAME!!!

"C:/TOS\_DI-20161026\_1219-V6.3.0/workspace"

Advanced separator (for numbers)

false

Thousands separator

","

Decimal separator

"."

Extract lines at random

false

Number of lines

10

Trim all columns

false

Check column to trim

\[{TRIM=false, SCHEMA\_COLUMN=Number}, {TRIM=false, SCHEMA\_COLUMN=Date}, {TRIM=false, SCHEMA\_COLUMN=Time}, {TRIM=false, SCHEMA\_COLUMN=Interface}, {TRIM=false, SCHEMA\_COLUMN=Origin}, {TRIM=false, SCHEMA\_COLUMN=Type}, {TRIM=false, SCHEMA\_COLUMN=Action}, {TRIM=false, SCHEMA\_COLUMN=Service}, {TRIM=false, SCHEMA\_COLUMN=Source\_Port}, {TRIM=false, SCHEMA\_COLUMN=Source}, {TRIM=false, SCHEMA\_COLUMN=Destination}, {TRIM=false, SCHEMA\_COLUMN=Protocol}, {TRIM=false, SCHEMA\_COLUMN=Rule}, {TRIM=false, SCHEMA\_COLUMN=Rule\_Name}, {TRIM=false, SCHEMA\_COLUMN=Current\_Rule\_Number}, {TRIM=false, SCHEMA\_COLUMN=User}, {TRIM=false, SCHEMA\_COLUMN=Information}, {TRIM=false, SCHEMA\_COLUMN=Product}, {TRIM=false, SCHEMA\_COLUMN=Source\_Machine\_Name}, {TRIM=false, SCHEMA\_COLUMN=Source\_User\_Name}\]

Check each row structure against schema

false

Check date

false

Encoding

"UTF-8"

Split row before field

false

Permit hexadecimal (0xNNN) or octal (0NNNN) for numeric types

false

Decode table

\[{DECODE=false, SCHEMA\_COLUMN=Number}, {DECODE=false, SCHEMA\_COLUMN=Date}, {DECODE=false, SCHEMA\_COLUMN=Time}, {DECODE=false, SCHEMA\_COLUMN=Interface}, {DECODE=false, SCHEMA\_COLUMN=Origin}, {DECODE=false, SCHEMA\_COLUMN=Type}, {DECODE=false, SCHEMA\_COLUMN=Action}, {DECODE=false, SCHEMA\_COLUMN=Service}, {DECODE=false, SCHEMA\_COLUMN=Source\_Port}, {DECODE=false, SCHEMA\_COLUMN=Source}, {DECODE=false, SCHEMA\_COLUMN=Destination}, {DECODE=false, SCHEMA\_COLUMN=Protocol}, {DECODE=false, SCHEMA\_COLUMN=Rule}, {DECODE=false, SCHEMA\_COLUMN=Rule\_Name}, {DECODE=false, SCHEMA\_COLUMN=Current\_Rule\_Number}, {DECODE=false, SCHEMA\_COLUMN=User}, {DECODE=false, SCHEMA\_COLUMN=Information}, {DECODE=false, SCHEMA\_COLUMN=Product}, {DECODE=false, SCHEMA\_COLUMN=Source\_Machine\_Name}, {DECODE=false, SCHEMA\_COLUMN=Source\_User\_Name}\]

!!!DESTINATION.NAME!!!

 

Min column number of optimize code

100

Label format

CheckpointLogFormat

Hint format

<b>\_\_UNIQUE\_NAME\_\_</b><br>\_\_COMMENT\_\_

Connection format

row

Show Information

false

Comment

 

Use an existing validation rule

false

Validation Rule Type

 

**Schema for CLF-metadata :**

**Column**

**Key**

**Type**

**Length**

**Precision**

**Nullable**

**Comment**

Number

false

Integer

2

 

true

 

Date

false

String

 

 

true

 

Time

false

String

 

 

true

 

Interface

false

String

6

 

true

 

Origin

false

String

10

 

true

 

Type

false

String

7

 

true

 

Action

false

String

7

 

true

 

Service

false

String

10

 

true

 

Source\_Port

false

String

10

 

true

 

Source

false

String

43

 

true

 

Destination

false

String

51

 

true

 

Protocol

false

String

4

 

true

 

Rule

false

String

2

 

true

 

Rule\_Name

false

String

27

 

true

 

Current\_Rule\_Number

false

String

23

 

true

 

User

false

String

 

 

true

 

Information

false

String

69

 

true

 

Product

false

String

27

 

true

 

Source\_Machine\_Name

false

String

 

 

true

 

Source\_User\_Name

false

String

 

 

true

 

**Original Function Parameters:**  

**Component   tFileList**

 

UNIQUE NAME

tFileList\_1

INPUT(S)

none

LABEL

\_\_UNIQUE\_NAME\_\_

OUTPUT(S)

[tFileInputDelimited\_1](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#tFileInputDelimited_1)

**Component Parameters:**

**Properties**

**Values**

Unique Name

tFileList\_1

Component Name

tFileList

Version

0.102 (ALPHA)

Family

File/Management|Orchestration

Start

true

Startable

true

SUBTREE\_START

true

END\_OF\_FLOW

true

Activate

true

DUMMY

false

tStatCatcher Statistics

false

Help

org.talend.help.tFileList

Update components

true

IREPORT\_PATH

 

JAVA\_LIBRARY\_PATH

C:\\TOS\_DI-20161026\_1219-V6.3.0\\configuration\\lib\\java

Subjob color

 

Title color

 

Directory

"C:/Users/mkemp/Desktop/MISC/CPLogs"

FileList Type

FILES

Includes subdirectories

false

Case Sensitive

YES

Generate Error if no file found

true

Use Glob Expressions as Filemask (Unchecked means Perl5 Regex Expressions)

false

Files

\[\]

By default

false

By file name

true

By file size

false

By modified date

false

ASC

true

DESC

false

Use Exclude Filemask

false

Exclude Filemask

"\*.txt"

Format file path to slash(/) style (useful on Windows)

false

Label format

\_\_UNIQUE\_NAME\_\_

Hint format

<b>\_\_UNIQUE\_NAME\_\_</b><br>\_\_COMMENT\_\_

Connection format

row

Show Information

false

Comment

 

Use an existing validation rule

false

Validation Rule Type

 

**Original Function Parameters:**  

**Component   tFileOutputDelimited**

 

UNIQUE NAME

tFileOutputDelimited\_1

INPUT(S)

[tMap\_1](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#tMap_1)

LABEL

\_\_UNIQUE\_NAME\_\_

OUTPUT(S)

none

**Component Parameters:**

**Properties**

**Values**

Unique Name

tFileOutputDelimited\_1

Component Name

tFileOutputDelimited

Version

0.101 (ALPHA)

Family

File/Output

Startable

false

SUBTREE\_START

false

END\_OF\_FLOW

true

Activate

true

DUMMY

false

tStatCatcher Statistics

false

Help

org.talend.help.tFileOutputDelimited

Update components

true

IREPORT\_PATH

 

JAVA\_LIBRARY\_PATH

C:\\TOS\_DI-20161026\_1219-V6.3.0\\configuration\\lib\\java

Subjob color

 

Title color

 

Property Type

Built-In

Use Output Stream

false

Output Stream

outputStream

File Name

"C:/Users/mkemp/Desktop/MISC/CPLogs/CSV/"+ ((String)globalMap.get("tFileList\_1\_CURRENT\_FILE")) +".csv"

Row Separator

"\\n"

Use OS line separator as row separator when CSV Row Separator is set to CR,LF or CRLF.

true

CSV Row Separator

"\\n"

Field Separator

","

Append

false

Include Header

true

Compress as zip file

false

REPOSITORY\_ALLOW\_AUTO\_SWITCH

true

Schema

Built-In

Advanced separator (for numbers)

false

Thousands separator

","

Decimal separator

"."

CSV options

true

Escape char

"""

Text enclosure

"""

Create directory if does not exist

true

Split output in several files

false

Rows in each output file

1000

Custom the flush buffer size

false

Row number

1

Output in row mode

false

Encoding

"ISO-8859-15"

Don't generate empty file

false

Min column number of optimize code

90

Label format

\_\_UNIQUE\_NAME\_\_

Hint format

<b>\_\_UNIQUE\_NAME\_\_</b><br>\_\_COMMENT\_\_

Connection format

row

Show Information

false

Comment

 

Use an existing validation rule

false

Validation Rule Type

 

**Schema for tFileOutputDelimited\_1 :**

**Column**

**Key**

**Type**

**Length**

**Precision**

**Nullable**

**Comment**

Number

false

Integer

2

 

true

 

timestamp

false

String

20

 

true

 

Date

false

String

8

 

true

 

Time

false

String

8

 

true

 

Interface

false

String

6

 

true

 

Origin

false

String

10

 

true

 

Type

false

String

7

 

true

 

Action

false

String

7

 

true

 

Service

false

String

10

 

true

 

Source\_Port

false

String

10

 

true

 

Source

false

String

43

 

true

 

Destination

false

String

51

 

true

 

Protocol

false

String

4

 

true

 

Rule

false

String

2

 

true

 

Rule\_Name

false

String

27

 

true

 

Current\_Rule\_Number

false

String

23

 

true

 

User

false

String

 

 

true

 

Information

false

String

69

 

true

 

Product

false

String

27

 

true

 

Source\_Machine\_Name

false

String

 

 

true

 

Source\_User\_Name

false

String

 

 

true

 

**Original Function Parameters:**  

**Component   tMap**

 

UNIQUE NAME

tMap\_1

INPUT(S)

[tFileInputDelimited\_1](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#tFileInputDelimited_1)

LABEL

\_\_UNIQUE\_NAME\_\_

OUTPUT(S)

[tFileOutputDelimited\_1](file:///C:\joboutput\CPL\LogTransform_0.1HTML\LogTransform\LogTransform_0.1.html#tFileOutputDelimited_1)

**Component Parameters:**

**Properties**

**Values**

tStatCatcher Statistics

```
false
```

Mapping links display as:

```
AUTO
```

Temp data directory path:

 

Max buffer size (nb of rows):

```
2000000
```

Ignore trailing zeros for BigDecimal

```
false
```

Show Information

```
false
```

Comment

 

Use an existing validation rule

```
false
```

**Mapper table for tMap\_1 ( input ):**

 

**Mapper table Properties( row1 ):**

**Properties**

**Values**

Name

row1

Matching-mode

UNIQUE\_MATCH

isMinimized

false

isReject

false

isRejectInnerJoin

false

isInnerJoin

false

expressionFilter

null

**Metadata Table Entries( row1 ):**

**Name**

**Type**

**Expression**

**isNullable**

Number

Integer

 

true

Date

String

 

true

Time

String

 

true

Interface

String

 

true

Origin

String

 

true

Type

String

 

true

Action

String

 

true

Service

String

 

true

Source\_Port

String

 

true

Source

String

 

true

Destination

String

 

true

Protocol

String

 

true

Rule

String

 

true

Rule\_Name

String

 

true

Current\_Rule\_Number

String

 

true

User

String

 

true

Information

String

 

true

Product

String

 

true

Source\_Machine\_Name

String

 

true

Source\_User\_Name

String

 

true

**Constraint Table Entries( row1 ):**

**Name**

**Type**

**Expression**

**isNullable**

**Mapper table for tMap\_1 ( output ):**

 

**Mapper table Properties( out1 ):**

**Properties**

**Values**

Name

out1

Matching-mode

 

isMinimized

false

isReject

false

isRejectInnerJoin

false

isInnerJoin

false

expressionFilter

null

**Metadata Table Entries( out1 ):**

**Name**

**Type**

**Expression**

**isNullable**

Number

Integer

row1.Number

true

timestamp

String

TalendDate.formatDate("yyyy-MM-dd",(TalendDate.parseDateLocale("ddMMMyyyy",row1.Date,"EN")) ) + ("T") + row1.Time

true

Date

String

row1.Date

true

Time

String

row1.Time

true

Interface

String

row1.Interface

true

Origin

String

row1.Origin

true

Type

String

row1.Type

true

Action

String

row1.Action

true

Service

String

row1.Service

true

Source\_Port

String

row1.Source\_Port

true

Source

String

row1.Source

true

Destination

String

row1.Destination

true

Protocol

String

row1.Protocol

true

Rule

String

row1.Rule

true

Rule\_Name

String

row1.Rule\_Name

true

Current\_Rule\_Number

String

row1.Current\_Rule\_Number

true

User

String

row1.User

true

Information

String

row1.Information

true

Product

String

row1.Product

true

Source\_Machine\_Name

String

row1.Source\_Machine\_Name

true

Source\_User\_Name

String

row1.Source\_User\_Name

true

**Constraint Table Entries( out1 ):**

**Name**

**Type**

**Expression**

**isNullable**

**Mapper table for tMap\_1 ( var ):**

 

**Mapper table Properties( Var ):**

**Properties**

**Values**

Name

Var

Matching-mode

 

isMinimized

true

isReject

false

isRejectInnerJoin

false

isInnerJoin

false

expressionFilter

null

**Metadata Table Entries( Var ):**

**Name**

**Type**

**Expression**

**isNullable**

**Constraint Table Entries( Var ):**

**Name**

**Type**

**Expression**

**isNullable**