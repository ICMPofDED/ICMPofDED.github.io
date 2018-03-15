# [](#HTB Mirai)HTB Mirai

Hey, this is my write-up of **Mirai**. 
I'm fairly new to hacking, but I'm writing this blog mainly to get better and to more effectivly internalize the lessons I learn along the way. 

I have already hit a few other boxes, but I'm circling around back to Mirai because it is an older box, so this means it will probably be my first post on my new blog. 
So let's begin. 

I started out by using nmap on the IP (which was `10.10.10.48` at the time). However, I was trying something new out...the `--script=default` so let's see how it goes and if it works out for me. I'm writing this as I go.

Here's the result of the NMAP Default script: 
>└──╼ $nmap --script=default 10.10.10.48

> Starting Nmap 7.60 ( https://nmap.org ) at 2018-01-16 13:50 MST
> Nmap scan report for 10.10.10.48
> Host is up (0.47s latency).
> Not shown: 997 closed ports
> PORT   STATE SERVICE
> 22/tcp open  ssh
> | ssh-hostkey: 
> |   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)
> |   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
> |   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
> |_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (EdDSA)
> 53/tcp open  domain
> | dns-nsid: 
> |_  bind.version: dnsmasq-2.76
> 80/tcp open  http
> |_http-title: Site doesn't have a title (text/html; charset=UTF-8).

> Nmap done: 1 IP address (1 host up) scanned in 46.55 seconds

Ok, so quite a bit less information than normal, but we have a few things to check out right off the bat. Says there is a website with no title. DNS server and SSH. 
Well, I learned about another script I want to try as well before we continue. The Nmap Vulners script. However, looks like the host is down now. Perhaps someone is issuing a reset.
![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/img1.jpg?raw=true)



Ok, it didn't want to come back up so I was the one who had to issue a reset. 
Anyway...back to business. 
![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/biz1.jpg?raw=true)


> └──╼ $nmap -sV --script vulners 10.10.10.48

