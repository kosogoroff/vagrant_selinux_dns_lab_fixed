# Исправленный стенд с устранённой проблемой (решение см. ниже)

#### SELinux: проблема с удаленным обновлением зоны DNS

Инженер настроил следующую схему:

- ns01 - DNS-сервер (192.168.50.10);
- client - клиентская рабочая станция (192.168.50.15).

При попытке удаленно (с рабочей станции) внести изменения в зону ddns.lab происходит следующее:
```bash
[vagrant@client ~]$ nsupdate -k /etc/named.zonetransfer.key
> server 192.168.50.10
> zone ddns.lab
> update add www.ddns.lab. 60 A 192.168.50.15
> send
update failed: SERVFAIL
>
```
Инженер перепроверил содержимое конфигурационных файлов и, убедившись, что с ними всё в порядке, предположил, что данная ошибка связана с SELinux.

В данной работе предлагается разобраться с возникшей ситуацией.


#### Задание

- Выяснить причину неработоспособности механизма обновления зоны.
- Предложить решение (или решения) для данной проблемы.
- Выбрать одно из решений для реализации, предварительно обосновав выбор.
- Реализовать выбранное решение и продемонстрировать его работоспособность.


#### Формат

- README с анализом причины неработоспособности, возможными способами решения и обоснованием выбора одного из них.
- Исправленный стенд или демонстрация работоспособной системы скриншотами и описанием.


# Решение

Для исправления проблемы файлы конфигурации динамических зон, в которые должен записывать сам процесс named, перенесены из директории /etc/named/dynamic (имеет контекст named_conf_t) в директорию /var/named/dynamic (имеет контекст named_cache_t), а файлы статических зон и конфигурации оставлены в директории /etc/named - в эти файлы процесс named записывать не должен.
Соответственно в репозитории изменён файл playbook.yml провизионинга Ansible (файлы динамических зон копируются в директорию /var/named/dynamic ,
а также изменены пути в файлам динамических зон в файле конфигурации named.conf).

При использовании нового репозитория ошибки при изменении динамических зон не возникает. После перезагрузки виртуальных машин сделанные изменения зон сохраняются:

