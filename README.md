# pen-leviathan
pentesting notes

## Priv Escalation
- If we grab hashes for a system, we can use mimikatx to grab them from memory. This needs to be done in a 64 bit process so that meterpreter can load the module without problem.
- Run `sysinfo` and check Meterpreter value. (x86/win32 aint gonna cut it)
- To find a process to migrate to, run `ps -A x86_64 -s` when you are a `system`. This will display available compatible processes to migrate to.
