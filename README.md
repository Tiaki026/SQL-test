# SQL-test

### Задачи по SQL

Оставлю на память 🤸

---

★ Задача №1

Написать запрос, выводящий имя и фамилию самых бедных клиентов -
среди замужних женщин, не проживающих ни в Японии, ни в Бразилии,
ни в Италии. Богатство определяется по кредитному лимиту.
[Отсортировать по CUST_LAST_NAME].

```sql
SELECT
	c.CUST_FIRST_NAME,
	c.CUST_LAST_NAME
FROM
	sh.CUSTOMERS c
WHERE
	CUST_GENDER = 'F'
	AND CUST_MARITAL_STATUS = 'Married'
	AND COUNTRY NOT IN ('Japan', 'Brazil', 'Italy')
	AND c.CUST_CREDIT_LIMIT = (
		SELECT MIN(CUST_CREDIT_LIMIT)
		FROM sh.CUSTOMERS
	)
ORDER BY c.CUST_LAST_NAME;
```

---

★ Задача №2

Написать запрос, выводящий клиента с самым длинным домашним адресом,
чей телефонный номер заканчивается на 77. Вывести результат в одном столбце,
в формате “Name: [cust_first_name] [cust_last_name]; city: [cust_city];
address: [cust_street_address]; number:[cust_main_phone_number];
email: [cust_email]; ”. (всё, что обернуто в [] – названия полей (столбцов) таблицы).
*/

```sql
SELECT
	'Name: ' || CUST_FIRST_NAME  || ' ' || CUST_LAST_NAME  || '; ' ||
	'city: ' || CUST_CITY  || '; ' ||
	'address: ' || CUST_STREET_ADDRESS  || '; ' ||
	'number: ' || CUST_MAIN_PHONE_NUMBER  || '; ' ||
	'email: ' || CUST_EMAIL  || ';'  AS LONG_ADRESS
FROM
	sh.CUSTOMERS
WHERE
	CUST_MAIN_PHONE_NUMBER LIKE '%77'
	AND LENGTH(CUST_STREET_ADDRESS) = (
		SELECT MAX(LENGTH(CUST_STREET_ADDRESS))
		FROM sh.CUSTOMERS
		WHERE CUST_MAIN_PHONE_NUMBER LIKE '%77'
	);
```

---

★ Задача №3

Написать запрос, выводящий всех клиентов, которые купили самый
дешевый продукт (цена считается от цены продажи - cust_list_price)
в субкатегории 'Sweaters - Men' или 'Sweaters - Women'
(связка таблиц CUSTOMERS -> SALES -> PRODUCTS), среди тех, кто родился
позже 1980 года, вывод должен быть отсортирован по cust_id.

```sql
SELECT DISTINCT
	c.CUST_ID,
	c.CUST_FIRST_NAME,
	c.CUST_LAST_NAME
FROM
	sh.CUSTOMERS c
JOIN sh.SALES s ON c.CUST_ID = s.CUST_ID
JOIN sh.PRODUCTS p ON s.PROD_ID = p.PROD_ID
WHERE
	p.PROD_SUBCATEGORY IN ('Sweaters - Men', 'Sweaters - Women')
	AND c.CUST_YEAR_OF_BIRTH > 1980
	AND p.PROD_LIST_PRICE = (
		SELECT MIN(p.PROD_LIST_PRICE)
		FROM sh.SALES s
		JOIN sh.PRODUCTS p ON s.PROD_ID = p.PROD_ID
		WHERE p.PROD_SUBCATEGORY IN ('Sweaters - Men', 'Sweaters - Women')
	)
ORDER BY c.CUST_ID;
```

---

★ Задача №4

Написать запрос, выводящий всех клиентов-мужчин с уровнем дохода "D",
у которых не заполнено поле "семейное положение" и которые проживают
в США или Германии (с использованием EXISTS). Отсортировать по cust_id.

```sql
SELECT
	c.CUST_ID,
	c.CUST_FIRST_NAME,
	c.CUST_LAST_NAME
FROM
	sh.CUSTOMERS c
WHERE
	c.CUST_GENDER = 'M'
	AND c.CUST_INCOME_LEVEL LIKE 'D%'
	AND c.CUST_MARITAL_STATUS IS NULL
	AND EXISTS (
		SELECT 1
		FROM sh.COUNTRIES co
		WHERE co.COUNTRY_ID = c.COUNTRY_ID
		AND co.COUNTRY_NAME IN ('United States of America', 'Germany')
	)
ORDER BY c.CUST_ID;
```

---

★ Задача №5

