# pandas_challenge1

# Data Analysis and Transformation Project

## Overview

This project involves exploring, transforming, and analyzing a client dataset. The steps include loading data, examining the dataset, performing transformations to prepare for analysis, and summarizing key metrics. The dataset includes information such as client ID, job, category, subcategory, units purchased, shipping price, line price, and profit.

## Table of Contents

1. [Part 1: Explore the Data](#part-1-explore-the-data)
2. [Part 2: Transform the Data](#part-2-transform-the-data)
3. [Part 3: Confirm your work](#part-3-confirm-your-work)
4. [Part 4: Summarize and Analyze](#part-4-summarize-and-analyze)
5. [Usage Instructions](#usage-instructions)

## Part 1: Explore the Data

In this section, we import the data and use Pandas to understand the dataset.

### Code

```python
import pandas as pd

df = pd.read_csv('Resources/client_dataset.csv')
df.head()
df.columns
df.describe()
df.count()
df.dtypes
df['job'].unique()
df['category'].value_counts().head(3)
df['subcategory'][df['category'] == 'consumables'].value_counts().head(1)
df['client_id'].value_counts().head(5)
top_5_clients = df['client_id'].value_counts().head(5).index.tolist()
top_5_clients
df['qty'][df['client_id'] == top_5_clients[0]].sum()
```

## Part 2: Transform the Data

In this part, we transform the data to prepare it for analysis.

### Code

```python
df['line_subtotal'] = df['unit_price'] * df['qty']
df['total_weight'] = df['qty'] * df['unit_weight']

def ship_price(ttl_weight):
    if ttl_weight > 50:
        return ttl_weight * 7
    return ttl_weight * 10

df['shipping_price'] = df['total_weight'].apply(ship_price)
df['line_price'] = (df['line_subtotal'] + df['shipping_price']) * 1.0925
df['line_price'] = df['line_price'].apply(lambda x: round(x, 2))
df['line_cost'] = (df['unit_cost'] * df['qty']) + df['shipping_price']
df['line_profit'] = df['line_price'] - df['line_cost']
df['line_profit'] = df['line_profit'].apply(lambda x: round(x, 2))
```

## Part 3: Confirm your work

Here, we confirm that our calculations match given values.

### Code

```python
def print_order_totals(df, order_ids):
    for order_id in order_ids:
        total_price = df.loc[df['order_id'] == order_id, 'line_price'].sum()
        print(f"Order ID {order_id} had a total price of ${total_price:,.2f}")

order_ids = [2742071, 2173913, 6128929]
print_order_totals(df, order_ids)
```

## Part 4: Summarize and Analyze

In this final part, we summarize the data and analyze key metrics.

### Code

```python
top_clients_orders_df = df[df['client_id'].isin(top_5_clients)]

client_totals = top_clients_orders_df.groupby('client_id')['line_price'].sum()

def print_client_totals(client_totals):
    for client_id, total_price in client_totals.items():
        print(f"Client ID {client_id}: ${total_price:,.2f}")

print_client_totals(client_totals)

summary = top_clients_orders_df.groupby('client_id').agg(
    total_units_purchased=('qty', 'sum'),
    total_shipping_price=('shipping_price', 'sum'),
    total_revenue=('line_price', 'sum'),
    total_profit=('line_profit', 'sum')
)

summary_sorted = summary.sort_values(by='total_profit', ascending=False).reset_index()

summary_sorted.columns = ['Client', 'Total Units Purchased', 'Total Shipping Price (Millions $)', 'Total Revenue (Millions $)', 'Total Profit (Millions $)']
summary_sorted['Total Shipping Price (Millions $)'] = summary_sorted['Total Shipping Price (Millions $)'].apply(lambda x: f"${x:,.2f}M")
summary_sorted['Total Revenue (Millions $)'] = summary_sorted['Total Revenue (Millions $)'].apply(lambda x: f"${x:,.2f}M")
summary_sorted['Total Profit (Millions $)'] = summary_sorted['Total Profit (Millions $)'].apply(lambda x: f"${x:,.2f}M")

summary_sorted
```

## Usage Instructions

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/repository-name.git
   ```

2. **Navigate to the project directory:**
   ```bash
   cd repository-name
   ```

3. **Install the necessary packages:**
   ```bash
   pip install pandas
   ```

4. **Run the script:**
   Ensure you have the `client_dataset.csv` file in the `Resources` directory, then execute the script.

5. **Verify the results:**
   Check the outputs for the summaries and analysis as specified in the steps above.
