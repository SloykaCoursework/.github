
# Курсовая работа с использованием MPI .Net

Данная курсовая работа реализует систему для работы с базой данных MS SQL. В данной работе используются такие программы и сервисы как:
1. MPI - для распределения нагрузки на сервисы на стороне сервера;
2. MS SQL Server - для хранения данных;
3. .Net Core 5.0 - платформа для разработки клиентского и серверного приложения;
4. RabbitMQ - брокер сообщений для получения результата на клиентской стороне.

## Сетевая инфраструктура

Тестовый стенд состоит из 2 машин и 1 коммуникатора. Коммуникатор работает в подсети 192.168.1.0/24. К нему подключено 2 машины, на первой запущено еще 4 виртуальных машины, которые подключены к сети через сетевой мост.

Ip адреса в сети задает DHCP сервер в коммуникаторе. Он заранее настроен задавать определенные ip адреса конкретным MAC адресам.

```

                \                   /
                 \                 /
              +-----------------------+
              |        Router         |
              |      192.168.1.1      |
              +-----------------------+
                |                   |
                |                   |
                v                   v
 +-----------------+             +-----------------+
 |      Pc # 1     |             |      Pc # 2     |
 |   192.168.1.35  |             |   192.168.1.22  |
 +-----------------+             +-----------------+
       |
       |    +-----------------+
       +--> |      Samba      | 
       |    |   192.168.1.30  |
       |    +-----------------+
       |
       |    +-----------------+
       +--> |       Mpi1      | 
       |    |   192.168.1.20  |
       |    +-----------------+
       |
       |    +-----------------+
       +--> |       Mpi2      | 
       |    |   192.168.1.21  |
       |    +-----------------+
       |
       |    +-----------------+
       +--> |       Sql       | 
            |   192.168.1.25  |
            +-----------------+
           
```

## Первая машина

Данная машина выполняет роль сервера. Она позволяет запускать нужные службы и сервисы.

Технические характеристики:
```
ОС: Linux Ubuntu 24.04.1 LTS  
Процессор: AMD Ryzen 7 4700U (8 ядер, 4.1 ГГц)
ОЗУ: 24 Гб  
Ip: 192.168.1.35
```
  
#### На ней есть следующие виртуальные машины:  
1. samba - Служит для хранения файлов проекта и дальнейшего запуска их на всех машинах
```
ОС: Linux Ubuntu Server 24.04.1 LTS
Процессор: 1 ядро
ОЗУ: 2Гб
Ip: 192.168.1.30
```
  
2. mpi1 - Служит как один из потоков приложения сервера
```
ОС: Windows 10 Pro 22H2
Процессор: 2 ядра
ОЗУ: 4Гб
Ip: 192.168.1.20
```
  
3. mpi2 - Служит как один из потоков приложения сервера
```
ОС: Windows 10 Pro 22H2
Процессор: 2 ядра
ОЗУ: 4Гб
Ip: 192.168.1.21
```
  
4. sql - Служит для хранения данных в системе
```
ОС: Windows 10 Pro 22H2
Процессор: 2 ядра
ОЗУ: 2Гб
Ip: 192.168.1.25
```
  
#### Также запущены следующие docker контейнеры:  
  
1. RabbitMQ - Служит для отправки результатов обратно на клиент
```bash
docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4.0-management 
```

## Вторая машина

Данная машина выполняет роль компьютера пользователя (клиентской машины). На ней запускаеться клиентское консольное приложение и идет подключение к сервисам на серверах (Первая машина).

Технические характеристики:
```
ОС: Windows 10 Pro 22H2
Процессор: AMD Athlon 300U (2 ядра, 2.4 ГГц)
ОЗУ: 2 Гб
Ip: 192.168.1.22
```

В целях оптимизации нагрузки на данной машине также запущен Mpi клиент (smpd).

## Программная часть

Проект написан на .Net Core 5.0 и состоит из 3 репозиториев:
1. [ServerApp](https://github.com/SloykaCoursework/ServerApp) - Содержит инструкции по работе сервера;
```
Поднимает TCP/IP сервер
Подключается к базе данных
Обрабатывает в среде MPI запросы пользователя
Отправляет результат при помощи RabbitMQ
```
2. [ConsoleApp](https://github.com/SloykaCoursework/ConsoleApp) - Содержит инструкции по взаимодействию пользователя и данного сервиса
```
Ждет выбора пользователя
Подключаеться по TCP/IP к серверу
Ждет результат
```
3. [CourseWorkLibrary](https://github.com/SloykaCoursework/CourseWorkLibrary) - Содержит инструкции по взаимодействию сервера и клиента на уровне кода
```
Содержит классы задач, которые решаются сервером
Содержит базовые обработчики TCP/IP взаимодействия
```

Библиотека выполнена в формате библиотеки классов для .Net Core 5.0. Собираеться библиотека в Nuget пакет и загружаеться на [github](https://github.com/SloykaCoursework/CourseWorkLibrary/pkgs/nuget/CourseworkLibrary), откуда она импортируется в другие проекты.

## UML диаграммы

### Deployment Diagram
![Deployment_Diagram](https://github.com/user-attachments/assets/67eee62d-e6a3-4560-b1bc-5a0a3ee2a4d3)

### Sequence Diagram

#### Общая диаграмма последовательности
![s](https://github.com/user-attachments/assets/3db6e8bf-15d7-4faf-95de-0bb1381e221a)

#### Детализация процесса "Обработка запроса" на стороне MPI Сервер
![d](https://github.com/user-attachments/assets/24717e39-c41b-4748-8f4b-da6d22f19acd)

## Результаты выполнения

Посмотреть результаты выполнения можно по [ссылке](https://kairu-my.sharepoint.com/:x:/g/personal/kuznetsovmr_stud_kai_ru/EeJ2ZQ-EmaJLsq6IrAhx5scBMgcLY4L04g2ty1nq0PytpA).

Все результаты представлены в виде таблиц с подсчетом времени обработки и выполнения команды на стороне MPI сервера. 

> [!IMPORTANT]
> Данные о времени создания таблиц недействительны из-за ошибок сохранения данных в некоторых случаях.

![image](https://github.com/user-attachments/assets/274efdcf-85e8-4588-b12f-57b283cde089)

![image](https://github.com/user-attachments/assets/db010a0f-9ed1-423b-86be-869b298a6c53)

![image](https://github.com/user-attachments/assets/58756bd6-199d-4344-bdc5-b8b07f12bb84)

![image](https://github.com/user-attachments/assets/f7987459-1f1a-4c9b-a170-48c3f4397290)

![image](https://github.com/user-attachments/assets/d3d48fe4-057c-4764-baa5-7e247a3d6cf1)

## Вывод

Из результатов выполнения видно, что чем больше потоков работает над задачей, тем быстрее она решается. Но данное правило работает только на ресурсозатратные задачи (2, 3 и 4). В случае задач, не требующих больших мощностей для их решения (5 и 6), время наоборот растет с ростом числа потоков. 

Также видно, что избыточное число потоков, а именно 4, в среднем справляется либо также, либо лучше с поставленными задачами по сравнению с 2 потоками.

В ходе реализации курсовой работы я столкнулся с трудностями использования MPI.Net, связанными с плохой поддержкой этого продукта и отсутсвием какой-либо документации. В ходе работы выяснилось, что MPI не может запускать свои сервисы на разных машинах, если у них не совпадают пользователи и версии сборок ОС. Для более комфортного решения проблемы паралельной обработки SQL запросов лучше использовать распределение нагрузки между машинами по средствам Hangfire.
