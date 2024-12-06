# Курсовая работа с использованием MPI .Net

Данная курсовая работа реализует систему для работы с распределенной базой данных PostgreSql. В данной работе используются такие программы и сервисы как:
1. MPI - для распределения нагрузки на сервисы на стороне сервера;
2. PostgreSql - для хранения данных;
3. .Net Core 5.0 - платформа для разработки клиентского и серверного приложения;
4. RabbitMQ - брокер сообщений для получения результата на клиентской стороне.

## Сетевая инфраструктура

Тестовый стенд состоит из 2 машин и 1 коммуникатора. Коммуникатор работает в подсети 192.168.1.0/24. К нему подключено 2 машины, на первой запущено еще 4 виртуальных машины, которые подключены к сети через сетевой мост.

Ip адреса в сети задает DHCP сервер в коммуникаторе. Он заранее настроен задавать определенные ip адреса конкретным MAC адресам.

```

              +-----------------------+
              |        Router         |
              |      192.168.1.1      |
              +-----------------------+
                |                   |
                |                   |
                v                   v
 +-----------------+             +-----------------+
 |      Pc # 1     |             |      Pc # 2     |
 |   192.168.1.35  |             |   192.168.1.    |
 +-----------------+             +-----------------+
       |
       |    +-----------------+
       +--> |      Samba      | 
       |    |   192.168.1.    |
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
       +--> |   Sql_Cluster   | 
            |   192.168.1.    |
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
1. samba 
```
ОС: Linux Ubuntu Server 24.04.1 LTS
Процессор: 1 ядро
ОЗУ: 2Гб
Ip: 192.168.1.
```
  
2. mpi1  
```
ОС: Windows 10
Процессор: 2 ядра
ОЗУ: 4Гб
Ip: 192.168.1.20
```
  
3. mpi2  
```
ОС: Windows 10
Процессор: 2 ядра
ОЗУ: 4Гб
Ip: 192.168.1.21
```
  
4. sql_cluster  (пока нет)
```
ОС: -
Процессор: - ядра
ОЗУ: -Гб
Ip: 192.168.1.
```
  
#### Также запущены следующие docker контейнеры:  
  
1. RabbitMQ  
``` bash
docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4.0-management 
```

## Вторая машина

Данная машина выполняет роль компьютера пользователя (клиентской машины). На ней запускаеться клиентское консольное приложение и идет подключение к сервисам на серверах (Первая машина).

Технические характеристики:
```
ОС: Windows 10 Pro 22H2
Процессор: AMD Athlon 300U (4 ядра, 2.4 ГГц)
ОЗУ: 12 Гб
Ip: 192.168.1.
```

В целях оптимизации нагрузки на данной машине также запущен Mpi клиент (smpd), но работать он будет в меньших обьемах.

