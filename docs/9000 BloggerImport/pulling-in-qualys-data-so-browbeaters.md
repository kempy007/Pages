---
title: 'Pulling in Qualys data with python and Talend Studio, so it can be dashboarded in excel :('
date: 2016-03-09T22:34:00.000Z
draft: false
aliases: [ "/2016/03/pulling-in-qualys-data.html" ]
parent: Blogger
---
#### date: 2016-03-09

First you enable ticket policy.  
For some reason somebody forgot to include tickets in version 2 of the API, you can only see ticket data with version 1 of the API, and you cannot set the truncate limit. That means you need to program extra logic to iterate over the entire dataset burning up API requests. Every smart person I spoke to regarding this cannot understand this illogical step in the improvement of their product. To top it off, try and download the KB data, and the API may say no… but the web portal says yes.  
Well I was left with no choice but to do the horrible act of webscraping, thankfully I was able to use the python extension in visual studio and take advantage of some intellisence. So after installing selenium python libraries and firefox, I was able to make some progress and build up a job for grabbing the ticket and KB data in one request’ish.  
I’m not a data analyst in fact it’s not in my personality, I can do it, but it’s just not fun. So thanks to Andrew for the recommendation to use Talend Studio to do the legwork, I mean every time I use it, I realise just how awesome it is.  
So then the plan is to use my python script to webscrape some data, then in talend studio I call that script, make a webrequest to the API for some other data, and then with some data mapping and a custom IP to host mapping file I can pull all this data together into a nice big flatfile for Management. Of course I’m a few KPI short but not bad for a start.  
My folder structure is;  
D:\\Data\\Qualys  

