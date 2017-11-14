# systemd_tutorial
understanding-systemd-units-and-unit-files

https://www.freedesktop.org/software/systemd/man/systemd.service.html
https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files
--> 

https://www.digitalocean.com/community/tutorials/ --> Tutorials on other tech topics

// Core Pattern:
1. Introduction to Core Dumps
In most GNU/Linux systems (all of those I personally have used, at least), core dump files generated after an uncaught signal in a process (as a SIGSEGV or SIGQUIT), are generated in the base directory where the program was executed, and named as “core” or “core.PID”.
For example:

$> cd /home/user
$> ulimit -c unlimited
$> kill -s SIGSEGV $$
This will trigger a segmentation fault in your current shell (you probably guessed it after seeing that the shell session where you executed it was closed), and generate a core file in:
/home/user/core
Now… is it possible to change where that file is generated by default instead of the current directory? And is it possible to change the name of that generated file? The answer is YES! to both. Let’s see how we can get this.
2. The Core Pattern in Kernel
Since some years ago, the kernel configuration includes a file named “core_pattern”:
/proc/sys/kernel/core_pattern
In my system, that file contains just this single word:
core
As expected, this pattern shows how the core file will be generated. Two things can be understood from the previous line: The filename of the core dump file generated will be “core”; and second, the current directory will be used to store it (as the path specified is completely relative to the current directory).
Now, if we change the contents of that file… (as root, of course)

$> mkdir -p /tmp/cores
$> chmod a+rwx /tmp/cores
$> echo "/tmp/cores/core.%e.%p.%h.%t" > /proc/sys/kernel/core_pattern
And we run the same as before:

$> cd /home/user
$> ulimit -c unlimited
$> $> kill -s SIGSEGV $$
We get… voilá!
/tmp/cores/core.bash.8539.drehbahn-mbp.1236975953
Not only the program name (“bash“) or the PID (“8539“), but also the hostname (“drehbahn-mbp“) and the unix time (“1236975953“) are appended in the name of the core file!! And of course, it is stored in the absolute path we specified (“/tmp/cores/“).
You can use the following pattern elements in the core_pattern file:

%p: pid
%: '%' is dropped
%%: output one '%'
%u: uid
%g: gid
%s: signal number
%t: UNIX time of dump
%h: hostname
%e: executable filename
%: both are dropped
Isn’t is great?! Imagine that you have a cluster of machines and you want to use a NFS directory to store all core files from all the nodes. You will be able to detect which node generated the core file (with the hostname), which program generated it (with the program name), and also when did it happen (with the unix time).
3. Configure it forever
The changes done before are only applicable until the next reboot. In order to make the change in all future reboots, you will need to add the following in “/etc/sysctl.conf“:

# Own core file pattern...
kernel.core_pattern=/tmp/cores/core.%e.%p.%h.%t
sysctl.conf is the file controlling every configuration under /proc/sys

From <https://sigquit.wordpress.com/2009/03/13/the-core-pattern/> 

