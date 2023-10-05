---
title: Linux 安装和使用 supervisor
key: 2023-10-05
tags: Linux supervisor
---

> 本文主要记录了怎样在 Linux 上安装 supervisor， 使用的是 CentOS 7。

Search suggest: linux supervisor

<!--more-->
## 0x01 介绍

supervisor 是用 Python 开发的一套通用的进程管理程序，能将一个普通的命令行进程变为后台 daemon，并监控进程状态，异常退出时能自动重启。


## 0x02 安装

```bash
[root@matrix002 ~]# dnf search supervisor
Last metadata expiration check: 8:28:17 ago on Thu 05 Oct 2023 01:02:52 PM CST.
=================================== Name Exactly Matched: supervisor ====================================
supervisor.noarch : A System for Allowing the Control of Process State on UNIX
================================== Name & Summary Matched: supervisor ===================================
nodejs-supervisor.noarch : A supervisor program for running nodejs programs
====================================== Summary Matched: supervisor ======================================
python-simplevisor.noarch : Python simple daemons supervisor
[root@matrix002 ~]# dnf install supervisor
Last metadata expiration check: 8:28:43 ago on Thu 05 Oct 2023 01:02:52 PM CST.
Dependencies resolved.
=========================================================================================================
 Package                                      Arch            Version                Repository     Size
=========================================================================================================
Installing:
 supervisor                                   noarch          3.4.0-1.el7            epel          498 k
Installing dependencies:
 python-backports                             x86_64          1.0-8.el7              base          5.8 k
 python-backports-ssl_match_hostname          noarch          3.5.0.1-1.el7          base           13 k
 python-ipaddress                             noarch          1.0.16-2.el7           base           34 k
 python-setuptools                            noarch          0.9.8-7.el7            base          397 k
 python-meld3                                 x86_64          0.6.10-1.el7           epel           73 k

Transaction Summary
=========================================================================================================
Install  6 Packages

Total download size: 1.0 M
Installed size: 5.1 M
Is this ok [y/N]: y
Downloading Packages:
(1/6): python-backports-1.0-8.el7.x86_64.rpm                              50 kB/s | 5.8 kB     00:00    
(2/6): python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch.rpm      109 kB/s |  13 kB     00:00    
(3/6): python-ipaddress-1.0.16-2.el7.noarch.rpm                          178 kB/s |  34 kB     00:00    
(4/6): python-setuptools-0.9.8-7.el7.noarch.rpm                          1.1 MB/s | 397 kB     00:00    
(5/6): python-meld3-0.6.10-1.el7.x86_64.rpm                              139 kB/s |  73 kB     00:00    
(6/6): supervisor-3.4.0-1.el7.noarch.rpm                                 792 kB/s | 498 kB     00:00    
---------------------------------------------------------------------------------------------------------
Total                                                                    396 kB/s | 1.0 MB     00:02     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                 1/1 
  Installing       : python-meld3-0.6.10-1.el7.x86_64                                                1/6 
  Installing       : python-ipaddress-1.0.16-2.el7.noarch                                            2/6 
  Installing       : python-backports-1.0-8.el7.x86_64                                               3/6 
  Installing       : python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch                        4/6 
  Installing       : python-setuptools-0.9.8-7.el7.noarch                                            5/6 
  Installing       : supervisor-3.4.0-1.el7.noarch                                                   6/6 
  Running scriptlet: supervisor-3.4.0-1.el7.noarch                                                   6/6 
  Verifying        : python-backports-1.0-8.el7.x86_64                                               1/6 
  Verifying        : python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch                        2/6 
  Verifying        : python-ipaddress-1.0.16-2.el7.noarch                                            3/6 
  Verifying        : python-setuptools-0.9.8-7.el7.noarch                                            4/6 
  Verifying        : python-meld3-0.6.10-1.el7.x86_64                                                5/6 
  Verifying        : supervisor-3.4.0-1.el7.noarch                                                   6/6 

Installed:
  supervisor-3.4.0-1.el7.noarch                                 python-backports-1.0-8.el7.x86_64        
  python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch      python-ipaddress-1.0.16-2.el7.noarch     
  python-setuptools-0.9.8-7.el7.noarch                          python-meld3-0.6.10-1.el7.x86_64         

Complete!
```

## 0x03 配置



启动 

```bash
sudo systemctl status supervisord
sudo systemctl enable supervisord
cd /etc
sudo chown -R chengchao /etc/supervisord.d
cd  /etc/supervisord.d
touch demo-promethues.ini
vim demo-promethues.ini

[program:demo]
command=/usr/local/jvm/jdk-17/bin/java -jar /home/chengchao/demo-prometheus-0.0.1-SNAPSHOT.jar
directory=/home/chengchao
user=chengchao
stopsignal=Kill
autostart=true
autorestart=true
startsecs=3
stderr_logfile=/home/chengchao/demo-prometheus.err.log
stdout_logfile=/home/chengchao/demo-prometheus.out.log
```

## 0x04 命令

