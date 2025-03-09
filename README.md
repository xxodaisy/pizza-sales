# **Pizza Sales Report**

Analysis of Pizza Sales Report based on my personal project

## **Background**

I was requested to do a data analysis on one of the Pizza Place Restaurants. The things that I analyzed were the performance of the restaurant in 2015. 

## **Data Collection**

The dataset I got from Kaggle and already in .csv (comma separated value). Also, this dataset already clean (was cleaning by Kaggle)

The dataset consist are:
1. Orders: contains data customer, such an order ID, and when customers buy pizza (date and time)
2. Order details: what the customer bought
3. Pizzas: menu items of Pizza
4. Pizza types: type of pizza 

## **Exploratory Data Analysis (EDA)**

I analyzed this dataset using Python

Here are some things that I analyzed from this task:
1. How many orders did we get every month? which month has the highest sales?
2. Are there peak hours?
3. What is the most ordered pizza? Compare the number of orders for each type of pizza
4. How does each pizza size contribute to revenue?
5. How does revenue contribute to each type of pizza?

### **Data preparation**
The first step I did was to import all the libraryand read the data with pandas DataFrame

```python
import pandas as pd
pd.set_option('display.max_columns', 50000)
pd.set_option('display.max_rows', 50000)
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
```
After that, I performed a data inspection to do an initial check on the DataFrame
```python
def inspect_data(df, col=None, n_rows=5):
    #check shape data
    print(f'data shape: {df.shape}')
    
    #define column
    if col is None:
        col = df.columns

    #check head data, use the display function to display the DataFrame
    display(df[col].head(n_rows))
```
Also, I checked for missing data
```python
def check_missing(df, cut_off=0, sort=True):
    freq=df.isnull().sum()
    percent=df.isnull().sum()/df.shape[0]*100
    types=df.dtypes
    unique = df.apply(pd.unique).to_frame(name='Unique Values')['Unique Values']
    df_miss=pd.DataFrame({'missing_percentage':percent, 'missing_frequency':freq, 'types':types, 'unique_values':unique})
    if sort: df_miss.sort_values(by='missing_frequency',ascending= False, inplace=True)
    return df_miss[df_miss['missing_percentage'] >= cut_off]
```
I also do the function fillna_by_metric to fill the missing values in a given column in a DataFrame with various metrics such as average, median, mode, zero value, or a given custom value
```python
def fillna_by_metric(df, column_name, metric='mean', custom_value=None):
    #Retrieve metric values according to input
    if metric == 'mean':
        metric_value = df[column_name].mean()
    elif metric == 'median':
        metric_value = df[column_name].median()
    elif metric == 'mode':
        metric_value = df[column_name].mode().iloc[0]
    elif metric == 'zero':
        metric_value = 0
    elif metric == 'custom':
        metric_value = custom_value
    else:
        raise ValueError("Invalid metric type")
    
    #Fill the missing value in the column with the retrieved metric value
    df[column_name].fillna(value=metric_value, inplace=True)
    
    return df
```
Then I entered each data wll be read with pandas DataFrame
```python
#load data pizzas
df_pizzas = pd.read_csv('pizzas.csv')
```

```python
#load data orders
df_orders = pd.read_csv('orders.csv')
```

```python
#load data order details
df_order_details = pd.read_csv('order_details.csv')
```

