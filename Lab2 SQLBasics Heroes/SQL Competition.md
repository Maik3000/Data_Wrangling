```python
import numpy as np
import pandas as pd
import duckdb

heroes_information = pd.read_csv("heroes_information.csv")
```


```python
duck_query = lambda q: duckdb.query(q)
pysqldf = lambda q: sqldf(q,globals())
```


```python
###instalar desde anaconda navigator######
####pandasql
####mysql
####mysql-connector-c
####mysql-connector-python
from pandasql import *
```


```python
####para enlistar los documentos en el directorio#####
os.listdir()
```




    ['.ipynb_checkpoints', 'heroes_information.csv', 'SQL Competition.ipynb']




```python
query1 = " SELECT * FROM heroes_information LIMIT 10"
```


```python
pysqldf(query1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>name</th>
      <th>Gender</th>
      <th>Eye color</th>
      <th>Race</th>
      <th>Hair color</th>
      <th>Height</th>
      <th>Publisher</th>
      <th>Skin color</th>
      <th>Alignment</th>
      <th>Weight</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>A-Bomb</td>
      <td>Male</td>
      <td>yellow</td>
      <td>Human</td>
      <td>No Hair</td>
      <td>203.0</td>
      <td>Marvel Comics</td>
      <td>None</td>
      <td>good</td>
      <td>441.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>Abe Sapien</td>
      <td>Male</td>
      <td>blue</td>
      <td>Icthyo Sapien</td>
      <td>No Hair</td>
      <td>191.0</td>
      <td>Dark Horse Comics</td>
      <td>blue</td>
      <td>good</td>
      <td>65.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>Abin Sur</td>
      <td>Male</td>
      <td>blue</td>
      <td>Ungaran</td>
      <td>No Hair</td>
      <td>185.0</td>
      <td>DC Comics</td>
      <td>red</td>
      <td>good</td>
      <td>90.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>Abomination</td>
      <td>Male</td>
      <td>green</td>
      <td>Human / Radiation</td>
      <td>No Hair</td>
      <td>203.0</td>
      <td>Marvel Comics</td>
      <td>None</td>
      <td>bad</td>
      <td>441.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>Abraxas</td>
      <td>Male</td>
      <td>blue</td>
      <td>Cosmic Entity</td>
      <td>Black</td>
      <td>-99.0</td>
      <td>Marvel Comics</td>
      <td>None</td>
      <td>bad</td>
      <td>-99.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5</td>
      <td>Absorbing Man</td>
      <td>Male</td>
      <td>blue</td>
      <td>Human</td>
      <td>No Hair</td>
      <td>193.0</td>
      <td>Marvel Comics</td>
      <td>None</td>
      <td>bad</td>
      <td>122.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6</td>
      <td>Adam Monroe</td>
      <td>Male</td>
      <td>blue</td>
      <td>None</td>
      <td>Blond</td>
      <td>-99.0</td>
      <td>NBC - Heroes</td>
      <td>None</td>
      <td>good</td>
      <td>-99.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>7</td>
      <td>Adam Strange</td>
      <td>Male</td>
      <td>blue</td>
      <td>Human</td>
      <td>Blond</td>
      <td>185.0</td>
      <td>DC Comics</td>
      <td>None</td>
      <td>good</td>
      <td>88.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8</td>
      <td>Agent 13</td>
      <td>Female</td>
      <td>blue</td>
      <td>None</td>
      <td>Blond</td>
      <td>173.0</td>
      <td>Marvel Comics</td>
      <td>None</td>
      <td>good</td>
      <td>61.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>9</td>
      <td>Agent Bob</td>
      <td>Male</td>
      <td>brown</td>
      <td>Human</td>
      <td>Brown</td>
      <td>178.0</td>
      <td>Marvel Comics</td>
      <td>None</td>
      <td>good</td>
      <td>81.0</td>
    </tr>
  </tbody>
</table>
</div>



1) Cuál es el nombre del primer superhéroe de la tabla heroes?


```python
duck_query(
    """
    SELECT name
    FROM heroes_information
    LIMIT 1;
    """
)
```




    ┌─────────┐
    │  name   │
    │ varchar │
    ├─────────┤
    │ A-Bomb  │
    └─────────┘



2) Cuántas casas publicadoras existen?


```python
duck_query(
    """
    SELECT COUNT(DISTINCT Publisher) AS 'Conteo Casas' 
    FROM heroes_information
    """
)
```




    ┌──────────────┐
    │ Conteo Casas │
    │    int64     │
    ├──────────────┤
    │           24 │
    └──────────────┘



3) Cuántos superhéroes miden más de 2 metros (200 cms)


```python
duck_query(
    """
    SELECT COUNT(name) AS 'Superheroes 2m'
    FROM heroes_information
    WHERE height > 200
    """
)
```




    ┌────────────────┐
    │ Superheroes 2m │
    │     int64      │
    ├────────────────┤
    │             59 │
    └────────────────┘



4) Cuántos superhéroes son humanos y miden más de 2 metros (200 cms)?


```python
duck_query(
    """
    SELECT COUNT(name) AS 'Superheroes Humanos y 2m'
    FROM heroes_information
    WHERE height > 200
    AND Race='Human'
    """
)

