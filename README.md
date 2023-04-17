# Сбор и анализ логов.
## Развертывание стенда для демострации работы пары серверов Web/Log

Стэнд состоит из хостовой машины под управлением ОС Ubuntu 20.04 на которой развернут Ansible и двух виртуальных машин CentOS/7 2004.01 (для проигрывания на них сценариев).

Название серверов в `Vagrantfile`:

`logW` - сервер Web с NGINX

`logS` - сервер удаленного хранения лога и аудита.

`Playbook` (nginx.yml) вызывается в процессе развертывания машин `Vagrant`.

Разворачиваем инфраструктуру в `Vagrant` исключительно через `Ansible`.

Все коментарии по каждому блоку указаны в тексте  `Playbook`.

## Результат работы 
Логи NGINX с Web сервера `logw` на сервере хранения логов `logS`:

```
[root@logS logw]# ll
total 12
-rw-------. 1 root root 6490 Apr 17 09:09 nginx_access.log
-rw-------. 1 root root  307 Apr 17 09:05 nginx_error.log
[root@logS logw]# cat nginx_access.log 
Apr 17 09:05:12 logw nginx_access: 192.168.1.12 - - [17/Apr/2023:09:05:12 +0000] "GET / HTTP/1.1" 200 4833 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 YaBrowser/22.11.3.832 (beta) Yowser/2.5 Safari/537.36"
Apr 17 09:05:12 logw nginx_access: 192.168.1.12 - - [17/Apr/2023:09:05:12 +0000] "GET /img/centos-logo.png HTTP/1.1" 200 3030 "http://192.168.1.73:8080/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 YaBrowser/22.11.3.832 (beta) Yowser/2.5 Safari/537.36"
Apr 17 09:05:12 logw nginx_access: 192.168.1.12 - - [17/Apr/2023:09:05:12 +0000] "GET /img/html-background.png HTTP/1.1" 200 1801 "http://192.168.1.73:8080/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 YaBrowser/22.11.3.832 (beta) Yowser/2.5 Safari/537.36"
Apr 17 09:05:12 logw nginx_access: 192.168.1.12 - - [17/Apr/2023:09:05:12 +0000] "GET /img/header-background.png HTTP/1.1" 200 82896 "http://192.168.1.73:8080/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 YaBrowser/22.11.3.832 (beta) Yowser/2.5 Safari/537.36"
Apr 17 09:05:12 logw nginx_access: 192.168.1.12 - - [17/Apr/2023:09:05:12 +0000] "GET /favicon.ico HTTP/1.1" 404 3650 "http://192.168.1.73:8080/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 YaBrowser/22.11.3.832 (beta) Yowser/2.5 Safari/537.36"
Apr 17 09:09:11 logw nginx_access: 192.168.1.12 - - [17/Apr/2023:09:09:11 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 YaBrowser/22.11.3.832 (beta) Yowser/2.5 Safari/537.36"
```
Логи ошибок NGINX с Web сервера `logw` на сервере хранения логов `logS`:

```
[root@logS logw]# cat nginx_error.log 
Apr 17 09:05:12 logw nginx_error: 2023/04/17 09:05:12 [error] 4568#4568: *2 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 192.168.1.12, server: default_server, request: "GET /favicon.ico HTTP/1.1", host: "192.168.1.73:8080", referrer: "http://192.168.1.73:8080/
```
Проверяем логи аудита. На Web сервере меняем права на файл `/etc/nginx/nginx.conf`:

```
[root@logW vagrant]# ls -l /etc/nginx/nginx.conf
-rwxrwxrwx. 1 root root 828 Apr 17 09:25 /etc/nginx/nginx.conf
[root@logW vagrant]# chmod 755 /etc/nginx/nginx.conf
[root@logW vagrant]# ls -l /etc/nginx/nginx.conf
-rwxr-xr-x. 1 root root 828 Apr 17 09:25 /etc/nginx/nginx.conf
```
Идем на сервер хранения логов проверяем переданные логи аудита:

```
[root@logS logw]grep logW /var/log/audit/audit.log
node=logW type=DAEMON_START msg=audit(1681723027.536:6932): op=start ver=2.8.5 format=raw kernel=3.10.0-1127.el7.x86_64 auid=4294967295 pid=4740 uid=0 ses=4294967295 subj=system_u:system_r:auditd_t:s0 res=success
node=logW type=CONFIG_CHANGE msg=audit(1681723027.709:1623): audit_backlog_limit=8192 old=8192 auid=4294967295 ses=4294967295 subj=system_u:system_r:unconfined_service_t:s0 res=1
node=logW type=CONFIG_CHANGE msg=audit(1681723027.711:1624): audit_failure=1 old=1 auid=4294967295 ses=4294967295 subj=system_u:system_r:unconfined_service_t:s0 res=1
node=logW type=CONFIG_CHANGE msg=audit(1681723027.711:1625): auid=4294967295 ses=4294967295 subj=system_u:system_r:unconfined_service_t:s0 op=add_rule key="nginx_conf" list=4 res=1
node=logW type=CONFIG_CHANGE msg=audit(1681723027.711:1626): auid=4294967295 ses=4294967295 subj=system_u:system_r:unconfined_service_t:s0 op=add_rule key="nginx_conf" list=4 res=1
node=logW type=SERVICE_START msg=audit(1681723027.711:1627): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=auditd comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
node=logW type=SYSCALL msg=audit(1681723043.074:1628): arch=c000003e syscall=268 success=yes exit=0 a0=ffffffffffffff9c a1=9f0420 a2=1ff a3=7fff71d275e0 items=1 ppid=4618 pid=4766 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=5 comm="chmod" exe="/usr/bin/chmod" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_conf"
node=logW type=CWD msg=audit(1681723043.074:1628):  cwd="/home/vagrant"
node=logW type=PATH msg=audit(1681723043.074:1628): item=0 name="/etc/nginx/nginx.conf" inode=67522517 dev=08:01 mode=0100755 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=NORMAL cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
node=logW type=PROCTITLE msg=audit(1681723043.074:1628): proctitle=63686D6F6400373737002F6574632F6E67696E782F6E67696E782E636F6E66
node=logW type=SERVICE_START msg=audit(1681723091.885:1629): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=systemd-tmpfiles-clean comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
node=logW type=SERVICE_STOP msg=audit(1681723091.885:1630): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=systemd-tmpfiles-clean comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'

...

node=logW type=CWD msg=audit(1681728292.776:2237):  cwd="/home/vagrant"
node=logW type=PATH msg=audit(1681728292.776:2237): item=0 name="/etc/nginx/nginx.conf" inode=80 dev=08:01 mode=0100777 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=NORMAL cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
node=logW type=PROCTITLE msg=audit(1681728292.776:2237): proctitle=63686D6F6400373535002F6574632F6E67696E782F6E67696E782E636F6E66
```
### P.S. 

Очень долго бился с считыванием ip (изначально получал его с DHCP) сервера логов для передачи их в качестве переменной в файлы конфигураций (jinja) на Web сервер. При проигрывании pkaybook из vagran ip не считывался, а при повторном запуске playbook c консоли хостовой машины, всё выходило удачно. ip считывал следующим образом: 

`{{ hostvars['logS']['ansible_facts']['eth1']['ipv4']['address'] }}`. 

Хотелось бы услышать эксперное мнение. Спасибо.