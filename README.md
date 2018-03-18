## О чем это?

Это ежегодный конкурс IT-планета, а именно задания и мой вариант решения по конкурсу ["Администрирование Linux"](http://world-it-planet.org/projects/competition_detail.php?ID=62208)

Тут в общем-то всего три этапа:
* первый этап, ответы на теоретические вопросы разной сложности (20-30 штук), вопросы разные, начиная от "Кто является владельцем торговой марки Linux" и заканчивая ACL-ками в squid и маршрутизацией трафика в IPTables, кстати вопросы довольно редко обновляются, часть вопросов еще в 2012 были, точно помню
* второй этап — решение заданий удаленно, на выделенной для каждого участника виртуалке, решение именно этих заданий будет описано ниже
* ну и третий этап, он очный и каждый год проводится в разных городах, там примерно задания как на втором этапе, но немного сложнее и решать их уже нужно очно, сидя в лаборатории за выделенным компьютером

Несколько лет назад было немного по-другому.
Вместо второго, заочного этапа, проводились очные этапы на базе университетов.
Например, для того чтобы пройти на Международный финал, нужно было сначала показать хороший результат на Всеукраинском этапе (в зависимости от страны обучения студента), а чтобы попасть на Всеукраинский этап, нужно в своей области (административном округе, республике) показать хороший результат.

## Задания прошлых годов

Я участвовал в конкурсе еще с 2012 года и в то время я тоже записывал задания и в некоторых случаях ход решения:
* задание Всеукраинского финала в Харькове в 2013 году и мой вариант решения: [тыц](https://blog.amet13.name/2013/04/linux-12042013.html)
* задание Международного финала в Киеве в 2013 году: [тыц](https://blog.amet13.name/2013/06/linux-31052013-04062013.html)
* задание 2 этапа (заочное) в 2014 году: [тыц](https://blog.amet13.name/2014/03/linux-it-19032014.html)
* задание Международного финала в Симферополе в 2014 году: [тыц](https://blog.amet13.name/2014/09/linux-20092014.html)
* задание 2 этапа (заочное) в 2015 году: [тыц](https://blog.amet13.name/2015/03/linux-16032015.html)

## Задание 2 этапа (заочное) в 2018 году

Для решения заданий конкурса созданы виртуальные машины для каждого участника отдельно. 
На указанную Вами во время регистрации на сайте world-it-planet.org электронную почту были высланы реквизиты доступа к виртуальной машине.

Инструкция доступа к ней:
1. Используя свой логин и пароль нужно подключиться к ssh-прокси
2. В домашнем каталоге прочитать файл `readme.txt`

Смотрим, что там находится:
```
$ cat readme.txt 
see 83.149.198.151:8080/qqq.txt

tg: telegram организатора конкурса
tg group: https://t.me/contestlinux
```

### 1. Motd и SSH

Для начала нужно представиться в `/etc/motd` в формате:
```
name: Ivan Ivanov
Edu: где учишься
IP: eth0 - 192.168.1.123
```

Не забыть включить опцию, чтобы `motd` отображался при логине на сервер

### 2. Скрипт управления пользователями

[Тут](http://83.149.198.151:8080/users.txt) есть список пользователей, которых нужно добавить в систему, а чтобы не делать этого руками, нужно написать скрипт, который:
* добавляет пользователя в систему
* создает ему случайный пароль
* записывает пару `логин:пароль` в `/var/userpassword.txt`, сам скрипт положить в `/usr/local/bin/bulkadd` c `+x` атрибутом

(Скрипт можно писать на чем хочется)

### 3. Права пользователей

Из новосозданных пользователей:
* аккаунты `plasticinesmith` и `koshermosher` должны после истечения 30 дней перейти в статус `expired`
* пользователь `koshkanorm` должен иметь полный доступ к домашней директории `mothyabrodsky`
* в директории `/usr/share/documentstuff` разрешить пользователям `mothyabrodsky` `koshkanorm` из группы `joboffers` читать, писать, создавать файлы, всем остальным запретить

### 4. Systemd timer

Написать скрипт на bash, который по systemd таймеру раз в 30 минут собирает информацию об использовании системных ресурсов в формате:
```
Memory Usage: 5638 MB of 16039 MB
Processes: 297
Load Averages: 2.97 2.55 2.20
Logged users: root ivan masha
```

И записывает все это в `/var/log/my_report.log`, сам таймер положить в `/etc/systemd/system/my_timer.timer`

### 5. LXC

* установить `bridge-utils`, `libvirtd` и все что нужно для функционирования lxc
* не поломать существующую сеть и таблицу маршрутизации
* создать контейнер c `centos7` внутри которого установить `sshd`
* настроить ssh port forwarding, при подключении к порту `30005` на машине участника нужно попадать на `22` порт контейнера

### 6. Установить и настроить docker

* собрать (скачать, написать, украсть) бинарник https://github.com/klange/nyancat.git
* написать `Dockerfile` для образа `nyancat` (результаты положить в `/home/centos/nyancatdocker/`), собрать контейнер на базе `centos7`
* в контейнере с именем `nyancat` запустить бинарник на `23` порту
* при обращении к хост-машине на `23` порт должна быть видна анимация кошки
* после рестарта системы контейнер должен самостоятельно запуститься и работать

### 7. Chattr

В домашней директории есть файл `ohmyfile`.
Переместите его в `/var/`

### Не забывайте, что все должно самостоятельно запуститься и работать после рестарта виртуальной машины

## Мой вариант решения заданий

Сразу оговорюсь, на решение заданий у меня ушло ровно 3 часа и полчаса на перезагрузки и контрольные проверки.
Некоторые решения кажутся мне костыльными, но я их делал в угоду скорости, не очень хотелось засиживаться.
Некоторые тонкости обнародовались уже после того как я закончил задание, например то, что nyancat надо собирать не на хост-ноде, а в контейнере, я же просто один раз скомпилил и скопировал бинарник в образ контейнера.

Дабы не дублировать вопросы снова, рекомендую открыть два окна браузера рядом и читать параллельно и вопросы и решение, чтобы не мотать колесиком туда-сюда.

Поехали!

### 1. Motd и SSH

Тут ничего сложного нет.
```
$ sudo -i
# ip a show eth0 (получаем айпишник)
# vim /etc/motd
name: Amet Umerov
Edu: Sevastopol State University
IP: eth0 - 172.31.51.115
```

Проверка:
```
# exit
$ ssh -p 3119 centos@83.149.198.150
name: Amet Umerov
Edu: Sevastopol State University
IP: eth0 - 172.31.51.115
```

В автозапуск я ничего больше не добавлял, оно автоматически подхватило конфиг motd и кушать не просит.

С этим заданием все, идем дальше.

### 2. Скрипт управления пользователями

Посмотрим что у нас по этой ссылке есть:
```
$ curl http://83.149.198.151:8080/users.txt
CookieMonster
Liza
TerribleIvan
skipercacao
koshermosher
koshkanorm
notsogayasuexpect
plasticinesmith
diabloiiplayer
funwithyourmom
ballisticbeer
mothyabrodsky
onewordauthor
untrustedsubnet
rataratrat
schoolslacker
ohwhatyoudoing
```

Там просто список пользователей.
Пишем скрипт, я писал на bash, потому что это проще и быстрее для меня:
```
# vim /usr/local/bin/bulkadd 
#!/bin/bash

if [[ ${EUID} -ne 0 ]]; then
	echo "Please run as root or use sudo"
	exit 1
fi

URL=http://83.149.198.151:8080/users.txt
FILE=users.txt
ENDFILE=/var/userpassword.txt

curl -s ${URL} -o ${FILE}
while read USER ; do
	useradd ${USER}
	PASSWORD=$(head /dev/urandom | tr -dc A-Za-z0-9 | head -c 15 ; echo '')
	echo ${USER}:${PASSWORD} | chpasswd
	echo ${USER}:${PASSWORD} | tee -a ${ENDFILE}
done < ${FILE}

echo "Succesfully added users, check file ${ENDFILE}"
exit 0
```

В принципе тут все понятно, ничего сложного:
* сначала проверяем юзера который запускает скрипт, это может делать только `root`
* скачиваем файл с пользователями, сохраняем в файл
* в цикле смотрим все имена пользователей в файле
* генерим для каждого пароль, создаем юзера, устанавливаем пароль
* доступы записываем в файл

Запускаем скрипт и проверяем что все отработало:
```
# chmod +x /usr/local/bin/bulkadd 
# /usr/local/bin/bulkadd

# cat /var/userpassword.txt 
CookieMonster:yOio4D9IWdr0GWp
Liza:uYTb8FFldeyTDqW
TerribleIvan:qdXpMt63laEG86P
...

# cat /etc/passwd
# su Liza
```

Тут вроде тоже все хорошо, идем дальше.

### 3. Права пользователей

Устанавливаем для указанных юзеров срок окончания действия их аккаунтов через 30 дней:
```
# chage -E `date -d "30 days" +"%Y-%m-%d"` plasticinesmith
# chage -E `date -d "30 days" +"%Y-%m-%d"` koshermosher
```

Проверка:
```
# chage -l plasticinesmith | grep "Acc"
Account expires           : Apr 17, 2018
# chage -l koshermosher | grep "Acc"
Account expires           : Apr 17, 2018
```

Все в порядке.

Далее добавляем юзера `koshkanorm` в группу `mothyabrodsky`, чтобы он мог иметь доступ к нужным файлам и даем группе права `770` на каталог:
```
# usermod -a -G mothyabrodsky koshkanorm
# chmod 770 /home/mothyabrodsky/ -R
```

Проверка:
```
# su koshkanorm
$ cd /home/mothyabrodsky/
$ touch test && rm test
```

Файл успешно создался и удалился, значит все ок.

Для того, чтобы юзеры `mothyabrodsky` и `koshkanorm` могли получить общий доступ в один каталог, надо добавить их в общую группу `joboffers` и указать для каталога корректные права:
```
# groupadd joboffers
# usermod -a -G joboffers mothyabrodsky
# usermod -a -G joboffers koshkanorm
# mkdir /usr/share/documentstuff
# chown root:joboffers /usr/share/documentstuff/
# chmod 770 -R /usr/share/documentstuff/
```

Проверка:
```
# ls -l /usr/share/ | grep stuff
drwxrwx---.   2 root joboffers     6 Mar 18 07:44 documentstuff
# su koshkanorm
$ cd /usr/share/documentstuff/
$ mkdir test && rmdir test
```

Каталог успешно создается и удаляется для этого пользователя.

Проверим, сможет ли другой юзер зайти в каталог, который не имеет отношения к нему:
```
# su koshermosher
$ cd /usr/share/documentstuff/
bash: cd: /usr/share/documentstuff/: Permission denied
```

Все в порядке, для других юзеров доступ закрыт, как и задумывали.

### 4. Systemd timer

> Раньше никогда не использовал systemd timer, а в этот раз буду. Планировщик заданий от народа.

Первым делом напишем скрипт:
```
# vim /usr/local/bin/system_information
#!/bin/bash

USED=$(free -m | grep Mem | awk '{print $3}')
TOTAL=$(free -m | grep Mem | awk '{print $2}')
PROCS=$(ps -Al | wc -l)
LA=$(cat /proc/loadavg | awk '{print $1 " " $2 " " $3}')
USERS=$(users)
OUTFILE=/var/log/my_report.log

{
echo "Memory Usage: ${USED} MB of ${TOTAL} MB"
echo "Processes: ${PROCS}"
echo Load Averages: ${LA}
echo "Logged users: ${USERS}"
echo ""
} >> ${OUTFILE}

exit 0
```

Возможно тут могут возникнуть некоторые споры по поводу того, как именно подсчитывать количество процессов, но думаю что суть задания не в этом.
По скрипту можно было еще [шеллчеком](https://github.com/koalaman/shellcheck) пройтись, для красоты, но это не обязательно.

Устанавливаем права на скрипт и лог, проверяем работоспособность скрипта:
```
# chmod 777 /usr/local/bin/system_information
# touch /var/log/my_report.log && chmod 777 /var/log/my_report.log 
# /usr/local/bin/system_information

# cat /var/log/my_report.log 
Memory Usage: 85 MB of 992 MB
Processes: 84
Load Averages: 0.00 0.01 0.05
Logged users: centos centos
```

Все как в техзадании.

Создаем сервисный файл для таймера:
```
# vim /usr/lib/systemd/system/my_timer.service
[Unit]
Description=Get sysinfo for timer

[Service]
Type=simple
ExecStart=/usr/local/bin/system_information

[Install]
WantedBy=multi-user.target
```

Создаем сам таймер:
```
# vim /etc/systemd/system/my_timer.timer
[Unit]
Description=Run script every 30min

[Timer]
OnCalendar=*:0/30
Unit=my_timer.service

[Install]
WantedBy=multi-user.target
```

Добавляем таймер в автозагрузку:
```
# systemctl daemon-reload
# systemctl enable my_timer.timer
```

Проверка:
```
# systemctl list-timers | grep my_timer
Sun 2018-03-18 08:30:00 UTC  3min 46s left Sun 2018-03-18 08:25:11 UTC  1min 2s ago my_timer.timer               my_timer.service
# systemctl is-enabled my_timer.timer && systemctl is-active my_timer.timer
enabled
active
```

Все хорошо, через полчаса можно перепроверить, что все отработало и в лог записало.

### 5. LXC

Ставим LXC, добавляем в автозапуск, все как обычно:
```
# yum install -y epel-release
# yum install -y lxc lxc-templates libvirt perl lxc-extra
# systemctl enable lxc && systemctl start lxc && systemctl start libvirtd
```

Создаем наш контейнер:
```
# lxc-create -n centos7_lxc -t /usr/share/lxc/templates/lxc-centos
# lxc-start -n centos7_lxc -d
# lxc-ls --active
centos7_lxc  
```

Заходим в контейнер и проверяем что `sshd` есть в автозагрузке, заодно и узнаем адрес виртуалки:
```
# lxc-attach -n centos7_lxc

[root@centos7_lxc /]# systemctl is-enabled sshd
enabled
[root@centos7_lxc /]# lsof -i :22
COMMAND PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd    304 root    3u  IPv4 100926      0t0  TCP *:ssh (LISTEN)
sshd    304 root    4u  IPv6 100928      0t0  TCP *:ssh (LISTEN)
[root@centos7_lxc /]# ip a show eth0 | grep "inet "
    inet 192.168.122.173/24 brd 192.168.122.255 scope global dynamic eth0
[root@centos7_lxc /]# exit
```

Добавляем контейнер в автозапуск (скорее всего есть для этого команда какая-то, но я быстрее нагуглил именно вариант с конфигом):
```
# vim /var/lib/lxc/centos7_lxc/config
lxc.start.auto = 1
lxc.start.delay = 1
```

Systemd с каждым разом меня удивляет все больше и больше, можно не городить DNAT/SNAT в netfilter или ставить haproxy, можно использовать штатный `systemd-socket-proxyd` для нашей конкретной ситуации.
А именно прокинуть порт `30005` внутрь контейнера на порт `22`.

Создаем файл сокета:
```
# vim /etc/systemd/system/ssh_proxy.socket
[Socket]
ListenStream=30005

[Install]
WantedBy=sockets.target
```

И сервис для его запуска:
```
# vim /etc/systemd/system/ssh_proxy.service
[Unit]
Requires=lxc.service
After=lxc.service
Requires=ssh_proxy.socket
After=ssh_proxy.socket

[Service]
ExecStart=/usr/lib/systemd/systemd-socket-proxyd 192.168.122.173:22
```

Добавляем в автозапуск все и проверяем:
```
# systemctl start ssh_proxy.socket && systemctl enable ssh_proxy.socket
# systemctl enable ssh_proxy && systemctl start ssh_proxy
# lsof -i :30005
COMMAND    PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd      1 root   27u  IPv6  30466      0t0  TCP *:30005 (LISTEN)
systemd-s 1827 root    3u  IPv6  30466      0t0  TCP *:30005 (LISTEN)
# ssh localhost -p 30005
root@localhost's password: 
```

Все работает.

### 6. Установить и настроить docker

Ставим Docker:
```
# yum install -y yum-utils device-mapper-persistent-data lvm2
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# yum install -y docker-ce
# systemctl enable docker && systemctl start docker
```

Качаем исходники `nyancat` и собираем бинарник:
```
# yum install -y gcc make automake autoconf
# mkdir /home/centos/nyancatdocker/ && cd nyancatdocker/
# wget https://github.com/klange/nyancat/archive/1.5.1.tar.gz && tar xzf 1.5.1.tar.gz 
# cd nyancat-1.5.1/src/
# make
# mv nyancat ../../ && cd ../../
# rm -f 1.5.1.tar.gz && rm -rf nyancat-1.5.1/
```

Далее я немного призадумался, так как не знал как заставить эту кошку работать в качестве telnet-сервера, спустя некоторое время до меня дошло, что это делается через xinetd-подобные штуки (не все поймут, не многие вспомнят).

Вот [этот](https://github.com/ddhhz/nyancat-server) репозиторий подсказал мне что можно использовать `onenetd`, так и сделаем пожалуй.

Качаем исходники и компилим:
```
# wget https://github.com/ddhhz/onenetd/archive/v12.tar.gz && tar xzf v12.tar.gz 
# autoreconf -vfi && ./configure && make
# mv onenetd ../ && cd ../
# rm -rf onenetd-12/ && rm -f v12.tar.gz 
```

Пишем `Dockerfile`:
```
FROM centos:7
MAINTAINER Amet Umerov

COPY nyancat /usr/local/bin/
COPY onenetd /usr/local/bin/

EXPOSE 23
ENTRYPOINT ["/usr/local/bin/onenetd", "-v1", "0", "23", "nyancat", "--telnet"]
```

Билдим образ и запускаем контейнер:
```
# docker pull centos:7
# docker build -t nyancat .
# docker run -i -d --restart always --name=nyancat -p 23:23 nyancat
```

Проверяем что контейнер работает:
```
# docker ps
CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS              PORTS                NAMES
4d5acf3a43ee        nyancat             "/root/nyancat -t"   3 seconds ago       Up 2 seconds        0.0.0.0:23->23/tcp   nyancat
```

Коннектимся к нашей кошке:
```
# lsof -i :23
COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
docker-pr 18765 root    4u  IPv6 121777      0t0  TCP *:telnet (LISTEN)
# telnet localhost 23
```

![](https://raw.githubusercontent.com/Amet13/linux-contest-2018/master/nyancat.png)

### 7. Chattr

Тут все просто, атрибут неизменяемости стоит, убираем его и переносим файл:
```
# lsattr ohmyfile 
----i----------- ohmyfile
# chattr -i ohmyfile 
# mv ohmyfile /var/
```

Собственно говоря, вот и все.
После этого можно перезагрузить сервер и пробежаться по всем заданиям снова, удостоверившись что все поднялось.
В моем случае вроде все было хорошо, если где-то и есть ошибки, то из-за невнимательности.

Спасибо организаторам, за то что нет на этот раз LDAP, IPTables и прочих почтовых серверов. Многие их недолюбливают :)

## Дополнения

* исправления — Fork и [Pull requests](https://github.com/Amet13/linux-contest-2018/pulls)
* вопросы можно в [Issues](https://github.com/Amet13/linux-contest-2018/issues) писать
* свои варианты решений тоже можете предложить в [Pull requests](https://github.com/Amet13/linux-contest-2018/pulls)
* вопросы наверное можно сюда задавать: https://t.me/contestlinux

Те кто дочитал до конца и прострелил себе ногу на правилах IPTables, может вам пригодится [мой опыт](https://blog.amet13.name/2014/04/iptables.html) как этого можно избежать.

![](https://raw.githubusercontent.com/Amet13/linux-contest-2018/master/thats_all_folks.jpg)
