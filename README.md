# TIL - Today I Learned

Notes about C, Bash, Unix, and Vim.

### Meta
This is a page is a selection of notes and tips to make life easier when working in a POSIX environment. Some tips are elementary, and some are fairly specific, sophisticated, or obscure. Most of these notes will work in a vanilla *nix environment.

---
## Table of Contents
* [Bash](#bash)
* [libreadline](#libreadline)
* [Networking](#networking)
* [POSIX](#posix)
 * [Debian](#debian)
* [Vim](#vim)

## Bash
### Go to the previous directory
`cd -` will take you to the previous directory. The previous directory is stored in the `$OLD_PWD` env var.

### Pidof
`pidof` does what it says on the label, it returns a list of pids matching the program name. E.g.,
```
pidof ssh
29421 28283 28180 28031 27122 22432 21558 21480 21230...
```
`pidof` is useful for scripting and I also use it for this gdb oneliner to attach to a running process by name.
```
gdb attach $(pidof <daemon name>)
```
Caveat on the gdb onliner, in my use case I'm attaching a to a daemon that has a single instance. If your attaching to one process of many that share the same name the `-s` which returns only 1 pid may or may not be helpful. The man page fails to document which pid it returns; in my expirience it returns the largest pid, your milage may vary.
 
### Ranges and String Expansions
Bash supports creating range of integers with the syntax `{start...end}`. Bash will expand the conents into a string  delimited with spaces. This is particular usefuly for for loops which recognizes the space delimited format. In our simplistic example we echo the values from 1 to 10.
```
$ echo {1..10}
1 2 3 4 5 6 7 8 9 10
```
Bash also supports string exansion using the same syntax. E.g., `echo a{d,c,b}e` yeilds `ade ace abe`.
Bash also supports empty parameters `cp /etc/some/deep/path/some.file{,.bak}` becomes `cp /etc/some/deep/path/some.file /etc/some/deep/path/some.file.bak`. 

### Redirect only stderr.
The trick is to redirect stderr to stdout with `2>&1` and then redirect stdout to the bit bucket with `>/dev/null`. E.g.,
```
<command> 2>&1 >/dev/null | grep "or somthing" 
```
Foobarrrrrrr
### Searching Man Pages
Use the apropos command to search through man pages.
```
$ apropos /proc
slabinfo (5)         - kernel slab allocator statistics
nfsiostat (8)        - Emulate iostat for NFS mount points using /proc/self/mountstats
xqmstats (8)         - Display XFS quota manager statistics from /proc
```

---
## libreadline - the transparent interface.
At first glance the the following tips look like they belong in the bash section. Here's the thing, bash uses libreadline which provides the cursor, line editing, and history. This is a long and convoluted way of saying that these tips should be valid for any shell or program that uses libreadline not just bash. 

### History Expansion
History expansions are built up from a series of subcommand, but they all must be prefixed with the `!`.

#### Index subcommands
* **Absolute Index:** When you want to repeat a history entry you simple call `!<index>`. In this example I want to repeat `ssh -Q cipher` query after mofiying my ssh config. 
```
$ history | head
    1  ssh -Q cipher
    2  vim ~/.ssh/config
    ...
$ !1
ssh -Q cipher
...
```
* **Relative Index:** History expansions also support relative indexes using the syntax `!-<how many commad ago>`. So in our previous example I could have also done.
```
$ !-2
ssh -Q cipher
aes256-gcm@openssh.com
chacha20-poly1305@openssh.com
```
* **Last Command Short Cut** We could use `!-1` to reference the previous command but that a drag so the `!!` shortcut exists. The most common use case is to prefix sudo to the bad idea you didn't have permissions for.
```
$ apt-get remove e2fsprogs
error: you cannot perform this operation unless you are root.
$ sudo !!
```

#### Search Subcommands 
* **Command Begins With:** `!<query>` will execute first command that starts with the query.
```
root@snoded834:~# !ps
ps -ef | grep rsyslog
root     29310     1  0 10:55 ?        00:00:00 rsyslogd -c5 -f /root/rsyslog.sec -i /tmp/foo
root     29327     1  0 10:55 ?        00:00:00 /usr/sbin/rsyslogd -c5
root     29335 29284  0 10:56 pts/1    00:00:00 grep rsyslog
root@snoded834:~# 
```
* **Command Contains:** `!?<query>` will execute the first command that contains the query.
```
root@snoded834:~# !?rsyslog
/etc/init.d/rsyslog restart
Stopping enhanced syslogd: rsyslogd.
Starting enhanced syslogd: rsyslogd.
```

---
## Make
### Debugging shelled out commands.
A shell command failing on you? Not sure what make is exactly executing? simply run. `make SHELL='sh -x'`

---
## Networking
### Examine a SSL/TLS Handshake
When debbuginng encyption problems my first step is to examine the handshake paramesters using `openssl s_client -connect`.
At first the output can be over whelming; in particular pay attention to certificate structure, protocols, and ciphers used. Here's an example handshake to www.google.com.
```
openssl s_client -connect www.google.com:443
CONNECTED(00000003)
depth=3 C = US, O = Equifax, OU = Equifax Secure Certificate Authority
verify return:1
depth=2 C = US, O = GeoTrust Inc., CN = GeoTrust Global CA
verify return:1
depth=1 C = US, O = Google Inc, CN = Google Internet Authority G2
verify return:1
depth=0 C = US, ST = California, L = Mountain View, O = Google Inc, CN = www.google.com
verify return:1
---
Certificate chain
 0 s:/C=US/ST=California/L=Mountain View/O=Google Inc/CN=www.google.com
   i:/C=US/O=Google Inc/CN=Google Internet Authority G2
 1 s:/C=US/O=Google Inc/CN=Google Internet Authority G2
   i:/C=US/O=GeoTrust Inc./CN=GeoTrust Global CA
 2 s:/C=US/O=GeoTrust Inc./CN=GeoTrust Global CA
   i:/C=US/O=Equifax/OU=Equifax Secure Certificate Authority
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIEgDCCA2igAwIBAgIIXDR9H6fDVBgwDQYJKoZIhvcNAQELBQAwSTELMAkGA1UE
BhMCVVMxEzARBgNVBAoTCkdvb2dsZSBJbmMxJTAjBgNVBAMTHEdvb2dsZSBJbnRl
cm5ldCBBdXRob3JpdHkgRzIwHhcNMTYwMjE3MTAyMDE3WhcNMTYwNTE3MDAwMDAw
WjBoMQswCQYDVQQGEwJVUzETMBEGA1UECAwKQ2FsaWZvcm5pYTEWMBQGA1UEBwwN
TW91bnRhaW4gVmlldzETMBEGA1UECgwKR29vZ2xlIEluYzEXMBUGA1UEAwwOd3d3
Lmdvb2dsZS5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC8+Ugs
pBXm3zFVRCA6k8DEXqpCf4Zw79y1dbgPuGHdw1NXawEvy8M4K3slQAwRBbGJO34Y
mVQEeJRK98kJ+dBAajlKGbOkqfk7ZdPpl50zSb+OmM5As4+w1K6gWo9CPt525PyS
/g/vdSj81XgCFQPNSLeTP2Uj6ZlXZpSyc1Ti+P6QZ/omOHtC/Lo1b9baQyQf7E7h
MOyTh8TAqJjTeVwg50SKhjzTRiY8t94JBXMknDL0eczEMtZRt5+Fwxe0li3xg5Aw
0bESlWU7qGluvjz+GFbSTdHfAIzYXxp86+zVvdyDTWGC5344GGtYCr5PRDNalV5o
wBxUVe6l1VYXBKDVAgMBAAGjggFLMIIBRzAdBgNVHSUEFjAUBggrBgEFBQcDAQYI
KwYBBQUHAwIwGQYDVR0RBBIwEIIOd3d3Lmdvb2dsZS5jb20waAYIKwYBBQUHAQEE
XDBaMCsGCCsGAQUFBzAChh9odHRwOi8vcGtpLmdvb2dsZS5jb20vR0lBRzIuY3J0
MCsGCCsGAQUFBzABhh9odHRwOi8vY2xpZW50czEuZ29vZ2xlLmNvbS9vY3NwMB0G
A1UdDgQWBBTiRG9FdyKQOTNltPaXqgJRKlSlPjAMBgNVHRMBAf8EAjAAMB8GA1Ud
IwQYMBaAFErdBhYbvPZotXb1gba7Yhq6WoEvMCEGA1UdIAQaMBgwDAYKKwYBBAHW
eQIFATAIBgZngQwBAgIwMAYDVR0fBCkwJzAloCOgIYYfaHR0cDovL3BraS5nb29n
bGUuY29tL0dJQUcyLmNybDANBgkqhkiG9w0BAQsFAAOCAQEAIUTrfaaB+cJSk20L
RHqDwaLWe8cyLR8Ks4Vee/ZxLQDcPuxItvlho0N+/j5ZUnU1XseyiE9yD6ezmY7e
ChyXUlzKzMdLyvjy7/EzTViW28Czbnp/JepBUipMDhJz7EMLdvqkw2cs0BwevRkU
6jzbQoYzOCalmWs1Mt4S8AyklbMHUjo/vOcs4+RePG9evxV0yWxCDNgLZbMckxcg
vL4S5P8C4cY96+qhRwR/ErYHFRkuniQleLz1tEMkei5sK3tY5Sae0uTGH2Z30fs0
RViv9SFdfjMQDMFmEabPoNermhUx9hjENfMvWqJ1r+dbDTl3ANt/feNa+d6Z3Zpz
MUtO9Q==
-----END CERTIFICATE-----
subject=/C=US/ST=California/L=Mountain View/O=Google Inc/CN=www.google.com
issuer=/C=US/O=Google Inc/CN=Google Internet Authority G2
---
No client certificate CA names sent
Peer signing digest: SHA256
Server Temp Key: ECDH, P-256, 256 bits
---
SSL handshake has read 3727 bytes and written 444 bytes
---
New, TLSv1/SSLv3, Cipher is ECDHE-RSA-AES128-GCM-SHA256
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: 1600B126722022838CE23D2D2B6F0E1C0BC9CDAA4D1D8DBE29EB187A8129DD72
    Session-ID-ctx:
    Master-Key: 9D8CB0C003640EBFE120006F3D27CDD888ED78AC64151A155F52F05BF12EF87D9443FB9A16EAB533A38F12D6C0CB52DA
    Key-Arg   : None
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 100800 (seconds)
    TLS session ticket:
    0000 - b7 6f 9f a9 72 4c a1 fa-d0 ec fa 68 e0 71 fc 68   .o..rL.....h.q.h
    0010 - 4d 55 16 f0 f2 58 0f e6-c4 55 92 30 a2 05 2a 34   MU...X...U.0..*4
    0020 - 4b a9 83 eb 1e 73 15 9e-e8 b4 ed 1d 8f 31 2e db   K....s.......1..
    0030 - 37 4d 86 5f 1f c8 2b 85-f7 ee aa a6 30 0a 61 47   7M._..+.....0.aG
    0040 - fe a7 d2 a6 49 b8 d4 6e-73 7c 30 53 1b b1 5a 95   ....I..ns|0S..Z.
    0050 - 90 c3 78 06 e3 7e 72 57-86 c5 a3 8b 1e fb 88 e6   ..x..~rW........
    0060 - ad e0 7d 93 1d 2a 1e cd-e1 a7 ef 1f cd b2 e9 f7   ..}..*..........
    0070 - 16 91 16 75 d6 d1 b9 4d-1a be 59 0a cd 87 ea 22   ...u...M..Y...."
    0080 - fe 9d e3 21 d2 1c 16 47-a3 5f f4 b0 25 11 ee e4   ...!...G._..%...
    0090 - 04 f3 3b ee b7 a0 3a 48-7e 9f 37 60 66 84 a5 05   ..;...:H~.7`f...
    00a0 - 5a 80 18 ec                                       Z...

    Start Time: 1457374693
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
```

---
## POSIX
---
## Debian
### Getting the last squeeze-lts updates
Squeeze-lts was end of lifed at the being of this month (03/15). The problem is that all of the packages have been migrated over to the debian-archive so without modification `apt-get update` is useless. Add the following line to your `/etc/apt/sources.list` then run `apt-get -o Acquire::Check-Valid-Until=false update`.
```
deb http://archive.debian.org/debian-archive/debian/ squeeze-lts main contrib non-free
```
### Modifing a `.deb` package
There are serveral ways of modifying a `.deb` while my method of using `ar` is a little fidly it has the benefit of perserving file permissions without messing about with fakeroot.
#### Extracting
 * Copy the package to your working directory
 * `mkdir -p <package_name>/cntrl`
 * `cd <package_name>`
 * `ar x ../<package_name>.deb`
 * `cd ctrl`
 * `tar xvzf ../control.tar.gz` 

#### Make your changes
 * `control` specifies dependencies and conflicts.
 * `prerm` and `postint` control restarting the daemon. 

#### Compacting
 * `tar czvf control.tar.gz *`
 * `mv control.tar.gz ..`
 * `cd ..`
 * `ar r <package_name>~<description_of_your_change>.deb debian-binary control.tar.gz data.tar.gz`
 
 **Note the ar options must be in the above order.**

### Making a device driver
If you ever need to create a Linux device driver, in a chroot for example, use `mknod`. In our example here we're creating a second null device.
```
mknod /tmp/null/ 1 3
```
The magic numbers `1` and `3` are major and minor device numbers respectively. These device numbers are defined ![here](https://www.kernel.org/doc/Documentation/devices.txt).

---
## Vim
### Man pages in Vim
Vim since version 6.0 has had support for displaying syntax highlighted withing vim, but becuase new things are bad and therefore disabled it's not widely used/known about. To enable man pages add `runtime! ftplugin/man.vim` to your vimrc. You can then invoke man pages like so.
```
Man 3 printf
```
The biggest advantage I see to using man pages with in vim is that tags are automatically supported, so ^] away. 

---
## 80's Nostalgia 
![80's Nostalgia](https://raw.githubusercontent.com/jparris/til/master/imgs/the_more_you_know.jpg)
