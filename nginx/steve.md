Nginx settings
============

## 1.1 nginx: All servers (both frontend and api)
Change global settings in /etc/nginx/nginx.conf

1. Disable cookie
For API server 
```
server {
    fastcgi_hide_header Set-Cookie;
}
```
For proxy
```
proxy_hide_header       Set-Cookie;
proxy_ignore_headers    Set-Cookie;
# important! Remember the special inheritance rules for proxy_set_header:
# http://nginx.org/ru/docs/http/ngx_http_proxy_module.html#proxy_set_header
proxy_set_header        Cookie "";
```

2. Increase nofile : 
```
worker_rlimit_nofile 500000;
```
3. Increase worker_processes

```
worker_processes [number of processor cores];
```
To check how many CPUs we have
```
cat /proc/cpuinfo |grep processor
```
4. Increase worker_connections

```
worker_connections  10000; # 10K
```
5. Increase maximum upload files..
```
server {
    client_max_body_size 1000M;
}
```
6. Deny access to hidden files 
```
location ~ /\. {
    access_log off;
    log_not_found off; 
    deny all;
}
```

## 1.2 nginx: Frontend

- Cache static files. We don't cache .html . Only .js, .css, .font...
- Chọn level gzip cho phù hợp (from 1-9), choose 6
- Turn off logging for static files access. And turn on **caching for static files**
```

location ~* \.(jpg|jpeg|gif|png|css|js|ico|xml)$ {
    access_log        off;
    log_not_found     off;
    expires           360d;
}
```

## 1.3 nginx: API 

* Increase fastcgi timeout for PHP
Trong web servers cho PHP set nginx header . This is the time the socket is allowed to server. If we leave it too low, then some PHP scripts might not return to nginx on time and nginx will return error: "Socket timeout" or something like that.
```
fastcgi_read_timeout 60; //seconds
```

## 1.4 Run nginx as a different user 'steve'
Sometimes we wanna scp files between different servers, thus we wanna run nginx as some logged in user so we don't have the problem of permissions and such...


```
# vim /etc/php/7.0/fpm/pool.d/www.conf
change all www-data to steve
# vim /etc/nginx/nginx.conf
change www-data to steve
```

## 1.5 Nginx custom log formats

Sometimes we wanna check response time, which is $upstream_response_time, in the log
https://www.tecmint.com/configure-custom-access-and-error-log-formats-in-nginx/

Basically define the 
```
log_format my_format '$upstream_response_time $remote_addr'
```
in nginx.conf

Then use it in specific site
```
#file: /etc/nginx/sites-available/api
server {
   access_log /path/to/log my_format;
}
```
Same for error logs

Soft link on api: /api-error.log /api-access.log /op.php /lts /overview 


PHP-FPM settings
=================

## 2.1 different pool conf for different websites
normally we can set the conf in global config in /etc/php/7.0/fpm/pool.d/www.conf
But we can always create multiple files like vlms.conf .... so that settings only affect vlms website.

## 2.2 php-fpm: emergency restart threshold
Set up emergency_restart_threshold, emergency_restart_interval and process_control_timeout. Default values for these options are totally off, but I think it’s better use these options example like following:
```
emergency_restart_threshold 10
emergency_restart_interval 1m
process_control_timeout 10s
```

What this mean? So if 10 PHP-FPM child processes exit with SIGSEGV or SIGBUS within 1 minute then PHP-FPM restart automatically. This configuration also sets 10 seconds time limit for child processes to wait for a reaction on signals from master.

Tăng số lượng sockets open bằng cách 
- tăng file max (xem trong /proc/sys/fs/file-max)
- Tăng soft & hard limit cho cả 2 user: root & user chạy php-fpm trong /etc/security/limits.conf


