##Users and Groups 
- Users are identified by their User ID(UID)
- UID 0 is a special privilege user called root, typically the administrator
- Unprivileged user IDs typically start at 500 or 1000 
- A group is a set of users sharing resources.(file, device,programs)
- Each group has a Group ID(GID)
- Each user belongs to at least one group, and potentially supplementary groups
![unix_id.png](https://upload-images.jianshu.io/upload_images/14793864-c5d4eeedcb94434d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## etc/passwd etc/shadow
etc/passwd contains the list of users.
![image.png](https://upload-images.jianshu.io/upload_images/14793864-0c593ee8db5e3ef5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    1. Username: It is used when user logs in. It should be between 1 and 32 characters in length.
    2. Password: An x character indicates that encrypted password is stored in /etc/shadow file. Please note that you need to use the passwd command to computes the hash of a password typed at the CLI or to store/update the hash of the password in /etc/shadow file.
    3. User ID (UID): Each user must be assigned a user ID (UID). UID 0 (zero) is reserved for root and UIDs 1-99 are reserved for other predefined accounts. Further UID 100-999 are reserved by system for administrative and system accounts/groups.
    4. Group ID (GID): The primary group ID (stored in /etc/group file)
    5. User ID Info: The comment field. It allow you to add extra information about the users such as user’s full name, phone number etc. This field use by finger command.
    6.  Home directory: The absolute path to the directory the user will be in when they log in. If this directory does not exists then users directory becomes /
    7. Command/shell: The absolute path of a command or shell (/bin/bash). Typically, this is a shell. Please note that it does not have to be a shell.
- etc/shadow contains (salted) password hashed, can only be read by root. 
![image.png](https://upload-images.jianshu.io/upload_images/14793864-d163af127f72be95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
1. Username : It is your login name.
2. Password : It is your encrypted password. The password should be minimum 8-12 characters long including special characters, digits, lower case alphabetic and more. Usually password format is set to $id$salt$hashed, The $id is the algorithm used On GNU/Linux as follows:
        1 is MD5
        2a is Blowfish
        2y is Blowfish
        5 is SHA-256
        6 is SHA-512
 3. Last password change (lastchanged) : Days since Jan 1, 1970 that password was last changed
 4.   Minimum : The minimum number of days required between password changes i.e. the number of days left before the user is allowed to change his/her password
  5.  Maximum : The maximum number of days the password is valid (after that user is forced to change his/her password)
 6.   Warn : The number of days before password is to expire that user is warned that his/her password must be changed
 7.   Inactive : The number of days after password expires that account is disabled
 8. Expire : days since Jan 1, 1970 that account is disabled i.e. an absolute date specifying when the login may no longer be used.
- The password field which starts with a exclamation mark (!) means that the password is locked.
- The last 6 fields provides password aging and account lockout features. You need to use the **chage** command to setup password aging.


## File system 
- Information organized in a directory tree rooted at "/", where each directory contains filesystem objects.
- Filesystem objects can be ordinary files and directories, also symbolic links, named pipes and sockets. 
- Usual directories. 
  - /etc : configuration files .
  - /home: user files and application. 
  - /bin: executables that are part of OS. stored essential user binaries, important system program and untilities such as bash shell. 
  
  - /sbin : executables for superusers.  
  - /var : log files, temporary files. variable(writable) data files
  - /boot: static boot files.
  - /dev: device files. 

  - /lib: essential shared libraries.
  - /lost+found: recovered files. 
  - /mnt: temporary mount points.
  - /opt: optional packages. 
  - /proc: Kernel&Process files. 
  - /root: root home directory. 
  - /usr: user binaries & read-only data. 
##File and Directory access control
### FIle permissions
- 4 3-bit numbers. 
- First 3 bits for setuid, setgid and sticky bits. 
- then read-write-execute permissions for owner. 
- then read-write-execute permissions for group. 
- then read-write-execute permissions for other.
- Executable file runs its program with the privileges of the executer's uid by default. However, when setuid or setgit bits is set on a executable file, this program runs with privileges of the file owner instead of executor. 
- can be transformed to demical form, e.g. 111->7, 100->4, 101->5
- the command "chmod 4757 some_file" means that this file is executable and run with owner's privilege. File owner can read, write, and execute.(7->111). group owner can read, not write, execute.(5->101). Other users can read, not write, not execute(4=100)

 - 1. The read bit (r) allows the affected user to list the files within the directory
 - 2.   The write bit (w) allows the affected user to create, rename, or delete files within the directory, and modify the directory's attributes
 - 3.    The execute bit (x) allows the affected user to enter the directory, and access files and directories inside
 - 4.   The sticky bit (T, or t if the execute bit is set for others) states that files and directories within that directory may only be deleted or renamed by their owner (or root). 
### Stick bit. 
Think of a scenario where you create a Linux directory that can be used by all the users of the Linux system for creating,deleting,renaming files, which means that the write bit for other users is set for this directory, like /tmp directory. 
 ![image.png](https://upload-images.jianshu.io/upload_images/14793864-c301d3dc0fc8b265.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
if root user create a file in such a directory, any normal users can delete that.  
So sticky bit is set to prevent that, which only permit the owner of the file or the root user to delete or rename the file. 
The sticky bit is indicated by "t" or "T", which shows in /tmp access bit on the figure above. 
##Process attributes
### Real-effective-saved user ID and group ID. 
- Real ID is ID of user executing the process 
-  Effective ID determines current privileges, and ownership of files created by the process 
- By default effective ID is real ID 
- Sometimes you need to use another user’s identity, typically root, for privileged operations
- Setuid programs : effective ID is ID owning the file, as opposed to ID running it
- Saved ID used for temporarily dropping permissions
  - Store effective ID in saved ID 
  - Change effective ID to real ID 
  - Later change effective ID back to saved ID 
-  An unprivileged process can only change its effective ID to saved ID or real ID(no-root users can't do setuid(0))
- usmask is for permissions for new files which process creates.(createfile(umask)) 
- it tell which permission must be denied on the new file.
- Resulting file permission are (!umask)&mode. if umask=022 and mode=777 we get 755. 
- setuid program: when going from root to unprivileged, cannot go back to root while seteuid can. 
Safe usage of setuid
-  Always check setuid return code 
- Use seteuid to temporarily drop permissions 
- Can/should drop additional permissions with setsid and setgroups 
- Close all file descriptors you do not need anymore (permissions checked at open() but not checked at each read and write)

###umask

### Resource limits
![image.png](https://upload-images.jianshu.io/upload_images/14793864-3259aeec3a937bda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14793864-813e1e387450af52.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14793864-63863d4a58ff33ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14793864-f26dbc0da122164b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14793864-1cb7224f50ac0568.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14793864-ea7a7a563fdcca31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14793864-789cdf033bc1f35f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14793864-1e8a4492c539512c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14793864-38cab49ebe6ef7ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14793864-0899a7296337e420.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14793864-26f4de221ed6997f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14793864-b87d4321dfbce1eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14793864-dce00c1f108d98c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/14793864-ba239a24517ac451.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)











  

 

