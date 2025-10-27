# Chula-Scopus-data

## Extracting the data out of the zip
```python 
import zipfile
import json
import sqlite3
import pandas as pd

conn = sqlite3.connect("scopus.db")
cur = conn.cursor()

cur.execute("""
CREATE TABLE IF NOT EXISTS papers_raw (
    file_id INTEGER,
    year INTEGER,
    raw_json TEXT
)
""")

conn.commit()
conn.close()

keep = "ScopusData2018-2023/2018/201800000"
zip_path = "ScopusData2018-2023.zip"
years = ["2018", "2019", "2020", "2021", "2022", "2023"]

def loader(year: int, start_id: int, end_id: int):
    conn = sqlite3.connect("scopus.db")
    cur = conn.cursor()

    with zipfile.ZipFile(zip_path, "r") as z:
        for file_id in range(start_id, end_id + 1):
            inner_path = f"ScopusData2018-2023/{year}/{file_id}"
            try:
                with z.open(inner_path) as f:
                    try:
                        obj = json.load(f)
                    except Exception:
                        continue
            except KeyError:
                continue
            raw_text = json.dumps(obj, ensure_ascii=False)
            cur.execute(
                """
                INSERT INTO papers_raw (file_id, year, raw_json)
                VALUES (?, ?, ?)
                """,
                (file_id, year, raw_text)
            )
    conn.commit()
    conn.close()
```


```python 
# 2018
loader(2018, 201800000, 201802761)

# 2019
loader(2019, 201900000, 201903081)

# 2020
loader(2020, 202000000, 202003392)

# 2021
loader(2021, 202100000, 202103814)

# 2022
loader(2022, 202200000, 202204243)

# 2023
loader(2023, 202300000, 202302889)
```

## Run the first two part of the code first to create a Database and the reason for this is because its much faster than saving in a list or dictionary. (I don't have to load 5.63 GB from extracting the zip file)

```python 
conn = sqlite3.connect("scopus.db")

df_2018 = pd.read_sql_query(
    """
    SELECT file_id, year, raw_json
    FROM papers_raw
    WHERE year = 2018
    """,
    conn
)

conn.close()

print(df_2018.shape)
print(df_2018.head())
```
