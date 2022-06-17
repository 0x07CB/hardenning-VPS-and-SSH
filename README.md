# hardenning-VPS-and-SSH
made your warsip now !



## How to improve security of the SSH of your Debian VPS
The little guide of hardening about SSH security settings for Vultr instances usages for every levels of competences.

## Introduction
> This tutorial explain how to setup SSH access correctly by standard usages in cryptography, rules and strategy to block and/or auto detect attacks from unethical hacker’s to your vps ,steps by steps and by priority. That include little parts of explains about the pirates strategy to take control of your server overs SSH.
> But I write an precise thing: in this tutorial, all settings are explained to understand risks and exploit possibility if an mistake do during settings.
> This not the warranty of risk zero because that can’t be possible. But more you follow and learn with new technicals setup in cybersecurity more you can have chance to block attacks before the success of them.

## Prerequisites
You need an good environnement dry common for train yourself first, try with debian 11 vpns fr cloud computing or directly if you ca your dedicated machine:

* Debian 11 is OS required
* 25GB of space are good for train
* Required ssh-audit, ufw, fail2ban, googlelib auth, cryptsetup
* Intermediate levels on linux systems
* Have the reflex to follow an logical order in the hardening process

## Before all, prepare yourself
**Don’t create your vps now, read first this.**

Passwords:
**First** take in head that: your vps created at the first minutes he is naked (zero security). And if you have registered keys (elliptical sure) that don’t disable automatically the permit to be authenticated by password so, weak or stronger password that have big importance you can understand but it’s not an security. This an fact with the protocol SSH the standards it’s to disallow password because that can’t be resistant side of brute force attacks and dictionary based brute force, MITM(Man of The Middle attacks) : an attack to intercept information and password it’s just clear in the tunnelled connection over SSH… and a lot of attacks possibles, like crazy case but really easy way for bad hacker’s : the key loggers on the victim computer record and be transmit or be taked (if it’s an physical device, because that can be just software)…

