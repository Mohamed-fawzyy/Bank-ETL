# Project Scenario 🎩
This project requires you to compile the list of the top 10 largest banks in the world ranked by market capitalization in billion USD. Further, you need to transform the data and store it in USD, GBP, EUR, and INR per the exchange rate information made available to you as a CSV file. You should save the processed information table locally in a CSV format and as a database table. Managers from different countries will query the database table to extract the list and note the market capitalization value in their own currency.

# Reach/Follow me on <br>
<p align="left">
  <a href="https://www.linkedin.com/in/mohamed-fawzy-936b661b8/" target="_blank" rel="noreferrer"> <img src="https://img.icons8.com/fluency/2x/linkedin.png" alt="linkedIn" width="50" height="50"/> </a>&nbsp&nbsp
  <a href="mailto:fwzymohamed90@gmail.com" target="_blank" rel="noreferrer"> <img src="https://img.icons8.com/fluency/2x/google-logo.png" alt="googleEmail" width="50" height="50"/> </a>&nbsp&nbsp
  <a href="https://www.facebook.com/mohamed.fwzy.14" target="_blank" rel="noreferrer"> <img src="https://cdn.iconscout.com/icon/free/png-256/facebook-262-721949.png" alt="facebook" width="50" height="50"/> </a>
</p>
<br>

# Objectives📝
* You have to complete the following tasks for this project
  - Write a data extraction function to retrieve the relevant information from the required URL.
  - Transform the available GDP information into 'Billion USD' from 'Million USD'.
  - Load the transformed information to the required CSV file and as a database file.
  - Run the required query on the database.
  - Log the progress of the code with appropriate timestamps.

## 📦 Install

- First of all, install the required Libraries for ETL:

```bash
pip3 install pandas
```

```bash
pip3 install beautifulsoup4
```

## Implementation
- After installation, we go through the code to discuss the progress briefly

- then, we need to initialize our libraries:
```python
import pandas as pd 
import numpy as np 
import requests
import sqlite3
from bs4 import BeautifulSoup
from datetime import datetime
```  

## Directions 🗺
1. This function extracts the tabular information from the given URL under the heading By Market Capitalization by using bs4 and saves it to a data frame.
```python
def extract(url, table_attribs):
    df = pd.DataFrame(columns = table_attribs)

    page = requests.get(url).text
    data = BeautifulSoup(page, 'html.parser')

    tables = data.find_all('tbody')[0]
    rows = tables.find_all('tr')

    for row in rows:
        col = row.find_all('td')
        if len(col) != 0:
            ancher_data = col[1].find_all('a')[1]
            if ancher_data is not None:
                data_dict = {
                    'Name': ancher_data.contents[0],
                    'MC_USD_Billion': col[2].contents[0]
                }
                df1 = pd.DataFrame(data_dict, index = [0])
                df = pd.concat([df, df1], ignore_index = True)

    USD_list = list(df['MC_USD_Billion'])
    USD_list = [float(''.join(x.split('\n'))) for x in USD_list]
    df['MC_USD_Billion'] = USD_list

    return df
```

2. This function transforms the data frame by adding columns for Market Capitalization in GBP, EUR, and INR, rounded to 2 decimal places, based on the exchange rate information shared as a CSV file.
```python
def transform(df, exchange_rate_path):
    csvfile = pd.read_csv(exchange_rate_path)

    # i made here the content for currenct is the keys and the content of 
    # the rate is the values to the crossponding keys
    dict = csvfile.set_index('Currency').to_dict()['Rate']

    df['MC_GBP_Billion'] = [np.round(x * dict['GBP'],2) for x in df['MC_USD_Billion']]
    df['MC_INR_Billion'] = [np.round(x * dict['INR'],2) for x in df['MC_USD_Billion']]
    df['MC_EUR_Billion'] = [np.round(x * dict['EUR'],2) for x in df['MC_USD_Billion']]

    return df
```

3. This function loads the transformed data frame to an output CSV file.
```python
def load_to_csv(df, output_path):
    df.to_csv(output_path)
```

4. This function loads the transformed data frame to an SQL database server as a table.
```python
def load_to_db(df, sql_connection, table_name):
    df.to_sql(table_name, sql_connection, if_exists = 'replace', index = False)
```

5. This function runs queries on the database table.
```python
def run_query(query_statements, sql_connection):
    for query in query_statements:
        print(query)
        print(pd.read_sql(query, sql_connection), '\n')
```

6. This function logs the progress of the code.
```python
def log_progress(msg):
    timeformat = '%Y-%h-%d-%H:%M:%S'
    now = datetime.now()
    timestamp = now.strftime(timeformat)

    with open(logfile, 'a') as f:
        f.write(timestamp + ' : ' + msg + '\n')
```

7. Here, you define the required entities and call the relevant functions in the correct order to complete the project. Note that this portion is not inside any function.
```python
url = 'https://web.archive.org/web/20230908091635/https://en.wikipedia.org/wiki/List_of_largest_banks'
exchange_rate_path = 'exchange_rate.csv'

table_attribs = ['Name', 'MC_USD_Billion']
db_name = 'Banks.db'
table_name = 'Largest_banks'
conn = sqlite3.connect(db_name)
query_statements = [
        'SELECT * FROM Largest_banks',
        'SELECT AVG(MC_GBP_Billion) FROM Largest_banks',
        'SELECT Name from Largest_banks LIMIT 5'
    ]

logfile = 'code_log.txt'
output_csv_path = 'Largest_banks_data.csv'

log_progress('Preliminaries complete. Initiating ETL process.')

df = extract(url, table_attribs)
log_progress('Data extraction complete. Initiating Transformation process.')


df = transform(df, exchange_rate_path)
log_progress('Data transformation complete. Initiating loading process.')

load_to_csv(df, output_csv_path)
log_progress('Data saved to CSV file.')

log_progress('SQL Connection initiated.')


load_to_db(df, conn, table_name)
log_progress('Data loaded to Database as table. Running the query.')

run_query(query_statements, conn)
conn.close()
log_progress('Process Complete.')
```

## Result Snapshots 📸
![Screen Shot 2023-11-16 at 18 15 54 PM](https://github.com/Mohamed-fawzyy/Bank-ETL/assets/111665714/7b28e8ae-44ca-4b77-9dde-6c90f8c56cd7)
![Screen Shot 2023-11-16 at 18 16 37 PM](https://github.com/Mohamed-fawzyy/Bank-ETL/assets/111665714/d174bb70-5a6c-4bcb-aa6c-87a457961305)