```




    ┌──────────────────────────┐
    │ Superheroes Humanos y 2m │
    │          int64           │
    ├──────────────────────────┤
    │                       12 │
    └──────────────────────────┘



5) Cuántos superhéroes pesan más de 100 lbs y menos de 200 lbs?


```python
duck_query(
    """
    SELECT COUNT(name) AS 'Superheroes 100lb - 200lb'
    FROM heroes_information
    WHERE Weight BETWEEN 100 AND 200
    """
)

```




    ┌───────────────────────────┐
    │ Superheroes 100lb - 200lb │
    │           int64           │
    ├───────────────────────────┤
    │                        98 │
    └───────────────────────────┘



6) Cuántos superhéroes tienen los ojos de color azul o rojo?


```python
duck_query(
    """
    SELECT COUNT(name) AS 'Ojos verdes o azules'
    FROM heroes_information
    WHERE "Eye color" ='green'
    OR "Eye color" ='blue'   
    """
)
```




    ┌──────────────────────┐
    │ Ojos verdes o azules │
    │        int64         │
    ├──────────────────────┤
    │                  298 │
    └──────────────────────┘



7) Cuántos superhéroes son Human, Mutant y tienen el pelo color Verde o son Vampiros con pelo Negro


```python
duck_query(
    """
    SELECT COUNT(name)
    FROM heroes_information
    WHERE (Race='Human' OR Race='Mutant') AND "Hair color"='green' 
    OR (Race='Vampire' AND "Hair color"='Black')
      
    """
)
```




    ┌───────────────┐
    │ count("name") │
    │     int64     │
    ├───────────────┤
    │             1 │
    └───────────────┘



8) Cuál es el nombre del primer superhéroe si ordeno la tabla por raza en orden descendente?


```python
duck_query(
    """
    SELECT Name 
    FROM heroes_information
    ORDER BY Race DESC
    LIMIT 1
    """
)
```




    ┌────────────────┐
    │      name      │
    │    varchar     │
    ├────────────────┤
    │ Solomon Grundy │
    └────────────────┘



9) Cuántos superhéroes son de género masculino y cuántas de género femenino?


```python
duck_query(
    """
    SELECT COUNT(Name) AS 'Males'
    FROM heroes_information
    WHERE gender='Male'
    """
)
```




    ┌───────┐
    │ Males │
    │ int64 │
    ├───────┤
    │   505 │
    └───────┘




```python
duck_query(
    """
    SELECT COUNT(Name) AS 'Female'
    FROM heroes_information
    WHERE gender='Female'
    """
)
```




    ┌────────┐
    │ Female │
    │ int64  │
    ├────────┤
    │    200 │
    └────────┘



10) Cuántas casas publicadoras tienen más de 15 superhéroes?


```python
duck_query(
    """
    SELECT COUNT(Publisher)
        FROM (
            SELECT Publisher
            FROM heroes_information
            GROUP BY Publisher
            HAVING COUNT(name) > 15
    ) AS subquery;

    """
)

```




    ┌──────────────────┐
    │ count(Publisher) │
    │      int64       │
    ├──────────────────┤
    │                4 │
    └──────────────────┘




```python

```
