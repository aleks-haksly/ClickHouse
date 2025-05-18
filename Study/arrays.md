## Массивы в ClickHouse

1. Создание массива (ручное)
```sql
SELECT
[2, 3, 4, 5, 8, 10] AS array1, -- массив int
array('k', '.', 'c') AS array2, -- массив строк
[today(), yesterday(), today() + 1] AS array3, -- массив дат
array(tuple(1, 'a'), tuple(3, 'd')) AS array4 -- массив таплов
```

2. Доступ к элементам массива

```sql
...
array1[1] AS first_el, -- первый элемент
array1[-1] AS last_el -- последний элемент
```
 - Доп функции
```sql
length(array1), -- длина массива
arrayEnumerate(array1) -- список индексов массива
```

3. Создание массива из данных
```sql
SELECT 
user_id,
groupArray(created_at) ts_array,
groupArray(total_sum) amount_array,
groupArray(tuple(created_at, total_sum)) ta_array,
ta_array.1,  -- только даты
ta_array.2, -- только суммы
groupUniqArray(created_at) uniq_dates, -- только уникальные
FROM t
GROUP BY user_id
```
4. Сортифровка
```sql
...
arraySort(x -> x.2, ta_array) sorted_by_amount_asc, -- с lambda
arrayReverse(uniq_dates) date_desc
FROM t
GROUP BY user_id
```

5. Фильтрация (lambda)
```sql
...
arrayFilter(x -> x.2 > 2500, ta_array) filtered_array, 
arrayFilter(x -> x.2 > 2500, ta_array).1 filtered_dates, -- можно брать только один столбец
arrayFilter((x, y) -> x < today() - 14 AND y > 2500, ts_array, amount_array) 
```
6. Преобразование массива (arrayMap)

```sql
...
arrayMap(x -> if(x.1 > today() - 100, x.2 * 2, x.2 * 3), ta_array),
arrayMap(x -> if(x.1 > today() - 100, tuple(x.1, x.2 * 2), tuple(x.1, x.2 * 3)), ta_array)
```
7. Кумулятивная сумма

```sql
...
arrayCumsum(ta_array.2) as cumsum,
arrayCumsum(arraySort(x -> x.1, ta_array).2) as cumsum_ordered_by date_asc
```
8. Reduce
```sql
...
arrayReduce('sum', amount_array) as summ,
arrayReduce('avg', amount_array) as avgg,
arrayReduce('sumIf', amount_array, condition_array) as summ

```