## 2.3 update pm.max_children & workers
```
; Choose how the process manager will control the number of child processes.
; Possible Values:
;   static  - a fixed number (pm.max_children) of child processes;
;   dynamic - the number of child processes are set dynamically based on the
;             following directives:
;             pm.max_children      - the maximum number of children that can
;                                    be alive at the same time.
;             pm.start_servers     - the number of children created on startup.
;             pm.min_spare_servers - the minimum number of children in 'idle'
;                                    state (waiting to process). If the number
;                                    of 'idle' processes is less than this
;                                    number then some children will be created.
;             pm.max_spare_servers - the maximum number of children in 'idle'
;                                    state (waiting to process). If the number
;                                    of 'idle' processes is greater than this
;                                    number then some children will be killed.
; Note: This value is mandatory.
```
```
Note that child / children means the number of processes. php-fpm will start multiple processes 
from its pool to max_children number.

on startup, the number of children (or processes/servers) will be pm.start_servers
if the web is consuming almost number of current processes, we will create enough 
so pm.min_spare_servers will be available.
Then, if load gets lower, we should kill idle processes so we can have pm.max_spare_servers only.
```


static vs dynamic workers: https://haydenjames.io/php-fpm-tuning-using-pm-static-max-performance/

Calculate and Settings for pm.max_children
https://www.kinamo.be/en/support/faq/determining-the-correct-number-of-child-processes-for-php-fpm-on-nginx

To check how much mem each php request takes up (usually from 15M - 300M)
```
ps -ylC php-fpm --sort:rss
ps --no-headers -o "rss,cmd" -C php-fpm | awk '{ sum+=$1 } END { printf ("%d%s\n", sum/NR/1024,"M") }'
```
then calculate pm.max_children as follows 
```
pm = dynamic
pm.max_children = Total RAM dedicated to the web server / Max child process size 

pm.start_servers = 3 # todo : review this
pm.min_spare_servers = 2 #todo:review this
pm.max_requests = 200 #todo:review this

```
Max request per process is unlimited by default, but it’s good to set some low value, like 200 and avoid some memory issues. This style setup could handle large amount of requests, even if the numbers seems to be small.


## 2.4 other settings 
```
listen.backlog = -1
rlimit_files=100000
rlimit_core = unlimited
```

## 2.5 enable php opcode

https://stackoverflow.com/questions/36050023/optimize-nginx-php-fpm-for-5-million-daily-pageviews
http://php.net/manual/en/opcache.installation.php
https://www.scalingphpbook.com/blog/2014/02/14/best-zend-opcache-settings.html

## 2.6 Background Links
Some useful course on php-fpm: https://serversforhackers.com/s/managing-php-fpm

You can monitor php-fpm status via web: https://easyengine.io/tutorials/php/fpm-status-page/

https://www.if-not-true-then-false.com/2011/nginx-and-php-fpm-configuration-and-optimizing-tips-and-tricks/

## 2.7 network port reuse 
PHP có thể bị overload khi connect tới redis.

```
sudo vim /etc/sysctl.conf 
net.ipv4.tcp_tw_reuse = 1
sudo sysctl -p
```

netstat -n | grep TIME_WAIT | wc -l


Operating System
===============
## 4.1 Disable systemd-timesycd or ntp service
```
# systemctl stop systemd-timesyncd
# systemctl disable systemd-timesyncd
```

để nó khỏi ping linh tinh, nhất là các servers behind firewall

## 4.2 Increase Process file limit

Set max number of possible file descriptors. This will be used by both **mongo/redis server & nginx servers**

```
ps -u username  # look up processes owned by user
sudo grep 'open files' /proc/${id}/limits  # find "Max open files" line for process ID
```

Follow this: https://www.cyberciti.biz/faq/linux-increase-the-maximum-number-of-open-files/
basically
```
su - www-data
ulimit -Hn
ulimit -Sn
``` 
to check 

System wide settings
```
cat /proc/sys/fs/file-max
vi /etc/sysctl.conf
fs.file-max = 100000
```

Specific user settings
```
vi /etc/security/limits.conf

vim /etc/security/limits.conf 

//Remember that we have to separate by TAB not spaces
nginx        soft nofile 500000 
nginx        hard nofile 500000
*       soft nofile 500000
*       hard nofile 500000
root    soft nofile 500000
root    hard nofile 500000
nobody  soft nofile 65536
nobody  hard nofile 65536


Edit /etc/pam.d/login file and add/modify the following line (make sure you get pam_limts.so):

session required pam_limits.so

Tăng maximum connection cho MongoDB: https://stackoverflow.com/a/51105832/1784858

Cần tăng cho user root nữa vì root là user chạy master php-fpm process
```
Then login & out


