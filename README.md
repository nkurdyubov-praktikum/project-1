# project-1
Самостоятельный проект первого скрипта
# Витрина RFM

## 1.1. Выясните требования к целевой витрине.

Постановка задачи выглядит достаточно абстрактно - постройте витрину. Первым делом вам необходимо выяснить у заказчика детали. Запросите недостающую информацию у заказчика в чате.

Зафиксируйте выясненные требования. Составьте документацию готовящейся витрины на основе заданных вами вопросов, добавив все необходимые детали.

-----------

Название витрины dm_rfm_segments
Витрина располагается в схеме analysis
Витрина не обновляется

## 1.2. Изучите структуру исходных данных.

Полключитесь к базе данных и изучите структуру таблиц.

Если появились вопросы по устройству источника, задайте их в чате.

Зафиксируйте, какие поля вы будете использовать для расчета витрины.

-----------

Для расчета витрины будут использоваться следующие таблицы и поля:
orders.user_id
orders.payment
orders.status
orders.order_ts

## 1.3. Проанализируйте качество данных

Изучите качество входных данных. Опишите, насколько качественные данные хранятся в источнике. Так же укажите, какие инструменты обеспечения качества данных были использованы в таблицах в схеме production.

Данным скриптом проверил наличие дублей а так же незаполненных полей в таблице
```SQL
SELECT count(*), 
count(distinct order_id),
count(order_id), 
count(order_ts), 
count(user_id), 
count(bonus_payment), 
count(payment), 
count(cost), 
count(bonus_grant), 
count(status)
FROM production.orders o

```
Для таблицы production.orders следующие ограничения:
1) CHECK cost = payment + bonus_payment
2) PRIMARY KEY (order_id)
Для таблицы production.orderitems следующие ограничения:
1) CHECK (((discount >= (0)::numeric) AND (discount <= price)))
2) UNIQUE (order_id, product_id)
3) PRIMARY KEY (id)
4) CHECK ((price >= (0)::numeric))
5) CHECK ((quantity > 0))
6) FOREIGN KEY (order_id) REFERENCES production.orders(order_id)
7) (product_id) REFERENCES production.products(id)

В целом, используются следующие инсрументы обеспечения качества: Проверки, первичные ключи, внешние ключи, уникальные ключи

## 1.4. Подготовьте витрину данных

Теперь, когда требования понятны, а исходные данные изучены, можно приступить к реализации.

### 1.4.1. Сделайте VIEW для таблиц из базы production.**

Вас просят при расчете витрины обращаться только к объектам из схемы analysis. Чтобы не дублировать данные (данные находятся в этой же базе), вы решаете сделать view. Таким образом, View будут находиться в схеме analysis и вычитывать данные из схемы production. 

Напишите SQL-запросы для создания пяти VIEW (по одному на каждую таблицу) и выполните их. Для проверки предоставьте код создания VIEW.

```SQL
create view analysis.orderitems as 
select * from production.orderitems;
create view analysis.orders as 
select * from production.orders;
create view analysis.orderstatuses as 
select * from production.orderstatuses;
create view analysis.orderstatuslog as 
select * from production.orderstatuslog;
create view analysis.products as 
select * from production.products;
create view analysis.users as 
select * from production.users;
```

### 1.4.2. Напишите DDL-запрос для создания витрины.**

Далее вам необходимо создать витрину. Напишите CREATE TABLE запрос и выполните его на предоставленной базе данных в схеме analysis.

```SQL
create table analysis.dm_rfm_segments 
(user_id int,
 recency smallint,
 frequency smallint,
 monetary_value smallint)
```

### 1.4.3. Напишите SQL запрос для заполнения витрины

Наконец, реализуйте расчет витрины на языке SQL и заполните таблицу, созданную в предыдущем пункте.

Для решения предоставьте код запроса.

```SQL
--Впишите сюда ваш ответ


```