- 查看进程状态： `sudo supervisorctl status`
- 启动服务： `sudo supervisorctl start Test`
- 停止服务： `sudo supervisorctl stop Test`
- 重启服务： `sudo supervisorctl restart Test`
- 配置文件修改后重新加载： `sudo supervisorctl update`
- 重启配置中的所有进程： `sudo supervisorctl reload`

## 0x05 其他

- 查看 java 的位置： `which java`

普通用户可运行, 需要修改 sock 文件的路径和 owner

```bash
sudo systemctl stop supervisord
sudo vim /etc/supervisord.conf

[unix_http_server]
file=/works/run/supervisor/supervisor.sock
chmod=0700
chown=chengchao:supervisor

[supervisorctl]
serverurl=unix:///works/run/supervisor/supervisor.sock

sudo systemctl start supervisord
supervisorctl status
demo                             RUNNING   pid 2735, uptime 0:04:41

```

## Appendix

service 文件

```bash
[root@matrix002 system]# cat supervisord.service 
[Unit]
Description=Process Monitoring and Control Daemon
After=rc-local.service nss-user-lookup.target

[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf

[Install]
WantedBy=multi-user.target
```

配置文件：

```bash
[root@matrix002 etc]# cat supervisord.conf 
; Sample supervisor config file.

[unix_http_server]
file=/var/run/supervisor/supervisor.sock   ; (the path to the socket file)
;chmod=0700                 ; sockef file mode (default 0700)
;chown=nobody:nogroup       ; socket file uid:gid owner
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

;[inet_http_server]         ; inet (TCP) server disabled by default
;port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

[supervisord]
logfile=/var/log/supervisor/supervisord.log  ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=50MB       ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10          ; (num of main logfile rotation backups;default 10)
loglevel=info               ; (log level;default info; others: debug,warn,trace)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
nodaemon=false              ; (start in foreground if true;default false)
minfds=1024                 ; (min. avail startup file descriptors;default 1024)
minprocs=200                ; (min. avail process descriptors;default 200)
;umask=022                  ; (process file creation umask;default 022)
;user=chrism                 ; (default is current user, required if root)
;identifier=supervisor       ; (supervisord identifier, default is 'supervisor')
;directory=/tmp              ; (default is not to cd  during start)
;nocleanup=true              ; (don't clean up tempfiles at start;default false)
;childlogdir=/tmp            ; ('AUTO' child log dir, default $TEMP)
;environment=KEY=value       ; (key value pairs to add to environment)
;strip_ansi=false            ; (strip ansi escape codes in logs; def. false)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///var/run/supervisor/supervisor.sock ; use a unix:// URL  for a unix socket
;serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket
;username=chris              ; should be same as http_username if set
;password=123                ; should be same as http_password if set
;prompt=mysupervisor         ; cmd line prompt (default "supervisor")
;history_file=~/.sc_history  ; use readline history if available

; The below sample program section shows all possible program subsection values,
; create one or more 'real' program: sections to be able to control them under
; supervisor.

;[program:theprogramname]
;command=/bin/cat              ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;directory=/tmp                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=999                  ; the relative start priority (default 999)
;autostart=true                ; start at supervisord start (default: true)
;autorestart=true              ; retstart at unexpected quit (default: true)
;startsecs=10                  ; number of secs prog must stay running (def. 1)
;startretries=3                ; max # of serial start failures (default 3)
;exitcodes=0,2                 ; 'expected' exit codes for process (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=true          ; redirect proc stderr to stdout (default false)
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A=1,B=2           ; process environment additions (def no adds)
;serverurl=AUTO                ; override serverurl computation (childutils)

; The below sample eventlistener section shows all possible
; eventlistener subsection values, create one or more 'real'
; eventlistener: sections to be able to handle event notifications
; sent by supervisor.

;[eventlistener:theeventlistenername]
;command=/bin/eventlistener    ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;events=EVENT                  ; event notif. types to subscribe to (req'd)
;buffer_size=10                ; event buffer queue size (default 10)
;directory=/tmp                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=-1                   ; the relative start priority (default -1)
;autostart=true                ; start at supervisord start (default: true)
;autorestart=unexpected        ; restart at unexpected quit (default: unexpected)
;startsecs=10                  ; number of secs prog must stay running (def. 1)
;startretries=3                ; max # of serial start failures (default 3)
;exitcodes=0,2                 ; 'expected' exit codes for process (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=true          ; redirect proc stderr to stdout (default false)
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups        ; # of stderr logfile backups (default 10)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A=1,B=2           ; process environment additions
;serverurl=AUTO                ; override serverurl computation (childutils)

; The below sample group section shows all possible group values,
; create one or more 'real' group: sections to create "heterogeneous"
; process groups.

;[group:thegroupname]
;programs=progname1,progname2  ; each refers to 'x' in [program:x] definitions
;priority=999                  ; the relative start priority (default 999)

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

[include]
files = supervisord.d/*.ini
```


以下是参考链接。

- [Linux上用supervisor运行java的方法](https://blog.csdn.net/grfstc/article/details/127425170)
- 
__EOF__

