# Pandas Challenge 1

## Part 1: Explore the Data

Import the data and use Pandas to learn more about the dataset.

```python
import pandas as pd

client_df = pd.read_csv("Resources/client_dataset.csv")
client_df.head()
```
<figure>
    <img src="images\img_1.png">
</figure>


```python
# View the column names in the data
client_df.columns
```
<figure>
    <img src="images\img_2.png">
</figure>

```python
# Use the describe function to gather some basic statistics
client_df.describe()
```
<figure>
    <img src="images\img_3.png">
</figure>

```python
# Use this space to do any additional research
# and familiarize yourself with the data.
client_df.info()
```
<figure>
    <img src="images\img_4.png">
</figure>

```python
# What three item categories had the most entries?
client_df["category"].value_counts().nlargest(3)
```
<figure>
    <img src="images\img_5.png" width="225" height="100">
</figure>

```python
# For the category with the most entries, which subcategory had the most entries?
# client_df["subcategory"].value_counts().nlargest(1)
top_category = client_df["category"].value_counts().nlargest(1).index[0]
cat_group = client_df.groupby("category")
cat_group.get_group(top_category)["subcategory"].value_counts().nlargest(1)
```

<figure>
    <img src="images\img_6.png" height=75 width=180>
</figure>

```python
# Which five clients had the most entries in the data?
client_df["client_id"].value_counts().nlargest(5)
```

<figure>
    <img src="images\img_7.png">
</figure>

```python
# Store the client ids of those top 5 clients in a list.
top_5_client_id = client_df["client_id"].value_counts().nlargest(5).index.to_list()
top_5_client_id
```

<figure>
    <img src="images\img_8.png">
</figure>

```python
# How many total units (the qty column) did the client with the most entries order order?
top_client_id = top_5_client_id[0]
client_grp = client_df.groupby("client_id")
client_grp.get_group(top_client_id)["qty"].sum()
```

<figure>
    <img src="images\img_9.png">
</figure>

## Part 2: Transform the Data
Do we know that this client spent the more money than client 66037? If not, how would we find out? Transform the data using the steps below to prepare it for analysis.

```python
# Create a column that calculates the subtotal for each line using the unit_price and the qty
def cal_line_subtotal(row):
    return row["unit_price"] * row["qty"]


line_subtotal = client_df.apply(cal_line_subtotal, axis=1)
client_df["line_subtotal"] = line_subtotal
client_df.loc[0:1, ["unit_price", "qty", "line_subtotal"]]
```

<figure>
    <img src="images\img_10.png">
</figure>

```python
# Create a column for shipping price.
# Assume a shipping price of $7 per pound for orders over 50 pounds and $10 per pound for items 50 pounds or under.


# calculate total weight
def cal_total_weight(row):
    return row["unit_weight"] * row["qty"]


total_weight = client_df.apply(cal_total_weight, axis=1)
client_df["total_weight"] = total_weight


# calculate shipping price
def cal_shipping_price(row):
    weight = row["total_weight"]
    if weight > 50:
        return weight * 7
    return weight * 10


# Shipping price
shipping_price = client_df.apply(cal_shipping_price, axis=1)
client_df["shipping_price"] = shipping_price

client_df.loc[
    0:2, ["unit_price", "unit_weight", "qty", "total_weight", "shipping_price"]
]
```

<figure>
    <img src="images\img_11.png">
</figure>

```python
# Create a column for the total price using the subtotal and the shipping price along with a sales tax of 9.25%


# calculate line price
def cal_line_price(row):
    tax_amt = (row["line_subtotal"] + row["shipping_price"]) * 0.0925
    return round(row["line_subtotal"] + tax_amt + row["shipping_price"], 2)


line_price = client_df.apply(cal_line_price, axis=1)
client_df["line_price"] = line_price
client_df.loc[0:2, ["line_subtotal", "shipping_price", "line_price"]]
```

<figure>
    <img src="images\img_12.png">
</figure>

```python
# Create a column for the cost of each line using unit cost, qty, and
# shipping price (assume the shipping cost is exactly what is charged to the client).


# calculate line cost
def cal_line_cost(row):
    total = row["unit_cost"] * row["qty"]
    return total + row["shipping_price"]


line_cost = client_df.apply(cal_line_cost, axis=1)
client_df["line_cost"] = line_cost
client_df.head(3)
```

