# java.net.SocketException-Too-many-open-files
Every time a socket connection is opened, it is treated like a file, so it uses a file descriptor. The file descriptors are set as a resource by the OS. It seems like at the time of the error there was heavy user activity where sockets were being opened/closed at a high rate in a smaller window of time. So due to this, there are really three things we can do to help understand/resolve the issue. 

#Purpose
This document discusses how to narrow down and resolve issues involving file descriptors, often reported in Java exceptions with the terms "too many open files."

Problem Description

The following two stack traces indicate the same issue and report the same message: Too many open files

Exception 1
java.net.SocketException: Too many open files
at java.net.PlainSocketImpl.accept(Compiled Code)
at java.net.ServerSocket.implAccept(Compiled Code)
at java.net.ServerSocket.accept(Compiled Code)
at weblogic.t3.srvr.ListenThread.run(Compiled Code)

Exception 2
java.io.IOException: Too many open files
at java.lang.UNIXProcess.forkAndExec(Native Method)
at java.lang.UNIXProcess.(UNIXProcess.java:54)
at java.lang.UNIXProcess.forkAndExec(Native Method)
at java.lang.UNIXProcess.(UNIXProcess.java:54)
at java.lang.Runtime.execInternal(Native Method)
at java.lang.Runtime.exec(Runtime.java:551)
at java.lang.Runtime.exec(Runtime.java:477)
at java.lang.Runtime.exec(Runtime.java:443)

Command to check Ulimits:

Command: ulimit -a|grep "open files"
Output :open files    (-n) 131072

Check for "can't identify protocol" count using below command:

/usr/sbin/lsof |grep "can't identify"
java      30960       grd   60u     sock                0,5          1699180460 can't identify protocol
java      31255       grd   47u     sock                0,5          1728229882 can't identify protocol
java      31758       grd   47u     sock                0,5          1728231559 can't identify protocol
java      32194       grd   47u     sock                0,5          1728232225 can't identify protocol
java      32741       grd   47u     sock                0,5          1728235074 can't identify protocol

This count should be minimal. If its huge, there is some issue in the application code.

TROUBLESHOOTING STEPS:

#1. Understand the user activity during the time this occurred. Is this normal or expected behavior?
#2. If this is normal/expected behavior, then we can look at setting the ulimit values.
/etc/security/limits.conf

The Admin user can set their file descriptor limits in the /etc/security/limits.conf configuration file, as shown in following example.
soft nofile 1024
hard nofile 4096

A system-wide file descriptor limit can also be set by adding the following three lines to the /etc/rc.d/rc.local startup script:
# Increase system-wide file descriptor limit.
echo 4096 > /proc/sys/fs/file-max
echo 16384 > /proc/sys/fs/inode-max

I would start out with doubling whatever the current values are and go from there.

#3. You can also change the TIME_WAIT timeout value. To do this I recommend working with your system administrators, as this needs to be tuned based on the type of network transactions/activity.
Just keep in mind by default this is set to 60 seconds in AIX. This means when a user opens a socket then quickly closes the socket, that resource will not be freed up until it hits the 60 second threshold.

