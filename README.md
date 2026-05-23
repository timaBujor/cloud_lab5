# Лабораторная работа №5  
# Облачные базы данных. Amazon RDS, DynamoDB

---

## Цель работы

Целью работы является знакомство с сервисами Amazon RDS (Relational Database Service) и Amazon DynamoDB, а также получение практических навыков:

- создание и настройка экземпляров реляционных баз данных в AWS;
- работа с Amazon RDS;
- понимание концепции Read Replicas;
- подключение к базе данных через EC2;
- выполнение CRUD-операций;
- знакомство с NoSQL базами данных DynamoDB.

---

# Шаг 1. Подготовка среды (VPC / Subnets / Security Groups)

Создана VPC:

`project-vpc`

Внутри VPC были созданы:
- 2 публичные подсети;
- 2 приватные подсети;
- подсети расположены в разных Availability Zones (AZ).

<p align="center">
  <img src="https://github.com/user-attachments/assets/a53e266a-affb-4dee-a153-0ee0d395a894" alt="vpc" width="900"/>
</p>

---

## Создание Security Group для приложения

Создана группа безопасности:

`web-security-group`

<p align="center">
  <img src="https://github.com/user-attachments/assets/e8283b64-9a3b-4008-97f9-e3daeba1dffc" alt="security_group" width="900"/>
</p>

### Входящий трафик

| Тип | Порт | Источник |
|---|---|---|
| HTTP | 80 | Anywhere |
| SSH | 22 | My IP / Anywhere |

---

## Создание Security Group для базы данных

Создана группа безопасности:

`db-mysql-security-group`

<p align="center">
  <img src="https://github.com/user-attachments/assets/bd28ce03-331c-4068-8bc1-de4ad1316b34" alt="security_group1" width="900"/>
</p>

### Входящий трафик

| Тип | Порт | Источник |
|---|---|---|
| MySQL/Aurora | 3306 | web-security-group |

---

## Исходящий трафик

В `web-security-group` добавлено правило:

| Тип | Порт | Назначение |
|---|---|---|
| MySQL/Aurora | 3306 | db-mysql-security-group |

---

# Шаг 2. Развертывание Amazon RDS

Открыта консоль:

`Amazon Aurora and RDS`

---

## Создание DB Subnet Group

Создана группа подсетей:

`project-rds-subnet-group`

<p align="center">
  <img src="https://github.com/user-attachments/assets/c26c2ebc-0e40-4962-9073-0fc5e8e4d4f7" alt="subnet" width="900"/>
</p>

В группу добавлены:
- 2 приватные подсети;
- из разных Availability Zones.

---

## Контрольный вопрос

### Что такое Subnet Group и зачем она нужна?

### Ответ

DB Subnet Group — это набор подсетей внутри VPC, который используется Amazon RDS для размещения экземпляров базы данных.

Subnet Group необходима для:
- обеспечения отказоустойчивости;
- размещения БД в нескольких Availability Zones;
- изоляции базы данных в приватных подсетях;
- повышения безопасности.

Без DB Subnet Group Amazon RDS не сможет корректно развернуть базу данных внутри VPC.

---

# Создание экземпляра базы данных Amazon RDS

Выбран режим:

`Standard Create`

---

## Параметры базы данных

| Параметр | Значение |
|---|---|
| Engine type | MySQL |
| Version | MySQL 8.0.42 |
| Template | Free tier |
| Deployment | Single-AZ |
| DB identifier | project-rds-mysql-prod |
| Username | admin |
| Instance class | db.t3.micro |
| Storage type | gp3 |
| Storage size | 20 GB |
| Autoscaling | Enabled |
| Max storage | 100 GB |

---

## Настройки подключения

| Параметр | Значение |
|---|---|
| Connect to EC2 | No |
| VPC | project-vpc |
| DB subnet group | project-rds-subnet-group |
| Public access | No |
| Security group | db-mysql-security-group |

---

## Additional configuration

| Параметр | Значение |
|---|---|
| Initial database | project_db |
| Automated backups | Enabled |
| Encryption | Disabled |
| Auto minor upgrades | Disabled |

<p align="center">
  <img src="https://github.com/user-attachments/assets/c8a03570-cf37-47be-b3f7-5262cf88251e" alt="database" width="900"/>
</p>

---

После завершения создания статус базы данных изменился на:

`Available`

Был скопирован Endpoint базы данных для последующего подключения.

---

# Шаг 3. Создание EC2 для подключения к RDS

Создан экземпляр EC2 в публичной подсети VPC.

Для виртуальной машины использована группа безопасности:

`web-security-group`

<p align="center">
  <img src="https://github.com/user-attachments/assets/acc4987f-6022-42df-ab64-cb01f86ca231" alt="launch" width="900"/>
</p>

---

## Установка MySQL клиента

На EC2 был установлен MariaDB/MySQL client.

Используемая команда:

`dnf install -y mariadb105`

---

# Шаг 4. Подключение к базе данных и выполнение CRUD операций