```
admin_insta11@mv334 ~]$ git clone https://github.com/kosogoroff/vagrant_selinux_dns_lab_fixed.git
[admin_insta11@mv334 ~]$ cd vagrant_selinux_dns_problems_fixed
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ vagrant up
Bringing machine 'ns01' up with 'virtualbox' provider...
Bringing machine 'client' up with 'virtualbox' provider...
==> ns01: This machine used to live in /home/admin_insta11/vagrant_selinux_dns_problems_kvm_fixed but it's now at /home/admin_insta11/vagrant_selinux_dns_problems_fixed.
==> ns01: Depending on your current provider you may need to change the name of
==> ns01: the machine to run it as a different machine.
==> ns01: Clearing any previously set forwarded ports...
==> ns01: Clearing any previously set network interfaces...
==> ns01: Preparing network interfaces based on configuration...
    ns01: Adapter 1: nat
    ns01: Adapter 2: intnet
==> ns01: Forwarding ports...
    ns01: 22 (guest) => 2222 (host) (adapter 1)
==> ns01: Running 'pre-boot' VM customizations...
==> ns01: Booting VM...
==> ns01: Waiting for machine to boot. This may take a few minutes...
    ns01: SSH address: 127.0.0.1:2222
    ns01: SSH username: vagrant
    ns01: SSH auth method: private key
==> ns01: Machine booted and ready!
==> ns01: Checking for guest additions in VM...
    ns01: The guest additions on this VM do not match the installed version of
    ns01: VirtualBox! In most cases this is fine, but in rare cases it can
    ns01: prevent things such as shared folders from working properly. If you see
    ns01: shared folder errors, please make sure the guest additions within the
    ns01: virtual machine match the version of VirtualBox you have installed on
    ns01: your host and reload your VM.
    ns01: 
    ns01: Guest Additions Version: 7.2.16
    ns01: VirtualBox Version: 7.1
==> ns01: Setting hostname...
==> ns01: Configuring and enabling network interfaces...
==> ns01: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> ns01: flag to force provisioning. Provisioners marked to run always will still run.
==> client: This machine used to live in /home/admin_insta11/vagrant_selinux_dns_problems_kvm_fixed but it's now at /home/admin_insta11/vagrant_selinux_dns_problems_fixed.
==> client: Depending on your current provider you may need to change the name of
==> client: the machine to run it as a different machine.
==> client: Clearing any previously set forwarded ports...
==> client: Fixed port collision for 22 => 2222. Now on port 2200.
==> client: Clearing any previously set network interfaces...
==> client: Preparing network interfaces based on configuration...
    client: Adapter 1: nat
    client: Adapter 2: intnet
==> client: Forwarding ports...
    client: 22 (guest) => 2200 (host) (adapter 1)
==> client: Running 'pre-boot' VM customizations...
==> client: Booting VM...
==> client: Waiting for machine to boot. This may take a few minutes...
    client: SSH address: 127.0.0.1:2200
    client: SSH username: vagrant
    client: SSH auth method: private key
==> client: Machine booted and ready!
==> client: Checking for guest additions in VM...
    client: The guest additions on this VM do not match the installed version of
    client: VirtualBox! In most cases this is fine, but in rare cases it can
    client: prevent things such as shared folders from working properly. If you see
    client: shared folder errors, please make sure the guest additions within the
    client: virtual machine match the version of VirtualBox you have installed on
    client: your host and reload your VM.
    client: 
    client: Guest Additions Version: 7.2.16
    client: VirtualBox Version: 7.1
==> client: Setting hostname...
==> client: Configuring and enabling network interfaces...
==> client: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> client: flag to force provisioning. Provisioners marked to run always will still run.
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ 
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ vagrant status
Current machine states:

ns01                      running (virtualbox)
client                    running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ vagrant ssh client
###############################
### Welcome to the DNS lab! ###
###############################

- Use this client to test the enviroment
- with dig or nslookup. Ex:
    dig @192.168.50.10 ns01.dns.lab

- nsupdate is available in the ddns.lab zone. Ex:
    nsupdate -k /etc/named.zonetransfer.key
    server 192.168.50.10
    zone ddns.lab 
    update add www.ddns.lab. 60 A 192.168.50.15
    send

- rndc is also available to manage the servers
    rndc -c ~/rndc.conf reload

###############################
### Enjoy! ####################
###############################
Last login: Sat Sep 12 15:18:16 2026 from 10.0.2.2
[vagrant@client ~]$ nsupdate -k /etc/named.zonetransfer.key
> server 192.168.50.10
> zone ddns.lab
> update add www.ddns.lab. 60 A 192.168.50.15
> send
> quit
[vagrant@client ~]$ dig www.ddns.lab

; <<>> DiG 9.16.23-RH <<>> www.ddns.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51117
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 39a2d75257c03cc9010000006aa56dfe905ecaa814aa1e09 (good)
;; QUESTION SECTION:
;www.ddns.lab.			IN	A

;; ANSWER SECTION:
www.ddns.lab.		60	IN	A	192.168.50.15

;; Query time: 3 msec
;; SERVER: 192.168.50.10#53(192.168.50.10)
;; WHEN: Sat Sep 12 15:21:34 UTC 2026
;; MSG SIZE  rcvd: 85

[vagrant@client ~]$ exit
logout
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$
[admin_insta11@mv334 vagrant_selinux_dns_problems_kvm_fixed]$ vagrant halt
==> client: Attempting graceful shutdown of VM...
==> ns01: Attempting graceful shutdown of VM...
[admin_insta11@mv334 vagrant_selinux_dns_problems_kvm_fixed]$
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ vagrant up
Bringing machine 'ns01' up with 'virtualbox' provider...
Bringing machine 'client' up with 'virtualbox' provider...
<...>
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ 
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ vagrant ssh client
###############################
### Welcome to the DNS lab! ###
###############################

- Use this client to test the enviroment
- with dig or nslookup. Ex:
    dig @192.168.50.10 ns01.dns.lab

- nsupdate is available in the ddns.lab zone. Ex:
    nsupdate -k /etc/named.zonetransfer.key
    server 192.168.50.10
    zone ddns.lab 
    update add www.ddns.lab. 60 A 192.168.50.15
    send

- rndc is also available to manage the servers
    rndc -c ~/rndc.conf reload

###############################
### Enjoy! ####################
###############################
Last login: Sat Sep 12 15:18:16 2026 from 10.0.2.2
[vagrant@client ~]$
[vagrant@client ~]$ dig www.ddns.lab

; <<>> DiG 9.16.23-RH <<>> www.ddns.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51117
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 39a2d75257c03cc9010000006aa56dfe905ecaa814aa1e09 (good)
;; QUESTION SECTION:
;www.ddns.lab.			IN	A

;; ANSWER SECTION:
www.ddns.lab.		60	IN	A	192.168.50.15

;; Query time: 3 msec
;; SERVER: 192.168.50.10#53(192.168.50.10)
;; WHEN: Sat Sep 12 15:21:34 UTC 2026
;; MSG SIZE  rcvd: 85

[vagrant@client ~]$ exit
logout
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$
```
