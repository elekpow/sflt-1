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
4. Просмотрены конфигурационные файлы, рассматриваемые на лекции, которые находятся по [ссылке](https://github.com/netology-code/sflt-homeworks/blob/main/1)

---

### Задание 1

- Дана [схема](https://github.com/netology-code/sflt-homeworks/blob/main/1/hsrp_advanced.pkt) для Cisco Packet Tracer, рассматриваемая в лекции.
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


[Схема решения](https://github.com/elekpow/sflt-1/blob/main/hsrp_advanced.pkt)



|Столбец 1|Столбец 2|
|:-|-:|
|<img src="https://github.com/elekpow/sflt-1/blob/main/вариант1.gif" alt="вариант1.gif" width="300">|
|<img src="https://github.com/elekpow/sflt-1/blob/main/вариант2.gif" alt="вариант2.gif" width="300">|<img src="https://github.com/elekpow/sflt-1/blob/main/вариант3.gif" alt="вариант3.gif" width="300">|

|Столбец 1|Столбец 2|
|:-|-:|
|<img src="https://github.com/elekpow/sflt-1/blob/main/вариант4.gif" alt="вариант4.gif" width="300">|<img src="https://github.com/elekpow/sflt-1/blob/main/вариант5.gif" alt="вариант5.gif" width="300">|



 ---

### Задание 2

- Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного [файла](https://github.com/netology-code/sflt-homeworks/blob/main/1/keepalived-simple.conf).
- Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
- Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
- Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
- На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html

### Выполнения задания 2

 ---

### Задание 3 *

- Изучите дополнительно возможность Keepalived, которая называется vrrp_track_file
- Напишите bash-скрипт, который будет менять приоритет внутри файла в зависимости от нагрузки на виртуальную машину (можно разместить данный скрипт в cron и запускать каждую минуту). Рассчитывать приоритет можно, например, на основании Load average.
- Настройте Keepalived на отслеживание данного файла.
- Нагрузите одну из виртуальных машин, которая находится в состоянии MASTER и имеет активный виртуальный IP и проверьте, чтобы через некоторое время она перешла в состояние SLAVE из-за высокой нагрузки и виртуальный IP переехал на другой, менее нагруженный сервер.
- Попробуйте выполнить настройку keepalived на третьем сервере и скорректировать при необходимости формулу так, чтобы плавающий ip адрес всегда был прикреплен к серверу, имеющему наименьшую нагрузку.
- На проверку отправьте получившийся bash-скрипт и конфигурационный файл keepalived, а также скриншоты логов keepalived с серверов при разных нагрузках

 ---

### Выполнения задания 3