> \\Xml  
> \\maps  
> \\ExternalJob  
> \\Documentation  
>   
>   

  
The Python code is below;  
\# Requirements  
#  
\# Install Python 2.7.9 or greater, added to path variable.  
\# Install additional python libraries;  
\# C:\\Python27\\Scripts\\pip.exe install selenium  
\# C:\\Python27\\Scripts\\pip.exe install bcrypt  
\# Install firefox 43.0.1 or higher  
\# Invoke this script from talend studio job; C:\\Python27\\python.exe D:\\Data\\Qualys\\ExternalJob\\QualysDownloadSelenium.py  
\# Created 22-02-2016  
\# Last Tested 09-03-2016  
\# Script likely to break upon password expiry and or changes to the qualys site  
\# -\*- coding: utf-8 -\*-  
from selenium import webdriver  
from selenium.webdriver.common.by import By  
from selenium.webdriver.common.keys import Keys  
from selenium.webdriver.support.ui import Select  
from selenium.common.exceptions import NoSuchElementException  
from selenium.common.exceptions import NoAlertPresentException  
from selenium.webdriver.firefox.firefox\_profile import FirefoxProfile  
import unittest, time, re, os  
import base64  
SAVE\_TO\_DIRECTORY = "D:\\\\Data\\\\Qualys\\\\Xml\\\\" # change this to the correct path  
userAccount = "Z29kbW9kZQ==" # change for real account, base64 to stop shoulder snoops  
Password = "dTgwMDhwYXNzd2Rub3RsaWtlbHl0b2JlaGVyZS4uLi4uMTIz" # change for real password, base64 to stop shoulder snoops  
tmpAcc = base64.b64decode(userAccount)  
spltUser = tmpAcc.split(r'-')  
tmpAcc = ""  
tmpKBfile = SAVE\_TO\_DIRECTORY+"DL\_vulnerabilities\_"+spltUser\[0\]+"\_"+spltUser\[1\]+"\_"+time.strftime("%Y%m%d")+".xml"  
tmpTLfile = SAVE\_TO\_DIRECTORY+"DL\_tickets\_"+spltUser\[0\]+"\_"+spltUser\[1\]+"\_"+time.strftime("%Y%m%d")+".xml"  
docName = "2tmpVulns"  
docName2 = "2tmpTickets"  
\# check for files we don't want to trip over  
if os.path.isfile(SAVE\_TO\_DIRECTORY+docName+".xml"):  
os.remove(SAVE\_TO\_DIRECTORY+docName+".xml")  
if os.path.isfile(SAVE\_TO\_DIRECTORY+docName2+".xml"):  
os.remove(SAVE\_TO\_DIRECTORY+docName2+".xml")  
if os.path.isfile(tmpKBfile):  
os.remove(tmpKBfile)  
if os.path.isfile(tmpTLfile):  
os.remove(tmpTLfile)  
profile = webdriver.FirefoxProfile()  
profile.set\_preference("browser.download.folderList", 2)  
profile.set\_preference("browser.download.dir", SAVE\_TO\_DIRECTORY)  
profile.set\_preference("browser.download.manager.alertOnEXEOpen", False);  
profile.set\_preference("browser.helperApps.neverAsk.saveToDisk", "text/xml, application/msword, application/csv, application/ris, text/csv, image/png, application/pdf, text/html, text/plain, application/zip, application/x-zip, application/x-zip-compressed, application/download, application/octet-stream" )  
profile.set\_preference("browser.helperApps.neverAsk.openFile", "text/xml, application/msword, application/csv, application/ris, text/csv, image/png, application/pdf, text/html, text/plain, application/zip, application/x-zip, application/x-zip-compressed, application/download, application/octet-stream" )  
profile.set\_preference("browser.download.manager.showWhenStarting", False);  
profile.set\_preference("browser.download.manager.focusWhenStarting", False);  
profile.set\_preference("browser.download.useDownloadDir", True);  
profile.set\_preference("browser.helperApps.alwaysAsk.force", False);  
profile.set\_preference("browser.download.manager.alertOnEXEOpen", False);  
profile.set\_preference("browser.download.manager.closeWhenDone", True);  
profile.set\_preference("browser.download.manager.showAlertOnComplete", False);  
profile.set\_preference("browser.download.manager.useWindow", False);  
profile.set\_preference("services.sync.prefs.sync.browser.download.manager.showWhenStarting", False);  
profile.set\_preference("pdfjs.disabled", True);  
class QualysSeleniumScript(unittest.TestCase):  
def setUp(self):  
self.driver = webdriver.Firefox(firefox\_profile=profile)  
self.driver.implicitly\_wait(30)  
self.base\_url = "https://qualysguard.qualys.eu"  
self.verificationErrors = \[\]  
self.accept\_next\_alert = True  
def test\_qualys\_selenium\_script(self):  
driver = self.driver  
driver.get(self.base\_url + "/")  
driver.find\_element\_by\_id("myform\_UserLogin").clear()  
driver.find\_element\_by\_id("myform\_UserLogin").send\_keys(base64.b64decode(userAccount))  
driver.find\_element\_by\_id("myform\_UserPasswd").clear()  
driver.find\_element\_by\_id("myform\_UserPasswd").send\_keys(base64.b64decode(Password))  
driver.find\_element\_by\_name("\_form\_action1").click()  
driver.get(self.base\_url + "/fo/tools/kbase.php")  
time.sleep(3)  
driver.find\_element\_by\_id("ext-gen202").click()  
driver.find\_element\_by\_id("ext-gen248").click()  
for i in range(60):  
try:  
if self.is\_element\_present(By.ID, "format\_handle"): break  
except: pass  
time.sleep(1)  
else: self.fail("time out")  
driver.find\_element\_by\_id("select\_download\_format\_xml").click()  
driver.find\_element\_by\_id("dload\_btn").click()  
\# should loop around watching for finsihed download KB  
\# DL\_vulnerabilities\_mdand\_mk\_20160309.xml  
\# DL\_vulnerabilities\_mdand\_mk\_20160309.xml.part  
X = 0  
while True:  
time.sleep(5)  
X += 1  
if os.path.isfile(tmpKBfile):  
if os.stat(tmpKBfile).st\_size > 0:  
break  
if X > 60:  
break # 60 iterations of 5 seconds will be 5 minutes  
\# rename downloaded file for KB.  
os.chdir(SAVE\_TO\_DIRECTORY)  
if os.path.isfile(docName+".xml"):  
os.remove(docName+".xml")  
files = filter(os.path.isfile, os.listdir(SAVE\_TO\_DIRECTORY))  
files = \[os.path.join(SAVE\_TO\_DIRECTORY, f) for f in files\] # add path to each file  
files.sort(key=lambda x: os.path.getmtime(x))  
newest\_file = files\[-1\]  
os.rename(newest\_file, docName+".xml")  
#Now we pull down the tickets because the API is poo poo  
driver.get(self.base\_url + "/fo/remedy/index.php")  
time.sleep(3)  
driver.find\_element\_by\_id("ext-gen68").click() #search button  
time.sleep(3)  
driver.find\_element\_by\_id("dl\[search\]\[state\_search\]\[OPEN\]").click()  
driver.find\_element\_by\_id("dl\[search\]\[state\_search\]\[RESOLVED\]").click()  
driver.find\_element\_by\_id("dl\[search\]\[state\_search\]\[CLOSED\]").click()  
driver.find\_element\_by\_id("dl\[search\]\[state\_search\]\[IGNORED\]").click()  
driver.find\_element\_by\_id("search\_btn").click()  
time.sleep(6)  
driver.find\_element\_by\_id("ext-gen65").click() #new button  
driver.find\_element\_by\_id("ext-gen114").click() #new menu download...  
driver.find\_element\_by\_id("select\_download\_format\_xml").click()  
driver.find\_element\_by\_id("dload\_btn").click()  
\# need to do loop looking for completed download  
\# DL\_tickets\_mdand\_mk\_20160309.xml  
\# DL\_tickets\_mdand\_mk\_20160309.xml.part  
X = 0  
while True:  
time.sleep(5)  
X += 1  
if os.path.isfile(tmpTLfile):  
if os.stat(tmpTLfile).st\_size > 0:  
break  
if X > 60:  
break # 60 iterations of 5 seconds will be 5 minutes #check file is greater than 0kb  
#>>to do rename downloaded file.  
os.chdir(SAVE\_TO\_DIRECTORY)  
if os.path.isfile(docName2+".xml"):  
os.remove(docName2+".xml")  
files = filter(os.path.isfile, os.listdir(SAVE\_TO\_DIRECTORY))  
files = \[os.path.join(SAVE\_TO\_DIRECTORY, f) for f in files\] # add path to each file  
files.sort(key=lambda x: os.path.getmtime(x))  
newest\_file = files\[-1\]  
os.rename(newest\_file, docName2+".xml")  
#finish off by logging off  
driver.find\_element\_by\_id("ext-gen78").click()  
time.sleep(5)  
def is\_element\_present(self, how, what):  
try: self.driver.find\_element(by=how, value=what)  
except NoSuchElementException as e: return False  
return True  
def is\_alert\_present(self):  
try: self.driver.switch\_to\_alert()  
except NoAlertPresentException as e: return False  
return True  
def close\_alert\_and\_get\_its\_text(self):  
try:  
alert = self.driver.switch\_to\_alert()  
alert\_text = alert.text  
if self.accept\_next\_alert:  
alert.accept()  
else:  
alert.dismiss()  
return alert\_text  
finally: self.accept\_next\_alert = True  
def tearDown(self):  
self.driver.quit()  
self.assertEqual(\[\], self.verificationErrors)  
if \_\_name\_\_ == "\_\_main\_\_":  
unittest.main()  
  
  
Now it’s time to document the Talend Studio job, which just a few click…. see below;  
  

