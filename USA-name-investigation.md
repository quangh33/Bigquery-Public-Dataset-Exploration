#### Dataset: bigquery-public-data.usa_names.usa_1910_current
#### Schema:

|Field name	|Type	|Mode	  |Description                      
|-----------|-------|---------|---------------------
|state	    |STRING	|NULLABLE |	2-digit state code
|gender	    |STRING	|NULLABLE |	Sex (M=male or F=female)
|year	    |INTEGER|NULLABLE |	4-digit year of birth
|name	    |STRING	|NULLABLE |	Given name of a person at birth
|number	    |INTEGER|	NULLABLE |	Number of occurrences of the name

#### Insight

1. Number of distinct name

```SQL
    select count(distinct(name))
    from `bigquery-public-data.usa_names.usa_1910_current`
```

`31595`

2. 10 most common name of all time
```SQL
    select name, sum(number) as count
    from `bigquery-public-data.usa_names.usa_1910_current`
    where gender = 'M'
    group by name
    order by count desc
    limit 10;
```
- male

**name**|**count**
:-----:|:-----:
James|4997327
John|4869607
Robert|4734038
Michael|4349307
William|3890923
David|3597725
Richard|2539873
Joseph|2522812
Charles|2273068
Thomas|2245124

- female

**name**|**count**
:-----:|:-----:
Mary|3741196
Patricia|1569022
Elizabeth|1537684
Jennifer|1466161
Linda|1447943
Barbara|1424221
Margaret|1130920
Susan|1109309
Dorothy|1053390
Jessica|1043442


3. Most common name of 2018

- male

```SQL
    select name, sum(number) as count
    from `bigquery-public-data.usa_names.usa_1910_current`
    where year = 2018 and gender = "M"
    group by name
    order by count desc
    limit 1;
```

`Liam 19837`

- female

```SQL
    select name, sum(number) as count
    from `bigquery-public-data.usa_names.usa_1910_current`
    where year = 2018 and gender = "F"
    group by name
    order by count desc
    limit 1;
```

`Emma 18688`

4. Most common name in each decade

```SQL
    WITH name_decade AS
    (SELECT 
    CASE WHEN CAST(year as STRING) like '188%' THEN '1880-1889'
        WHEN CAST(year as STRING) like '189%' THEN '1890-1899'
        WHEN CAST(year as STRING) like '190%' THEN '1900-1909'
        WHEN CAST(year as STRING) like '191%' THEN '1910-1919'
        WHEN CAST(year as STRING) like '192%' THEN '1920-1929'
        WHEN CAST(year as STRING) like '193%' THEN '1930-1939'
        WHEN CAST(year as STRING) like '194%' THEN '1940-1949'
        WHEN CAST(year as STRING) like '195%' THEN '1950-1959'
        WHEN CAST(year as STRING) like '196%' THEN '1960-1969'
        WHEN CAST(year as STRING) like '197%' THEN '1970-1979'
        WHEN CAST(year as STRING) like '198%' THEN '1980-1989'
        WHEN CAST(year as STRING) like '199%' THEN '1990-1999'
        WHEN CAST(year as STRING) like '200%' THEN '2000-2009'
        WHEN CAST(year as STRING) like '201%' THEN '2010-2019'
    END AS decade,
    Name, SUM(number) AS count
    FROM `bigquery-public-data.usa_names.usa_1910_current`
    WHERE Gender = 'F'
    GROUP BY decade, Name)

    SELECT t1.decade, t1.name, t1.count
    FROM name_decade t1
    INNER JOIN 
    (SELECT decade, MAX(count) as max_count from name_decade group by decade) t2
    ON t1.decade = t2.decade AND t1.count = t2.max_count
```

- female


**decade**|**name**|**count**
:-----:|:-----:|:-----:
1910-1919|Mary|478639
1920-1929|Mary|701754
1930-1939|Mary|572956
1940-1949|Mary|640031
1950-1959|Mary|625568
1960-1969|Lisa|496976
1970-1979|Jennifer|581763
1980-1989|Jessica|469487
1990-1999|Jessica|303094
2000-2009|Emily|223690
2010-2019|Emma|177410

- male

**decade**|**name**|**count**
:-----:|:-----:|:-----:
1910-1919|John|376318
1920-1929|Robert|576363
1930-1939|Robert|590733
1940-1949|James|795680
1950-1959|James|843531
1960-1969|Michael|833216
1970-1979|Michael|707645
1980-1989|Michael|663741
1990-1999|Michael|462327
2000-2009|Jacob|273844
2010-2019|Noah|163657

