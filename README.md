
Задание:
взято от сюда: https://docs.google.com/document/d/1H2LePo4ThiBJZGYnwD0744TL5NDQb-xdSfyvEYaDWyU/edit#

! текст (задание) походу меняют для разных испытуемых.

Исходное задание:

База данных (**mongodb**):
Создать коллекцию в которой хранятся чеки:
- номер чека (int)
- номер кассы (int)
- дата и время чека (время на чеке)
- сумма чека
Создать коллекцию в которой хранятся кассы:
- номер кассы (int)
- часовой пояс кассы (можно int – отклонение от GMT) 
Создать 3 кассы (GMT-8, GMT 0, GMT+8)
Создать сервис очередей (RabbitMQ)
Через сервис очередей заполнить 365 дней тестовыми данными исходя из следующих условий:
- 400-800 чеков в день
- 80-150 сумма чека
- продажи идут с 6 до 22 местного времени
Бэк:
Сервер возвращает на фронт JSON
Все чеки отображаем по времени GMT+3
На фронте ajax – выводим в любом виде (без красоты) следующие страницы
 Шахматка – строки месяцы, столбцы кассы. На пересечении сумма чеков. 
 При нажатии на месяц – строки дни + дни недели, столбцы кассы. На пересечении сумма чеков.
 При нажатии на день – чеки этого дня.
 



**Описание, как делаем, как все работает:**

Для менеджера очередей используем RabbitMQ, для СУБД NoSQL MongoDB.
- Подымаем VPS, ставим Ubuntu, Apache, PHP, библиотеки для PHP.
- Подымаем RabbitMQ подключение в локалке (localhost) под пользователем quest.
- Подымаем MongoDB, открываем доступ для внешки (так удобно работать с любой машины), 
но! делаем отдельный логин, с доступом только в нашу базу, переменная для подключения 
к MongoDB хранится в файле **login_vars.php** здесь этот файл отсутствует (в репозиторий не закачан),
Без отдельного внешнего логина (открытый доступ с внешки), за несколько дней базу угоняют и вымогают биткоины!
Пример, формат файла:
```
	$MongoDB_Login = "UserName:UserPassword@localhost:27017";
```

- Проверяем и убеждаемся что все сервисы подняты (RabbitMQ, Apache, PHP, MongoDB) и работают.
- В MongoDB создаем БД **TestSales**, создаем коллекции **RetailCollection** и **SalesCollection**, 
пользователь из **login_vars.php** имеет доступ только к **TestSales**, админских прав к **MongoDB** у него нет.
- Для наполнения БД чеками, первым делом в CLI (в комнадной строке) запускаем ```consumer.php```, 
запуск как  ```php consumer.php```. Consumer это приемка и обработка чеков из очереди RabbitMQ.
- После запуска consumer можно запускать генератор чеков, так же в CLI, пример: 
```
php cash.php 2019 1 1 America/Anchorage```

Параметры:
```
	CLI: php cash.php Year ShopNumber CashNumber TimeZone
		Year:          2016 .. 2020, default 1
		ShopNumber:    1 .. 99, default 1
		CashNumber:      1 .. 99, default 1
		TiemZone: TiemZoneName (Linux Timezone ID), default "Europe/Samara"
		https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
```

имеется исполняемый файл заготовок для закачки чеков в БД сразу за несколько лет, 2016, 2017, 2018, 2019 годы, занимает это несколько минут,
файл **GenChecks.sh**


Пример работы cash.php
```
   2019-09-04 20:50:56  Создан чек: 105   Shop:1;  Cash:1;  GoodsCount:4;  BuySum:53453;  Sum:86890;
   2019-09-04 20:59:30  Создан чек: 106   Shop:1;  Cash:1;  GoodsCount:5;  BuySum:50334;  Sum:81600;
   2019-09-04 21:08:04  Создан чек: 107   Shop:1;  Cash:1;  GoodsCount:10;  BuySum:98441;  Sum:162010;
   2019-09-04 21:16:38  Создан чек: 108   Shop:1;  Cash:1;  GoodsCount:5;  BuySum:67137;  Sum:106710;
   2019-09-04 21:25:12  Создан чек: 109   Shop:1;  Cash:1;  GoodsCount:6;  BuySum:28984;  Sum:56370;
   2019-09-04 21:33:46  Создан чек: 110   Shop:1;  Cash:1;  GoodsCount:13;  BuySum:108707;  Sum:168900;
   2019-09-04 21:42:20  Создан чек: 111   Shop:1;  Cash:1;  GoodsCount:6;  BuySum:74793;  Sum:113580;
   2019-09-04 21:50:54  Создан чек: 112   Shop:1;  Cash:1;  GoodsCount:3;  BuySum:31588;  Sum:55500;
   2019-09-04 21:59:28  Создан чек: 113   Shop:1;  Cash:1;  GoodsCount:2;  BuySum:19950;  Sum:33260;


Всего чеков: 1499
```

Пример работы consumer.php
```   Чек обработан: "CreateDT_Local":"2019-09-04 20:50:56","CreateDT_Stamp":1567659056,"TimeZoneName...
   Чек обработан: "CreateDT_Local":"2019-09-04 20:59:30","CreateDT_Stamp":1567659570,"TimeZoneName...
   Чек обработан: "CreateDT_Local":"2019-09-04 21:08:04","CreateDT_Stamp":1567660084,"TimeZoneName...
   Чек обработан: "CreateDT_Local":"2019-09-04 21:16:38","CreateDT_Stamp":1567660598,"TimeZoneName...
   Чек обработан: "CreateDT_Local":"2019-09-04 21:25:12","CreateDT_Stamp":1567661112,"TimeZoneName...
   Чек обработан: "CreateDT_Local":"2019-09-04 21:33:46","CreateDT_Stamp":1567661626,"TimeZoneName...
   Чек обработан: "CreateDT_Local":"2019-09-04 21:42:20","CreateDT_Stamp":1567662140,"TimeZoneName...
   Чек обработан: "CreateDT_Local":"2019-09-04 21:50:54","CreateDT_Stamp":1567662654,"TimeZoneName...
   Чек обработан: "CreateDT_Local":"2019-09-04 21:59:28","CreateDT_Stamp":1567663168,"TimeZoneName...
```

пример шахматки: http://151.80.14.150/  
(на время жизни VPS)