## 4.3 sysctl params
```
net.core.somaxconn=1024
sysctl net.ipv4.ip_local_port_range="15000 61000"
sysctl net.ipv4.tcp_fin_timeout=30
sysctl net.ipv4.tcp_tw_recycle=1
sysctl net.ipv4.tcp_tw_reuse=1 

```

and then to reload sysctl 
```
sysctp -p 
```
or simply restart server

## 4.4 network interface queue length
```
ifconfig eth0 txqueuelen 5000
echo "/sbin/ifconfig eth0 txqueuelen 5000" >> /etc/rc.local
```
Remember to make sure /etc/rc.local is **executable** 
Then restart 

See https://stackoverflow.com/questions/410616/increasing-the-maximum-number-of-tcp-ip-connections-in-linux


## 4.5 Services
```
systemctl enable mongod.service
systemctl enable redis-server.service
systemctl enable redis.service
systemctl list-unit-files | grep enabled
```
Remember to make sure** sshd is enabled** as well

## 4.6. Other utilities & tools
network monitoring: https://www.binarytides.com/linux-commands-monitor-network/

system monitoring: [sysusage](http://sysusage.darold.net/)

'top' command
toggle 1 to show multiple cores
toggle 'e' to show human readable numbers


Redis & Mongo
==============
## Setup firewalls

Seting ufw and use this code to open port for web server access to DB server via mongo port and redis port

```
sudo ufw allow proto tcp from 10.10.3.227 (LAN IP for web server) to any port 6379 (redis port)
sudo ufw allow proto tcp from 10.10.3.227 (LAN IP for web server) to any port 27017 (mongo port)
```

Setting to open Mongo port for web serer, please refer: https://www.mkyong.com/mongodb/mongodb-allow-remote-access/

```
vim /etc/mongod.conf
bindIp = 127.0.0.1,10.10.3.227,10.10.3.226 (LAN IP of web server) 
```

Use `ufw status` to check server & port is opened

## High Performance Redis
Fix redis https://www.techandme.se/performance-tips-for-redis-cache-server/

```
redis-cli
config get maxclients
config set maxclients 10000
```

Config *maxclients* in /etc/redis/redis.conf

Check no files limit for user _redis_ 
https://superuser.com/questions/810951/how-do-i-check-the-ulimit-for-another-user-and-change-open-files


## Troubleshoot Redis
In case redis is eating too much RAM

https://stackoverflow.com/questions/4006324/how-to-atomically-delete-keys-matching-a-pattern-using-redis

```

[notice] Starting worker cdn-aws-taphuan:17455:*
PHP Fatal error:  Uncaught RedisException: MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk. Commands that may modify the data set are disabled, because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option). Please check the Redis logs for details about the RDB error. in /var/www/vlms/vendor/chrisboulton/php-resque/lib/Resque/Worker.php:479
Stack trace:
```
https://gist.github.com/kapkaev/4619127

Other Notes
============

## File permissions
* Make sure public/**ufiles** exists and **writable** in Web servers


## Resque proxy nginx config

Resque-web is using a different port (for example 8282) to view the status of resque job queues.
After this we could see the resque web in http://elearning.evn.com.vn:8282/overview
We need to by pass the following URLs so CSS/JS/HTML/Images assets of the resque-web load properly

```
server {# other configs etc....

      set $r "http://192.168.0.138:80";
      # by pass all the links/css links/js links in resque web
      location  ~* ^/(overview\.poll|overview|working|failed|queues|workers|stats|stats\/resque|reset\.css|style\.css|jquery(.*)|ranger\.js)$ {
           proxy_pass $r/$1$is_args$args;
           proxy_set_header HOST $host;
      }

      location ~* ^/queues/(.*) {
           proxy_pass $r/queues/$1$is_args$args;
           proxy_set_header HOST $host;
      }
}
```

## Proxy for scorm
To play SCORM 1.2 in an iframe, the scorm player (HTML5) is embedded on elearning.evn.com.vn, but its assets/resources currently stay in web API server. Therefore we need to proxy some URLs from web frontend to web api servers.

Use the same technique as the resque-web detailed above.

## list of cron jobs to setup
see cli/bash/ folder


Network Servers Architecture Improvements
============
* Consider redis db different from mongodb
* php-fpm different from web nginx servers
* Nginx proxy to connect to php through IP/Ports in stead of unix sockets. this way the web servers don't have to have nginx setup any more.