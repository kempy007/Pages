---
title: 'SQL Sha1 cracking'
date: 2017-12-29T12:10:00.000Z
draft: false
aliases: [ "/2017/12/sql-sha1-cracking.html" ]
nav_order: 9000
description: "blogger"
has_children: false
has_toc: false
permalink: docs/blogger
has_children: false
has_toc: false
---

We can base64decode the passwords table then Ascii to Hex to reveal a hash value.

  

Having found some details of how the password hashing mechanism may work at a forum the function below was extracted. [https://forums.asp.net/t/1336657.aspx](https://forums.asp.net/t/1336657.aspx)?

ASP+NET+Membership+and+User+Password+Hashing+SHA1+Issues

  

  

  

        static string EncodePassword(string pass, string saltBase64)

        {

            byte\[\] bytes = Encoding.Unicode.GetBytes(pass);

            byte\[\] src = Convert.FromBase64String(saltBase64);

            byte\[\] dst = new byte\[src.Length + bytes.Length\];

            Buffer.BlockCopy(src, 0, dst, 0, src.Length);

            Buffer.BlockCopy(bytes, 0, dst, src.Length, bytes.Length);

            HashAlgorithm algorithm = HashAlgorithm.Create("SHA1");

            byte\[\] inArray = algorithm.ComputeHash(dst);

            return Convert.ToBase64String(inArray);

        }

  

  

// {CDB719C9-38AA-47D0-BF1E-58CEC7F90AD2}

IMPLEMENT\_OLECREATE(<<class>>, <<external\_name>>,

0xcdb719c9, 0x38aa, 0x47d0, 0xbf, 0x1e, 0x58, 0xce, 0xc7, 0xf9, 0xa, 0xd2);

  

exaZyTmpR9C/HljOx/kK0g==

  

Salt is created with a GUID with the curly brackets and hyphens stripped, then we do hex to ascii > base64 encode.