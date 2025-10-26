# Лабораторная работа №3 — Облачные сети

**Студент:** Irina Cristeva  
**Вариант:** kXX  
**Дата:** 26.10.2025  

---

## 1. Описание лабораторной работы
В данной лабораторной работе изучается процесс создания и настройки облачной сети в AWS.  
Студент создаёт виртуальную сеть (VPC), настраивает подсети, таблицы маршрутов, Internet Gateway и NAT Gateway, а также взаимодействие между публичным веб-сервером и приватной базой данных.

<img width="1273" height="1342" alt="image" src="https://github.com/user-attachments/assets/4a2d6e0b-cbb9-4ba3-b8bb-3e4f7ccf22da" />
Подпись: “Выбран регион Frankfurt, открыт сервис VPC”.
---

## 2. Постановка задачи
- Создать VPC и разделить её на публичную и приватную подсети.  
- Настроить маршрутизацию и шлюзы для выхода в интернет.  
- Создать Security Groups для контроля трафика.  
- Развернуть EC2-инстансы: веб-сервер, сервер базы данных и Bastion Host.  
- Проверить корректность работы сети и взаимодействие инстансов.


<img width="1766" height="1338" alt="image" src="https://github.com/user-attachments/assets/91464ba1-78a2-46fb-ae32-e66fc4587b74" />

---

## 3. Цель и основные этапы работы
**Цель:** Научиться создавать изолированные облачные сети и настраивать безопасное взаимодействие ресурсов.  

**Основные этапы:**
1. Подготовка среды AWS.  
2. Создание VPC.  
3. Создание подсетей (публичной и приватной).  
4. Настройка таблиц маршрутов.  
5. Создание Internet Gateway и NAT Gateway.  
<img width="1767" height="1335" alt="image" src="https://github.com/user-attachments/assets/4c492e5d-16fd-48b8-aa6f-ac06378f048b" />
Подпись: “Internet Gateway student-igw-8 прикреплён к VPC student-vpc-8”.
<img width="2559" height="1337" alt="image" src="https://github.com/user-attachments/assets/1d462f59-e5d8-46d5-9ad9-b049f2f23a36" />
Публичная подсеть → подпись: “Public Subnet student-vpc-8 создана, CIDR 10.8.1.0/24”.

Приватная подсеть → подпись: “Private Subnet student-vpc-8 создана, CIDR 10.8.2.0/24”.

Контрольный вопрос:

На этом этапе приватная подсеть изолирована от интернета, потому что ещё нет NAT Gateway.


<img width="1802" height="1339" alt="image" src="https://github.com/user-attachments/assets/44614a62-f089-4318-a511-e54f5da79102" />
Публичная таблица маршрутов с привязкой к public-subnet-8.

Подпись: “Публичная таблица маршрутов настроена с маршрутом 0.0.0.0/0 → IGW”.

<img width="1795" height="1325" alt="image" src="https://github.com/user-attachments/assets/eb549f7f-7597-4186-8583-2606412ce7c5" />
Приватная таблица маршрутов с привязкой к private-subnet-8.

Подпись: “Приватная таблица маршрутов создана для private-subnet-8”.




<img width="1802" height="1343" alt="image" src="https://github.com/user-attachments/assets/441a0bbb-d099-4e26-84f2-8a906f32c849" />
Приватная таблица маршрутов с маршрутом 0.0.0.0/0 → NAT Gateway

Подпись: “Приватная подсеть теперь может выходить в интернет через NAT Gateway, оставаясь недоступной извне”.



7. Настройка Security Groups.  

<img width="2536" height="1339" alt="image" src="https://github.com/user-attachments/assets/f8bdd40c-f21b-4441-84e6-3f381972ad96" />

8. Создание EC2-инстансов и скриптов установки.  
9. Проверка работы сети и подключения.  
10. Использование Bastion Host для доступа к приватной подсети.  

---

## 4. Практическая часть

### Шаг 1. Подготовка среды
- Вошла в AWS Management Console, выбрала регион **Frankfurt (eu-central-1)**.  
- Перешла в сервис **VPC**.

