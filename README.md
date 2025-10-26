# Лабораторная работа №3 — Облачные сети

**Студент:** Irina Cristeva  
**Вариант:** 8  
**Дата:** 26.10.2025  



## 1. Описание лабораторной работы
В данной лабораторной работе изучаю процесс создания и настройки облачной сети в AWS.  
Создаю виртуальную сеть (VPC), настраивает подсети, таблицы маршрутов, Internet Gateway и NAT Gateway, а также взаимодействие между публичным веб-сервером и приватной базой данных.

<img width="1273" height="1342" alt="image" src="https://github.com/user-attachments/assets/4a2d6e0b-cbb9-4ba3-b8bb-3e4f7ccf22da" />
Выбираю регион Frankfurt, открыт сервис VPC.


## 2. Постановка задачи
- Создать VPC и разделить её на публичную и приватную подсети.  
- Настроить маршрутизацию и шлюзы для выхода в интернет.  
- Создать Security Groups для контроля трафика.  
- Развернуть EC2-инстансы: веб-сервер, сервер базы данных и Bastion Host.  
- Проверить корректность работы сети и взаимодействие инстансов.


<img width="1766" height="1338" alt="image" src="https://github.com/user-attachments/assets/91464ba1-78a2-46fb-ae32-e66fc4587b74" />



## 3. Цель и основные этапы работы
**Цель:** Научиться создавать изолированные облачные сети и настраивать безопасное взаимодействие ресурсов.  

**Основные этапы:**
1. Подготовка среды AWS.  
2. Создание VPC.  
3. Создание подсетей (публичной и приватной).  
4. Настройка таблиц маршрутов.  
5. Создание Internet Gateway и NAT Gateway.  
<img width="1767" height="1335" alt="image" src="https://github.com/user-attachments/assets/4c492e5d-16fd-48b8-aa6f-ac06378f048b" />
Internet Gateway student-igw-8 прикреплён к VPC student-vpc-8
<img width="2559" height="1337" alt="image" src="https://github.com/user-attachments/assets/1d462f59-e5d8-46d5-9ad9-b049f2f23a36" />
Public Subnet student-vpc-8 создана, CIDR 10.8.1.0/24

Приватная подсеть → подпись: “Private Subnet student-vpc-8 создана, CIDR 10.8.2.0/24”.

Контрольный вопрос: На этом этапе приватная подсеть изолирована от интернета, потому что ещё нет NAT Gateway.


<img width="1802" height="1339" alt="image" src="https://github.com/user-attachments/assets/44614a62-f089-4318-a511-e54f5da79102" />
Публичная таблица маршрутов настроена с маршрутом 0.0.0.0/0 → IGW

<img width="1795" height="1325" alt="image" src="https://github.com/user-attachments/assets/eb549f7f-7597-4186-8583-2606412ce7c5" />

Подпись: “Приватная таблица маршрутов создана для private-subnet-8”.




<img width="1802" height="1343" alt="image" src="https://github.com/user-attachments/assets/441a0bbb-d099-4e26-84f2-8a906f32c849" />
Приватная подсеть теперь может выходить в интернет через NAT Gateway, оставаясь недоступной извне



7. Настройка Security Groups.  

<img width="2536" height="1339" alt="image" src="https://github.com/user-attachments/assets/f8bdd40c-f21b-4441-84e6-3f381972ad96" />

8. Создание EC2-инстансов и скриптов установки.  
9. Проверка работы сети и подключения.  
10. Использование Bastion Host для доступа к приватной подсети.  


