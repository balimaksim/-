# Создание собственного vpn-сервера на основе технологии OpenVPN и свервера от рег.ру


## Создаем собственный VPN-сервер: установка OpenVPN

Мы производили установку на сервер с ОС Ubuntu 20.04 — для других версий процесс установки и настройки может отличаться. Чтобы начать установку, подключитесь к серверу по SSH.

### Установка пакетов
1.   Обновите списки пакетов с помощью команды:
	
		sudo apt update

2. Установите пакеты, необходимые для установки и настройки OpenVPN:
		
		sudo apt install openvpn easy-rsa net-tools

### Настройка удостоверяющего центра
После установки пакетов необходимо развернуть на сервере центр сертификации и сгенерировать корневой сертификат. Для этого: 
1. Скопируйте необходимые файлы в папку /etc/openvpn/, а затем перейдите в нее:

		sudo cp -R /usr/share/easy-rsa /etc/openvpn/
		cd /etc/openvpn/easy-rsa 

2. Создайте удостоверяющий центр и корневой сертификат:

		export EASYRSA=$(pwd)  
		sudo ./easyrsa init-pki  
		sudo ./easyrsa build-ca

После ввода последней команды вам будет предложено ввести кодовую фразу. Указанный вами пароль в дальнейшем будет использоваться для подписи сертификатов и ключей.

У вас появятся 2 файла:
	+ /etc/openvpn/easy-rsa/pki/ca.crt — сертификат CA
	+ /etc/openvpn/easy-rsa/pki/private/ca.key — приватный ключ CA. 

1. Создайте директорию, в которой будут храниться ключи и сертификаты:

		mkdir /etc/openvpn/certs/
2. Переместите в созданную папку сертификат CA:
		
		cp /etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/certs/ca.crt

## Создание запроса сертификата и закрытого ключа сервера OpenVPN
1. Создайте закрытый ключ для сервера и файл запроса сертификата с помощью команды:

		./easyrsa gen-req server nopass
2. Подпишите созданный сертификат ключом сертификационного центра:

		./easyrsa sign-req server server

Затем введите «yes» и нажмите **Enter**. После введите кодовую фразу, которую вы задали при настройке удостоверяющего центра. 

3. Скопируйте сгенерированный сертификат и ключ в директорию /etc/openvpn/certs/:
		cp /etc/openvpn/easy-rsa/pki/issued/server.crt /etc/openvpn/certs/
		cp /etc/openvpn/easy-rsa/pki/private/server.key /etc/openvpn/certs/

4. Сгенерируйте файл параметров Diffie–Hellman:
		
		openssl dhparam -out /etc/openvpn/certs/dh2048.pem 2048

5. Создайте ключ HMAC с помощью команды:

		openvpn --genkey --secret /etc/openvpn/certs/ta.key

Теперь в директории /etc/openvpn/certs/ должно быть 5 файлов. Проверить это можно с помощью команды:

		ls -l /etc/openvpn/certs/

Вывод должен выглядеть следующим образом:

![Изображение](https://img.reg.ru/news/blog_25082023_image2.png)