*Скриншот:*  
![Регион AWS](https://user-images.githubusercontent.com/твой_id/скриншот1.png)

---

### Шаг 2. Создание VPC
- Имя: `student-vpc-kXX`  
- CIDR: `10.(k%30).0.0/16`  
- Tenancy: Default  

*Скриншот создания VPC:*  
![Создание VPC](https://user-images.githubusercontent.com/твой_id/скриншот2.png)

**Контрольный вопрос:**  
Маска /16 — первые 16 бит адреса сети фиксированы, остальные 16 бит — для хостов. /8 слишком большой диапазон, неэффективно.

---

### Шаг 3. Создание Internet Gateway (IGW)
- Имя: `student-igw-kXX`  
- Прикрепила к VPC `student-vpc-kXX`

*Скриншот IGW:*  
![Internet Gateway](https://user-images.githubusercontent.com/твой_id/скриншот3.png)

---

### Шаг 4. Создание подсетей
#### 4.1 Публичная подсеть
- Имя: `public-subnet-kXX`  
- CIDR: `10.(k%30).1.0/24`  
- Availability Zone: `eu-central-1a`  

*Скриншот:*  
![Public Subnet](https://user-images.githubusercontent.com/твой_id/скриншот4.png)

#### 4.2 Приватная подсеть
- Имя: `private-subnet-kXX`  
- CIDR: `10.(k%30).2.0/24`  
- Другая зона доступности  

*Скриншот:*  
![Private Subnet](https://user-images.githubusercontent.com/твой_id/скриншот5.png)

**Контрольный вопрос:**  
Пока нет NAT Gateway и маршрута к интернету — подсеть приватная.

---

### Шаг 5. Таблицы маршрутов
#### 5.1 Публичная таблица
- Имя: `public-rt-kXX`  
- Route: `0.0.0.0/0 → IGW`  
- Привязка к `public-subnet-kXX`  

*Скриншот:*  
![Public Route Table](https://user-images.githubusercontent.com/твой_id/скриншот6.png)

#### 5.2 Приватная таблица
- Имя: `private-rt-kXX`  
- Пока нет NAT Gateway, маршрутов наружу нет  
- Привязка к `private-subnet-kXX`  

*Скриншот:*  
![Private Route Table](https://user-images.githubusercontent.com/твой_id/скриншот7.png)

---

### Шаг 6. NAT Gateway
1. Создала Elastic IP.  
2. Создала NAT Gateway в публичной подсети с Elastic IP.  
3. Добавила маршрут `0.0.0.0/0 → NAT Gateway` в приватную таблицу.

*Скрин:*  
![NAT Gateway](https://user-images.githubusercontent.com/твой_id/скриншот8.png)

---

### Шаг 7. Security Groups
- `web-sg-kXX`: HTTP 80, HTTPS 443 для всех  
- `bastion-sg-kXX`: SSH 22 только с моего IP  
- `db-sg-kXX`: MySQL 3306 только с web-sg и bastion-sg; SSH 22 только с bastion

*Скрин:*  
![Security Groups](https://user-images.githubusercontent.com/твой_id/скриншот9.png)

**Контрольный вопрос:**  
Bastion Host нужен для безопасного доступа к приватной подсети.

---

### Шаг 8. EC2-инстансы
- **web-server** — публичная подсеть, HTTP/PHP  
- **db-server** — приватная подсеть, MariaDB  
- **bastion-host** — публичная подсеть, MariaDB client  

*Скрин:*  
![EC2 Instances](https://user-images.githubusercontent.com/твой_id/скриншот10.png)

---

### Шаг 9. Проверка работы
- Web-server → страница PHP открывается  
- Подключение к bastion → ping google.com успешен  
- Через bastion → подключение к db-server и MySQL успешно

*Скрины:*  
![Web Server](https://user-images.githubusercontent.com/твой_id/скриншот11.png)  
![SSH Bastion](https://user-images.githubusercontent.com/твой_id/скриншот12.png)  

---

### Шаг 10. Дополнительно: подключение через Bastion
```bash
eval "$(ssh-agent -s)"
ssh-add student-key-kXX.pem
ssh -A -J ec2-user@<Bastion-Host-IP> ec2-user@<DB-Server-IP>

