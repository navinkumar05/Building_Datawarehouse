# Star Schema

- [Star Schema](#star-schema)
  - [Agenda](#agenda)
  - [Dimention tables](#dimention-tables)
    - [dimDate](#dimdate)
      - [Create table](#create-table)
      - [Insert records](#insert-records)
    - [dimCustomer](#dimcustomer)
      - [Create table](#create-table-1)
      - [Insert Records](#insert-records-1)
    - [dimMovie](#dimmovie)
      - [create table](#create-table-2)
      - [Insert records](#insert-records-2)
    - [dimstore](#dimstore)
      - [create table](#create-table-3)
      - [insert records](#insert-records-3)
  - [fact table](#fact-table)
    - [factsales](#factsales)
      - [create table](#create-table-4)
      - [insert records](#insert-records-4)
  - [Query](#query)
  - [Reference](#reference)

## Agenda
<img src="attachments\dvdrental - public - factsales.png" width="700">

## Dimention tables

### dimDate

#### Create table

```sql
CREATE TABLE dimDate
    (
      date_key integer NOT NULL PRIMARY KEY,
        date date NOT NULL,
        year smallint NOT NULL,
        quarter smallint NOT NULL,
        month smallint NOT NULL,
        day smallint NOT NULL,
        week smallint NOT NULL,
        is_weekend boolean
    );
```

#### Insert records

```sql
INSERT INTO dimDate (date_key, date, year, quarter, month, day, week, is_weekend)
SELECT DISTINCT(TO_CHAR(payment_date :: DATE, 'yyyyMMDD')::integer) AS date_key,
       date(payment_date)                                           AS date,
       EXTRACT(year FROM payment_date)                              AS year,
       EXTRACT(quarter FROM payment_date)                           AS quarter,
       EXTRACT(month FROM payment_date)                             AS month,
       EXTRACT(day FROM payment_date)                               AS day,
       EXTRACT(week FROM payment_date)                              AS week,
       CASE WHEN EXTRACT(ISODOW FROM payment_date) IN (6, 7) THEN true ELSE false END AS is_weekend
FROM payment;
```

### dimCustomer

#### Create table

```sql
CREATE TABLE dimCustomer
    (
      customer_key SERIAL PRIMARY KEY,
      customer_id  smallint NOT NULL,
      first_name   varchar(45) NOT NULL,
      last_name    varchar(45) NOT NULL,
      email        varchar(50),
      address      varchar(50) NOT NULL,
      address2     varchar(50),
      district     varchar(20) NOT NULL,
      city         varchar(50) NOT NULL,
      country      varchar(50) NOT NULL,
      postal_code  varchar(10),
      phone        varchar(20) NOT NULL,
      active       smallint NOT NULL,
      create_date  timestamp NOT NULL,
      start_date   date NOT NULL,
      end_date     date NOT NULL
    );
```

#### Insert Records

```sql
INSERT INTO dimCustomer (customer_key, customer_id, first_name, last_name, email, address, 
                         address2, district, city, country, postal_code, phone, active, 
                         create_date, start_date, end_date)
SELECT  c.customer_id as customer_key,
        c.customer_id,
        c.first_name,
        c.last_name,
        c.email,
        a.address,
        a.address2,
        a.district,
        ci.city,
        co.country,
        postal_code,
        a.phone,
        c.active,
        c.create_date,
       now()         AS start_date,
       now()         AS end_date
FROM customer c
JOIN address a  ON (c.address_id = a.address_id)
JOIN city ci    ON (a.city_id = ci.city_id)
JOIN country co ON (ci.country_id = co.country_id);
```

### dimMovie

#### create table

```sql
CREATE TABLE dimMovie
    (
      movie_key          SERIAL PRIMARY KEY,
      film_id            smallint NOT NULL,
      title              varchar(255) NOT NULL,
      description        text,
      release_year       year,
      language           varchar(20) NOT NULL,
      original_language  varchar(20),
      rental_duration    smallint NOT NULL,
      length             smallint NOT NULL,
      rating             varchar(5) NOT NULL,
      special_features   varchar(60) NOT NULL
    );
```

#### Insert records

```sql
INSERT INTO dimMovie (movie_key, film_id, title, description, release_year, language, original_language, rental_duration, length, rating, special_features)
SELECT 
    f.film_id as movie_key,
    f.film_id,
    f.title, 
    f.description,
    f.release_year,
    l.name as language,
    orig_lang.name AS original_language,
    f.rental_duration,
    f.length,
    f.rating,
    f.special_features
FROM film f
JOIN language l              ON (f.language_id=l.language_id)
LEFT JOIN language orig_lang ON (f.language_id = orig_lang.language_id);
```

### dimstore

#### create table

```sql
CREATE TABLE dimStore
    (
      store_key           SERIAL PRIMARY KEY,
      store_id            smallint NOT NULL,
      address             varchar(50) NOT NULL,
      address2            varchar(50),
      district            varchar(20) NOT NULL,
      city                varchar(50) NOT NULL,
      country             varchar(50) NOT NULL,
      postal_code         varchar(10),
      manager_first_name  varchar(45) NOT NULL,
      manager_last_name   varchar(45) NOT NULL,
      start_date          date NOT NULL,
      end_date            date NOT NULL
    );
```

#### insert records

```sql
INSERT INTO dimStore (store_key, store_id, address, address2, district, city, country, postal_code, manager_first_name, manager_last_name, start_date, end_date)
SELECT
    s.store_id as store_key,
    s.store_id,
    a.address,
    a.address2,
    a.district,
    c.city,
    co.country,
    a.postal_code,
    st.first_name as manager_first_name,
    st.last_name  as manager_last_name,
    now() as start_date,
    now() as end_date
FROM store s
JOIN staff st     ON    (s.manager_staff_id = st.staff_id)
JOIN address a    ON    (s.address_id = a.address_id)
JOIN city c       ON    (a.city_id = c.city_id)
JOIN country co   ON    (c.country_id = co.country_id);
```

## fact table

### factsales
#### create table

```sql
CREATE TABLE factSales
    (
        sales_key SERIAL PRIMARY KEY,
        date_key integer REFERENCES dimDate (date_key),
        customer_key integer REFERENCES dimCustomer (customer_key),
        movie_key integer REFERENCES dimMovie (movie_key),
        store_key integer REFERENCES dimStore (store_key),
        sales_amount numeric
    );
```

#### insert records

```sql
INSERT INTO factSales (date_key, customer_key, movie_key, store_key, sales_amount)
SELECT 
        TO_CHAR(payment_date :: DATE, 'yyyyMMDD')::integer AS date_key,
        p.customer_id  as customer_key,
        i.film_id as movie_key,
        i.store_id as store_key,
        p.amount as sales_amount
FROM payment p 
JOIN rental r ON (p.rental_id = r.rental_id)
JOIN inventory i ON (r.inventory_id = i.inventory_id);
```

## Query

```sql
-- start schema
SELECT dimMovie.title, dimDate.month, dimCustomer.city, sum(sales_amount) as revenue
FROM factSales 
JOIN dimMovie    on (dimMovie.movie_key      = factSales.movie_key)
JOIN dimDate     on (dimDate.date_key         = factSales.date_key)
JOIN dimCustomer on (dimCustomer.customer_key = factSales.customer_key)
group by (dimMovie.title, dimDate.month, dimCustomer.city)
order by dimMovie.title, dimDate.month, dimCustomer.city, revenue desc;


-- 3nf
SELECT f.title, EXTRACT(month FROM p.payment_date) as month, ci.city, sum(p.amount) as revenue
FROM payment p
JOIN rental r    ON ( p.rental_id = r.rental_id )
JOIN inventory i ON ( r.inventory_id = i.inventory_id )
JOIN film f ON ( i.film_id = f.film_id)
JOIN customer c  ON ( p.customer_id = c.customer_id )
JOIN address a ON ( c.address_id = a.address_id )
JOIN city ci ON ( a.city_id = ci.city_id )
group by (f.title, month, ci.city)
order by f.title, month, ci.city, revenue desc;
```

## Reference

- [How to build a start schema & Understanding Query time analysis - video](https://www.youtube.com/watch?v=vob1PBhyng8&list=PLBJe2dFI4sgukOW6O0B-OVyX9c6fQKJ2N&index=7)

- [Design star schema for relational database - Blog](https://dba.stackexchange.com/questions/224520/design-star-schema-for-relatonal-database)

- [what is star schema - video](https://www.youtube.com/watch?v=hQvCOBv_-LE&t=357s)