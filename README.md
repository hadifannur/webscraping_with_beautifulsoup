# webscraping_with_beautifulsoup

# Introduction
Web scraping is used for extraction of relevant data from web pages. If we require some data from a web page in a public domain, web scraping makes the process of data extraction quite convenient. The use of web scraping requires some basic knowledge of the structure of HTML pages. 

# Objective
By the end of this, I will be able to:
- Use requests and BeautifulSoup libraries to extract the contents of a web page
- Analyze the HTML code of a webpage to find the relevant information
- Extract the relevant information and save it in the required form

# Scenario
I have been hired by a Multiplex management organization to extract the information of the top 50 movies with the best average rating from the web link shared below.
```python
https://web.archive.org/web/20230902185655/https://en.everybodywiki.com/100_Most_Highly-Ranked_Films
```
The information required is Average Rank, Film, and Year. I am required to write a Python script webscraping_movies.py that extracts the information and saves it to a csv file top_50_fim.csv. I am also required to save the same information to a database Movies.db under the table name Top_50.

# Initial steps
I require the following libraries for this mission.
1. pandas library for data storage and manipulation.
2. BeautifulSoup library for interpreting the HTML document.
3. requests library to communicate with the web page.
4. SQLite3 for creating the database instance
While requests and sqlite3 come bundled with Python3, I need to install pandas and BeautifulSoup libraries to the IDE.
For this, I ran the following commands in a terminal window.
```bash
python3 -m pip install pandas
python3 -m pip install bs4
```
Now, I have created a new file by the name of webscraping_movies.py in the path /home/projects/webscrap/ .
I will write all of my code in this file

# Importing Libraries
I import the following four libraries by adding lines of code noted below to my webscraping_movies.py file.
```python
import pandas as pd
import sqlite3
import requests
from bs4 import BeautifulSoup
```

# Initialization of known entities
I must declare a few entities at the beginning. I know the required URL, the CSV name for saving the record, the database name, and the table name for storting the record. I also know the entities to be saved. Additionally, since I require only the top 50 results, I will require a loop counter initialized to 0. 
```python
url = 'https://web.archive.org/web/20230902185655/https://en.everybodywiki.com/100_Most_Highly-Ranked_Films'
db_name = 'Movies.db'
table_name = 'Top_50'
csv_path = 'top_50_films.csv'
df = pd.DataFrame(columns=["Average Rank", "Film", "Year"])
count = 0
```

# Loading the webpage for Webscraping
To access the required information from the web page, I first need to load the entire web page as an HTML document in python using the requests.get().text function and then parse the text in the HTML format using BeautifulSoup to enable extraction of relevant information.
```python
html_page = requests.get(url).text
data = BeautifulSoup(html_page, 'html.parser')
```

# Analyze the webpage
I open the web page in a browser and locate the required table by scrolling down to it. I enter the inspect panel by clicking fn + f12, then I search for 'td'. 
I notice that all rows under this table are mentioned as tr objects under the table. Clicking one of them would show that the data in each row is further saved as td object, as seen in the image below. I require the information under the first three headers of this stored data.
It is also important to note that this is the first table on the page. I must identify the required table when extracting information.

# Scraping of the required information
I now need to write a loop to extract the appropriate information from the web page. The rows of the table needed can be accessed using the find_all() function with the BeautifulSoup object using the statements below.
```python
tables = data.find_all('tbody')
rows = tables[0].find_all('tr')
```
Here, the variable tables gets the body of all the tables on the web page and the variable rows gets all the rows of the first table. I can now iterate over the rows to find the required data.
```python
for row in rows:
    if count < 50:
        col = row.find_all('td')
        if len(col) != 0:
            data_dict = {"Average Rank": col[0].contents[0],
                         "Film": col[1].contents[0],
                         "Year": col[2].contets[0]}
            df1 = pd.DataFrame(data_dict, index=[0])
            df = pd.concat([df,df1], ignore_index=True)
            count+=1
    else:
        break
```
The code functions as follows:
1. Iterate over the contents of the variable rows.
2. Check for the loop counter to restrict it to 50 entries.
3. Extract all the td data objects in the row and save them to col.
4. Check if the length of col is 0, that is, if there is no data in the current row. This is important since, many times, there are merged rows that are not apparent in the web page appearance.
5. Create a dictionary data_dict with keys, same as the columns of the dataframe created for recording the output earlier and corresponding values from the first three headers of data.
6. Convert the dictionary to a dataframe and concatenate it with the existing one. This way, the data keeps getting appended to the dataframe with every iteration of the loop
7. Increment the loop counter.
8. Once the counter hits 50, stop iterating over rows and break the loop
Print the contents of the dataframe using the following:
```python
print(df)
```

# Storing the data
After the dataframe has been created, I can save it to a CSV file using the following command:
```python
df.to_csv(csv_path)
```

To store the required data in a database, I first need to initialize a connection to the database, save the dataframe as a table, and then close the connection. This can be done using the following code:
```python
conn = sqlite3.connect(db_name)
df.to_sql(table_name, conn, if_exists='replace', index=False)
conn.close()
```
This database can now be used to retrieve the relevant information using SQL queries.

# Execute the code
```python
python3 webscraping_movies.py
```
