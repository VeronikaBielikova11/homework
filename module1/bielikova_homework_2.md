
Task 1

```Last login: Mon Mar 30 18:28:52 on ttys000
(base) veronikabelikova@MacBook-Pro-Veronika ~ % cd /
(base) veronikabelikova@MacBook-Pro-Veronika / % ls
Applications	Volumes		etc		sbin
Library		bin		home		tmp
System		cores		opt		usr
Users		dev		private		var


(base) veronikabelikova@MacBook-Pro-Veronika / % cd / etc
/etc
(base) veronikabelikova@MacBook-Pro-Veronika /etc % ls
afpovertcp.cfg				ntp_opendirectory.conf
aliases					openldap
aliases.db				pam.d
apache2					passwd
asl					paths
asl.conf				paths.d
auto_home				pf.anchors
auto_master				pf.conf
autofs.conf				pf.os
bashrc					postfix
bashrc_Apple_Terminal			ppp
com.apple.screensharing.agent.launchd	profile
csh.cshrc				protocols
csh.login				racoon
csh.logout				rc.common
cups					rc.netboot
find.codes				resolv.conf
ftpusers				rmtab
gettytab				rpc
group					rtadvd.conf
hosts					security
hosts.equiv				services
irbrc					shells
kern_loader.conf			snmp
krb5.keytab				ssh
localtime				ssl
locate.rc				sudo_lecture
mail.rc					sudoers
man.conf				sudoers.d
manpaths				syslog.conf
manpaths.d				ttys
master.passwd				uucp
networks				wfs
newsyslog.conf				xtab
newsyslog.d				zprofile
nfs.conf				zshrc
notify.conf				zshrc_Apple_Terminal
ntp.conf


(base) veronikabelikova@MacBook-Pro-Veronika /etc % cd ..
(base) veronikabelikova@MacBook-Pro-Veronika / % cd /home
(base) veronikabelikova@MacBook-Pro-Veronika /home % ls
(base) veronikabelikova@MacBook-Pro-Veronika /home % pwd
/home


Task 2

```Last login: Mon Mar 30 18:29:14 on ttys000
(base) veronikabelikova@MacBook-Pro-Veronika ~ % pwd
/Users/veronikabelikova
(base) veronikabelikova@MacBook-Pro-Veronika ~ % ls
2025		Documents	Library		Node.js		VirtualBox VMs
Applications	Downloads	Movies		Pictures
Desktop		IdeaProjects	Music		Public
(base) veronikabelikova@MacBook-Pro-Veronika ~ % mkdir practical2
(base) veronikabelikova@MacBook-Pro-Veronika ~ % ls
2025		Documents	Library		Node.js		VirtualBox VMs
Applications	Downloads	Movies		Pictures	practical2
Desktop		IdeaProjects	Music		Public
(base) veronikabelikova@MacBook-Pro-Veronika ~ % cd practical2
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % touch file.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % ls
file.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % cp file.txt file_copy.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % ls
file.txt	file_copy.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % mv file_copy.txt file_renamed.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % ls
file.txt		file_renamed.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % ln file.txt hardlink.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % ls
file.txt		file_renamed.txt	hardlink.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % ln -s file.txt symlink.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % ls
file.txt		hardlink.txt
file_renamed.txt	symlink.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % find /Users/veronikabelikova -name file.txt
/Users/veronikabelikova/practical2/file.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 %

Task 3

```Last login: Mon Mar 30 18:36:43 on ttys000
(base) veronikabelikova@MacBook-Pro-Veronika ~ % pwd
/Users/veronikabelikova
(base) veronikabelikova@MacBook-Pro-Veronika ~ % ls
2025		Documents	Library		Node.js		VirtualBox VMs
Applications	Downloads	Movies		Pictures	practical2
Desktop		IdeaProjects	Music		Public
(base) veronikabelikova@MacBook-Pro-Veronika ~ % cd practical2 
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % ls -l file.txt
-rw-r--r--  2 veronikabelikova  staff  0 Mar 30 18:38 file.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % chmod 444 file.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % ls -l file.txt
-r--r--r--  2 veronikabelikova  staff  0 Mar 30 18:38 file.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % chmod 644 file.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % ls -l file.txt
-rw-r--r--  2 veronikabelikova  staff  0 Mar 30 18:38 file.txt
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % umask
022
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % umask 022
(base) veronikabelikova@MacBook-Pro-Veronika practical2 % 

Task 4
```bash
student@ubuntu:~$ sudo useradd -m -s /bin/bash trainee
student@ubuntu:~$ sudo usermod -aG sudo trainee
student@ubuntu:~$ cat /etc/passwd | grep trainee
trainee:x:1001:1001::/home/trainee:/bin/bash
student@ubuntu:~$