<figure>
    <img src="images\img_13.png">
</figure>

```python
# Create a column for the profit of each line using line cost and line price


# calculate line profit
def cal_line_profit(row):
    return row["line_price"] - row["line_cost"]


line_profit = client_df.apply(cal_line_profit, axis=1)
client_df["line_profit"] = line_profit
client_df.head(3)
```

<figure>
    <img src="images\img_14.png">
</figure>

## Part 3: Confirm your work
You have email receipts showing that the total prices for 3 orders. Confirm that your calculations match the receipts. Remember, each order has multiple lines.

Order ID 2742071 had a total price of \$152,811.89

Order ID 2173913 had a total price of \$162,388.71

Order ID 6128929 had a total price of \$923,441.25

```python
# Check your work using the totals above
order_id_2742071 = 2742071
order_id_2173913 = 2173913
order_id_6128929 = 6128929

ord_id_group = client_df.groupby("order_id")

order_id_2742071_sum = ord_id_group.get_group(order_id_2742071)["line_price"].sum()
order_id_2173913_sum = ord_id_group.get_group(order_id_2173913)["line_price"].sum()
order_id_6128929_sum = ord_id_group.get_group(order_id_6128929)["line_price"].sum()

print(f"Order ID {order_id_2742071} has a total price of ${order_id_2742071_sum:,.2f} ")
print(f"Order ID {order_id_2173913} has a total price of ${order_id_2173913_sum:,.2f} ")
print(f"Order ID {order_id_6128929} has a total price of ${order_id_6128929_sum:,.2f} ")
```

<figure>
    <img src="images\img_15.png">
</figure>

## Part 4: Summarize and Analyze
Use the new columns with confirmed values to find the following information.

```python
# How much did each of the top 5 clients by quantity spend? Check your work from Part 1 for client ids.

client_id_33615 = top_5_client_id[0]
client_id_66037 = top_5_client_id[1]
client_id_46820 = top_5_client_id[2]
client_id_38378 = top_5_client_id[3]
client_id_24741 = top_5_client_id[4]

client_id_group = client_df.groupby("client_id")

client_id_33615_sum = client_id_group.get_group(client_id_33615)["line_price"].sum()
client_id_66037_sum = client_id_group.get_group(client_id_66037)["line_price"].sum()
client_id_46820_sum = client_id_group.get_group(client_id_46820)["line_price"].sum()
client_id_38378_sum = client_id_group.get_group(client_id_38378)["line_price"].sum()
client_id_24741_sum = client_id_group.get_group(client_id_24741)["line_price"].sum()

print(f"Client ID {client_id_33615} has a total price of ${client_id_33615_sum:,.2f} ")
print(f"Client ID {client_id_66037} has a total price of ${client_id_66037_sum:,.2f} ")
print(f"Client ID {client_id_46820} has a total price of ${client_id_46820_sum:,.2f} ")
print(f"Client ID {client_id_38378} has a total price of ${client_id_38378_sum:,.2f} ")
print(f"Client ID {client_id_24741} has a total price of ${client_id_24741_sum:,.2f} ")
```

<figure>
    <img src="images\img_16.png">
</figure>