Написать запрос, выводящий среднюю сумму покупки (сумма покупки
является произведением цены товара (prod_list_price) на количество
проданного товара (quantity_sold)) в каждой стране, полное название
страны. Отсортировать в порядке убывания средней суммы.

```sql
SELECT
	co.COUNTRY_NAME AS "Страна",
	AVG(p.PROD_LIST_PRICE * s.QUANTITY_SOLD) AS "Средняя сумма"
FROM
	sh.SALES s
JOIN sh.CUSTOMERS c ON s.CUST_ID = c.CUST_ID
JOIN sh.COUNTRIES co ON c.COUNTRY_ID = co.COUNTRY_ID
JOIN sh.PRODUCTS p ON s.PROD_ID = p.PROD_ID
GROUP BY
	co.COUNTRY_NAME
ORDER BY "Средняя сумма" DESC;
```
---

★ Задача №6

Написать запрос, выводящий "популярность" почтовых доменов 
клиентов, т.е. количество клиентов с почтой в каждом из доменов.

```sql
SELECT
	SUBSTR(c.CUST_EMAIL, INSTR(c.CUST_EMAIL, '@') + 1) AS "Домен",
	COUNT(*) AS "Количество"
FROM
	sh.CUSTOMERS c
WHERE
	c.CUST_EMAIL IS NOT NULL
GROUP BY
	SUBSTR(c.CUST_EMAIL, INSTR(c.CUST_EMAIL, '@') + 1)
ORDER BY "Количество" DESC;
```

---

★ Задача №7

Написать запрос, выводящий распределение суммы проданных товаров
в единицах (quantity_sold) категории "Men" по странам (т.е.
распределение по странам, в которых проживают клиенты), в конечной
выборке оставить те страны, в которых общее количество проданных
товаров в единицах выше, чем средняя сумма проданных товаров в
единицах этой категории в стране (по всему миру). Упорядочить по
полному названию стран.

```sql
WITH country_sums AS (
	SELECT
		co.COUNTRY_NAME,
		SUM(s.QUANTITY_SOLD) AS total
	FROM
		sh.SALES s
		JOIN sh.CUSTOMERS c ON s.CUST_ID = c.CUST_ID
		JOIN sh.COUNTRIES co ON c.COUNTRY_ID = co.COUNTRY_ID
		JOIN sh.PRODUCTS p ON s.PROD_ID = p.PROD_ID
	WHERE
		p.PROD_CATEGORY = 'Men'
	GROUP BY co.COUNTRY_NAME
)
SELECT
	COUNTRY_NAME AS "Страна",
	total AS "Количество товаров"
FROM
	country_sums
WHERE
	total > (SELECT AVG(total) FROM country_sums)
ORDER BY "Страна";
```

---

★ Задача №8

Написать запрос, выводящий процентное соотношение мужчин и женщин,
проживающих в каждой стране, отсортированное по названию страны в
алфавитном порядке. Столбцы в выводе должны быть такими: «Страна»,
«% мужчин», «% женщин» [использовать WITH]. Упорядочить по полному
названию стран.

```sql
WITH customer_stats AS (
	SELECT
		co.COUNTRY_NAME,
		COUNT(
			CASE WHEN c.CUST_GENDER = 'M' THEN 1 END
		) AS male_count,
		COUNT(
			CASE WHEN c.CUST_GENDER = 'F' THEN 1 END
		) AS female_count,
		COUNT(*) AS total
	FROM
		sh.CUSTOMERS c
	JOIN
		sh.COUNTRIES co ON c.COUNTRY_ID = co.COUNTRY_ID
	GROUP BY
		co.COUNTRY_NAME
)
SELECT
	COUNTRY_NAME AS "Страна",
	ROUND(male_count * 100.0 / total, 2) AS "% мужчин",
	ROUND(female_count * 100.0 / total, 2) AS "% женщин"
FROM
	customer_stats
ORDER BY "Страна";
```

---

★ Задача №9

Написать запрос, выводящий максимальное суммарное количество
проданных единиц товара (quantity_sold) за день для каждого
продукта (т.е. продукты в выводе не должны повторяться).
Запрос должен выводить TOP 20 строк, отсортированных по убыванию
количества проданных единиц товара (Столбцы должны быть такими:
"Макс покуп/день", prod_name) [Под первым столбцом подразумевается
объединение в одно поле количества покупок и последней даты,
за которую сделаны эти покупки].

