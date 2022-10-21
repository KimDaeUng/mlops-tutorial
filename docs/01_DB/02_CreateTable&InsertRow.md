# 목표

- 앞서 띄운 postgres 서버에 테이블을 생성하고 row를 삽입해 봅시다.

# 요구사항

1. Python의 `psycopg2` 패키지를 이용합니다
    - `pip install psycopg2-binary`
2. psycopg2 를 사용해 `iris_data` table을 만들어 봅시다.
    |id|sepal length(cm)|sepal width(cm)|petal length(cm)|petal width(cm)|target|
    |------|---|---|---|---|---|---|---|
    serial(primary key)|float|float|float|float|int|

- 다음 내용을 참고
  - float64 dtype 처리: double precision
    - 15 decimal digits precision
    - 1 sign bit / 11 bits exponent / 52 bits mantissa
    - [ref](https://jakevdp.github.io/PythonDataScienceHandbook/02.01-understanding-data-types.html)
    - [ref](https://www.postgresql.org/docs/current/datatype-numeric.html)
  - column naming convention
    - must begin with a letter (`a-z`) or underscore(`_`).
    - subsequent characters in a name can be letter, digit, or underscores.
    - maximum length: `NAMEDATALEN` - 1, By default `NAMEDATALEN`' is 32.
    - non case-sensitive.
    - if quoted, Quoting a name makes it case-sensitive, and also make it possible to contain disallowed characters such as spaces, ampersands, etc.
      - e.g `FOO` and `foo` and `"foo"` are considered the same by Postgres. but `"Foo"` is different name.
  - 이미 테이블이 있는 경우: `DuplicateTable: relation "iris_data" already exists` error
  -> IF 

3. psycopg2 를 사용해 iris 데이터 하나를 삽입해 봅시다.
4. `psql` 등을 이용해 생성한 테이블과 삽입한 데이터를 확인합니다.


# Solution
- M1 Mac 기준

0. `psycopg2` 설치
```bash
pip install psycopg2-binary
```

1. psycopg2 를 사용해 iris 데이터를 이용해 table을 만들고 데이터 하나 삽입
```python
import psycopg2
import pandas as pd
import numpy as np

from psycopg2.extensions import ISOLATION_LEVEL_AUTOCOMMIT
from sklearn import datasets

conn = psycopg2.connect(user='postgres',
    password='mypassword',
    host='localhost',
    port=5432,
)
conn.set_isolation_level(ISOLATION_LEVEL_AUTOCOMMIT)
cur = conn.cursor()

iris = datasets.load_iris()
df = pd.DataFrame(
    data=iris.data,
    columns=[
        "sepal length (cm)",
        "sepal width (cm)",
        "petal length (cm)",
        "petal width (cm)",
    ]
)
df["target"] = iris.target

# Create Table
sql_create_table = f"""CREATE TABLE iris_data (
  id serial PRIMARY KEY,
  \"sepal length (cm)\" DOUBLE PRECISION NOT NULL,
  \"sepal width (cm)\" DOUBLE PRECISION NOT NULL,
  \"petaal length (cm)\" DOUBLE PRECISION NOT NULL,
  \"petaal width (cm)\" DOUBLE PRECISION NOT NULL,
  target INT NOT NULL
);"""
cur.execute(sql_create_table)

# Insert one row to the table
sql_insert_table = f"""INSERT INTO {table_name} (
  \"sepal length (cm)\",
  \"sepal width (cm)\",
  \"petaal length (cm)\",
  \"petaal width (cm)\",
  target
) VALUES (%s, %s, %s, %s, %s)"""
record_to_insert = df.iloc[0].values
cur.execute(sql_insert_table, record_to_insert)
conn.commit()

count = cur.rowcount
print(count, "Record inserted successfully into postgres.iris_data")
```

3. `psql`를 이용해 테이블과 삽입된 데이터 확인

```bash
psql -h localhost -U Postgres -p 5432
```

테이블 확인: `\dt`
```bash
postgres=# \dt
           List of relations
 Schema |   Name    | Type  |  Owner   
--------+-----------+-------+----------
 public | iris_data | table | postgres
(1 row)
```

삽입된 데이터 확인: `SELECT * FROM iris_data;`
```bash
postgres=# SELECT * FROM iris_data;
 id | sepal length (cm) | sepal width (cm) | petaal length (cm) | petaal width (cm) | target 
----+-------------------+------------------+--------------------+-------------------+--------
  1 |               5.1 |              3.5 |                1.4 |               0.2 |      0
(1 row)
```

테이블 상세 확인: `\d+ iris_data;`
``` bash
postgres=# \d+ iris_data;
                                                                 Table "public.iris_data"
       Column       |       Type       | Collation | Nullable |                Default                | Storage | Compression | Stats target | Description 
--------------------+------------------+-----------+----------+---------------------------------------+---------+-------------+--------------+-------------
 id                 | integer          |           | not null | nextval('iris_data_id_seq'::regclass) | plain   |             |              | 
 sepal length (cm)  | double precision |           | not null |                                       | plain   |             |              | 
 sepal width (cm)   | double precision |           | not null |                                       | plain   |             |              | 
 petaal length (cm) | double precision |           | not null |                                       | plain   |             |              | 
 petaal width (cm)  | double precision |           | not null |                                       | plain   |             |              | 
 target             | integer          |           | not null |                                       | plain   |             |              | 
Indexes:
    "iris_data_pkey" PRIMARY KEY, btree (id)
Access method: heap
```