Подключение к EC2 выполнено по SSH.

После этого выполнено подключение к Amazon RDS через MySQL client.

---

## Подключение к RDS

Используемая команда:

`mysql -h <RDS_ENDPOINT> -u admin -p`

---

## Выбор базы данных

Используемая команда:

`USE project_db;`

<p align="center">
  <img src="https://github.com/user-attachments/assets/785f6aff-10ef-4254-ad4e-50f4f7a9bb51" alt="connect_db" width="900"/>
</p>

---

# Создание таблиц

Созданы две таблицы со связью one-to-many:

- categories
- todos

---

## Связь таблиц

`todos.category_id -> categories.id`

<p align="center">
  <img src="https://github.com/user-attachments/assets/4bb4e51b-2351-4332-b38d-f520a2f9415a" alt="create_table" width="900"/>
</p>

---

## Наполнение таблиц

В каждую таблицу добавлено минимум по 3 записи.

<p align="center">
  <img src="https://github.com/user-attachments/assets/aee79757-d630-4a25-be37-79a204f8a63d" alt="table_data" width="900"/>
</p>

---

## Выполнение SELECT и JOIN запросов

Были выполнены:
- SELECT запросы;
- JOIN между таблицами;
- выборка данных по категориям.

<p align="center">
  <img src="https://github.com/user-attachments/assets/0aab24ed-a94e-4613-a6ef-a4bfaf7d2fdb" alt="select" width="900"/>
</p>

---

# Шаг 5. Создание Read Replica

Для основного экземпляра RDS была создана Read Replica.

---

## Параметры Read Replica

| Параметр | Значение |
|---|---|
| Identifier | project-rds-mysql-read-replica |
| Instance class | db.t3.micro |
| Storage type | gp3 |
| Public access | No |
| Security group | db-mysql-security-group |

---

После создания реплика получила статус:

`Available`

Был получен отдельный endpoint для подключения.

<p align="center">
  <img src="https://github.com/user-attachments/assets/ee9b4c3f-6f28-497e-abc9-778d2d4b763b" alt="replica" width="900"/>
</p>

---

# Подключение к Read Replica

С EC2 выполнено подключение к read replica и выполнены SELECT-запросы.

---

## Контрольный вопрос

### Какие данные отображаются на реплике? Почему?

### Ответ

На Read Replica отображаются те же данные, что и на основном экземпляре базы данных.

Это происходит потому, что реплика автоматически синхронизируется с master instance посредством асинхронной репликации.

---

## Проверка INSERT/UPDATE

Была выполнена попытка выполнить INSERT и UPDATE запросы на Read Replica.

<p align="center">
  <img src="https://github.com/user-attachments/assets/e1f7e11d-33bc-40b4-9625-7013972a64b5" alt="insert" width="900"/>
</p>

---

## Контрольный вопрос

### Получилось ли выполнить запись на Read Replica? Почему?

### Ответ

Нет, запись выполнить не удалось.

Read Replica предназначена только для операций чтения (`SELECT`).

---

## Проверка репликации

На основном экземпляре была добавлена новая запись.

<p align="center">
  <img src="https://github.com/user-attachments/assets/f583fe4f-2085-4bb8-9eda-fee298ecbcb4" alt="add_data" width="900"/>
</p>

После этого выполнен SELECT на Read Replica.

<p align="center">
  <img src="https://github.com/user-attachments/assets/95e01f74-4080-45a1-b3c0-af16a862d3dd" alt="check_db" width="900"/>
</p>

---

## Контрольный вопрос

### Появилась ли новая запись на реплике? Почему?

### Ответ

Да, новая запись появилась спустя небольшое время из-за асинхронной репликации.

---

## Контрольный вопрос

### Для чего нужны Read Replicas?

### Ответ

Read Replicas используются для:
- масштабирования чтения;
- снижения нагрузки на master instance;
- повышения производительности;
- отказоустойчивости.

---

# Шаг 6. Подключение приложения к базе данных

Разработано простое CRUD веб-приложение.

<p align="center">
  <img src="https://github.com/user-attachments/assets/d2684b3e-997a-44b2-b208-5fdabea737c0" alt="web" width="700"/>
</p>

---

## Архитектура подключения

### Master instance:
- INSERT
- UPDATE
- DELETE

### Read Replica:
- SELECT

---

# DynamoDB

Дополнительно изучен сервис Amazon DynamoDB:
- NoSQL storage;
- key-value architecture;
- высокая масштабируемость;
- serverless подход.

---

# Список использованных источников

1. AWS RDS Documentation  
2. AWS DynamoDB Documentation  
3. AWS EC2 Documentation  
4. AWS Security Groups Documentation  
5. AWS Read Replicas Documentation

---

# Вывод

В ходе лабораторной работы были изучены:
- Amazon RDS;
- Read Replicas;
- CRUD операции;
- подключение EC2 к RDS;
- масштабирование чтения;
- DynamoDB.

Amazon RDS позволяет быстро развертывать масштабируемые и отказоустойчивые облачные базы данных.
