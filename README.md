# Домашнее задание к занятию «Индексы» - Карпов Роман

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

#### Ответ: 

```sql
SELECT ((SUM(INDEX_LENGTH) / SUM(DATA_LENGTH)) * 100) 
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'sakila'
```
![Скрин](https://github.com/Karhq/12.5_hw_indexes/blob/main/Nom1.png)

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.


#### Ответ: 

Время на вывод стандартного запроса занимает - 6.806 секунд.  
![Скрин](https://github.com/Karhq/12.5_hw_indexes/blob/main/Nom2.png)  

Судя по выводу, после запроса, на мой взгляд? в данном запросе лишним будет запрос к таблице film и к f.tite - сключаем их, и время вывода необходимой таблицы сокращается до 17ms, что уже является не плохим результатом.   
![Скрин](https://github.com/Karhq/12.5_hw_indexes/blob/main/Nom3.png)  
