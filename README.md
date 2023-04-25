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

Судя по выводу, после запроса, на мой взгляд, в данном запросе лишним будет запрос к таблице film и к f.tite - сключаем их, и время вывода необходимой таблицы сокращается до 17ms, что уже является не плохим результатом.   
![Скрин](https://github.com/Karhq/12.5_hw_indexes/blob/main/Nom3.png)  

Далее просматриваем повторно, и видим, что идентичный вывод таблицы можно получить при использовании группировки (убираем портянку из условия Where). Применим ее. 
```sql
select concat(c.last_name, ' ', c.first_name), sum(p.amount)
from payment p
join rental r ON p.payment_date = r.rental_date
join customer c ON r.customer_id = c.customer_id
join inventory i ON i.inventory_id = r.inventory_id
where date(p.payment_date) = '2005-07-30'
group by c.customer_id
```
![Скрин](https://github.com/Karhq/12.5_hw_indexes/blob/main/Nom4.png)  
После проведенной группировки, время на запрос уменьшилось до 8ms.

Проведем повторный Explain Analize
![Скрин](https://github.com/Karhq/12.5_hw_indexes/blob/main/Nom6.png)  

Создаем индекс, для таблицы payment, поля payment_date. Изменяем запрои и проводим анализ повторно. 
```sql
CREATE INDEX pd_index ON payment(payment_date);
```

```sql
EXPLAIN ANALYZE
select concat(c.last_name, ' ', c.first_name), sum(p.amount)
from payment p
join rental r ON p.payment_date = r.rental_date
join customer c ON r.customer_id = c.customer_id
join inventory i ON i.inventory_id = r.inventory_id
where p.payment_date >= '2005-07-30' and p.payment_date < DATE_ADD('2005-07-30', INTERVAL 1 DAY)
group by c.customer_id
```
![Скрин](https://github.com/Karhq/12.5_hw_indexes/blob/main/Nom7.png)  
