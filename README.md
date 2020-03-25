# OTUS-devops
DevOps практики и инструменты

## HW #5: Знакомство с облачным сервисом GCP
#### Запуск VM Bastion host:
* Генерация пары ключей для ssh соединения `ssh-keygen -f ~/.ssh/<private_key_file_name>`
* Далее необходимо добавить публичный ключ в раздел Метаданны > SSH-ключи
* Создаем экземпляр виртуальной машины, далее привязываем статический айпи адрес VM инстанс > Изменить > Выбираем нужный сетевой
интерфейс > меняем тип адреса и указываем имя
* Установить ssh соединение с тачкой с внешним айпи адресом можно командой
`ssh -i ~/.ssh/<private_key_file_name` username@<external-ip> где -i - указатель на приватный ключ юзера
* Далее создаем экземпляр ВМ без внешнего айпи адреса, в одной сети с bastion host
* Чтобы до этого экземпляра можно было достучаться с локальной тачки можно зайти на bastion host
после чего установить соединение командой `ssh username@<internal_ip_address>`, но так как приватного ключа на хосте бастиона не лежит, то
  необходимо пробросить его с помощью ssh агента
* `ssh-add -L` - проверка добавленных ключей в агент; `ssh-add ~/.ssh/<private_key_file_name>` добавление ключа в агента
* Теперь можно подключиться к внутренней тачке с помощью команды `ssh -i ~/.ssh/<private_key_file_name> username@<external_ip> -A` и уже с бастион хоста кинуть соединение на внутренний инстанс
* Чтобы установить соединение напрямую без дополнительных команд с внутренним экземпляром ВМ, можно проксировать запросы через
  bastion host для этого:
  1. Создаем файл `~/.ssh/config` и в нем прописываем следующие конфиги:
```
Host alias1
  Hostname <external_ip>
  User username
  IdentityFile <absolut_path_to_private_bastion_key>
Host alias2
  Hostname <internal_ip>
  User username
  ProxyCommand ssh -W %h:%p alias1
```
* Далее можно подключиться по алиасу `ssh alias2`
  
  #### Создание VPN сервера на экземпляре VM bastion host
  В консоли bastion host выполняем следующие команды
```
$ cat <<EOF> setupvpn.sh
> #!/bin/bash
> echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" > /etc/
apt/sources.list.d/mongodb-org-3.4.list
> echo "deb http://repo.pritunl.com/stable/apt xenial main" > /etc/apt/sources.list.d/
pritunl.list
> apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv
0C49F3730359A14518585931BC711F9BA15703C6
> apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv
7568D9BB55FF9E5287D586017AE645C0CF8E292A
> apt-get --assume-yes update
> apt-get --assume-yes upgrade
> apt-get --assume-yes install pritunl mongodb-org
> systemctl start pritunl mongod
> systemctl enable pritunl mongod
> EOF
```
`$ sudo bash setupvpn.sh`
это создаст файл со скриптами и выполнит его, после чего можно перейти по ссылке `http://<адрес bastion VM>/setup`
* Сгенерировать на сервере ключ и ввести его, командой `sudo pritunl setup-key`
* Сгенерировать пароль и логин командо `sudo pritunl default-password`
