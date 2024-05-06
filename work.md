1. # Cоколовский Тимур, ИС 22/9-1

## Содержание
- [ER-диаграмма](#er-диаграмма)
- [Описание базы данных](#описание-базы-данных)
- [Демонстрация SQL-запросов](#демонстрация-sql-запросов)

## ER-диаграмма
![ER-диаграмма](img/de67caa6-5763-4c6e-9d43-58262f538ba9.jpg)

2. ## Описание базы данных
### Описание сущностей
1. #### Сущность "menu"
- menuid: int - Уникальный идентификатор записи в меню.
- dishid: int - Идентификатор блюда, внешний ключ.
- dishdescription: text - Подробное описание блюда.
- price: numeric - Цена блюда.
- cafeid: int - Идентификатор кафе, к которому принадлежит меню, внешний ключ.
  
![](img/Menu.png)

2. #### Сущность "customers"
- customerid: int - Уникальный идентификатор клиента.
- firstname: varchar - Имя клиента.
- lastname: varchar - Фамилия клиента.
- phone: varchar - Телефонный номер клиента.
- email: varchar - Электронная почта клиента.
- cafeid: int - Идентификатор кафе, которое клиент посещает, внешний ключ.
  
![](img/Customers.png)
3. #### Сущность "employees"
- employeeid: int - Уникальный идентификатор сотрудника.
- firstname: varchar - Имя сотрудника.
- lastname: varchar - Фамилия сотрудника.
- position: varchar - Должность сотрудника в кафе.
- phone: varchar - Телефонный номер сотрудника.
- cafeid: int - Идентификатор кафе, в котором работает сотрудник, внешний ключ.

![](img/Employees.png)
4. #### Сущность "cafe"
- cafeid: int - Уникальный идентификатор кафе.
- location: varchar - Адрес расположения кафе.
- phone: varchar - Телефонный номер кафе.
- email: varchar - Электронная почта кафе.
- openinghours: time - Часы работы кафе.

![](img/CafeScreen.png)

5. #### Сущность "orders"
- orderid: int - Уникальный идентификатор заказа.
- customerid: int - Идентификатор клиента, сделавшего заказ, внешний ключ.
- orderdatetime: timestamp - Дата и время совершения заказа.
- status: varchar - Статус заказа.
- cafeid: int - Идентификатор кафе, в котором сделан заказ, внешний ключ.

![](img/Orders.png)

6. #### Сущность "orderdetails"
- orderdetailid: int - Уникальный идентификатор детали заказа.
- orderid: int - Идентификатор заказа, внешний ключ.
- dishid: int - Идентификатор блюда в заказе, внешний ключ.

![](img/OrderDets.png)

## Демонстрация SQL-запросов
3. ### UNION
Для объединения результатов двух запросов, которые имеют одинаковое количество столбцов и совместимые типы данных, используется оператор `UNION`. Например:

```sql
SELECT firstname, lastname FROM customers
UNION
SELECT firstname, lastname FROM employees;
```
![Union](img/Union.png)
Этот запрос вернет список уникальных имен и фамилий клиентов и сотрудников.

...

4. ## ORDER BY
```sql
SELECT cafeid, COUNT(dishid) AS dish_count
FROM menu
GROUP BY cafeid
ORDER BY dish_count DESC;
```
![](img/OrderBy.png)
Этот запрос выводит список кафе с количеством блюд в каждом из них, упорядоченных от наибольшего к наименьшему. Это позволяет увидеть, какое кафе предлагает наибольшее разнообразие блюд.

...

5. ## HAVING
```sql
SELECT cafe.cafeid, cafe.location, COUNT(orders.orderid) AS number_of_orders
FROM cafe
JOIN orders ON cafe.cafeid = orders.cafeid
GROUP BY cafe.cafeid, cafe.location
HAVING COUNT(orders.orderid) > 5
ORDER BY number_of_orders DESC;
```
![](img/Having.png)
Запрос фильтрует кафе, где количество заказов превышает 1, и возвращает идентификаторы и местоположения таких кафе, упорядоченные по количеству заказов от большего к меньшему. HAVING здесь используется для фильтрации групп после группировки по cafeid, что невозможно выполнить с помощью WHERE.

...

6. ## Вложенные Запросы
### В SELECT
```sql
SELECT c.firstname, c.lastname,
  (SELECT COUNT(*) FROM orders o WHERE o.customerid = c.customerid) AS number_of_orders
FROM customers c;
```
![](img/Select.png)
Запрос возвращает имена и фамилии клиентов вместе с количеством сделанных ими заказов. Вложенный запрос подсчитывает количество заказов для каждого клиента.

### В WHERE
```sql
SELECT o.*
FROM orders o
WHERE EXISTS (
  SELECT 1 FROM orderdetails od
  JOIN menu m ON m.dishid = od.dishid
  WHERE od.orderid = o.orderid AND m.price = (
    SELECT MAX(price) FROM menu
  )
);
```
![](img/Where.png)
Запрос выбирает все заказы, в которых присутствует хотя бы одно блюдо с максимальной ценой в меню. Вложенный запрос с EXISTS проверяет наличие такого блюда в каждом заказе.

...

8. ### JOINs
Для демонстрации работы `JOIN` можно использовать следующий запрос, который соединяет информацию о заказах с информацией о клиентах:

```sql
SELECT o.orderid, c.firstname, c.lastname
FROM orders o
JOIN customers c ON o.customerid = c.customerid;
```
![](img/InnerJoin.png)
Этот запрос возвращает идентификаторы заказов вместе с именами и фамилиями клиентов, которые их сделали.

### LEFT JOIN
Использование `LEFT JOIN` для получения всех клиентов и их заказов, включая клиентов, у которых нет заказов.

```sql
SELECT customers.firstname, customers.lastname, orders.orderid
FROM customers
LEFT JOIN orders ON customers.customerid = orders.customerid;
```
![](img/LeftJoin.png)
### RIGHT JOIN
Использование `RIGHT JOIN` для получения всех заказов и информации о клиентах, включая заказы, не связанные с клиентами.

```sql
SELECT customers.firstname, customers.lastname, orders.orderid
FROM customers
RIGHT JOIN orders ON customers.customerid = orders.customerid;
```
![](img/RightJoin.png)
### CROSS JOIN
`CROSS JOIN` создаёт декартово произведение, т.е., соединяет каждую запись из одной таблицы с каждой записью из другой таблицы.

```sql
SELECT customers.firstname, customers.lastname, cafe.location
FROM customers
CROSS JOIN cafe;
```
![](img/CrossJoin.png)
### FULL OUTER JOIN
`FULL OUTER JOIN` используется для выборки всех записей, когда есть совпадение в одной из таблиц, включает всех клиентов и все заказы, даже если нет совпадения.

```sql
SELECT customers.firstname, customers.lastname, orders.orderid
FROM customers
FULL OUTER JOIN orders ON customers.customerid = orders.customerid;
```
![](img/FullOuJoin.png)
...
### Оконные функции
Оконные функции позволяют выполнять вычисления над набором строк, подобно агрегатным функциям, но без группировки. Например:

```sql
SELECT orderid, orderdatetime, SUM(price) OVER (PARTITION BY customerid) as total_spent
FROM orders 
    JOIN OrderDetails USING(orderid) 
    JOIN Menu USING(dishid);
```
![](img/AVGshka.png)
Этот запрос рассчитывает общую сумму, потраченную каждым клиентом.

...

9. ### CASE
Оператор `CASE` используется для реализации логики ветвления в SQL. Например:

```sql
SELECT
  c.firstname,
  c.lastname,
  o.orderdatetime,
  CASE 
    WHEN o.status = 'Принят' THEN 'В процессе'
    WHEN o.status = 'Готовится' THEN 'Готовится'
    WHEN o.status = 'Доставляется' THEN 'В пути'
    WHEN o.status = 'Завершен' THEN 'Завершен'
    WHEN o.status = 'Отдан' THEN 'Отдан'
    ELSE 'Неизвестно'
  END AS order_status
FROM customers AS c
JOIN orders AS o ON c.customerid = o.customerid;
```
![](img/Case.png)
Этот запрос возвращает идентификаторы заказов с описанием их статуса на русском языке.

...
