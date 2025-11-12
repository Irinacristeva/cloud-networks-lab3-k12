Лабораторная работа №3 - Облачные сети

Студент: Irina Cristeva  
Вариант:8  
Дата: 26.10.2025  

Цель и основные этапы работы

Основная цель - создать отказоустойчивую и безопасную сетевую среду. Основные этапы включали:

Этапы работы:
1.Создание VPC и подсетей.
2.Настройка Internet Gateway (IGW) и NAT Gateway.
3.Настройка таблиц маршрутизации для публичного и приватного трафика.
4.Конфигурация групп безопасности (Security Groups) для каждого слоя.
5.Запуск EC2-инстансов (Bastion Host, Web Server, DB Server) в соответствующих подсетях.
6.Проверка работоспособности Web Server и NAT Gateway.


1) Создание VPC и подсетей
Создаю VPC с CIDR блоком 10.8.0.0/16 (student-vpc-8). 
<img width="2174" height="1101" alt="image" src="https://github.com/user-attachments/assets/26dba0e9-5325-481a-9045-7ec6afa268fb" />


Создаю две подсети:
Публичная подсеть (public-subnet-8): CIDR 10.8.1.0/24. В ней размещены Bastion Host и Web Server.

Контрольный вопрос: Является ли подсеть "публичной" на данный момент? Почему?
На данный момент подсеть еще не является публичной, потому что не выполнены 2 главных условия еще - это маршрутизация (без маршрута (0.0.0.0/0) в таблице подсети, указывающего на IGW, трафик не знает, как покинуть VPC и выйти в инет) и назначение IP (без публичного IP - нет возможности сопоставить входящий интернет-трафик с приватным адресом инстанса)

Приватная подсеть (private-subnet-8): CIDR 10.8.2.0/24. В ней размещен DB Server.
<img width="2525" height="619" alt="image" src="https://github.com/user-attachments/assets/c83ccb02-d5f2-48ac-a1b2-d617aa604bd5" />

Контрольный вопрос: Является ли подсеть "приватной" на данный момент? Почему?
На данный момент подсеть является приватной, потому что ее трафик не направляется напрямую в инет через IGW.

2) Настраиваю Internet Gateway и NAT Gateway
Internet Gateway (IGW) был создан и присоединен к VPC (student-igw-8).
<img width="2162" height="863" alt="image" src="https://github.com/user-attachments/assets/55400edf-fd0b-4a06-9f9d-20193822140b" />

NAT Gateway был создан в публичной подсети с привязанным Elastic IP-адресом.
<img width="2173" height="302" alt="image" src="https://github.com/user-attachments/assets/8b469a79-8721-429d-acde-695a18814f95" />

Контрольный вопрос: Как работает NAT Gateway?
NAT Gateway создает временные записи для исходящих запросов, но отклоняет любые нежелательные входящие соединения, обеспечивая безопасность приватной подсети.

3) Настраиваю таблицы маршрутизации
Создавала и настраивала две таблицы маршрутизации (Route Tables):

Публичная таблица маршрутизации (public-rt-8):
Ассоциирована с public-subnet-8.
<img width="1268" height="1088" alt="image" src="https://github.com/user-attachments/assets/6c105eb4-a108-4f50-887b-d653e5ebd809" />

Маршрут по умолчанию (0.0.0.0/0) направлен на IGW (igw-0fffb982b884...), что обеспечивает доступ к Интернету.
Маршрут 10.8.0.0/16 направлен на local.
<img width="1269" height="485" alt="image" src="https://github.com/user-attachments/assets/3ee331b7-e130-4497-9082-1decefd62913" />

Контрольный вопрос: Зачем необходимо привязать таблицу мартшрутов к подсети? 
Потому что без привязки - Route Tables будет просто как список правил и не действует пока явно не будет ассоциирован с конкретной подсетью.