```python
# Create a summary DataFrame showing the totals for the for the top 5 clients with the following information:
# total units purchased, total shipping price, total revenue, and total profit.
top_5_client_id = client_df["client_id"].value_counts().nlargest(5).index.to_list()

# qty
qty_33615 = client_id_group.get_group(client_id_33615)["qty"].sum()
qty_66037 = client_id_group.get_group(client_id_66037)["qty"].sum()
qty_46820 = client_id_group.get_group(client_id_46820)["qty"].sum()
qty_38378 = client_id_group.get_group(client_id_38378)["qty"].sum()
qty_24741 = client_id_group.get_group(client_id_24741)["qty"].sum()

qty_list = [qty_33615, qty_66037, qty_46820, qty_38378, qty_24741]

# shipping price
ship_price_33615 = client_id_group.get_group(client_id_33615)["shipping_price"].sum()
ship_price_66037 = client_id_group.get_group(client_id_66037)["shipping_price"].sum()
ship_price_46820 = client_id_group.get_group(client_id_46820)["shipping_price"].sum()
ship_price_38378 = client_id_group.get_group(client_id_38378)["shipping_price"].sum()
ship_price_24741 = client_id_group.get_group(client_id_24741)["shipping_price"].sum()

ship_list = [
    ship_price_33615,
    ship_price_66037,
    ship_price_46820,
    ship_price_38378,
    ship_price_24741,
]

# line price
line_price_33615 = client_id_group.get_group(client_id_33615)["line_price"].sum()
line_price_66037 = client_id_group.get_group(client_id_66037)["line_price"].sum()
line_price_46820 = client_id_group.get_group(client_id_46820)["line_price"].sum()
line_price_38378 = client_id_group.get_group(client_id_38378)["line_price"].sum()
line_price_24741 = client_id_group.get_group(client_id_24741)["line_price"].sum()

line_price_list = [
    line_price_33615,
    line_price_66037,
    line_price_46820,
    line_price_38378,
    line_price_24741,
]

# line cost
line_cost_33615 = client_id_group.get_group(client_id_33615)["line_cost"].sum()
line_cost_66037 = client_id_group.get_group(client_id_66037)["line_cost"].sum()
line_cost_46820 = client_id_group.get_group(client_id_46820)["line_cost"].sum()
line_cost_38378 = client_id_group.get_group(client_id_38378)["line_cost"].sum()
line_cost_24741 = client_id_group.get_group(client_id_24741)["line_cost"].sum()

line_cost_list = [
    line_cost_33615,
    line_cost_66037,
    line_cost_46820,
    line_cost_38378,
    line_cost_24741,
]

# line profit
line_profit_33615 = client_id_group.get_group(client_id_33615)["line_profit"].sum()
line_profit_66037 = client_id_group.get_group(client_id_66037)["line_profit"].sum()
line_profit_46820 = client_id_group.get_group(client_id_46820)["line_profit"].sum()
line_profit_38378 = client_id_group.get_group(client_id_38378)["line_profit"].sum()
line_profit_24741 = client_id_group.get_group(client_id_24741)["line_profit"].sum()

line_profit_list = [
    line_profit_33615,
    line_profit_66037,
    line_profit_46820,
    line_profit_38378,
    line_profit_24741,
]

client_summary_df = pd.DataFrame(
    {
        "client_id": top_5_client_id,
        "qty": qty_list,
        "shipping_price": ship_list,
        "line_price": line_price_list,
        "line_cost": line_cost_list,
        "line_profit": line_profit_list,
    }
)

client_summary_df
```

<figure>
    <img src="images\img_17.png">
</figure>

```python
# Format the data and rename the columns to names suitable for presentation.

# Define the money columns.

rename_col = {
    "client_id": "Client ID",
    "qty": "Units",
    "shipping_price": "Shipping (millions)",
    "line_price": "Total Revenue (millions)",
    "line_cost": "Total Cost (millions)",
    "line_profit": "Total Profit (millions)",
}


def covert_to_millions(data):
    return data / 1000000


# Apply the currency_format_millions function to only the money columns.
client_summary_df["shipping_price"] = client_summary_df["shipping_price"].apply(
    covert_to_millions
)
client_summary_df["line_price"] = client_summary_df["line_price"].apply(
    covert_to_millions
)
client_summary_df["line_cost"] = client_summary_df["line_cost"].apply(
    covert_to_millions
)
client_summary_df["line_profit"] = client_summary_df["line_profit"].apply(
    covert_to_millions
)


client_summary_df = client_summary_df.rename(columns=(rename_col))

client_summary_df
```

<figure>
    <img src="images\img_18.png">
</figure>


```python
# Sort the updated data by "Total Profit (millions)" form highest to lowest and assign the sort to a new DatFrame.
final_summary_report = (
    client_summary_df.sort_values(by="Total Profit (millions)", ascending=False)
    .copy()
    .reset_index()
)

final_summary_report
```

<figure>
    <img src="images\img_19.png">
</figure>

