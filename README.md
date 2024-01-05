# RHEL Study Guide - Manage SELinux Security


[RHEL Study Guide - Table of Contents](https://github.com/pslucas0212/RHEL-Study-Guide)  

In the example we will see that we need to create a SELinx context for a file or folder.

Start by searching for an SEAlert in the /var/log/messages log file
```
# /var/log/messages
...
Jan  5 17:29:30 serverb systemd-logind[741]: New session 83 of user root.
Jan  5 17:29:30 serverb systemd[1]: Started Session 83 of User root.
Jan  5 17:29:30 serverb systemd[1]: session-83.scope: Deactivated successfully.
Jan  5 17:29:30 serverb systemd-logind[741]: Session 83 logged out. Waiting for processes to exit.
/var/log/messages
```

At the prompt type /sealert and copy the sealert -l <message-id>
```
/sealert
Jan  5 17:29:37 serverb setroubleshoot[30571]: SELinux is preventing /usr/sbin/httpd from getattr access on the file /lab-content/lab.html. For complete SELinux messages run: sealert -l 021384f0-66e2-4054-81d4-3b358d1f759e
Jan  5 17:29:37 serverb setroubleshoot[30571]: SELinux is preventing /usr/sbin/httpd from getattr access on
...

Type q to quit the less command and look up the sealert message
```
# sealert -l 021384f0-66e2-4054-81d4-3b358d1f759e
SELinux is preventing /usr/sbin/httpd from getattr access on the file /lab-content/lab.html.
...
# semanage fcontext -a -t FILE_TYPE '/lab-content/lab.html'...
Additional Information:
Source Context                system_u:system_r:httpd_t:s0
Target Context                unconfined_u:object_r:default_t:s0
Target Objects                /lab-content/lab.html [ file ]
Source                        httpd
Source Path                   /usr/sbin/httpd
Port                          <Unknown>
Host                          serverb.lab.example.com
Source RPM Packages           httpd-2.4.51-7.el9_0.x86_64
Target RPM Packages           
SELinux Policy RPM            selinux-policy-targeted-34.1.29-1.el9_0.noarch
Local Policy RPM              selinux-policy-targeted-34.1.29-1.el9_0.noarch
Selinux Enabled               True
Policy Type                   targeted
Enforcing Mode                Enforcing
Host Name                     serverb.lab.example.com
Platform                      Linux serverb.lab.example.com
                              5.14.0-70.13.1.el9_0.x86_64 #1 SMP PREEMPT Thu Apr
                              14 12:42:38 EDT 2022 x86_64 x86_64
Alert Count                   12
First Seen                    2024-01-05 17:00:45 EST
Last Seen                     2024-01-05 17:34:04 EST
Local ID                      021384f0-66e2-4054-81d4-3b358d1f759e

Raw Audit Messages
type=AVC msg=audit(1704494044.614:2342): avc:  denied  { getattr } for  pid=30341 comm="httpd" path="/lab-content/lab.html" dev="vda4" ino=9072802 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:default_t:s0 tclass=file permissive=0


type=SYSCALL msg=audit(1704494044.614:2342): arch=x86_64 syscall=newfstatat success=no exit=EACCES a0=ffffff9c a1=7f41380453e0 a2=7f41377fd830 a3=100 items=0 ppid=30339 pid=30341 auid=4294967295 uid=48 gid=48 euid=48 suid=48 fsuid=48 egid=48 sgid=48 fsgid=48 tty=(none) ses=4294967295 comm=httpd exe=/usr/sbin/httpd subj=system_u:system_r:httpd_t:s0 key=(null)

Hash: httpd,httpd_t,default_t,file,getattr
```

From the above output notice the issue - SELinux is preventing /usr/sbin/httpd from getattr access on the file /lab-content/lab.html. and suggest resolution  semanage fcontext -a -t FILE_TYPE '/lab-content/lab.html'...  Also notice the Source and Target Contexts and Target Object.  Finally look at the information in the Raw Audit Messages - type=AVC.  We will look at this next.

Use audit search look at the raw audit message with -m for message type and -ts for time stamp.  We will choose recent for the -ts option.

```
# ausearch -m AVC -ts recent
----
time->Fri Jan  5 17:34:04 2024
type=PROCTITLE msg=audit(1704494044.614:2341): proctitle=2F7573722F7362696E2F6874747064002D44464F524547524F554E44
type=SYSCALL msg=audit(1704494044.614:2341): arch=c000003e syscall=262 success=no exit=-13 a0=ffffff9c a1=7f4138045308 a2=7f41377fd830 a3=0 items=0 ppid=30339 pid=30341 auid=4294967295 uid=48 gid=48 euid=48 suid=48 fsuid=48 egid=48 sgid=48 fsgid=48 tty=(none) ses=4294967295 comm="httpd" exe="/usr/sbin/httpd" subj=system_u:system_r:httpd_t:s0 key=(null)
type=AVC msg=audit(1704494044.614:2341): avc:  denied  { getattr } for  pid=30341 comm="httpd" path="/lab-content/lab.html" dev="vda4" ino=9072802 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:default_t:s0 tclass=file permissive=0
----
time->Fri Jan  5 17:34:04 2024
type=PROCTITLE msg=audit(1704494044.614:2342): proctitle=2F7573722F7362696E2F6874747064002D44464F524547524F554E44
type=SYSCALL msg=audit(1704494044.614:2342): arch=c000003e syscall=262 success=no exit=-13 a0=ffffff9c a1=7f41380453e0 a2=7f41377fd830 a3=100 items=0 ppid=30339 pid=30341 auid=4294967295 uid=48 gid=48 euid=48 suid=48 fsuid=48 egid=48 sgid=48 fsgid=48 tty=(none) ses=4294967295 comm="httpd" exe="/usr/sbin/httpd" subj=system_u:system_r:httpd_t:s0 key=(null)
type=AVC msg=audit(1704494044.614:2342): avc:  denied  { getattr } for  pid=30341 comm="httpd" path="/lab-content/lab.html" dev="vda4" ino=9072802 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:default_t:s0 tclass=file permissive=0
```

Notice notice the denied getattr comm, path and tclass

Let's set a new SELinux rule based on what we've learned.  First we need to need to determin the proper file context.  We can do this by comparing a properly configured directory
```
# ls -dZ /lab-content/ /var/www/html/
      unconfined_u:object_r:default_t:s0 /lab-content/
system_u:object_r:httpd_sys_content_t:s0 /var/www/html/
```

Let's create our new rule.  We are adding a rule for files - fcontext with -a.  -t for the type and the location of the folder(s) and/or files
```
# semanage fcontext -a -t httpd_sys_content_t '/lab-content(/.*)?'
```

Finally run restore context recursively
```
# restorecon -R /lab-content/
```
