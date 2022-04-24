# project-1
Самостоятельный проект первого скрипта
# Витрина RFM
# Задание 1
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
### Это решение соглсано заданию (сегменты равны)
```SQL
insert into analysis.dm_rfm_segments (user_id, recency, frequency, monetary_value)
with o as
(select * from production.orders ord
 left join production.orderstatuses os on os.id = ord.status
 where os.key = 'Closed'
 and ord.order_ts > '2021-01-01'), --Отбираем заказы со статусом Closed и с начала 2021г
rfm as 
 (select u.id as user_id, 
  count(o.order_id) as orders_cnt,
  now() - max(o.order_ts) as timediff,
  max(o.order_ts),
  sum(o.payment) as total_sum
  from production.users u
  left join o on o.user_id = u.id
  group by 1) --Формируем таблицу |юзер|кол-во заказов|времени с полседнего заказа|заплачено денег|
  select user_id, 
ntile(5) OVER(order by timediff desc) as recency, --подходит для кластеризации по времени, т.к время непрерывно
ntile(5) OVER(order by orders_cnt asc) as frequency,
ntile(5) OVER(order by orders_cnt asc) as monetary_value
from rfm
```
### Это решение которое я бы сделал
```SQL
insert into analysis.dm_rfm_segments (user_id, recency, frequency, monetary_value)
with o as
(select * from production.orders ord
 left join production.orderstatuses os on os.id = ord.status
 where os.key = 'Closed'
 and ord.order_ts > '2021-01-01'), --Отбираем заказы со статусом Closed и с начала 2021г
rfm as 
 (select u.id as user_id, 
  count(o.order_id) as orders_cnt,
  now() - max(o.order_ts) as timediff,
  max(o.order_ts),
  sum(coalesce(o.payment, 0)) as total_sum --используется в таком виде тк по непонятной причине значения null попадают в 100й перцентиль
  from production.users u
  left join o on o.user_id = u.id
  group by 1) --Формируем таблицу |юзер|кол-во заказов|времени с полседнего заказа|заплачено денег|
select user_id, 
ntile(5) OVER(order by timediff desc) as recency, --подходит для кластеризации по времени, т.к время непрерывно
--ntile(5) OVER(order by orders_cnt asc),-- этот вариант отбросил, тк два юзера с одинаковым кол-вом заказов могут попасть в разные группы
--cume_dist() OVER(order by orders_cnt asc), --этот столбец расчитывает перцентели значений и исопльзуется для кластеризации пользователей
--по количеству заказов
case when cume_dist() OVER(order by orders_cnt asc) <= 0.2 then 1
     when cume_dist() OVER(order by orders_cnt asc) <= 0.4 then 2
     when cume_dist() OVER(order by orders_cnt asc) <= 0.6 then 3
     when cume_dist() OVER(order by orders_cnt asc) <= 0.8 then 4
     else 5 end as frequency,
--cume_dist() OVER(order by total_sum asc), --этот столбец расчитывает перцентели значений и исопльзуется для кластеризации пользователей
--по потраченным деньгам
case when cume_dist() OVER(order by total_sum asc) <= 0.2 then 1
     when cume_dist() OVER(order by total_sum asc) <= 0.4 then 2
     when cume_dist() OVER(order by total_sum asc) <= 0.6 then 3
     when cume_dist() OVER(order by total_sum asc) <= 0.8 then 4
     else 5 end as monetary_value
from rfm
```
# Задание 2
Для того, чтобы ваш скрипт по расчету витрины продолжил работать, вам необходимо внести изменения в то, как формируется представление (view) analysis.Orders - вам необходимо вделать так, чтобы в этом представлении по-прежнему присутствовало поле status. Значение в этом поле должно соответствовать последнему (по времени) значению статуса из таблицы production.OrderStatusLog.

Для проверки предоставьте код на языке SQL, выполняющий обновление представления analysis.Orders.
```SQL
CREATE OR REPLACE VIEW analysis.orders
AS 
with s as
(select order_id, status_id , row_number() OVER(partition by order_id order by dttm desc) rn  
from analysis.orderstatuslog) --Получаем для каждого заказа порядковый номер статуса в обратном поярдке (последний статус в строке 1)
SELECT 
    orders.order_id,
    orders.order_ts,
    orders.user_id,
    orders.bonus_payment,
    orders.payment,
    orders.cost,
    orders.bonus_grant,
    s.status_id
   FROM production.orders
   left join s on s.order_id = orders.order_id and s.rn = 1 --джойним по строкам, у которых номер = 1 (соотв последнему статусу заказа);
```
