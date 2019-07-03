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

5. Most female common name in each state 2009 - 2018

```SQL
    WITH state_name AS
    (SELECT state, name, sum(number) AS count
    FROM `bigquery-public-data.usa_names.usa_1910_curren`
    WHERE gender = 'F' AND year >= 2008
    GROUP BY state, name)

    SELECT t1.state, t1.name, t1.count
    FROM state_name t1
    INNER JOIN
    (SELECT state, max(count) as max_count
    FROM state_name
    GROUP BY state) t2
    ON t1.state = t2.state AND t1.count = t2.max_count
    ORDER BY t1.state
```

**state**|**name**|**count**
:-----:|:-----:|:-----:
AK|Emma|552
AL|Emma|3206
AR|Emma|2095
AZ|Sophia|5249
CA|Sophia|32754
CO|Olivia|3456
CT|Olivia|2504
DC|Olivia|455
DE|Ava|656
FL|Isabella|16462
GA|Ava|6204
HI|Sophia|624
IA|Emma|2203
ID|Emma|1249
IL|Olivia|8976
IN|Emma|4980
KS|Emma|2234
KY|Emma|3518
LA|Ava|3211
MA|Olivia|4838
MD|Olivia|3333
ME|Emma|941
MI|Olivia|6462
MN|Olivia|3940
MO|Emma|4465
MS|Ava|1898
MT|Emma|682
NC|Emma|6688
ND|Emma|717
NE|Emma|1412
NH|Olivia|955
NJ|Isabella|6505
NM|Isabella|1283
NV|Sophia|2023
NY|Sophia|13482
OH|Emma|8315
OK|Emma|2825
OR|Emma|2549
PA|Emma|8849
RI|Sophia|831
SC|Emma|2825
SD|Emma|661
TN|Emma|5079
TX|Isabella|21531
UT|Olivia|2947
VA|Emma|5256
VT|Emma|378
WA|Olivia|4789
WI|Olivia|3759
WV|Emma|1448
WY|Emma|369

6. Top 10 gender neutral names

```SQL
    WITH name_gender AS
    (
    SELECT name, SUM(IF(gender='F',number,0)) female, SUM(IF(gender='M',number,0)) male
    FROM `bigquery-public-data.usa_names.usa_1910_current`
    GROUP BY name
    )

    SELECT name, ABS(male-female)/(male+female) balance, female, male
    FROM name_gender
    WHERE male * female>1000000
    ORDER BY 2
    LIMIT 10;
```

**name**|**balance**|**female**|**male**
:-----:|:-----:|:-----:|:-----:
Santana|9.445843828715365E-4|3179|3173
Landry|9.946949602122016E-4|3013|3019
Lennon|0.0041753653444676405|3367|3339
Kris|0.006668493651228647|11020|10874
Kerry|0.007894221777613467|45620|46346
Justice|0.012610626641164456|15229|15618
Arden|0.015634580012262415|3211|3313
Quinn|0.01792151435942118|29791|28742
Oakley|0.025977733371395945|3412|3594


7. Average length of names over time

```SQL
    SELECT t1.year, male, female
    FROM
    (SELECT year, AVG(LENGTH(name)) as male
    FROM `bigquery-public-data.usa_names.usa_1910_current`
    WHERE gender = 'M'
    GROUP BY year) t1
    INNER JOIN
    (SELECT year, AVG(LENGTH(name)) as female
    FROM `bigquery-public-data.usa_names.usa_1910_current`
    WHERE gender = 'F'
    GROUP BY year) t2
    on t1.year = t2.year
```
![](https://i.imgur.com/T4205ue.png)
