### Proc

ls / 


one of them is /proc

man 5 proc

The proc filesystem is a pseudo-filesystem which provides an interface to kernel data

too hard 

(pseudo-filesystem ) virtual file system (like `/sys`) -> `df -ah` -> zero size
it shows us what is currently inside the kernel brain (what processes are running right now/ some information about hardware and kernel)

everything is a file -> kernel shows us what is inside its head


What is a process ?


why learn about it

fun
/proc -> the magic behind the ps, top command
proc filesystem is mounted in your container, you will be able to access this information



/proc/1 -> systemd





/proc/memeinfo (free -h)
/proc/uptime
/proc/version


$$ -> pid of current shell
/proc/self -> pid of current process






Executable file of process
/proc/X/exec

Command and arguments that were passed
/proc/X/cmdline

Environment variables a process is using you can do the following
/proc/X/environ

On which directory the current process is operating. 
/proc/X/cwd

/proc/X/limits




signals

kill -l

Interrupt,

ping -> graceful shutdown


echo $COLUMNS
echo $LINES



resources to check:

https://fernandovillalba.substack.com/p/a-journey-into-the-linux-proc-filesystem