![clip_image002](https://lh3.googleusercontent.com/-qPZIpEVvz0Y/VuCjMmmMJZI/AAAAAAAAABM/MzCTFdYMHB8/clip_image002%25255B3%25255D.jpg?imgmax=800 "clip_image002")

Job Documentation

Generated by Talend Open Studio for Big Data

Project Name

Qualys

GENERATION DATE

09-Mar-2016 16:06:01

AUTHOR

user@talend.com

Talend Open Studio VERSION

6.1.1.20151214\_1327

#### Summary

**Project Description**  
**Description**  
**Preview Picture**  
**Settings**  
**Context List**  
**Component List**  
**Components Description**  

#### Project Description

**Properties**

**Values**

Name

Qualys

Language

java

Description

#### Description

**Properties**

**Values**

Name

QualysDataProcessor

Author

user@talend.com

Version

0.1

Purpose

XML TO CSV

Status

Description

Creation

14-Jan-2016 14:52:57

Modification

09-Mar-2016 16:06:00

#### Preview Picture

[![](https://2.bp.blogspot.com/-Q6_ibkgsfMo/VuCkABOEcWI/AAAAAAAAACM/ZomWPtRmp0M/s1600/TalendQualysJob.png)](https://2.bp.blogspot.com/-Q6_ibkgsfMo/VuCkABOEcWI/AAAAAAAAACM/ZomWPtRmp0M/s1600/TalendQualysJob.png)

  

  

#### Settings

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

#### Context List

**ContextDefault**

**Name**

**Prompt**

**Need Prompt?**

**Type**

**Value**

**Source**

Password

Password?

false

id\_Password

\*\*\*\*

ContextVars

#### Component List

**Component Name**

**Component Type**

tFileInputDelimited\_1

tFileInputDelimited

tFileInputXML\_1

tFileInputXML

tFileInputXML\_2

tFileInputXML

tFileInputXML\_5

tFileInputXML

tFileOutputDelimited\_2

tFileOutputDelimited

tHttpRequest\_2

tHttpRequest

tSystem\_1

tSystem

tSystem\_2

tSystem

tXMLMap\_1

tXMLMap

#### Components Description

**Component   tFileInputDelimited**

![clip_image007](https://lh3.googleusercontent.com/-dmCwQJxUu4E/VuCjNpa6K7I/AAAAAAAAABY/Pr5_mzbtAOk/clip_image007_thumb.png?imgmax=800 "clip_image007")

UNIQUE NAME

tFileInputDelimited\_1

INPUT(S)

none

LABEL

IP2DeptMap

OUTPUT(S)

tXMLMap\_1

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

C:\\install\\TOS\_BD-20151214\_1327-V6.1.1\\configuration\\lib\\java

Subjob color

Title color

!!!PROPERTY.NAME!!!

!!!FILENAMETEXT.NAME!!!

"When the input source is a stream or a zip file,footer and random shouldn't be bigger than 0."

File name/Stream

"D:/Data/Qualys/maps/Host-to-Department.csv"

CSV options

false

Row Separator

"\\n"

CSV Row Separator

"\\n"

Field Separator

","

Escape char

"""

Text enclosure

"""

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

false

Schema

!!!SCHEMA\_REJECT.NAME!!!

null:errorCode errorMessage

!!!TEMP\_DIR.NAME!!!

"C:/install/TOS\_BD-20151214\_1327-V6.1.1/workspace"

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

\[{TRIM=false, SCHEMA\_COLUMN=IP}, {TRIM=false, SCHEMA\_COLUMN=CIDR}, {TRIM=false, SCHEMA\_COLUMN=Zone}, {TRIM=false, SCHEMA\_COLUMN=Department}, {TRIM=false, SCHEMA\_COLUMN=Function}\]

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

\[{DECODE=false, SCHEMA\_COLUMN=IP}, {DECODE=false, SCHEMA\_COLUMN=CIDR}, {DECODE=false, SCHEMA\_COLUMN=Zone}, {DECODE=false, SCHEMA\_COLUMN=Department}, {DECODE=false, SCHEMA\_COLUMN=Function}\]

!!!DESTINATION.NAME!!!

Min column number of optimize code

100

Label format

IP2DeptMap

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

  
**Schema for metadata :**  

**Column**

**Key**

**Type**

**Length**

**Precision**

**Nullable**

**Comment**

IP

false

String

11

true

CIDR

false

String

14

true

Zone

false

String

8

true

Department

false

String

3

true

Function

false

String

15

true

  
**Original Function Parameters:**  

**Component   tFileInputXML**

![clip_image009](https://lh3.googleusercontent.com/-hE87Pdfd_fc/VuCjN0tSXbI/AAAAAAAAABc/HFcDkrzXOvk/clip_image009_thumb.png?imgmax=800 "clip_image009")

UNIQUE NAME

tFileInputXML\_1

INPUT(S)

tHttpRequest\_2

LABEL

WsTicketList

OUTPUT(S)

tXMLMap\_1

  
**Component Parameters:**  

**Properties**

**Values**

Unique Name

tFileInputXML\_1

Component Name

tFileInputXML

Version

0.102 (ALPHA)

Family

File/Input|XML

Start

false

Startable

true

SUBTREE\_START

true

END\_OF\_FLOW

false

Activate

true

DUMMY

false

tStatCatcher Statistics

false

Help

org.talend.help.tFileInputXML

Update components

true

IREPORT\_PATH

JAVA\_LIBRARY\_PATH

C:\\install\\TOS\_BD-20151214\_1327-V6.1.1\\configuration\\lib\\java

Subjob color

Title color

!!!PROPERTY.NAME!!!

REPOSITORY\_ALLOW\_AUTO\_SWITCH

false

!!!SCHEMA.NAME!!!

File name/Stream

"D:/Data/Qualys/Xml/2tmpTickets.xml"

Loop XPath query

"/DATALIST/LIST/RECORD"

Mapping

\[{QUERY="KEY\[@name=\\"TICKET\_#\\"\]", NODECHECK=, SCHEMA\_COLUMN=NUMBER}, {QUERY="KEY\[@name=\\"STATE\\"\]", NODECHECK=, SCHEMA\_COLUMN=STATE}, {QUERY="KEY\[@name=\\"DUE\_DATE\\"\]", NODECHECK=, SCHEMA\_COLUMN=DUE\_DATE}, {QUERY="KEY\[@name=\\"IP\\"\]", NODECHECK=, SCHEMA\_COLUMN=IP}, {QUERY="KEY\[@name=\\"PORT\_#\\"\]", NODECHECK=, SCHEMA\_COLUMN=PORT}, {QUERY="KEY\[@name=\\"DNS\_HOSTNAME\\"\]", NODECHECK=, SCHEMA\_COLUMN=DNS\_HOSTNAME}, {QUERY="KEY\[@name=\\"SEVERITY\\"\]", NODECHECK=, SCHEMA\_COLUMN=SEVERITY}, {QUERY="KEY\[@name=\\"QID\\"\]", NODECHECK=, SCHEMA\_COLUMN=QID}, {QUERY="KEY\[@name=\\"VULNERABILITY\_TITLE\\"\]", NODECHECK=, SCHEMA\_COLUMN=VULNERABILITY\_TITLE}, {QUERY="KEY\[@name=\\"MODIFIED\\"\]", NODECHECK=, SCHEMA\_COLUMN=MODIFIED}, {QUERY="KEY\[@name=\\"CREATED\\"\]", NODECHECK=, SCHEMA\_COLUMN=CREATED}, {QUERY="KEY\[@name=\\"RESOLVED\\"\]", NODECHECK=, SCHEMA\_COLUMN=RESOLVED}\]

Limit

\-1

Die on error

false

!!!SCHEMA\_REJECT.NAME!!!

null:errorCode errorMessage

Advanced separator (for numbers)

false

Thousands separator

","

Decimal separator

"."

Ignore the namespaces

false

Ignore DTD file

false

Generate a temporary file

"C:/install/TOS\_BD-20151214\_1327-V6.1.1/workspace/temp.xml"

Use Separator for mode Xerces

false

Field Separator

","

Generation mode

Dom4j

Validate date

false

Encoding

"UTF-8"

Min column number of optimize code

100

Label format

WsTicketList

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

  
**Schema for metadata :**  

**Column**

**Key**

**Type**

**Length**

**Precision**

**Nullable**

**Comment**

NUMBER

false

Integer

6

true

STATE

false

String

12

true

DUE\_DATE

false

String

20

true

IP

false

String

15

true

PORT

false

String

5

true

DNS\_HOSTNAME

false

String

37

true

SEVERITY

false

String

46

true

QID

false

Integer

6

true

VULNERABILITY\_TITLE

false

String

197

true

MODIFIED

false

String

28

true

CREATED

false

String

28

true

RESOLVED

false

String

28

true

  
**Original Function Parameters:**  

**Component   tFileInputXML**

![clip_image009[1]](https://lh3.googleusercontent.com/-Vf6hjbQzWVo/VuCjOGdD28I/AAAAAAAAABg/5NGqvwOeb8E/clip_image009%25255B1%25255D_thumb.png?imgmax=800 "clip_image009[1]")

UNIQUE NAME

tFileInputXML\_2

INPUT(S)

none

LABEL

AssetList

OUTPUT(S)

tXMLMap\_1

  
**Component Parameters:**  

**Properties**

**Values**

Unique Name

tFileInputXML\_2

Component Name

tFileInputXML

Version

0.102 (ALPHA)

Family

File/Input|XML

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

org.talend.help.tFileInputXML

Update components

true

IREPORT\_PATH

JAVA\_LIBRARY\_PATH

C:\\install\\TOS\_BD-20151214\_1327-V6.1.1\\configuration\\lib\\java

Subjob color

Title color

!!!PROPERTY.NAME!!!

REPOSITORY\_ALLOW\_AUTO\_SWITCH

false

!!!SCHEMA.NAME!!!

File name/Stream

"D:/Data/Qualys/Xml/tmpAssetList.xml"

Loop XPath query

"/HOST\_LIST\_OUTPUT/RESPONSE/HOST\_LIST/HOST"

Mapping

\[{QUERY="ID", NODECHECK=, SCHEMA\_COLUMN=Qualys\_Asset\_ID}, {QUERY="IP", NODECHECK=, SCHEMA\_COLUMN=IP}, {QUERY="TRACKING\_METHOD", NODECHECK=, SCHEMA\_COLUMN=TRACKING\_METHOD}, {QUERY="NETWORK\_ID", NODECHECK=, SCHEMA\_COLUMN=NETWORK\_ID}, {QUERY="OS", NODECHECK=, SCHEMA\_COLUMN=OS}, {QUERY="LAST\_COMPLIANCE\_SCAN\_DATETIME", NODECHECK=, SCHEMA\_COLUMN=LAST\_COMPLIANCE\_SCAN\_DATETIME}, {QUERY="LAST\_VULN\_SCAN\_DATETIME", NODECHECK=, SCHEMA\_COLUMN=LAST\_VULN\_SCAN\_DATETIME}, {QUERY="NETBIOS", NODECHECK=, SCHEMA\_COLUMN=NETBIOS}, {QUERY="DNS", NODECHECK=, SCHEMA\_COLUMN=DNS}\]

Limit

\-1

Die on error

false

!!!SCHEMA\_REJECT.NAME!!!

null:errorCode errorMessage

Advanced separator (for numbers)

false

Thousands separator

","

Decimal separator

"."

Ignore the namespaces

false

Ignore DTD file

false

Generate a temporary file

"C:/install/TOS\_BD-20151214\_1327-V6.1.1/workspace/temp.xml"

Use Separator for mode Xerces

false

Field Separator

","

Generation mode

Dom4j

Validate date

false

Encoding

"UTF-8"

Min column number of optimize code

100

Label format

AssetList

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

  
**Schema for metadata :**  

**Column**

**Key**

**Type**

**Length**

**Precision**

**Nullable**

**Comment**

Qualys\_Asset\_ID

false

Integer

9

true

IP

false

String

15

true

TRACKING\_METHOD

false

String

11

true

NETWORK\_ID

false

Integer

5

true

OS

false

String

80

true

LAST\_COMPLIANCE\_SCAN\_DATETIME

false

java.util.Date

20

true

LAST\_VULN\_SCAN\_DATETIME

false

java.util.Date

20

true

NETBIOS

false

String

15

true

DNS

false

String

45

true

  
**Original Function Parameters:**  

**Component   tFileInputXML**

![clip_image009[2]](https://lh3.googleusercontent.com/-kKGPEiqOTnU/VuCjOfMa5RI/AAAAAAAAABo/P-ubiMK0AVs/clip_image009%25255B2%25255D_thumb.png?imgmax=800 "clip_image009[2]")

UNIQUE NAME

tFileInputXML\_5

INPUT(S)

none

LABEL

VulnerabilityList

OUTPUT(S)

tXMLMap\_1

  
**Component Parameters:**  

**Properties**

**Values**

Unique Name

tFileInputXML\_5

Component Name

tFileInputXML

Version

0.102 (ALPHA)

Family

File/Input|XML

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

org.talend.help.tFileInputXML

Update components

true

IREPORT\_PATH

JAVA\_LIBRARY\_PATH

C:\\install\\TOS\_BD-20151214\_1327-V6.1.1\\configuration\\lib\\java

Subjob color

Title color

!!!PROPERTY.NAME!!!

REPOSITORY\_ALLOW\_AUTO\_SWITCH

false

!!!SCHEMA.NAME!!!

File name/Stream

"D:/Data/Qualys/Xml/2tmpVulns.xml"

Loop XPath query

"/DATALIST/LIST/RECORD"

Mapping

\[{QUERY="KEY\[@name=\\"QID\\"\]", NODECHECK=, SCHEMA\_COLUMN=QID}, {QUERY="KEY\[@name=\\"TITLE\\"\]", NODECHECK=, SCHEMA\_COLUMN=TITLE}, {QUERY="KEY\[@name=\\"CATEGORY\\"\]", NODECHECK=, SCHEMA\_COLUMN=CATEGORY}, {QUERY="KEY\[@name=\\"CVE\_ID\\"\]", NODECHECK=, SCHEMA\_COLUMN=CVE\_ID}, {QUERY="KEY\[@name=\\"CVSS\_BASE\\"\]", NODECHECK=, SCHEMA\_COLUMN=CVSS\_BASE}, {QUERY="KEY\[@name=\\"BUGTRAQ\_ID\\"\]", NODECHECK=, SCHEMA\_COLUMN=BUGTRAQ\_ID}, {QUERY="KEY\[@name=\\"MODIFIED\\"\]", NODECHECK=, SCHEMA\_COLUMN=MODIFIED}, {QUERY="KEY\[@name=\\"PUBLISHED\\"\]", NODECHECK=, SCHEMA\_COLUMN=PUBLISHED}\]

Limit

\-1

Die on error

false

!!!SCHEMA\_REJECT.NAME!!!

null:errorCode errorMessage

Advanced separator (for numbers)

false

Thousands separator

","

Decimal separator

"."

Ignore the namespaces

false

Ignore DTD file

false

Generate a temporary file

"C:/install/TOS\_BD-20151214\_1327-V6.1.1/workspace/temp.xml"

Use Separator for mode Xerces

false

Field Separator

","

Generation mode

Dom4j

Validate date

false

Encoding

"UTF-8"

Min column number of optimize code

100

Label format

VulnerabilityList

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

  
**Schema for metadata :**  

**Column**

**Key**

**Type**

**Length**

**Precision**

**Nullable**

**Comment**

QID

false

Integer

6

true

TITLE

false

String

241

true

CATEGORY

false

String

27

true

CVE\_ID

false

String

2855

true

CVSS\_BASE

false

String

3

true

BUGTRAQ\_ID

false

String

263

true

MODIFIED

false

String

28

true

PUBLISHED

false

String

28

true

  
**Original Function Parameters:**  

**Component   tFileOutputDelimited**

![clip_image011](https://lh3.googleusercontent.com/-BdrKqopRdtU/VuCjOwlZYWI/AAAAAAAAABs/HMDQlk0URHs/clip_image011_thumb.png?imgmax=800 "clip_image011")

UNIQUE NAME

tFileOutputDelimited\_2

INPUT(S)

tXMLMap\_1,  tFileInputXML\_2,  tFileInputXML\_5,  tFileInputDelimited\_1

LABEL

\_\_UNIQUE\_NAME\_\_

OUTPUT(S)

tSystem\_2

  
**Component Parameters:**  

**Properties**

**Values**

Unique Name

tFileOutputDelimited\_2

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

C:\\install\\TOS\_BD-20151214\_1327-V6.1.1\\configuration\\lib\\java

Subjob color

Title color

!!!PROPERTY.NAME!!!

Use Output Stream

false

Output Stream

outputStream

File Name

"D:/Data/Qualys/VTM.csv"

Row Separator

"\\r\\n"

Use OS line separator as row separator when CSV Row Separator is set to CR,LF or CRLF.

true

CSV Row Separator

"\\r\\n"

Field Separator

","

Append

false

Include Header

true

Compress as zip file

false

REPOSITORY\_ALLOW\_AUTO\_SWITCH

false

Schema

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

  
**Schema for VTM :**  

**Column**

**Key**

**Type**

**Length**

**Precision**

**Nullable**

**Comment**

QUALYS\_TICKET\_NUMBER

false

Integer

10

true

TICKET\_CREATION\_DATETIME

false

String

20

true

TICKET\_RESOLVED\_DATETIME

false

String

28

true

TICKET\_MODIFIED\_DATETIME

false

String

28

true

TICKET\_DUE\_DATETIME

false

String

20

true

TICKET\_CURRENT\_STATE

false

String

6

true

IP

false

String

15

true

PORT

false

String

4

true

QUALYS\_SEVERITY

false

String

255

true

QID

false

Integer

6

true

TRACKING\_METHOD

false

String

11

true

OS

false

String

80

true

NETBIOS

false

String

15

true

DNS

false

String

45

true

LAST\_COMPLIANCE\_SCAN\_DATETIME

false

java.util.Date

20

true

LAST\_VULN\_SCAN\_DATETIME

false

java.util.Date

20

true

TITLE

false

String

241

true

CATEGORY

false

String

27

true

CVE\_ID

false

String

2855

true

CVSS\_BASE

false

String

3

true

BUGTRAQ\_ID

false

String

263

true

QID\_MODIFIED

false

String

28

true

QID\_PUBLISHED

false

String

28

true

CIDR

false

String

14

true

Zone

false

String

8

true

Department

false

String

3

true

Function

false

String

15

true

  
**Original Function Parameters:**  

**Component   tHttpRequest**

![clip_image013](https://lh3.googleusercontent.com/-2jskfBJ_uxs/VuCjPPaa6PI/AAAAAAAAABw/xapE3wvL5do/clip_image013_thumb.png?imgmax=800 "clip_image013")

UNIQUE NAME

tHttpRequest\_2

INPUT(S)

tSystem\_1

LABEL

\_\_UNIQUE\_NAME\_\_

OUTPUT(S)

tFileInputXML\_1

  
**Component Parameters:**  

**Properties**

**Values**

Unique Name

tHttpRequest\_2

Component Name

tHttpRequest

Version

0.101 (ALPHA)

Family

Internet

Start

false

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

org.talend.help.tHttpRequest

Update components

true

IREPORT\_PATH

JAVA\_LIBRARY\_PATH

C:\\install\\TOS\_BD-20151214\_1327-V6.1.1\\configuration\\lib\\java

Subjob color

Title color

REPOSITORY\_ALLOW\_AUTO\_SWITCH

false

Property

null:ResponseContent

URI

"https://qualysapi.qualys.eu/api/2.0/fo/asset/host/?action=list&truncation\_limit=1000000&details=All&vm\_scan\_since=2015-01-01"

Method

POST

Post parameters from file

Write response content to file

true

"D:/Data/Qualys/Xml/tmpAssetList.xml"

Create directory if not exists

false

Headers

\[{HEADER\_NAME="X-Requested-With", HEADER\_VALUE="Talend"}, {HEADER\_NAME="Authorization", HEADER\_VALUE="Basic Z29kbW9kZTp1ODAwOHBhc3N3ZG5vdGxpa2VseXRvYmVoZXJlLi4uLi4xMjM="}\]

Need authentication

false

user

""

password

\*\*

Die on error

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

  
**Schema for tHttpRequest\_2 :**  

**Column**

**Key**

**Type**

**Length**

**Precision**

**Nullable**

**Comment**

ResponseContent

false

String

true

  
**Original Function Parameters:**  

**Component   tSystem**

![clip_image015](https://lh3.googleusercontent.com/-ivqqR2SnQqk/VuCjPu4DoPI/AAAAAAAAAB0/MHJByFfZHXE/clip_image015_thumb.png?imgmax=800 "clip_image015")

UNIQUE NAME

tSystem\_1

INPUT(S)

none

LABEL

\_\_UNIQUE\_NAME\_\_

OUTPUT(S)

tHttpRequest\_2

  
**Component Parameters:**  

**Properties**

**Values**

Unique Name

tSystem\_1

Component Name

tSystem

Version

0.101 (ALPHA)

Family

System

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

org.talend.help.tSystem

Update components

true

IREPORT\_PATH

JAVA\_LIBRARY\_PATH

C:\\install\\TOS\_BD-20151214\_1327-V6.1.1\\configuration\\lib\\java

Subjob color

Title color

Use Home Directory

false

Home Directory

"C:/install/TOS\_BD-20151214\_1327-V6.1.1/workspace"

Use Single Command

true

Command

"C:\\\\Python27\\\\python.exe D:\\\\Data\\\\Qualys\\\\ExternalJob\\\\QualysDownloadSelenium.py"

Use Array Command

false

Command

\[\]

Standard Output

OUTPUT\_TO\_CONSOLE

Error Output

OUTPUT\_TO\_CONSOLE

REPOSITORY\_ALLOW\_AUTO\_SWITCH

false

!!!SCHEMA.NAME!!!

null:

Environment variables

\[\]

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

  
**Schema for tSystem\_1 :**  

**Column**

**Key**

**Type**

**Length**

**Precision**

**Nullable**

**Comment**

  
**Original Function Parameters:**  

**Component   tSystem**

![clip_image015[1]](https://lh3.googleusercontent.com/-foV_MLovWvk/VuCjP17BgmI/AAAAAAAAAB4/80UeYBZm56E/clip_image015%25255B1%25255D_thumb.png?imgmax=800 "clip_image015[1]")

UNIQUE NAME

tSystem\_2

INPUT(S)

tFileOutputDelimited\_2

LABEL

\_\_UNIQUE\_NAME\_\_

OUTPUT(S)

none

  
**Component Parameters:**  

**Properties**

**Values**

Unique Name

tSystem\_2

Component Name

tSystem

Version

0.101 (ALPHA)

Family

System

Start

false

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

org.talend.help.tSystem

Update components

true

IREPORT\_PATH

JAVA\_LIBRARY\_PATH

C:\\install\\TOS\_BD-20151214\_1327-V6.1.1\\configuration\\lib\\java

Subjob color

Title color

Use Home Directory

false

Home Directory

"C:/install/TOS\_BD-20151214\_1327-V6.1.1/workspace"

Use Single Command

true

Command

"cmd /c del D:\\\\Data\\\\Qualys\\\\Xml\\\\\*.xml"

Use Array Command

false

Command

\[\]

Standard Output

OUTPUT\_TO\_CONSOLE

Error Output

OUTPUT\_TO\_CONSOLE

REPOSITORY\_ALLOW\_AUTO\_SWITCH

false

!!!SCHEMA.NAME!!!

null:

Environment variables

\[\]

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

  
**Schema for VTM :**  

**Column**

**Key**

**Type**

**Length**

**Precision**

**Nullable**

**Comment**

QUALYS\_TICKET\_NUMBER

false

Integer

10

true

TICKET\_CREATION\_DATETIME

false

String

20

true

TICKET\_RESOLVED\_DATETIME

false

String

28

true

TICKET\_MODIFIED\_DATETIME

false

String

28

true

TICKET\_DUE\_DATETIME

false

String

20

true

TICKET\_CURRENT\_STATE

false

String

6

true

IP

false

String

15

true

PORT

false

String

4

true

QUALYS\_SEVERITY

false

String

255

true

QID

false

Integer

6

true

TRACKING\_METHOD

false

String

11

true

OS

false

String

80

true

NETBIOS

false

String

15

true

DNS

false

String

45

true

LAST\_COMPLIANCE\_SCAN\_DATETIME

false

java.util.Date

20

true

LAST\_VULN\_SCAN\_DATETIME

false

java.util.Date

20

true

TITLE

false

String

241

true

CATEGORY

false

String

27

true

CVE\_ID

false

String

2855

true

CVSS\_BASE

false

String

3

true

BUGTRAQ\_ID

false

String

263

true

QID\_MODIFIED

false

String

28

true

QID\_PUBLISHED

false

String

28

true

CIDR

false

String

14

true

Zone

false

String

8

true

Department

false

String

3

true

Function

false

String

15

true

  
**Original Function Parameters:**  

**Component   tXMLMap**

![clip_image017](https://lh3.googleusercontent.com/-UosJb62inxA/VuCjQHC9rSI/AAAAAAAAAB8/Ca_OFepCJ6Y/clip_image017_thumb.png?imgmax=800 "clip_image017")

UNIQUE NAME

tXMLMap\_1

INPUT(S)

tXMLMap\_1,  tFileInputXML\_2,  tFileInputXML\_5,  tFileInputDelimited\_1

LABEL

\_\_UNIQUE\_NAME\_\_

OUTPUT(S)

tFileOutputDelimited\_2

  
**Component Parameters:**  

**Properties**

**        Values**

Activate

         true

tStatCatcher Statistics

         false

Map Editor:

Keep order for document

         false

Show Information

         false

Comment

Use an existing validation rule

         false

  
**Schema for VTM :**  

**Column**

**Key**

**Type**

**Length**

**Precision**

**Nullable**

**Comment**

QUALYS\_TICKET\_NUMBER

false

Integer

10

true

TICKET\_CREATION\_DATETIME

false

String

20

true

TICKET\_RESOLVED\_DATETIME

false

String

28

true

TICKET\_MODIFIED\_DATETIME

false

String

28

true

TICKET\_DUE\_DATETIME

false

String

20

true

TICKET\_CURRENT\_STATE

false

String

6

true

IP

false

String

15

true

PORT

false

String

4

true

QUALYS\_SEVERITY

false

String

255

true

QID

false

Integer

6

true

TRACKING\_METHOD

false

String

11

true

OS

false

String

80

true

NETBIOS

false

String

15

true

DNS

false

String

45

true

LAST\_COMPLIANCE\_SCAN\_DATETIME

false

java.util.Date

20

true

LAST\_VULN\_SCAN\_DATETIME

false

java.util.Date

20

true

TITLE

false

String

241

true

CATEGORY

false

String

27

true

CVE\_ID

false

String

2855

true

CVSS\_BASE

false

String

3

true

BUGTRAQ\_ID

false

String

263

true

QID\_MODIFIED

false

String

28

true

QID\_PUBLISHED

false

String

28

true

CIDR

false

String

14

true

Zone

false

String

8

true

Department

false

String

3

true

Function

false

String

15

true

  
  
  
P.S. for the astute, no I did not leave any credentials of use :)  it's garbage...