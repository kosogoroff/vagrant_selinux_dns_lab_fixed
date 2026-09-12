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

При использовании нового репозитория ошибки при изменении динамических зон не возникает:

```
admin_insta11@mv334 ~]$ mv vagrant_selinux_dns_problems vagrant_selinux_dns_problems_fixed
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
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ 
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git remote add origin https://github.com/kosogoroff/vagrant_selinux_dns_lab_fixed.git
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git add .
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git status
Текущая ветка: main
Изменения, которые будут включены в коммит:
  (используйте «git restore --staged <файл>...», чтобы убрать из индекса)
	изменено:      Vagrantfile

[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git commit -m "ДЗ #15: клонированный стенд SELinux"
Author identity unknown

*** Пожалуйста, скажите мне кто вы есть.

Запустите

  git config --global user.email "you@example.com"
  git config --global user.name "Ваше Имя"

для указания идентификационных данных аккаунта по умолчанию.
Пропустите параметр --global для указания данных только для этого репозитория.

fatal: не удалось выполнить автоопределение адреса электронной почты (получено «admin_insta11@mv334.(none)»)
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git config --global user.name "Evgeny Kosogorov"
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git config --global user.email "kosogoroff@yandex.ru"
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git commit -m "ДЗ #15: клонированный стенд SELinux для исправления лабораторной ошибки"
[main 0476603] ДЗ #15: клонированный стенд SELinux для исправления лабораторной ошибки
 1 file changed, 28 insertions(+), 7 deletions(-)
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git branch -M main
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git push -u origin main
remote: Invalid username or token. Password authentication is not supported for Git operations.
fatal: Authentication failed for 'https://github.com/kosogoroff/vagrant_selinux_dns_lab_fixed.git/'
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git push -u origin main
remote: Invalid username or token. Password authentication is not supported for Git operations.
fatal: Authentication failed for 'https://github.com/kosogoroff/vagrant_selinux_dns_lab_fixed.git/'
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git push -u origin main
remote: Invalid username or token. Password authentication is not supported for Git operations.
fatal: Authentication failed for 'https://github.com/kosogoroff/vagrant_selinux_dns_lab_fixed.git/'
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git push -u origin main
remote: Invalid username or token. Password authentication is not supported for Git operations.
fatal: Authentication failed for 'https://github.com/kosogoroff/vagrant_selinux_dns_lab_fixed.git/'
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git push -u origin main
Перечисление объектов: 35, готово.
Подсчет объектов: 100% (35/35), готово.
При сжатии изменений используется до 4 потоков
Сжатие объектов: 100% (24/24), готово.
Запись объектов: 100% (35/35), 7.97 KiB | 3.98 MiB/s, готово.
Total 35 (delta 11), reused 30 (delta 9), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (11/11), done.
To https://github.com/kosogoroff/vagrant_selinux_dns_lab_fixed.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ ll
итого 16
-rw-rw-r-- 1 admin_insta11 admin_insta11 1501 сен 12 15:08 LICENSE
drwxrwxr-x 3 admin_insta11 admin_insta11 4096 сен 12 15:08 provisioning
-rwxrwxr-x 1 admin_insta11 admin_insta11 1931 сен 12 15:08 README.md
-rwxrwxr-x 1 admin_insta11 admin_insta11 1474 сен 12 15:39 Vagrantfile
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ rm -rf .git
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git init
hint: Using 'master' as the name for the initial branch. This default branch name
hint: will change to "main" in Git 3.0. To configure the initial branch name
hint: to use in all of your new repositories, which will suppress this warning,
hint: call:
hint:
hint: 	git config --global init.defaultBranch <name>
hint:
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint:
hint: 	git branch -m <name>
hint:
hint: Disable this message with "git config set advice.defaultBranchName false"
Инициализирован пустой репозиторий Git в /home/admin_insta11/vagrant_selinux_dns_problems_fixed/.git/
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git remote add origin https://github.com/kosogoroff/vagrant_selinux_dns_lab_fixed.git
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git add .
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ git status
Текущая ветка: master

Еще нет коммитов

Изменения, которые будут включены в коммит:
  (используйте «git rm --cached <файл>...», чтобы убрать из индекса)
	новый файл:    .gitignore
	новый файл:    LICENSE
	новый файл:    README.md
	новый файл:    Vagrantfile
	новый файл:    provisioning/files/client/motd
	новый файл:    provisioning/files/client/resolv.conf
	новый файл:    provisioning/files/client/rndc.conf
	новый файл:    provisioning/files/named.zonetransfer.key.special
	новый файл:    provisioning/files/ns01/named.50.168.192.rev
	новый файл:    provisioning/files/ns01/named.conf
	новый файл:    provisioning/files/ns01/named.ddns.lab
	новый файл:    provisioning/files/ns01/named.ddns.lab.view1
	новый файл:    provisioning/files/ns01/named.dns.lab
	новый файл:    provisioning/files/ns01/named.dns.lab.view1
	новый файл:    provisioning/files/ns01/named.newdns.lab
	новый файл:    provisioning/files/ns01/resolv.conf
	новый файл:    provisioning/playbook.yml

[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ vi provisioning/files/ns01/named.conf
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ vi provisioning/playbook.yml
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ vi provisioning/files/ns01/named.conf
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ 
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ 
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ 
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ 
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ vagrant up --provision
Bringing machine 'ns01' up with 'virtualbox' provider...
Bringing machine 'client' up with 'virtualbox' provider...
==> ns01: Running provisioner: ansible...
    ns01: Running ansible-playbook...

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [ns01]

TASK [install packages] ********************************************************
ok: [ns01]

PLAY [ns01] ********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [ns01]

TASK [copy named.conf] *********************************************************
changed: [ns01]

TASK [copy master zone dns.lab] ************************************************
ok: [ns01] => (item=/home/admin_insta11/vagrant_selinux_dns_problems_fixed/provisioning/files/ns01/named.dns.lab.view1)
ok: [ns01] => (item=/home/admin_insta11/vagrant_selinux_dns_problems_fixed/provisioning/files/ns01/named.dns.lab)

TASK [copy dynamic zone ddns.lab] **********************************************
changed: [ns01]

TASK [copy dynamic zone ddns.lab.view1] ****************************************
changed: [ns01]

TASK [copy master zone newdns.lab] *********************************************
ok: [ns01]

TASK [copy rev zones] **********************************************************
ok: [ns01]

TASK [copy resolv.conf to server] **********************************************
changed: [ns01]

TASK [copy transferkey to server] **********************************************
ok: [ns01]

TASK [set /etc/named permissions] **********************************************
ok: [ns01]

TASK [set /var/named/dynamic permissions] **************************************
changed: [ns01]

TASK [ensure named is running and enabled] *************************************
changed: [ns01]

PLAY [client] ******************************************************************
skipping: no hosts matched

PLAY RECAP *********************************************************************
ns01                       : ok=14   changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

==> client: Running provisioner: ansible...
    client: Running ansible-playbook...

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [client]

TASK [install packages] ********************************************************
ok: [client]

PLAY [ns01] ********************************************************************
skipping: no hosts matched

PLAY [client] ******************************************************************

TASK [Gathering Facts] *********************************************************
ok: [client]

TASK [copy resolv.conf to the client] ******************************************
changed: [client]

TASK [copy rndc conf file] *****************************************************
ok: [client]

TASK [copy motd to the client] *************************************************
ok: [client]

TASK [copy transferkey to client] **********************************************
ok: [client]

PLAY RECAP *********************************************************************
client                     : ok=7    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ 
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ 
[admin_insta11@mv334 vagrant_selinux_dns_problems_fixed]$ vagrant ssh cliant
The machine with the name 'cliant' was not found configured for
this Vagrant environment.
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
```