> Starting Nmap 7.60 ( https://nmap.org ) at 2018-01-16 14:05 MST
> Nmap scan report for 10.10.10.48
> Host is up (0.47s latency).
> Not shown: 997 closed ports
> PORT   STATE SERVICE VERSION
> 22/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
> | vulners: 
> |   cpe:/a:openbsd:openssh:6.7p1: 
> | 	CVE-2016-8858		7.8		[https://vulners.com/cve/CVE-2016-8858](https://vulners.com/cve/CVE-2016-8858).
> | 	CVE-2017-15906		5.0		[https://vulners.com/cve/CVE-2017-15906](https://vulners.com/cve/CVE-2017-15906).
> | 	CVE-2016-0778		4.6		[https://vulners.com/cve/CVE-2016-0778](https://vulners.com/cve/CVE-2016-0778).
> |_	CVE-2016-0777		4.0		[https://vulners.com/cve/CVE-2016-0777](https://vulners.com/cve/CVE-2016-0777).
> 53/tcp open  domain  dnsmasq 2.76
> 80/tcp open  http    lighttpd 1.4.35
> |_http-server-header: lighttpd/1.4.35
> | vulners: 
> |   cpe:/a:lighttpd:lighttpd:1.4.35: 
> |_	CVE-2015-3200		5.0		[https://vulners.com/cve/CVE-2015-3200](https://vulners.com/cve/CVE-2015-3200).
> Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> Nmap done: 1 IP address (1 host up) scanned in 43.83 seconds

Ok, didn't see anything of note in the CVEs, the one for lighttpd just allows you to write to the log file, but no RCE as far as I can tell. 

So I tried going to the IP in the browser, and it gave me page could not be displayed. So maybe it's in another directory? I added /admin and voila! I've got an admin portal for "pi-hole". 
Doing some digging on pi-hole led me to some vulnerabilities for the dnsmasq version they're using. So...Don't expect your tools to be 100% accurate all the time. 
![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/tmyk.jpg?raw=true)


Looks like a few of the CVEs are of the RCE variety (yay!)
![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/img2.jpg?raw=true)


All of these were found on the Google Security Blog [https://security.googleblog.com/2017/10/behind-masq-yet-more-dns-and-dhcp.html](here!)

Ok. I took a moment to think about this, and how I was going to attempt some of these POCs, however I ended up thinking more about Mirai and how it spread because of default credentials. Now that I know this is a raspberry pi, I found someone's post on reddit about how he wasn't sure how to SSH in. Someone mentioned how to reset it, and then mentioned the default login was login: pi password: raspberry 
Excellent, let's give that a shot. 
![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/img3.jpg?raw=true)


and we're in. Let's do some digging.

Alright! we got our user.txt on the desktop!

Ok, I'm going to pretend I didn't just copy a linux priv. escalation script over to the machine before trying "sudo su" without a password...

But they wouldn't make it that easy for us, would they? 
![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/img4.jpg?raw=true)


Apparently they did, but look we have another hoop to jump through already?
Ok, that should be simple enough...
Alright, so I did an "lsblk" to determine where the flash drive was mounted, and we found it was in the /media/usbstick directory.
![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/img5.jpg?raw=true)

-sigh- 

Alright, to be honest. I am not sure how to recover files. So I had to dig a bunch. I tried to install something through apt-get which was locked down. I'm thinking I'm going to have to recover it with grep if at all possible. 

After some digging, I was able to find this post: 

[http://blog.nullspace.io/recovering-deleted-files-using-only-grep.html](http://blog.nullspace.io/recovering-deleted-files-using-only-grep.html)

I did modify the command a bit to my liking. 

The command on the blog post was: 
grep --binary-files=text --context=x 'stringfromyourfile' \ /dev/whateverPartition > someFile.txt

I don't know a string from the file, because that's what I'm trying to find. So here were my modifications...

`grep --binary-files=text --context=10 '*' /dev/sdb > /root/root2.txt`

This was just a snippet of the END of the result: 

>�
>   o��!:2�Y:2�Y:2�Y
>�* �!9�����2�Y�2�Y�2�Y
>�+ �!9��;9�Y�3
>                   8PP
>(["�	  �1�Y��S��1�Y
>                         �<Byc[��B)�>r &�<�yZ�.Gu���m^��>
>                                                               �1�Y
>�|}*,.�����+-���3d3e483143ff12ec505d026fa13e020b
>Damnit! Sorry man I accidentally deleted your files off the USB stick.
>Do you know if there is any way to get them back?
>
>-James

So you can see, the file that was saved to the flash drive, and just before that in memory, you can see a key. I wasn't sure if that was the key we were looking for, but I tried it and it was!
It felt great to root this box, and to find this file. It always feels great to learn something new.

Again, to look over our lessons we learned...I'd say there is one major one...don't ALWAYS trust your tools. Maybe you don't have the right options, and maybe you're doing it wrong, maybe the tool doesn't understand the context of the situation. Each situation, each network, each system, is different. Trusting your tools 100% may result in your inability to break in. If I had focused all of my time and resources on SSH or lighttpd vulnerabilities, I think I wouldn't be where we are now (at the finish line), at least I wouldn't be here as quickly as I would have been. 

Hey, thanks for reading my write-up. It really means a lot. I'm really passionate about Infosec and am glad whenever I get some free time to work on it, to study it, and to learn more about it. I hope I helped someone/anyone get further on a problem they were stuck on or to just learn more about something they were curious about. Feel free to reach out, I'll have a contact page later on if you have any questions/comments/concerns about me, my methods, or my opinions. 

 Thanks again, and happy hacking.


Next day Edit. After all is said and done, a friend informed me that you can extract strings on the drive using the dd command. Specifically:

`dd if=/dev/sdb | strings`

Wow, certainly a much better result than my previous command. 
Here was a snippet of the result:
![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/img6.jpg?raw=true)


Look at that, plain as day. We are basically extracting all of the strings of text from the entire drive. I think the only thing that makes this command feasible though, is the fact that it was a USB drive and not a system partition. Very cool, even after rooting the box, Im still learning awesome Sh..tuff. Thanks again for reading!
