# Домашнее задание к занятию "`Disaster recovery и Keepalived`" - `Савин Алексей`

### Задание 1
* Дана схема для Cisco Packet Tracer, рассматриваемая в лекции.
* На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)
* Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).
* Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.
* На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.  

### Решение 1
* [Схема](https://github.com/AI-Savin/netology_hw_keepalived/blob/main/img/hsrp_advanced_savin.pkt) в формате pkt.  
* Скриншот процесса настройки маршрутизатора:  
![Router](https://github.com/AI-Savin/netology_hw_keepalived/blob/main/img/router_VRRP.jpg)
* Cкриншот отправки пакета на сервер и обратно при разорванной связи  
![HSRP](https://github.com/AI-Savin/netology_hw_keepalived/blob/main/img/HSRP.jpg)    
  
---

### Задание 2
* Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного файла.
* Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
* Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
* Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
* На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html

### Решение 2

* изменил содержание дефолтной веб страницы на MASTER server nginx:
```
<!DOCTYPE html>
<html>
<body>
<h1>Server MASTER 192.168.3.16</h1>
</body>
</html>
```
* изменил содержание дефолтной веб страницы на BACKUP server nginx:
```
<!DOCTYPE html>
<html>
<body>
<h1>Server BACKUP 192.168.3.17</h1>
</body>
</html>
```
* bash-скрипт проверки доступности http хоста по порту 80 и проверки наличия index.html файла
```
#!/bin/bash
if [[ $(netstat -tuln | grep LISTEN | grep :80) ]] && [[ -f /var/www/html/index.nginx-debian.html ]]; then
   exit 0
else
   exit 1
fi
```
* конфигурационный файл keepalived
```
vrrp_script check_script {
      script "/home/user/check_nginx.sh"
      interval 3
      fall 2       # require 2 failures for KO
      rise 2       # require 2 successes for OK
}

vrrp_instance VI_1 {
        state MASTER
        interface enp0s3
        virtual_router_id 25
        priority 255
        advert_int 1

        virtual_ipaddress {
              192.168.3.25/24
        }

        track_script {
                   check_script
                }
}
```
* Проверил работу MASTER сервера

![master_server](https://github.com/AI-Savin/netology_hw_keepalived/blob/main/img/master_server.jpg)  
* Проверил доступность MASTER server по плавающему IP

![master_server_floating](https://github.com/AI-Savin/netology_hw_keepalived/blob/main/img/master_server_floating.jpg) 

* скриншот наличия плавающего IP на сетевом интерфейсе MASTER сервера, с последующей остановкой **nginx** и проверкой отсутствия плавающего IP

![master_stop](https://github.com/AI-Savin/netology_hw_keepalived/blob/main/img/master_stop.jpg)  

* скриншот появления плавающего IP на сетевом интерфейсе BACKUP сервера  

![backup_ip](https://github.com/AI-Savin/netology_hw_keepalived/blob/main/img/backup_ip_a.jpg)  

* скриншот достуности BACKUP сервера по плавающему IP

![backup_floating](https://github.com/AI-Savin/netology_hw_keepalived/blob/main/img/Backup_floating.jpg)  


