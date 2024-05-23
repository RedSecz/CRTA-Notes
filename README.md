# CRTA Notes

## Initial Access
Running Nmap to gather server information:
- Scanning active target: `sudo nmap -sn <target IP Prefix>
- Scanning target service and version: `sudo nmap -sC -sV <Active target IP>`

Exploit the web application (input field) to get initial foot hold:
- LFI --> /etc/passwd
- XSS
- File upload to reverse shell
- SSRF
- Direct reverse shell
- etc

From the information of exploiting web application:
- reverse shell
- ssh credential

## Enumeration
After got SSH/reverse shell. Check the **IP Config** `ip a`  to check the network
- Usually will have 2 IP, 1 internal and 1 External
- Try to run nmap from the external machine to the Internal IP subnet 
	- `sudo nmap -sn <Internal IP Prefix` e.g: sudo nmap -sn 192.168.98.0/24
- Scanning target service and version: 
	- `sudo nmap -sC -sV <Active target IP>`
- After that try to check hidden file such as `ls -la` to find hidden interesting file
- Don't forget to check the browser bookmarks (e.g /.mozilla/firefox/xxx/places.sqlite), if the file extension is sqlite. Open it with `sqlite3` --> additional command:
	- `.tables` --> to show tables
	- `select * from <tables>;` --> to show data in the tables

## SSH Pivoting

Config the proxychains socks4 & 5 to 127.0.0.1 9050 (/etc/proxychains4)

Running `ssh -D 9050 <user>@<ip>`

## SMB Enumeration
Scanning target service and version: `sudo nmap -sC -sV <Active target IP>`

To perform SMB enumeration can user netexec/crackmapexec (deprecated)
- Enumerate Hosts: `nxc smb 192.168.1.0/24`
- Check for smb allow null session: 
	```
	- nxc smb 10.10.10.161 -u '' -p ''`
	- nxc smb 10.10.10.161 -u '' -p '' --shares
	- nxc smb 10.10.10.161 -u '' -p '' --pass-pol
	- nxc smb 10.10.10.161 -u '' -p '' --users
	- nxc smb 10.10.10.161 -u '' -p '' --groups
	````
- Reproduce this behavior with `smbclient` or `rpcclient`
	```
	smbclient -N -U "" -L \\10.10.10.161
	```

	```
	rpcclient -N -U "" -L \\10.10.10.161
	rpcclient $> enumdomusers
	user:[bonclay] rid:[0x46e]
	user:[zoro] rid:[0x46f]
	```
- Scan for Vulnerabilities
	```
	ZeroLogon
	nxc smb <ip> -u '' -p '' -M zerologon
	
	PetitPotam
	nxc smb <ip> -u '' -p '' -M petitpotam
	
	noPAC
	nxc smb <ip> -u 'user' -p 'pass' -M nopac
	```
