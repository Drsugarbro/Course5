#Q1

!pip install yfinance

import yfinance as yf

gme_ticker = yf.Ticker("GME")

gme_data = gme_ticker.history(period="max")
gme_data.reset_index(inplace=True)
print(gme_data.head())




#Q2
import requests
from bs4 import BeautifulSoup
import pandas as pd

# Task 1: Download webpage
url = "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0220EN-SkillsNetwork/labs/project/stock.html"
html_data_2 = requests.get(url).text

# Task 2: Parse HTML
soup = BeautifulSoup(html_data_2, 'html.parser')

# Task 3: Extract and clean table
table = soup.find_all('table')[1]

# Second table contains GameStop data
gme_revenue = pd.read_html(str(table))[0]
gme_revenue.columns = ['Date', 'Revenue']
# Fix for potential extra backslash and missing closing parenthesis
gme_revenue['Revenue'] = gme_revenue['Revenue'].str.replace(',|\\$', '', regex=True)
gme_revenue['Revenue'] = pd.to_numeric(gme_revenue['Revenue'], errors='coerce')
gme_revenue.dropna(inplace=True)

# Task 4: Display last five rows
print(gme_revenue.tail())




#Q5：
import pandas as pd
import yfinance as yf
import plotly.graph_objects as go
from plotly.subplots import make_subplots
from IPython.display import display, HTML

# Step 1: Define the graph function
def make_graph(stock_data, revenue_data, stock_name):
    fig = make_subplots(rows=2, cols=1, shared_xaxes=True,
                        subplot_titles=("Historical Share Price", "Historical Revenue"),
                        vertical_spacing=0.3)

    # Filter data up to June 2021
    stock_data_specific = stock_data[stock_data['Date'] <= '2021-06-30']
    revenue_data_specific = revenue_data[revenue_data['Date'] <= '2021-06-30']

    # Add share price trace
    fig.add_trace(go.Scatter(
        x=pd.to_datetime(stock_data_specific['Date']),
        y=stock_data_specific['Close'],
        mode='lines',
        name='Share Price'
    ), row=1, col=1)

    # Add revenue trace
    fig.add_trace(go.Scatter(
        x=pd.to_datetime(revenue_data_specific['Date']),
        y=revenue_data_specific['Revenue'],
        mode='lines',
        name='Revenue'
    ), row=2, col=1)

    # Update axes
    fig.update_xaxes(title_text="Date", row=1, col=1)
    fig.update_xaxes(title_text="Date", row=2, col=1)
    fig.update_yaxes(title_text="Price ($US)", row=1, col=1)
    fig.update_yaxes(title_text="Revenue ($US Millions)", row=2, col=1)

    # Layout
    fig.update_layout(
        showlegend=False,
        height=900,
        title=f"{stock_name} Historical Share Price and Revenue (Up to June 2021)",
        xaxis_rangeslider_visible=True
    )

    fig.show()
    display(HTML(fig.to_html()))

# Step 2: Download Tesla stock data using yfinance (2010 - mid 2021)
tesla = yf.Ticker("TSLA")
stock_data = tesla.history(start="2010-01-01", end="2021-06-30")
stock_data.reset_index(inplace=True)
stock_data = stock_data[['Date', 'Close']]

# Step 3: Mocked revenue data (manual quarterly values)
tesla_revenue = pd.DataFrame({
    "Date": pd.to_datetime([
        "2010-03-31", "2010-06-30", "2010-09-30", "2010-12-31",
        "2011-03-31", "2011-06-30", "2011-09-30", "2011-12-31",
        "2012-03-31", "2012-06-30", "2012-09-30", "2012-12-31",
        "2013-03-31", "2013-06-30", "2013-09-30", "2013-12-31",
        "2014-03-31", "2014-06-30", "2014-09-30", "2014-12-31",
        "2015-03-31", "2015-06-30", "2015-09-30", "2015-12-31",
        "2016-03-31", "2016-06-30", "2016-09-30", "2016-12-31",
        "2017-03-31", "2017-06-30", "2017-09-30", "2017-12-31",
        "2018-03-31", "2018-06-30", "2018-09-30", "2018-12-31",
        "2019-03-31", "2019-06-30", "2019-09-30", "2019-12-31",
        "2020-03-31", "2020-06-30", "2020-09-30", "2020-12-31",
        "2021-03-31"
    ]),
    "Revenue": [
        21, 28, 31, 36,
        49, 58, 57, 39,
        30, 27, 50, 306,
        561, 405, 431, 615,
        713, 769, 852, 956,
        939, 955, 937, 1214,
        1147, 1270, 2298, 2284,
        2696, 2790, 2985, 3288,
        3409, 4011, 6824, 7212,
        4541, 6350, 6303, 7384,
        5985, 6036, 8771, 10744,
        10389  # All in millions USD
    ]
})

# Step 4: Make the graph
make_graph(stock_data, tesla_revenue, "Tesla")
