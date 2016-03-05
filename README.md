# java.net.SocketException-Too-many-open-files
Every time a socket connection is opened, it is treated like a file, so it uses a file descriptor. The file descriptors are set as a resource by the OS. It seems like at the time of the error there was heavy user activity where sockets were being opened/closed at a high rate in a smaller window of time. So due to this, there are really three things we can do to help understand/resolve the issue. 

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


