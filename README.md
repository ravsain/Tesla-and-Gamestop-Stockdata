<h1>Tesla and Gamestop Stock Data</h1>



```python
!pip install yfinance
#!pip install pandas
#!pip install requests
!pip install bs4
#!pip install plotly
```


```python
import yfinance as yf
import pandas as pd
import requests
from bs4 import BeautifulSoup
import plotly.graph_objects as go
from plotly.subplots import make_subplots
```

## Define Graphing Function


Used plotly for graphs and made this function `make_graph`.



```python
def make_graph(stock_data, revenue_data, stock):
    fig = make_subplots(rows=2, cols=1, shared_xaxes=True, subplot_titles=("Historical Share Price", "Historical Revenue"), vertical_spacing = .3)
    fig.add_trace(go.Scatter(x=pd.to_datetime(stock_data.Date, infer_datetime_format=True), y=stock_data.Close.astype("float"), name="Share Price"), row=1, col=1)
    fig.add_trace(go.Scatter(x=pd.to_datetime(revenue_data.Date, infer_datetime_format=True), y=revenue_data.Revenue.astype("float"), name="Revenue"), row=2, col=1)
    fig.update_xaxes(title_text="Date", row=1, col=1)
    fig.update_xaxes(title_text="Date", row=2, col=1)
    fig.update_yaxes(title_text="Price ($US)", row=1, col=1)
    fig.update_yaxes(title_text="Revenue ($US Millions)", row=2, col=1)
    fig.update_layout(showlegend=False,
    height=900,
    title=stock,
    xaxis_rangeslider_visible=True)
    fig.show()
```

## Using yfinance to Extract Stock Data


Using the `Ticker` function, and the ticker symbol to extract the stock data. The stock is Tesla and its ticker symbol is `TSLA`.



```python
Tesla = yf.Ticker("TSLA")

```

Using the ticker object and the function `history`, extract stock information and save it in a dataframe named `tesla_data`.  The `period` parameter is set to `max` so I get information for the maximum amount of time.



```python
tesla_data = tesla.history(period="max")
tesla_data

```

**Reset the index** using the `reset_index(inplace=True)`


```python
tesla_data.reset_index(inplace=True)
tesla_data.head()

```
![image](https://github.com/ravsain/Tesla-and-Gamestop-Stock-Data/assets/155238122/8bd93471-b2c6-4caa-8f33-a4adb5d9d041)

## Webscraping to Extract Tesla Revenue Data

By using the `requests` library to download the webpage https://www.macrotrends.net/stocks/charts/TSLA/tesla/revenue And save the text of the response as a variable named `html_data`.

```python
url = "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0220EN-SkillsNetwork/labs/project/revenue.htm"
html_data = requests.get(url).text

```

Parse the html data using `beautiful_soup`.

```python
soup = BeautifulSoup(html_data, "html.parser")

```

Using `BeautifulSoup` or the `read_html` function extract the table with `Tesla Revenue` and store it into a dataframe named `tesla_revenue`. The dataframe should have columns `Date` and `Revenue`.

```python
tesla_revenue = pd.DataFrame(columns = ["Date","Revenue"])

table = soup.find_all("tbody")
table_2 = table[1]

for row in table_2.find_all("tr"):
    col = row.find_all("td")
    date = col[0].text
    revenue = col[1].text
    
    tesla_revenue = tesla_revenue.append({"Date":date, "Revenue":revenue}, ignore_index=True)
tesla_revenue

```
![image](https://github.com/ravsain/Tesla-and-Gamestop-Stock-Data/assets/155238122/008feb88-bc17-4903-b686-9ff48450c82a)

Execute the following line to remove the comma and dollar sign from the `Revenue` column, and to remove an null or empty strings in the Revenue column.

```python
tesla_revenue["Revenue"] = tesla_revenue['Revenue'].str.replace(',|\$',"")

tesla_revenue.dropna(inplace=True)
tesla_revenue = tesla_revenue[tesla_revenue['Revenue'] != ""]

```

```python
tesla_revenue.tail()

```
![image](https://github.com/ravsain/Tesla-and-Gamestop-Stock-Data/assets/155238122/15a9ecc2-c3ca-4924-b684-c50523ccdd36)

Use the `make_graph` function to graph the Tesla Stock Data, also provide a title for the graph. The structure to call the `make_graph` function is `make_graph(tesla_data, tesla_revenue, 'Tesla')`. Note the graph will only show data upto June 2021.

```python
make_graph(tesla_data, tesla_revenue, 'Tesladata')

```
![image](https://github.com/ravsain/Tesla-and-Gamestop-Stock-Data/assets/155238122/941c45b7-757a-4d21-ae40-f71e7d42ca28)

***Same code for other stock data as well***

##  Gamestop

ticker symbol is `GME`
URL : https://www.macrotrends.net/stocks/charts/GME/gamestop/revenue

all Codes:

```python

#yfinance api
gamestop = yf.Ticker("GME")

gme_data = gamestop.history(period="max")
gme_data

gme_data.reset_index(inplace = True)
gme_data.head()

#requests url
url = "https://www.macrotrends.net/stocks/charts/GME/gamestop/revenue"
html_data = requests.get(url).text

#parse html_data
soup = BeautifulSoup(html_data,"html.parser") 

df=pd.read_html(html_data,header=0) #read_html function
table = soup.find_all("table")[1]
gme_revenue = pd.DataFrame(columns =["Date","Revenue"])
for row in table.find("tbody").find_all("tr"):
    col = row.find_all("td")
    date = col[0].text
    revenue = col[1].text
    gme_revenue = gme_revenue.append({"Date":date, "Revenue":revenue}, ignore_index = True)

gme_revenue["Revenue"]= gme_revenue['Revenue'].str.replace('$','').str.replace(',','')   #remove $ sign
gme_revenue.dropna(subset=['Revenue'], inplace=True)  #drop blacnk rows
for i in gme_revenue : gme_revenue[i] = gme_revenue[i].astype(str)
print(gme_revenue)

#last five rows
gme_revenue.tail()

#Graph
make_graph(gme_data, gme_revenue, 'GameStopdata')
```
    
  





