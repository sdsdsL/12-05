# Домашнее задание к занятию 12.5. «Индексы» - Денис Беляев

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

![](https://github.com/sdsdsL/12-05/blob/main/img/1.png)

```sql
select (sum(index_length) / sum(data_length)) * 100 as percentage_ratio
from information_schema.TABLES
```

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```

![](https://github.com/sdsdsL/12-05/blob/main/img/2_1.png)

- перечислите узкие места;
   - долгое выполнение операции - 10.610 секунд
   - большой объем данных
  
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.
   - при добавлении индекса idx_payment_date время выполнения запроса составило 13 миллисекунд
   - добавлено join for rental, customer and inventory
   - удалено f.title and film f
   - переписано условие where to read as where p.payment_date >= '2005-07-30' and p.payment_date < date_add('2005-07-30', interval 1 day);

```sql
create index idx_payment_date on payment (payment_date);

explain analyze 
select distinct concat (c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
from payment p
join rental r on p.payment_date = r.rental_date
join customer c on r.customer_id = c.customer_id
join inventory i on r.inventory_id = i.inventory_id
where p.payment_date >= '2005-07-30' and p.payment_date < date_add('2005-07-30', interval 1 day);

INSERT INTO `explain analyze 
select distinct concat (c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
from payment p
join rental r on p.payment_date = r.rental_date
join customer c on r.customer_id = c.customer_id
join inventory i on r.inventory_id = i.inventory_id
where p.payment_date >= '2005-07-30' and p.payment_date < date_add('2005-07-30', interval 1 day)` (`EXPLAIN`) VALUES
	 ('-> Table scan on <temporary>  (cost=2.50..2.50 rows=0) (actual time=9.820..9.862 rows=391 loops=1)
    -> Temporary table with deduplication  (cost=0.00..0.00 rows=0) (actual time=9.819..9.819 rows=391 loops=1)
        -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id )   (actual time=8.311..9.579 rows=642 loops=1)
            -> Sort: c.customer_id  (actual time=8.284..8.352 rows=642 loops=1)
                -> Stream results  (cost=806.88 rows=661) (actual time=0.082..7.929 rows=642 loops=1)
                    -> Nested loop inner join  (cost=806.88 rows=661) (actual time=0.076..7.382 rows=642 loops=1)
                        -> Nested loop inner join  (cost=582.24 rows=661) (actual time=0.070..6.038 rows=642 loops=1)
                            -> Nested loop inner join  (cost=350.73 rows=634) (actual time=0.049..2.411 rows=634 loops=1)
                                -> Filter: ((r.rental_date >= TIMESTAMP''2005-07-30 00:00:00'') and (r.rental_date < <cache>((''2005-07-30'' + interval 1 day))))  (cost=128.83 rows=634) (actual time=0.032..0.828 rows=634 loops=1)
                                    -> Covering index range scan on r using rental_date over (''2005-07-30 00:00:00'' <= rental_date < ''2005-07-31 00:00:00'')  (cost=128.83 rows=634) (actual time=0.027..0.585 rows=634 loops=1)
                                -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.25 rows=1) (actual time=0.002..0.002 rows=1 loops=634)
                            -> Index lookup on p using idx_payment_date (payment_date=r.rental_date)  (cost=0.26 rows=1) (actual time=0.004..0.005 rows=1 loops=634)
                        -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=0.24 rows=1) (actual time=0.002..0.002 rows=1 loops=642)
');
```

![](https://github.com/sdsdsL/12-05/blob/main/img/2_2.png)

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

- GIN (Generalized Inverted Index) - это индекс, который используется для полнотекстового поиска и поиска по массивам или JSON-объектам.

- GiST (Generalized Search Tree) - это индекс, который может использоваться для поиска по геометрическим данным, полнотекстовому поиску, а также для поиска по произвольным типам данных.

- SP-GiST (Space-Partitioned Generalized Search Tree) - альтернативный вариант GiST-индекса, который может использоваться для более эффективного хранения и поиска пространственных данных.

- BRIN (Block Range INdex) - индекс, который используется для эффективного сканирования больших таблиц с упорядоченными данными.

- Hash - это быстрый индекс, который используется для точного поиска по значению, но он не подходит для диапазонных запросов.

- RUM (Range-UpperMark) - это индекс, который используется для поиска по диапазону значений с возможностью оптимизации для запросов, которые содержат частичное совпадение.

- KNN-GiST - это индекс, который используется для поиска ближайших соседей на плоскости или в пространстве.

*Приведите ответ в свободной форме.*
