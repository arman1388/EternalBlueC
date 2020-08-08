[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

## EternalBlueC
EternalBlue suite remade in C/C++ which includes: MS17-010 Exploit, EternalBlue/MS17-010 vulnerability detector, DoublePulsar detector and DoublePulsar UploadDLL & Shellcode 

## Project goals

[*] Convert to other languages such as Java and C# and implement a scanner & attack GUI

[*] Allow editing of EternalBlue exploit payload to remove DoublePulsar and allow custom payloads & shellcode to be sent instead.

[*] Add EternalRomance (Requires Named Pipe) support

## Extra education code

Repository also contains the following for educational purposes and are NOT supported by me:

[*] DoublePulsar x86/x64 Upload DLL python script

[*]EternalBlue All in one binary

[*]Multi arch kernel queue apc assembly code & Windows x86/x64 Multi-Arch Kernel Ring 0 to Ring 3 via Queued APC kernel code

[*]EternalBlue x64/x86 kernel payload shellcode from worawit

[*]Eternalblue replay file

## EternalBlue Suite

* **ms17_vuln_status.cpp** - This program sends 4 SMB packets.  1 negociation packet and 3 requests.  This program reads the NT_STATUS response from a TransNamedPipeRequest ( PeekNamedPipe request ) and determines if NT_STATUS = 0xC0000205 ( STATUS_INSUFF_SERVER_RESOURCES ).  If this is the correct response, then the target is vulnerable to MS17-010.  Tested on unpatched Windows 7 x64 bit.

![ms17vulnstatus](/images/ms17vulnstatus.PNG)

* **doublepulsar_check.cpp** - This program sends 4 SMB packets.  1 negociation packet and 3 requests.  The last request is a Trans2 SESSION SETUP request.  Doing so, the multiplex id can be compared against value: 0x51 or 81.  If that is the response, that means DoublePulsar is present on the machine.  Afterwards, you can send commands to burn the DoublePulsar backdoor.  ( NOTE: DoublePulsar becomes dormant and not removed ).  Tested on Windows 7 x64 bit.

![doublepulsar_check](/images/doublepulsar_check.PNG)

* **EternalBlue.cpp** - This program sends multiple SMB packets.  Negociation, SessionSetup, TreeConnect and multiple NT trans and Trans2 packets.  These NT trans packets are malformed which grooms the exploit in memory on the victim computer.  More whitespace or empty SMB packets are sent to the victim over multiple sockets to the same port on the victim.  Most of EternalBlue's base64 payload is being sent over socket 1 where the Negociation, SessionSetup & TreeConnect packets were sent on.  Then 20 other sockets are created and data is being sent to those sockets ( Socket 3 to Socket 21 ).  Afterwards DoublePulsar is sent on Socket 3 to Socket 21.  The sockets are then closed by the program which detonates EternalBlue & DoublePulsar on the victim computer.  A SMB disconnect and SMB logoff request is then sent and the connection closes.  This exploit works and was tested on Windows 7 x64 bit.  It took around 5 seconds for the backdoor to fully be operational, as already reported with EternalBlue writeups around the internet.  More exploitation attempts may be necessary.  However there currently is a bug with the TreeID & UserID not being correctly set in the packets, which will be fixed in a later release.
![EternalBlue](/images/eternalbluecpp.PNG)
![EternalBlue_cmd](/images/eternalbluecmd.PNG)
![eternalbluepackets](/images/eternalbluepackets.PNG)

* **DoublePulsarXORKeyCalculator.cpp** - This program sends 4 SMB packets.  1 negociation packet and 3 requests.  The last request is a Trans2 SESSION SETUP request.  A Trans2 SESSION_SETUP response is then recieved and the SMB signature is extracted at (Recvbuff[18] -> Recvbuff[22]) .  The SMB signature is converted from the hex characters into an unsigned integer.  This unsigned integer is ran through the DoublePulsar Xor key calculator function, which generates a XOR key that can be used to encrypt the shellcode or DLL payload that will be uploaded to DoublePulsar.  NOTE: The SESSION_SETUP data parameters must contain the char version of the calculated DoublePulsar XOR key in the payload upload portion of this repository.  Tested on Windows 7 x64 bit.
Sample screenshot:

![XORCalculator](/images/XORCalculator.PNG)

## Not finished

* **doublepulsar_upload.cpp** - This program sends 4 SMB packets.  1 negociation packet and 3 requests.  The last request is a Trans2 SESSION SETUP request to obtain the DoublePulsar XOR key via the returned signature field in the TRANS2 SMB response packet. This key is ran through the DoublePulsar XOR key calculator extracted from the DoublePulsar binary. Then the program reads a DLL file (Example: payload.dll) and combines it with userland shellcode (userland_shellcode.bin) and XORs the buffer with the DoublePulsar XOR key we calculated from the SMB signature.  A packet is generated by allocating memory, copying the Trans2 packet, editing the values needed for the SMB transaction to work ( UserID, ProcessID, TreeID, MultiplexID ) then copying the XORed data (shellcode + DLL) to the end and loop through it sending it at a total packet length of 4096 bytes at a time to DoublePulsar.  Total packet length = 4178.  4096 is for the XOR encrypted data.  This is still a work in progress.  

TODO: Might need to implement the Trans2 Upload function using a structure and not editing the Trans2 packet capture using hexadecimal.

[NOT FINISHED]
* **Doublepulsar_UploadShellcode.cpp** - This program sends 5 SMB packets.  1 negociation packet and 4 requests.  The last request is a Trans2 SESSION SETUP request to obtain the DoublePulsar XOR key via the returned signature field in the TRANS2 SMB response packet. This signature is ran through the DoublePulsar XOR key calculator (extracted from the DoublePulsar binary). Then the program copies sample shellcode from a buffer ( NOPs / x90 ), it XORs the buffer with the DoublePulsar XOR key we calculated from the SMB signature.  A packet is generated by allocating memory, copying the Trans2 exec packet (from Wannacry), edits the values needed for the SMB transaction to work ( UserID, ProcessID, TreeID, MultiplexID ) then copying the SESSION_PARAMETERS & XORed data (shellcode) to the end and send it at a total packet length of 4178 bytes to DoublePulsar.  Total packet length = 4178.  4096 is for the XOR encrypted data.  This is still a work in progress.  

TODO: Might need to implement the Trans2 Upload function using a structure and not editing the Trans2 packet capture using hexadecimal.
