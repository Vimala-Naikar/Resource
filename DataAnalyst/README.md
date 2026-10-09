# Data Ingestion, Extraction, and Exporting Pipeline

A comprehensive collection of practical examples and reference code for importing, exporting, fetching via APIs, and web scraping data using **Python**, **Pandas**, **SQLAlchemy**, **MySQL**, and **BeautifulSoup**.

---

## Table of Contents

1. [Overview](https://www.google.com/search?q=%23overview)
2. [Prerequisites & Requirements](https://www.google.com/search?q=%23prerequisites--requirements)
3. [Reading Data](https://www.google.com/search?q=%23reading-data)
* [CSV and Delimited Files](https://www.google.com/search?q=%23csv-and-delimited-files)
* [Excel Files](https://www.google.com/search?q=%23excel-files)
* [JSON Files & Web URLs](https://www.google.com/search?q=%23json-files--web-urls)
* [SQL Databases (XAMPP / MySQL)](https://www.google.com/search?q=%23sql-databases-xampp--mysql)


4. [Exporting Data](https://www.google.com/search?q=%23exporting-data)
* [CSV](https://www.google.com/search?q=%23csv)
* [Excel (Single & Multi-Sheet)](https://www.google.com/search?q=%23excel-single--multi-sheet)
* [HTML & JSON](https://www.google.com/search?q=%23html--json)
* [SQL Databases](https://www.google.com/search?q=%23sql-databases)


5. [Fetching Data via REST APIs](https://www.google.com/search?q=%23fetching-data-via-rest-apis)
6. [Web Scraping with BeautifulSoup](https://www.google.com/search?q=%23web-scraping-with-beautifulsoup)
7. [Workflow Best Practices](https://www.google.com/search?q=%23workflow-best-practices)

---

## Overview

This repository serves as a guide for data extraction, transformation, and storage[cite: 1]. It covers techniques to optimize memory usage using chunking[cite: 1], fetch paginated data from REST APIs[cite: 1], set up MySQL connections via XAMPP[cite: 1], and perform automated web scraping with standard HTTP user headers[cite: 1].

---

## Prerequisites & Requirements

Install the required Python packages before running the scripts:

```bash
pip install pandas requests bs4 lxml mysql-connector-python pymysql sqlalchemy openpyxl

```

---

## Reading Data

### CSV and Delimited Files

To handle text files or delimiter-separated files (e.g., TSV, custom separators like `;` or `:`):

```python
import pandas as pd

# Load standard CSV
df_csv = pd.read_csv('data.csv')

# Load custom delimited text files
df_txt = pd.read_csv('test.txt')  # Adjust sep parameter if using non-comma delimiters

# Use 'chunksize' parameter to reduce RAM overhead when loading large files
chunk_iter = pd.read_csv('large_dataset.csv', chunksize=10000)
for chunk in chunk_iter:
    # Process each chunk individually
    pass

```

### Excel Files

```python
# Read a specific sheet and set an index column
df_excel = pd.read_excel('output.xlsx', index_col='Unnamed: 0', sheet_name='sheet2')

```

### JSON Files & Web URLs

```python
# Load local JSON file
df_json_local = pd.read_json('train.json')

# Load JSON directly from a URL
df_json_url = pd.read_json('https://api.example.com/data.json')

```

### SQL Databases (XAMPP / MySQL)

1. **XAMPP Setup:**
* Launch XAMPP Control Panel[cite: 1].
* Start **Apache** and **MySQL** modules[cite: 1].
* Navigate to `http://localhost/phpmyadmin/` in your web browser[cite: 1].
* Create a new database and import your `.sql` file[cite: 1].


2. **Querying via Python:**

```python
import mysql.connector
import pandas as pd

# Establish connection
conn = mysql.connector.connect(
    host='localhost',  # Use IP address if connecting to a remote server
    user='root',
    password='',
    database='your_database_name'
)

# Read query directly into a Pandas DataFrame
df_sql = pd.read_sql_query("SELECT * FROM city WHERE CountryCode LIKE 'USA'", conn)
conn.close()

```

---

## Exporting Data

### CSV

```python
# Example aggregation before exporting
temp = df.groupby('batman')['batman_runs'].sum().reset_index()

# Export without the DataFrame index column
temp.to_csv('output.csv', index=False)

```

### Excel (Single & Multi-Sheet)

```python
# Export a single DataFrame sheet
df.to_excel('output.xlsx', sheet_name='Summary')

# Export multiple DataFrames to a single Excel workbook using ExcelWriter
with pd.ExcelWriter('multi_sheet_output.xlsx') as writer:
    temp.to_excel(writer, sheet_name='Sheet_1', index=False)
    temp2.to_excel(writer, sheet_name='Sheet_2', index=False)

```

### HTML & JSON

```python
# Export to HTML string/file
html_data = df.to_html()

# Unstack and export to JSON format
df.unstack().to_json('unstacked_output.json')

```

### SQL Databases

```python
from sqlalchemy import create_engine
import pandas as pd

# Create SQLAlchemy Engine: "mysql+pymysql://{user}:{password}@{host}/{database}"
engine = create_engine("mysql+pymysql://root:@localhost/ipl")

# Export DataFrames to SQL table
df.to_sql('ipl_delivery', con=engine, if_exists='append', index=False)
temp.to_sql('batman', con=engine, if_exists='append', index=False)

```

---

## Fetching Data via REST APIs

Example using **TMDB API** with pagination handling:

```python
import requests
import pandas as pd

# Single Request Example
api_url = "https://api.themoviedb.org/3/movie/popular?api_key=YOUR_API_KEY"
response = requests.get(api_url)
data = response.json()['results']
df_movies = pd.DataFrame(data)[['id', 'title', 'overview', 'release_date', 'popularity', 'vote_average']]

# Multi-Page Ingestion Example
all_movies_list = []

for page in range(1, 429):
    url = f"https://api.themoviedb.org/3/movie/popular?api_key=YOUR_API_KEY&page={page}"
    res = requests.get(url)
    if res.status_code == 200:
        page_data = res.json().get('results', [])
        temp_df = pd.DataFrame(page_data)[['id', 'title', 'overview', 'release_date', 'popularity', 'vote_average']]
        all_movies_list.append(temp_df)

# Concatenate all dataframes efficiently
final_movies_df = pd.concat(all_movies_list, ignore_index=True)
final_movies_df.to_csv('movies.csv', index=False)

```

---

## Web Scraping with BeautifulSoup

Example scraping company listings from AmbitionBox with User-Agent request headers:

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.162 Safari/537.36'
}

scraped_data_list = []

for page in range(1, 11):
    url = f"https://www.ambitionbox.com/list-of-companies?pages={page}"
    webpage = requests.get(url, headers=headers)
    soup = BeautifulSoup(webpage.text, 'lxml')
    
    companies = soup.find_all('div', class_='company-content-wrapper')
    
    for company in companies:
        name = company.find('h2').text.strip() if company.find('h2') else None
        rating = company.find('p', class_='rating').text.strip() if company.find('p', class_='rating') else None
        reviews = company.find('a', class_='review-count').text.strip() if company.find('a', class_='review-count') else None
        
        scraped_data_list.append({
            'name': name,
            'rating': rating,
            'reviews': reviews
        })

df_companies = pd.DataFrame(scraped_data_list)
df_companies.to_csv('companies_scraped.csv', index=False)

```

---

## Workflow Best Practices

1. **RAM Optimization:** Use `chunksize` in `pd.read_csv()`[cite: 1] or filter required columns upfront using `usecols` to handle memory limits efficiently.
2. **DataFrame Concatenation:** Avoid using `.append()` inside loops (deprecated in recent Pandas versions). Collect DataFrames in a list and call `pd.concat(list, ignore_index=True)`[cite: 1].
3. **Database Connections:** Prefer `sqlalchemy.create_engine` over raw driver connections when writing DataFrames to SQL via `.to_sql()`[cite: 1].
4. **Header Spoofing:** Always pass custom HTTP headers (e.g., `User-Agent`) during web scraping to avoid getting blocked by anti-bot controls[cite: 1].