Приватная таблица маршрутизации (private-rt-8):
Ассоциирована с private-subnet-8.
<img width="1265" height="1132" alt="image" src="https://github.com/user-attachments/assets/085d0b4d-9137-4f49-8599-2e94c1296723" />

Маршрут по умолчанию (0.0.0.0/0) направлен на NAT Gateway, что позволяет инстансам приватной подсети инициировать исходящие подключения в Интернет.
<img width="1272" height="1022" alt="image" src="https://github.com/user-attachments/assets/40209a31-b38f-4d6f-aaef-ce3db03c087e" />

4) Конфигурирую группы безопасности (Security Groups)
Создала три группы безопасности:
<img width="1270" height="725" alt="image" src="https://github.com/user-attachments/assets/1cfeddea-7654-4a9f-86ab-fccd0bb6c402" />

bastion-sg-8: Разрешаю входящий SSH (порт 22) трафик. Для диагностики временно установила Source: 0.0.0.0/0.
<img width="1268" height="1001" alt="image" src="https://github.com/user-attachments/assets/fcc8f4a1-3961-442b-bf89-9024dd646e53" />


web-sg-8: Разрешаем входящий HTTP (порт 80) трафик с 0.0.0.0/0 (для доступа из Интернета).
<img width="1268" height="1004" alt="image" src="https://github.com/user-attachments/assets/ba571dcb-6bb6-4e5d-b1c8-6d1a5ef99b87" />


db-sg-8: Разрешаем входящий MySQL (порт 3306) трафик только от web-sg-8, обеспечивая изоляцию базы данных.
<img width="1275" height="1069" alt="image" src="https://github.com/user-attachments/assets/cf42c7d3-31f2-4853-9675-b5de8aff825f" />

Контрольный вопрос: Что такое Bastion Host и зачем он нужен в архитектуре с приватными подсетями?
Bastion Host является единственным, защищенным сервером-посредником (прыжковый сервер), который находится в публичной подсети и имеет публичный IP-адрес. Его основная цель - безопасный удаленный доступ к ресурсам в приватных подсетях, которые не имеют публичного IP и не должны быть доступны из Интернета.
5) Запуск EC2-инстансов
Были запущены три инстанса, все типа t3.micro:
<img width="1266" height="473" alt="image" src="https://github.com/user-attachments/assets/61bb8c14-fa84-40cb-9920-ffcf8c5ed5f2" />

Bastion Host (bastion-host-k8): Запущен в public-subnet-8.
<img width="1238" height="758" alt="image" src="https://github.com/user-attachments/assets/fdfc6d34-efe3-4928-b298-19881e3a6ca6" />

Web Server (web-server-k8): Запущен в public-subnet-8, имел публичный IP-адрес 3.123.32.68.
<img width="1221" height="709" alt="image" src="https://github.com/user-attachments/assets/4b617eae-edd7-40fb-942a-616468c22c95" />

DB Server (db-server-k8): Запущен в private-subnet-8.
<img width="1253" height="806" alt="image" src="https://github.com/user-attachments/assets/2df96ff3-6860-4b18-96fb-0813d8a07db8" />

6) Проверка работоспособности
Проверка Web Server (HTTP): После успешного запуска Web Server и настройки Apache (HTTPD), доступ к публичному IP-адресу (http://3.123.32.68) был успешно получен, что подтверждает корректную работу IGW и web-sg-8.

Проверка NAT Gateway (Ping): После подключения к DB Server через Bastion Host, была выполнена команда ping -c 4 google.com. Тест успешно пройден, что подтверждает, что приватная подсеть имеет выход в Интернет через NAT Gateway.

Вывод

Все необходимые компоненты трехуровневой архитектуры VPC были успешно созданы и настроены согласно заданию (VPC, подсети, IGW, NAT GW, таблицы маршрутизации, группы безопасности). Маршрутизация трафика (публичный через IGW, приватный через NAT GW) настроена корректно, что подтверждается проверками Route Tables.

