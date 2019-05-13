# Bash

**Быстро скопировать права (именно права) одного файла на другой** можно вот так:

```
chmod --reference=alice.log bob.log
```
Так мы файлу bob.log зададим те же права что и у файла alice.log

Аналогично и с владельцем:

```
chown --reference=alice.log bob.log
```

При этом, можно делать, например, такие штуки:

```
find /some/dir -type f -print0 | xargs -O -I {} chmod --reference=/some/file.txt {}
```
---

***Получить имя домена из сертификата**, можно с помощью самого openssl:

```
openssl x509 -noout -subject -in ssl.<domain>.pem | cut -f3 -d=
<domain>
```

А тем, кто как и я не может осилить синтаксис openssl по памяти, я рекомендую воспользоваться утилитой certtool из пакета gnutls-utils:

```
certtool -i < ssl.<domain>.pem | grep "Subject: CN" | cut -f2 -d=
<domain>
```
---

**Увидеть какие процессы не дают освободить место на диске после удаления файла** можно с помощью lsof:

```
lsof -a +L1 /path/to/dir
```

---

**psacct\acct** - набор утилит для получения дополнительной информации о действиях других аккаунтов на сервере.

**ac** - время пребывания на сервере;

**sa** - вывод команд, выполняемых пользователями;

**lastcomm** - вывод последних выполненных команд.

---

Иногда возникает необходимость **проверить права на все директории в пути к определённому файлу**. В этом случае на помощь приходит namei:

```
namei -l /home/user/web/site/public_html/wp-content/themes/f2/inc/theme-options/theme-options.css 

f: /home/user/web/site/public_html/wp-content/themes/f2/inc/theme-options/theme-options.css
dr-xr-xr-x root root /
drwxr-xr-x root root home
drwx--x--x user user user
drwxr-xr-x user user web
drwxr-x--x user user site
drwxr-x--x user user public_html
drwxr-xr-x user user wp-content
drwxr-xr-x user user themes
drwxr-xr-x user user f2
drwxr-xr-x user user inc
drwxr-xr-x user user theme-options
-rw-r--r-- user user theme-options.css
```

---

Задачка **копирования структуры только каталогов, без файлов в них**. Решилось всё rsync'ом:

```
rsync -av -f"+ */" -f"- *" /path/to/src/dir /path/to/targ/dir/
```

В итоге, получаем дублирование структуры каталогов с сохранением атрибутов, прав, владельцев и т. п.

---

Быстро **сделать бекап всей системы, если она расположена на одной партации**:

```
tar -cvpzf system.tar.gz --exclude=/system.tar.gz --one-file-system /
```

Если партаций используется несколько (например, /var или /home на отдельных разделах), то --one-file-system не подойдёт для такого случая, тогда просто достаточно создать архив от корня, но при этом, с помощью --exclude исключить лишние директории.

---

**Копируем содержимое папки с dot-файлами**

Квантификатор * не учитывает файлы и папки, начинающиеся с точки, поэтому в список аргументов для копирования они не попадают.

Для решения этой проблемы  можно использовать такую конструкцию, которая скопирует все содержимое директории, включая dot-файлы и папки:

```
cp -r /opt/. /app
```

---

**Curl resolve w\o dns**

Иногда нужно протестировать какой-то домен здесь и сейчас (особенно когда он работает через прокси и\или https), лезть при этом делать dns неохото, прописывать в hosts - тоже долго, поскольку это можно сделать не отходя далеко от курла:

```
curl --resolve foo.example.com:443:127.0.0.1 https://foo.example.com:443/
```

---

**SSH**

