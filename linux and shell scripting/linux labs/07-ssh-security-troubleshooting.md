## SSH Security: An Audit Perspective

### Step 1: Seeing the "True" Configuration
- The /etc/ssh/sshd_config file only shows changes. To see the effective configuration (including all the hidden defaults), we use sshd -T .

**1. Dump the active configuration:**
```sh
sshd -T | grep -E "port|permitrootlogin|passwordauthentication"
```

**2. Analyze the output:**
  - port 22 : The standard entry point for attackers.
  - permitrootlogin yes : This means an attacker can try to brute-force the 'root' password directly.
  - passwordauthentication yes : This means the server accepts passwords, which are weaker than keys.

### Step 2: Understanding PermitRootLogin

- In your audit, you likely saw PermitRootLogin yes .

**Why is this a risk?**
   - Every Linux system has a user named root . This gives hackers 50% of the login credentials (the username) for free.
  
| Option              | Meaning                                                                                  | Security Level |
| ------------------- | ---------------------------------------------------------------------------------------- | -------------- |
| `yes`               | Root can log in directly using a **password or SSH key**                                 |  Least secure  |
| `prohibit-password` | Root login is allowed **only using SSH keys** (password login disabled)                  |  More secure   |
| `no`                | Root login is **completely disabled**. Users must log in as a normal user and use `sudo` |  Most secure   |

> Concept Check: By setting this to no , an attacker must first guess a valid username and then guess a password/key, then bypass sudo .

### Step 3: The Listening Surface
**Security isn't just about files; it's about what is happening on the network.**

**1. Check the socket:**
```sh
ss -tlpn | grep ssh
```
- Understand the output: The *:22 or 0.0.0.0:22 means SSH is listening on all available network interfaces on port 22.

**The "Obscurity" Concept**
> By changing the port to something like 2222 , you don't make the encryption stronger, but you stop 99% of automated "bot" scripts that only scan port 22. This keeps your logs clean and reduces load on the CPU.

### Step 4: Testing without Breaking
- You can test a new configuration file without restarting the service. This is how pros avoid getting locked out.

**1. Create a "dummy" config:**
```sh
cp /etc/ssh/sshd_config /tmp/test_sshd_config
echo "Port 9999" >> /tmp/test_sshd_config
```

**2. Run a test pass:**
```sh
sshd -t -f /tmp/test_sshd_config
```
> If there is no error, the config is safe to use. If you had a typo, it would tell you here.

```sh
ubuntu:~$ sshd -T | grep -E "port|permitrootlogin|passwordauthentication"
port 22
permitrootlogin yes
passwordauthentication yes
gatewayports no

ubuntu:~$ ss -tlpn | grep ssh
LISTEN 0      4096         0.0.0.0:22         0.0.0.0:*    users:(("sshd",pid=1140,fd=3),("systemd",pid=1,fd=93))
LISTEN 0      4096            [::]:22            [::]:*    users:(("sshd",pid=1140,fd=4),("systemd",pid=1,fd=94))

ubuntu:~$ cp /etc/ssh/sshd_config /tmp/test_sshd_config
ubuntu:~$ echo "Port 9999" >> /tmp/test_sshd_config
ubuntu:~$ sshd -t -f /tmp/test_sshd_config
ubuntu:~$
```