SSH PORT:
But it’s not the only thing for protect your SSH better because the first to do for an bad hacker’s it’s just gathering information about… the number of the PORT used for the service of your SSH server( by default it’s 22 but you can and highly recommend to change this and choose an upper number than 22 by exemple 2222 because it’s over 1024 reserved ports of the system but 2222 it’s too many common please use more randomly choice of them and keep that for you, and you use 2222 or 22 ports to place the service in this tutorial: endlessh (he’s an trap infinite for attack over the presumed SSH, that can be fun but that made difficult by increase the slower speed of an attacker to the success for exploit your SSH.

TENTATIVE DETECTION AND BANNISSEMENT OF ATTACKS
But yeah you have need to ban tentatives because a lot of praticables attacks exists, so in this tutorial the simple usage of fail2ban explained first fast and details at end of this article. (This is an software to read constantly the logs of sshd, the daemon of ssh, because in there you can found tentative of brute force and every fails of authentication; so fail2ban it’s in the name after n fails adequately with the configuration file of the jail-settings ,the IP address registered in the log and suspected to have made an illegitimate grant to access of the SSH on the VPS be bans immediately for an determined time by the same configuration.(in this case the pirates have need a lot of ip adresses, and machine, proxy, zombie, and else to have the possibility to re-try the attack it’s just heavy and not fun not easy and not the good way to success. ; But in the case of an attack inside an public or local network area : the DHCP and DNS locally can be less or more easy way by bad settings of them for an pirate device to change just the ip adresses MAC adresses (physical adresses, like an serial number of material, used for the telecommunications interface: bluetooth, wifi card , ether port ,…).
So an local filtering by IP or MAC or Hostname can’t be an full security.

CRYPTOGRAPHIE BAD USAGE OF NON STANDARDS
In common case regular vps are built ready to use but have pre generated ssh keys and cryptography settings, you need to fix that rapidly, to fix that you can refer you to an SSH hardening guide about this settings, in this tutorial I show to you how to audit your SSH to see if this an need and how to fix that on Debian 11.
That can impact directly the quality of cryptography and the secure connection between the client and the VPS.

FIREWALL FULL SETTINGS
In this tutorial I show to you how to lock fully your firewall

ELLIPTICAL KEYS
Because rsa key s not strong security today you need to use more strong keys : elliptical keys (ecdsa and ed25519) in this tutorial you can see who to do that.

2FA AND U2FA
Use double authentication and physical double authentication, in this tutorial.

ROOT AND LIMITED SUDOER USER
Because root expose your vps to too many attacker’s

AUDITING AND PENTEST YOUR SSH
With your ow computer or anther vps right you can use ssh-audit by exemple. This exemple in this tutorial.


## Let’s go !

1. Install an ssh client if you don’t have that on your computer.
2. create an debian vps if you have not do that.
3. You need to choose 25~50Gb of space system disk for your vps.
4. Main packages required are: ufw fail2ban libpam-google-authenticator, but you need to install more...
5. Follow this next steps:

## 1. Prepare your Credentials

Install pwgen
```
sudo apt insall pwgen
```

Because you have first and internally nee of passwords’ generate them ! 
Prepare on your computer 2 passwords.
First is for root and Second is for sudoers limited user
with pwgen that look like that:

```
pwgen -snc 49 2 > passwords && chmod 0600 password
nano password
```

Generate root and limited sudoers user SSH elliptical keys.
You have two choice of algoritms `ECDSA` or `ED25519`.

For `ECDSA` (note: for the U2F version it’s `ECDSA-SK`)
```
ssh-keygen -t ecdsa -b 521 -f id-ecdsa-root -C "to root"
ssh-keygen -t ecdsa -b 521 -f id-ecdsa-spongebob -C "to spongebob"
```

OR 

For `ED25519` (note: for the U2F version it’s `ED25519-SK`)
```
ssh-keygen -t ed25519 -a 1000 -f id-ed25519-root -C "to root"
ssh-keygen -t ed25519 -a 1000 -f id-ed25519-spongebob -C "to spongebob"
```


## 2. Modify SSH cryptography settings

If you are not in root use the `su` command like that:
```
sudo su - root
```

And paste that into the VPS terminal.
```
rm -f /etc/ssh/ssh_host_*

ssh-keygen -t rsa -b 4096 -f /etc/ssh/ssh_host_rsa_key -N ""

ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ""


sed -i 's/^\#HostKey \/etc\/ssh\/ssh_host_\(rsa\|ed25519\)_key$/HostKey \/etc\/ssh\/ssh_host_\1_key/g' /etc/ssh/sshd_config

awk '$5 >= 3071' /etc/ssh/moduli > /etc/ssh/moduli.safe

mv -f /etc/ssh/moduli.safe /etc/ssh/moduli

echo -e "\n# Restrict key exchange, cipher, and MAC algorithms, as per sshaudit.com\n# hardening guide.\nKexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group-exchange-sha256\nCiphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr\nMACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,umac-128-etm@openssh.com\nHostKeyAlgorithms ssh-ed25519,ssh-ed25519-cert-v01@openssh.com,sk-ssh-ed25519@openssh.com,sk-ssh-ed25519-cert-v01@openssh.com,rsa-sha2-256,rsa-sha2-512,rsa-sha2-256-cert-v01@openssh.com,rsa-sha2-512-cert-v01@openssh.com" > /etc/ssh/sshd_config.d/ssh-audit_hardening.conf
service ssh restart
```
try to open anoter new session and accept new fingerprints done !


## 3. Create an limited sudoer user

Create an user’ set an password for tis user, copy `.ssh` folder to him home folder.
Give full right on him folder after your modification and add the user to sudo group.
And restart sshd.

```
useradd spongebob
passwd spongebob
mkdir -p /home/spongebob/.ssh
touch /home/spongebob/.ssh/authorized_keys
cp ~/.ssh/authorized_keys /home/spongebob/.ssh/authorized_keys
chown -R spongebob:spongebob /home/spongebob/
chown -R spongebob:spongebob /home/spongebob/.ssh
usermod -aG sudo spongebob
systemctl restart sshd
```

Next change the shell attributed to him.
```
vim /etc/passwd
```
if he have SH change to BASH.
```
spongebob:x:1000:1000:Sponge Bob:/home/spongebob:/usr/bin/sh
```
TO
```
spongebob:x:1000:1000:Sponge Bob:/home/spongebob:/usr/bin/bash
```



## 4. Hardening around your SSH

modify the configuration file of sshd:

First allow only user you want to connect.
```
AllowUsers spongebob
```

Modify the port(up than 1024).
```
Port 18888
```

Change log level to `VERBOSE`.
```
LogLevel VERBOSE
```

Change `PermitRootLogin` to `no` for stop the exposure of the root user.
```
PermitRootLogin no
```

Change `MaxAuthTries` to `3`.
```
MaxAuthTries 3
```

Change `MaxSessions` to `2`.
```
MaxSessions 2
```

Uncomment and set to yes `PubkeyAuthentication`
```
PubkeyAuthentication yes
```

Uncomment `AuthorizedKeysFile`
```
# Expect .ssh/authorized_keys2 to be disregarded by default in future.
AuthorizedKeysFile    .ssh/authorized_keys .ssh/authorized_keys2
```

Disable `PasswordAuthentication` and `PermitEmptyPasswords`.
```
# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
PermitEmptyPasswords no
```

Disable `ChallengeResponseAuthentication`
```
# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication no
```

Set to no : `AllowAgentForwarding` and `AllowTcpForwarding`
```
AllowAgentForwarding no
AllowTcpForwarding no
```

Set to no the `X11Forwarding`.
```
X11Forwarding no
```

Disable `TCPKeepAlive`
```
TCPKeepAlive no
```

And Compression too.
```
Compression no
```

Set `ClientAliveInterval` to 300.
set `ClientAliveCountMax` to 0.
```
ClientAliveInterval 300
ClientAliveCountMax 0
```

set `ClientAliveInterval` to 120.
```
ClientAliveInterval 120
```

After settngs save and made `systemctl restart sshd`!

An complete file look like that:
```
#	$OpenBSD: sshd_config,v 1.103 2018/04/09 20:41:22 tj Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf
AllowUsers spongebob
Port 18888
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
LogLevel VERBOSE

# Authentication:

#LoginGraceTime 2m
PermitRootLogin no
#StrictModes yes
MaxAuthTries 3
MaxSessions 2

PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
AuthorizedKeysFile	.ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via ChallengeResponseAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and ChallengeResponseAuthentication to 'no'.
UsePAM yes

AllowAgentForwarding no
AllowTcpForwarding no
#GatewayPorts no
X11Forwarding no
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
PrintMotd no
PrintLastLog yes
TCPKeepAlive no
#PermitUserEnvironment no
Compression no
ClientAliveInterval 300
ClientAliveCountMax 0
#UseDNS no
#PidFile /var/run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem sftp	/usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#	X11Forwarding no
#	AllowTcpForwarding no
#	PermitTTY no
#	ForceCommand cvs server
ClientAliveInterval 120
```

## FIREWALL FULL SETTINGS

First allow your port of SSH protocol in inbound on TCP layer.
```
ufw allow in 18888/tcp
```
Because in this exemple the SSH are setted to listen the port 1888.
You can LIMIT this port like that:
```
ufw limit in 18888/tcp
```

Now you set deny incoming by default.
```
ufw default deny incoming
```
If you have don’t made mistake you can enable.
Reply y to this question.
```
ufw enable 
```
```
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

Now set outgoing rules
```
ufw allow out 123/udp
ufw allow out 80
ufw allow out 443
ufw allow out 5353
ufw allow out 53
ufw allow out 21
ufw allow out 22
ufw allow out 18888
```

Ready to made an default deny for outgoing!
```
ufw default deny outgoing
```

Now reload UFW
```
ufw reload
```

Don’t forgot to enable systemd service of UFW
```
systemctl enable ufw
```

Check is good.
```
systemctl status ufw
ufw status verbose
```



## 5. Setting up fail2ban

```
apt install -y fail2ban
```

Edit the file `/etc/fail2ban/jail.conf` and find the SSHD Jail.
To Add this:
```
enable = true
bantime = 4w
maxretry = 3
```
For Enable the jail set bantime and maxretry.


At the ehd the jail look like:
```
[sshd]

# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
#mode   = normal
enable = true
bantime = 4w
maxretry = 3
port    = ssh
logpath = %(sshd_log)s
backend = %(sshd_backend)s
```

## 6. Setting up endlessh

Install endlessh
```
apt install endlessh
```

Now Enable and start `endlessh`
```
systemctl enable --now endlessh
```

Check the status of endlessh
```
systemctl status endlessh
● endlessh.service - Endlessh SSH Tarpit
     Loaded: loaded (/lib/systemd/system/endlessh.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-06-15 04:37:19 UTC; 6h ago
       Docs: man:endlessh(1)
   Main PID: 1221 (endlessh)
      Tasks: 1 (limit: 38215)
     Memory: 216.0K
        CPU: 29ms
     CGroup: /system.slice/endlessh.service
             └─1221 /usr/bin/endlessh -p 4444

Jun 15 04:37:19 ns3117432 systemd[1]: Started Endlessh SSH Tarpit.

```
The file of the service are accessible on this path: `/lib/systemd/system/endlessh.service`


open the service file
```
vim
```

modify the port (default 2222)
by exemple I use the 4444 port in tis configuration exemple.
```
...

[Service]
Type=simple
Restart=always
RestartSec=30sec
ExecStart=/usr/bin/endlessh -p 4444
KillSignal=SIGTERM

...

```
Save and,
reload daemons of systemd.
```
systemctl daemon-reload
```

restart endlessh to change really this.
```
systemctl restart endlessh.service
```

and allow the tarpit to be accessible with UFW:
```
ufw allow in 4444/tcp
ufw reload
```

Check again.
```
systemctl status endlessh
ufw status
```
Done.


## 7. Install haveged for entropy

```
apt install haveged
echo ’DAEMON_ARGS="-w 1024"’ >> /etc/default/haveged
update-rc.d haveged defaults
```

## 8. Install SSH-AUDIT

installation :
```
sudo apt update -y && sudo apt install -y ssh-audit
```

## 9. Use SSH-AUDIT to your VPS

In this case you type the command `ssh-audit -p <ssh_port> <VPS_IP_OR_DOMAIN>` in your initial computer, like your office computer or another vps, but not the target himself.


## 10. Install lynis

install this and launch an audit of the system to continue.
```
apt install lynis
```
```
sudo lynis audit system
```
Congratulations you can continue to follow lynis.


# Source documents (ssh hardening guide) used for this article:
`https://www.sshaudit.com/hardening_guides.html`