After that, I tried to inspectthe data of each file with n_rows = 10
```python
#load data pizzas
df_pizzas = pd.read_csv('pizzas.csv')
inspect_data(df_pizzas, n_rows=10)
```
![image](https://github.com/user-attachments/assets/f9bae01f-12af-46c0-82a2-901122dbf481)

```python
#load data order details
df_order_details = pd.read_csv('order_details.csv')
inspect_data(df_order_details, n_rows=10)
```
![image](https://github.com/user-attachments/assets/30f38b15-334a-4b6f-85de-5d206a24538d)

```python
#load data orders
df_order_details = pd.read_csv('orders.csv')
inspect_data(df_orders, n_rows=10)
```
![image](https://github.com/user-attachments/assets/175f5d2a-26bf-4cfd-97f4-b9edaed38e54)

I checked duplicate for each dataset
```python
#check duplicate pizzas
df_pizzas[df_pizzas.duplicated()]
```
![image](https://github.com/user-attachments/assets/ddd13373-dc34-4c74-b676-4db9c970c06c)

```python
#check duplicate orders
df_orders[df_orders.duplicated()]
```
![image](https://github.com/user-attachments/assets/f7fa4dd4-1eb3-46aa-bc43-e85aefa5d2de)

```python
#check duplicate order details
df_order_details[df_order_details.duplicated()]
```
![image](https://github.com/user-attachments/assets/c768e0a9-79dc-460c-a3f1-51fe59bfc1e1)

### **Data Analysis**
1. How many orders did we get every month? which month has the highest sales?

```python
#read data from  orders table
df_orders = pd.read_csv('orders.csv')

#convert a date column to datetime data type
df_orders['date'] = pd.to_datetime(df_orders['date'])

#create a new column with year-month date format (YYYY-mm)
df_orders['tahun-bulan'] = df_orders['date'].dt.strftime('%Y-%m')

#count the number of order_ids by year-month
summary = df_orders['order_id'].groupby(df_orders['tahun-bulan']).count()

#create a line chart using summary data
summary.plot(kind='line')
plt.xlabel('Month')
plt.ylabel('Number of orders')
plt.title('Total Orders by Month')
plt.show()

#search for the summary of the month with the highest order
ringkasan_pesanan_tertinggi = summary.idxmax()
print("Bulan dengan pesanan tertinggi: ", ringkasan_pesanan_tertinggi)
```
![image](https://github.com/user-attachments/assets/d012409a-1f47-4133-88bd-a509d2a69720)

From the data that has been analyzed, the month with the highest order was obtained in July with a total order of around 1900.

2. Are there peak hours?
```python
# read data from orders tabel to Pandas DataFrame
df_orders = pd.read_csv('orders.csv')

# convert date column to datetime data type
df_orders['date'] = pd.to_datetime(df_orders['time'])

# extract hour from date column
df_orders['time'] = df_orders['date'].dt.hour

# calculate the number of order per hours
summary = df_orders.groupby('time').size().reset_index(name='jumlah_order')

# sort data based on number of orders and time
summary = summary.sort_values('time')

summary.plot(x='time', y='jumlah_order', kind='bar')

plt.xlabel('Hour of Day')
plt.ylabel('Number of Orders')
plt.title('Orders by Hour')

plt.show()
```
![image](https://github.com/user-attachments/assets/1f147af9-0404-4954-ac00-a925b334bdf9)

From the data that has been analyzed, the busiest hours start from 12–13 hours with each order reaching 2500 per day and 17–18 hours with each order reaching 2400 per day.

3. What is the most ordered pizza? Compare the number of orders for each type of pizza
```python
#read data from pizzas table
df_order_details = pd.read_csv('order_details.csv')

#calculate the total order for each pizza size
pizza_counts = df_order_details['pizza_id'].value_counts()

# sort data based on number of reviews and time
summary = summary.sort_values('time')

#create a bar plot using pizza counts
plt.figure(figsize=(10,6)) #set image size

pizza_counts.plot(kind='bar', width=0.8)

plt.xlabel('Pizza Type', fontsize=14)
plt.ylabel('Number of Orders', fontsize=14)
plt.title('Ranking of Most Popular Pizzas', fontsize=14)

#set x-axis label rotation
plt.xticks(rotation=90)

plt.xticks(ticks=range(len(pizza_counts.index)), labels=pizza_counts.index)

#so as not to overlap on the label
plt.tight_layout()

plt.show()
```
![image](https://github.com/user-attachments/assets/e6432809-10e6-49c2-9879-b4e20741b12a)

From the data that has been analyzed, the type of pizza that customers often order is big_meat_s with a total order of more than 1750.

4. How does each pizza size contribute to revenue?
```python
#read data from pizza table
df_pizzas = pd.read_csv('pizzas.csv')

#calculate the total revenue for each pizza size
merged_df = pd.merge(df_order_details, df_pizzas, on='pizza_id')
merged_df['revenue'] = merged_df['quantity'] * merged_df['price']
pizza_revenue = merged_df.groupby('size')['price'].sum().reset_index()

total_revenue = pizza_revenue['price'].sum()

pizza_revenue['percentage'] = pizza_revenue['price']/total_revenue * 100

#create a pie chart using pizza revenue data

fig, ax = plt.subplots()
ax.pie(pizza_revenue['percentage'], labels=pizza_revenue['size'], autopct='%1.1f%%')

plt.axis('equal')
plt.title ('Revenue Contribution by Pizza Size')

plt.show()
```
![image](https://github.com/user-attachments/assets/9a4013de-d2a2-4579-a3cd-a467f1f404db)

Pizza size L contributes the largest revenue contribution compared to other pizza sizes with a percentage of 45.7%. Followed by pizza size M with a revenue contribution of 30.6% and S of 21.8%. XL and XXL pizza sizes contributed the least revenue, which were 0.1% and 1.7% respectively.

5. How does revenue contribute to each type of pizza?
```python
#read data from pizza table
df_pizzas = pd.read_csv('pizzas.csv')

#calculate the total revenue for each pizza size
merged_df = pd.merge(df_order_details, df_pizzas, on='pizza_id')
merged_df['revenue'] = merged_df['quantity'] * merged_df['price']
pizza_revenue = merged_df.groupby('pizza_type_id')['price'].sum().reset_index()

total_revenue = pizza_revenue['price'].sum()

pizza_revenue['percentage'] = pizza_revenue['price']/total_revenue * 100

#sort dat based on highest contribution
pizza_revenue = pizza_revenue.sort_values('percentage', ascending=False)

#create a bar plot using pizza counts
pizza_revenue.plot(x='pizza_type_id', y='percentage', kind='bar', color=['#FF0000', '#00FF00', '#0000FF'])

plt.xlabel('Pizza Type')
plt.ylabel('Revenue Contribution(%)')
plt.title('Revenue Contribution by Pizza Type')

plt.tight_layout()

plt.show()
```
![image](https://github.com/user-attachments/assets/32fc43d6-7631-4dd0-a449-be350c8f070b)

The type of pizza that contributes the highest revenue contribution is thai_ckn pizza with a revenue contribution of 5%, followed by bbq_ckn and cali_ckn.