```
mkdir ~/.ssh
chmod ~/.ssh 600
echo "содержимое id_rsa.pub" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

**Закинуть свои ключи с сервера**

```
ssh-copy-id логин@адрес_клиента
```

---

**Обрезаем строку, выбирая диапазон для отображения**

```
cut -c 140-240 file
```

Или через **pipe** или напрямую из файла можно заставить отобразить именно какой-то конкретный кусок строки. Например это может понадобиться для дебага при просмотре логов mysql, чтобы видеть запросы.

---

**Отсоединение запущенного процесса от процесса-родителя**

1. Запустить программу, например gedit
2. Нажать комбинацию Ctrl+Z (в linux эта комбинация посылает сигнал  SIGTSTP процессу)
3. Ввести disown - родитель текущего процесса меняется
4. Посмотреть pid своего процесса - ps aux | grep [g]edit
5. Вернуть его к жизни kill -CONT $pid
6. Можно закрывать процесс-родитель.

Вообще способов существуют несколько, это один из вариантов.

---

**Назначаем права на файлы**

Дано:  
есть N файлов с неизвестными разными правами
Задача: для всех файлов и директорий сделать права группы = правам пользователя

Решение: 
```
chmod -R g=u ./*
```

---

**Определяем каталоги, которые занимают больше всего inode**

```
find . -printf "%h\n" | cut -d/ -f-2 | sort | uniq -c | sort -rn
```

---

**Ubuntu 18 и локальный dns**

Столкнулись с очень веселым поведением сегодня на бубунточке. Эта красивая и без сомнения удобная ОС использует локальный резолвер dns из коробки. т.е. вместо стандартных записей dns серверов полученных по dhcp в /etc/resolv.conf будет помещена запись типа 127.0.53.1. Локально на этом адресе будет слушать systemd-resolver. Казалось бы ну ок, к чему это может привести..?

А вот к чему. Когда вы запускаете локально docker, он монтирует /etc/resolv.conf с хоста внутрь контейнера. Угадайте, работает ли внутри контейнера после этого dns резолв, когда обращаться надо в 127.0.53.1? Правильно, нихуа!

В качестве решения - отключаем напрочь эту очень удобную фичу:

```
Disable and stop the systemd-resolved service:

# systemctl disable systemd-resolved.service
# systemctl stop systemd-resolved
```

Добавляем в секцию [main] запись в файл /etc/NetworkManager/NetworkManager.conf

```
dns=default
```

Delete the symlink /etc/resolv.conf

```
rm /etc/resolv.conf
```

Перезапускаем службу нетворк-манагера

```
service network-manager restart
```

---

**Создаем архив исключая файлы контроля версий**

Есть у tar замечательный ключ, который позволяет так сделать: exclude version control system directories

```
--exclude-vcs
```

---

**Вывод результата выполнения команды в переменной с сохранением переносов**

```
cmd=$(ps aux)
printf "%s\n" "$cmd"
```

---

**Grep'ай [у]добно.**

Не так:
```
$ ps aux | grep xfce4-terminal
user 10216  0.0  2.0 686344 79004 ?        Sl   фев14   0:12 /usr/bin/xfce4-terminal
user 14777  0.0  0.0 215740   828 pts/2    S+   09:05   0:00 grep --color=auto xfce4-terminal
```

Вот так:
```
$ ps aux | grep [x]fce4-terminal
user 10216  0.0  2.0 686344 79004 ?        Sl   фев14   0:13 /usr/bin/xfce4-terminal
```

---

**Время старта процесса**

```
$ ps -eo pid,lstart,cmd
  PID CMD                                          STARTED
    1 Tue Jun  7 01:29:38 2016 /sbin/init                  
    2 Tue Jun  7 01:29:38 2016 [kthreadd]                  
    3 Tue Jun  7 01:29:38 2016 [ksoftirqd/0]               
    5 Tue Jun  7 01:29:38 2016 [kworker/0:0H]              
    7 Tue Jun  7 01:29:38 2016 [rcu_sched]                 
    8 Tue Jun  7 01:29:38 2016 [rcu_bh]                    
    9 Tue Jun  7 01:29:38 2016 [migration/0]               
   10 Tue Jun  7 01:29:38 2016 [kdevtmpfs]                 
   11 Tue Jun  7 01:29:38 2016 [netns]                     
  277 Tue Jun  7 01:29:38 2016 [writeback]                 
  279 Tue Jun  7 01:29:38 2016 [crypto]   
```

---

**Разархивирование файлов из директории в архиве**

Допустим есть архив, в нем папка, а в папке файлы. Хотим разархивировать файлы из этой папки, а сама папка нам как бы не нужна.

```
tar --strip-components=1 -zxvf arhive.tar.gz
```

---

**Удаление старых файлов**

Подсчет занимаемого места:

```
nice find ./ -type f -mtime +365 | xargs du | awk '{ t+=$1 } END { print t }'
```

Просмотр:

```
find ./ -type f -mtime +365 -exec ls {} \; 
```

Удаление:

```
nice find ./ -type f -mtime +365 -exec rm {} \; 
```

---

**Decode entries in /proc/net/tcp**

```
Linux 5.x  /proc/net/tcp
Linux 6.x  /proc/PID/net/tcp

Given a socket:

$ ls -l  /proc/24784/fd/11
lrwx------ 1 jkstill dba 64 Dec  4 16:22 /proc/24784/fd/11 -> socket:[15907701]

Find the address

$ head -1 /proc/24784/net/tcp; grep 15907701 /proc/24784/net/tcp
  sl  local_address rem_address   st  tx_queue  rx_queue tr tm->when  retrnsmt   uid  timeout inode
  46: 010310AC:9C4C 030310AC:1770 01 0100000150:00000000  01:00000019 00000000  1000 0 54165785 4 cd1e6040 25 4 27 3 -1

46: 010310AC:9C4C 030310AC:1770 01 
|   |         |   |        |    |--> connection state
|   |         |   |        |------> remote TCP port number
|   |         |   |-------------> remote IPv4 address
|   |         |--------------------> local TCP port number
|   |---------------------------> local IPv4 address
|----------------------------------> number of entry

00000150:00000000 01:00000019 00000000 
|        |        |  |        |--> number of unrecovered RTO timeouts
|        |        |  |----------> number of jiffies until timer expires
|        |        |----------------> timer_active (see below)
|        |----------------------> receive-queue
|-------------------------------> transmit-queue

1000 0 54165785 4 cd1e6040 25 4 27 3 -1
|    | |        | |        |  | |  |  |--> slow start size threshold, 
|    | |        | |        |  | |  |       or -1 if the treshold
|    | |        | |        |  | |  |       is >= 0xFFFF
|    | |        | |        |  | |  |----> sending congestion window
|    | |        | |        |  | |-------> (ack.quick<<1)|ack.pingpong
|    | |        | |        |  |---------> Predicted tick of soft clock
|    | |        | |        |               (delayed ACK control data)
|    | |        | |        |------------> retransmit timeout
|    | |        | |------------------> location of socket in memory
|    | |        |-----------------------> socket reference count
|    | |-----------------------------> inode
|    |----------------------------------> unanswered 0-window probes
|---------------------------------------------> uid


timer_active:
0 no timer is pending
1 retransmit-timer is pending
2 another timer (e.g. delayed ack or keepalive) is pending
3 this is a socket in TIME_WAIT state. Not all field will contain data.
4 zero window probe timer is pending

==========================================
Perl script to decode the address

#!/usr/bin/perl

my $hexip=$ARGV[0];
my $hexport=$ARGV[1];

print "hex: $hexip\n";

my @ip = map hex($_), ( $hexip =~ m/../g );

my $ip = join('.',reverse(@ip));

my $port = hex($hexport);

print "IP: $ip  PORT: $port\n";

==========================================

$ hexip.pl 030310AC 1770
hex: 030310AC
IP: 172.16.3.3  PORT: 6000
```

---

