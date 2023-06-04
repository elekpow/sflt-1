# Домашнее задание к занятию «Disaster recovery и Keepalived» - Левин Игорь

### Цель задания
В результате выполнения этого задания вы научитесь:
1. Настраивать отслеживание интерфейса для протокола HSRP;
2. Настраивать сервис Keepalived для использования плавающего IP

------

### Чеклист готовности к домашнему заданию

1. Установлена программа Cisco Packet Tracer
2. Установлена операционная система Ubuntu на виртуальную машину и имеется доступ к терминалу
3. Сделан клон этой виртуальной машины, они находятся в одной подсети и имеют разные IP адреса
4. Просмотрены конфигурационные файлы, рассматриваемые на лекции, которые находятся по [ссылке](https://github.com/netology-code/sflt-homeworks/blob/main/sflt-1/1)

---

### Задание 1

- Дана [схема](https://github.com/netology-code/sflt-homeworks/blob/main/sflt-1/1/hsrp_advanced.pkt) для Cisco Packet Tracer, рассматриваемая в лекции.
- На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)
- Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).
- Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.
- На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.

 ---

### Выполнения задания 1


конфигурируем маршрутизатор: Router1
```
  enable
  configure terminal
```  
проверяем текущую конфигурацию командой 
```
  do show running-config
  do show standby brief
```
настраиваем интерфейс
```
  interface GigabitEthernet0/1
```
выставляем приоритет 
```
  standby 1 priority 105  
  standby 1 preempt
  standby 1 track GigabitEthernet 0/0
```
конфигурируем маршрутизатор: Router1

проверяем текущую конфигурацию командой 
```
  do show running-config
  do show standby brief
```
настраиваем интерфейс
```
  interface GigabitEthernet0/1
```
выставляем приоритет 
```
  standby 1 preempt
  standby 1 track GigabitEthernet 0/0
```



[Схема решения](https://github.com/elekpow/sflt-1/blob/main/sflt-1/hsrp_advanced1.pkt)

#### при разрыве одно из линий, второму маршрутизатору присваевается роль активного маршрутизатора 

|Вариант 1|
|:-:|
|<img src="https://github.com/elekpow/sflt-1/blob/main/sflt-1/вариант1.gif" alt="вариант1.gif" width="300">|

|Вариант 2|Вариант 3|
|:-|-:|
|<img src="https://github.com/elekpow/sflt-1/blob/main/sflt-1/вариант2.gif" alt="вариант2.gif" width="300">|<img src="https://github.com/elekpow/sflt-1/blob/main/sflt-1/вариант3.gif" alt="вариант3.gif" width="300">|

|Вариант 4|Вариант 5|
|:-|-:|
|<img src="https://github.com/elekpow/sflt-1/blob/main/sflt-1/Вариант4.gif" alt="Вариант4.gif" width="300">|<img src="https://github.com/elekpow/sflt-1/blob/main/sflt-1/Вариант5.gif" alt="Вариант5.gif" width="300">|






 ---

### Задание 2

- Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного [файла](https://github.com/netology-code/sflt-homeworks/blob/main/sflt-1/1/keepalived-simple.conf).
- Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
- Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
- Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
- На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html

### Выполнения задания 2

Создаю две виртуальные машины Linux

 ![Virtualbox](https://github.com/elekpow/sflt-1/blob/main/sflt-1/Virtualbox.JPG)  



конфигурационный файл keepalived.conf

```
global_defs {
    script_user root
    enable_script_security
}

vrrp_script check_srv {
    script "/etc/keepalived/scripts/check_testsrv.sh"
    interval 3
}

vrrp_instance VI_1 {
        state MASTER
        interface enp0s3
        virtual_router_id 15
        priority 100
        advert_int 1

        virtual_ipaddress {
                192.168.56.15/24
        }

        track_script {
           check_srv
        }
}

```


для выполения условия проверки наличия файла index.html и доступности сервера на 80 порту примене скрипт:

```
#!/bin/bash

FILE="/var/www/html/index.html"
PORT="80"

# netstat
if ! [ -x "$(command -v netstat)" ]; then
  echo 'netstat is not installed.' >&2
  exit 1
fi

# check port
if netstat -tuln | grep ":$PORT " > /dev/null; then
  if [ -e "$FILE" ]; then exit 0  # "0"
  else exit 1 # "1" "Port 80 is open, File \"index.html\" does not exist"
  fi
else
  if [ -e "$FILE" ]; then exit 1  # "1" "Port 80 is closed, File \"index.html\" exists"
  else exit 1 # "1" Port 80 is closed, File \"index.html\" does not exist"
  fi
fi

```


Веб-сервер Nginx, доступен на виртуальном ip-адресе 192.168.56.15


Состояние сервиса keepalived на сервере (Master)

 ![server_master](https://github.com/elekpow/sflt-1/blob/main/sflt-1/server_master.JPG)  
  
 При остановке сервера или при отсутсвии файла index.html 

 ![server_master_state](https://github.com/elekpow/sflt-1/blob/main/sflt-1/server_master_state.JPG)
 
 Web страница
 ![Nginx_server_2](https://github.com/elekpow/sflt-1/blob/main/sflt-1/Nginx_server_2.JPG)
 
 Состояние сервиса keepalived на втором серевере (Backup)
 
  ![server_backup](https://github.com/elekpow/sflt-1/blob/main/sflt-1/server_backup.JPG)
 
  
При возобновлении работы сервера и при наличии файла index.html 
 
  ![server_master_state_m](https://github.com/elekpow/sflt-1/blob/main/sflt-1/server_master_state_m.JPG)

 Web страница
 
  ![Nginx_serve](https://github.com/elekpow/sflt-1/blob/main/sflt-1/Nginx_server.JPG)