```sql
WITH product_daily_totals AS (
	SELECT
		p.PROD_NAME,
		t.TIME_ID,
		SUM(s.QUANTITY_SOLD) AS daily_total
	FROM
		sh.SALES s
		JOIN sh.PRODUCTS p ON s.PROD_ID = p.PROD_ID
		JOIN sh.TIMES t ON s.TIME_ID = t.TIME_ID
	GROUP BY
		p.PROD_NAME, t.TIME_ID
)
SELECT
	MAX(daily_total) || ' (' || 
	TO_CHAR(
		MAX(TIME_ID) KEEP (
			DENSE_RANK FIRST ORDER BY daily_total DESC
		),
		'DD.MM.YYYY'
	) || ')' AS "Макс покуп/день",
	PROD_NAME AS "prod_name"
FROM
	product_daily_totals
GROUP BY PROD_NAME
ORDER BY MAX(daily_total) DESC
FETCH FIRST 20 ROWS ONLY;
```

---

★ Задача №10

Написать запрос, выводящий максимальное суммарное количество проданных единиц товара за день
для каждой категории продуктов. Отсортировать по убыванию количества.
(Столбцы должны быть такими: "Макс за день", prod_category).
[Под первым столбцом подразумевается одно число].

```sql
WITH daily_sales AS (
	SELECT
		p.PROD_CATEGORY,
		t.TIME_ID,
		SUM(s.QUANTITY_SOLD) AS daily_quantity
	FROM
		sh.SALES s
		JOIN sh.PRODUCTS p ON s.PROD_ID = p.PROD_ID
		JOIN sh.TIMES t ON s.TIME_ID = t.TIME_ID
    GROUP BY
        p.PROD_CATEGORY, t.TIME_ID
)
SELECT
	MAX(daily_quantity) AS "Макс за день",
	PROD_CATEGORY AS "prod_category"
FROM
	daily_sales
GROUP BY PROD_CATEGORY
ORDER BY "Макс за день" DESC;
```

---

★ Задача №11

Написать запрос, который создаст таблицу с именем
sales_[имя пользователя в ОС]_[Ваше имя]_[Ваша фамилия],
содержащую строки из таблицы sh.sales за один пиковый месяц.
(Т.е. месяц, за который получена максимальная выручка).
Показать все поля таблицы в порядке возрастания дат.

```sql
CREATE TABLE sales_tiaki_Eugene_Kolotikov AS
WITH peak_month AS (
	SELECT t2.CALENDAR_MONTH_DESC
	FROM sh.SALES s2
	JOIN sh.TIMES t2 ON s2.TIME_ID = t2.TIME_ID
	GROUP BY t2.CALENDAR_MONTH_DESC
	ORDER BY SUM(s2.AMOUNT_SOLD) DESC
	FETCH FIRST 1 ROW ONLY
)
SELECT s.*
FROM sh.SALES s
WHERE s.TIME_ID IN (
	SELECT t.TIME_ID
	FROM sh.TIMES t
	WHERE t.CALENDAR_MONTH_DESC = (SELECT CALENDAR_MONTH_DESC FROM peak_month)
)
ORDER BY s.TIME_ID ASC;
```

---

★ Задача №12
Написать запрос, который для созданной в задании 11 таблицы изменит 
значение поля time_id на формат 'DD.MM.YYYY HH24:MI:SS' (см. NLS_DATE_FORMAT).
Значение hh24:mm:ss должно выбираться случайным образом. Сохранить сделанные 
изменения. Показать все поля таблицы в порядке возрастания дат.
SELECT dbms_random.value FROM DUAL возвращает случайное значение от 0 до 1;

```sql
UPDATE sales_tiaki_Eugene_Kolotikov
SET time_id = TRUNC(time_id) + DBMS_RANDOM.VALUE(0, 1);

COMMIT;

SELECT
	prod_id,
	cust_id,
	TO_CHAR(time_id, 'DD.MM.YYYY HH24:MI:SS') AS time_id,
	channel_id,
	promo_id,
	quantity_sold,
	amount_sold
FROM
	sales_tiaki_Eugene_Kolotikov
ORDER BY time_id;
```

---

★ Задача №13

Написать запрос, выводящий почасовую разбивку количества операций продажи для Вашей таблицы.

```sql
SELECT
	TO_CHAR(time_id, 'HH24') AS "Час",
	COUNT(*) AS "Количество продаж"
FROM
	sales_tiaki_Eugene_Kolotikov
GROUP BY TO_CHAR(time_id, 'HH24')
ORDER BY TO_CHAR(time_id, 'HH24');
```

---

★ Задача №14

Написать запрос, который удалит созданную в задании 11 таблицу. Сохранить сделанные изменения.

```sql
DROP TABLE sales_tiaki_Eugene_Kolotikov PURGE;

COMMIT;
```
