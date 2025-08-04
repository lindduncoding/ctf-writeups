## Linux and Windows OS CTF Challenges

Part of SMTP 2025

Written by fredora

## Out of Sight

Description:

```
Welcome to our very first OS challenge.
Somewhere on this system lies a flag... but not everything is visible at first glance.

Can you find it? I know you can, let's find it!
Connect here: nc 52.77.77.117 9040
```

Connecting to the remote address and listing the files return a readme.txt that gives a hint about ls capabilities:

```
fred@fedora:~$ nc 52.77.77.117 9040
-sh: 0: can't access tty; job control turned off
$ ls
ls
readme.txt
$ cat readme.txt
cat readme.txt
Nothing interesting here! ls has many options. One of them is worth to try, buddy :)
$ 
```

ls does have many options, one of them listing hidden files. I've used this before many times for my own shenanigans but anyways, doing that will return a list of hidden files:

```
ls -a
.  ..  .bash_logout  .bashrc  .flag.txt  .profile  readme.txt
```

Well there you go, the flag is marked as a hidden file. However, you can still print the content of a hidden file using the cat command as always.

```
$ cat .flag.txt
cat .flag.txt
SMT2025{hidden_permission_flag}
```

## Random Cities

```
Your objective exists among executables.

Connect here: nc 52.77.77.117 8087
```

Upon connecting, listing the files of current directory return an executable and some text files.

```
=====================================
         Welcome to Cities!
=====================================

Look for the flags through the executables!
'Be careful who you trust Sergeant' -Ghost

ctf@1fee2ffd73bd:~$ ls
ls
README.md  cities  cities.txt  notes.txt
ctf@1fee2ffd73bd:~$ 
```

I thought the solution was to run the executable or modifying its permission bit to get the flag I wanted. However, doing both will result in nothing since the binary can't be executed due to some errors, and the regular ctf user doesn't have the rights to grant file permission. 

Unlocking the hint makes the solution obvious, however.

```
This challenge favor fundamental analysis:
1. Identify the file type
2. Examine meaningful strings, not just noise
3. Search smartly, not blindly
```

Got the flag by using the strings binary to extract the hidden flag inside the binary.

```
ctf@1fee2ffd73bd:~$ strings cities | grep SMT
strings cities | grep SMT
SMT2025{congratzzzzzz}
ctf@1fee2ffd73bd:~$ 
```

## Kecleon

```
It looks like a regular spreadsheet, but can you find what’s hidden beneath the surface?

Remember that Office files aren’t always what they seem. Inspect beyond the cells. Try to peek inside.
```

Given a suspicious .xlsx file, I didn't know what to do. So I googled something about the .xlsx file format and turns out it's just an archive file containing multiple XML files. Try changing the extension of an .xlsx file to .zip will still make the file valid and you'll be able to extract information from it.

![xlsx](/smt-2025/images/hidden.png)

Inside the directory are various XML files, and inspecting one of them will reveal the flag.

```
.
├── Hidden
│   ├── [Content_Types].xml
│   ├── docProps
│   │   ├── app.xml
│   │   └── core.xml
│   ├── _rels
│   └── xl
│       ├── printerSettings
│       │   └── printerSettings1.bin
│       ├── _rels
│       │   └── workbook.xml.rels
│       ├── sharedStrings.xml
│       ├── styles.xml
│       ├── theme
│       │   └── theme1.xml
│       ├── workbook.xml
│       └── worksheets
│           ├── _rels
│           │   └── sheet1.xml.rels
│           └── sheet1.xml
```

Content of sharedStrings.xml:

```
<si>
<t>One-hour drive = 10 km + 50 patience points.</t>
</si>
<si>
<t>Online Shopping</t>
</si>
<si>
<t>"Cuma lihat-lihat" ends in checkout.</t>
</si>
<si>
<t>Public Holidays</t>
</si>
<si>
<t>1 real holiday + 3 "hari kejepit nasional."</t>
</si>
</sst>
<!--  SMT2025{could_be_anywhere_so_beware}  -->
```

Alternatively, you can use grep with the -r option to find the SMT pattern in the current directory (.) or other directories. The sharedStrings.xml is a structure to contain all string data of the Open Office format, so it's often used to hide data. 

## Scheduler Shenanigans

Description:

```
You've entered a compromised Linux box.
Something’s ticking every minute — and it’s not a clock. The adversary might drop something inside, something time-sensitive, and you need to inspect it.
Ironically, its familiarity is what makes it stealthy. 

Find the flag before it resets!!!
Connect here: nc 52.77.77.117 9000
```

Trying to inspect the shenanigans return something like this:

```
fred@fedora:~$ nc 52.77.77.117 9000
====================================
 Welcome to Scheduler Shenanigans!
====================================

Someone's been here before you...
A scheduler is running in the shadows.

Can you catch the beacon before it's gone?

Good luck, hacker!

14
ctfuser@9a07c6f2347b:/$ ls
ls
banner.txt  dev            etc       lib    libx32  opt   run   sys  var
bin         dropper.sh     flag.txt  lib32  media   proc  sbin  tmp
boot        entrypoint.sh  home      lib64  mnt     root  srv   usr
ctfuser@9a07c6f2347b:/$ cat flag.txt
cat flag.txt
cat: flag.txt: Permission denied
ctfuser@9a07c6f2347b:/$ cat dropper.sh
cat dropper.sh
#!/bin/bash
cp /flag.txt /tmp/flag
chmod 644 /tmp/flag
chown ctfuser:ctfuser /tmp/flag
sleep 10
rm /tmp/flag
ctfuser@9a07c6f2347b:/$
```

Seems like the dropper.sh is a script used to automate copying the flag.txt file to the tmp folder where the ctf user has access to read and write the file. However, it only stays there for only 10 seconds. Turns out the dropper command is scheduled to run every minute using crontab.

```
ctfuser@9a07c6f2347b:/$ cat /etc/crontab
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
# You can also override PATH, but by default, newer versions inherit it from the environment
#PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* * * * * root curl -s http://localhost:8000/dropper.sh | bash
```

Essentially what happened is the flag is only available for 10 seconds every minute. I think I got lucky first try because I get to cat the flag once and then when I try to recreate the solution to write this WU it doesn't work... However, there's an automation command to get the flag as soon as it's available:

```
ctfuser@9a07c6f2347b:/$ while true; do cat /tmp/flag 2>/dev/null && break; sleep 1; done
<o cat /tmp/flag 2>/dev/null && break; sleep 1; done
SMT2025{automated_persistence_daemon}
ctfuser@9a07c6f2347b:/$ 
```

```
while true {
	do cat /tmp/flag 2>/dev/null # redirect error to dev/null
	break # when the above command is successful, break out of the loop
	sleep 1 # wait for one second before retrying
}
```