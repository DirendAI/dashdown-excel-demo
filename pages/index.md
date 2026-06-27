# 🛍️ Super Store Sales Dashboard

A retail analytics dashboard built from a single **Excel workbook** with
[Dashdown](https://github.com/DirendAI/dashdown). Every chart below is plain Markdown plus SQL,
querying `SuperStoreUS-2015.xlsx` through the Excel connector — its `Orders`,
`Returns`, and `Users` sheets each become a SQL table you can JOIN across.

## 📊 Business Overview

:::query name=sales_overview connector=superstore
SELECT
    COUNT(*) as total_orders,
    COUNT(DISTINCT "Customer ID") as unique_customers,
    SUM("Sales") as total_sales,
    SUM("Profit") as total_profit,
    AVG("Sales") as avg_order_value
FROM Orders
:::

<Grid cols=4>
<Counter
    data={sales_overview}
    column="total_orders"
    label="Orders"
    prefix="📦 "
    format="number"
/>
<Counter
    data={sales_overview}
    column="unique_customers"
    label="Customers"
    prefix="👥 "
    format="number"
/>
<Counter
    data={sales_overview}
    column="total_sales"
    label="Total Sales"
    format="currency"
    decimals=0
/>
<Counter
    data={sales_overview}
    column="total_profit"
    label="Total Profit"
    format="currency"
    decimals=0
/>
</Grid>

---

## 🚀 Sales Performance

:::query name=sales_by_category connector=superstore
SELECT
    "Product Category" as category,
    SUM("Sales") as total_sales,
    SUM("Profit") as total_profit,
    ROUND(SUM("Profit") / SUM("Sales") * 100, 2) as profit_margin_pct
FROM Orders
GROUP BY "Product Category"
ORDER BY total_sales DESC
:::

<Grid cols=2>
<BarChart
    data={sales_by_category}
    x="category"
    y="total_sales"
    title="Sales by Category"
    subtitle="Total revenue per product category"
    color="#6366f1"
    format="currency"
    decimals=0
/>
<BarChart
    data={sales_by_category}
    x="category"
    y="profit_margin_pct"
    title="Profit Margin by Category"
    subtitle="Profit as a percentage of sales"
    color="#22c55e"
    format="percent"
    decimals=1
/>
</Grid>

### Monthly Sales & Profit Trend

:::query name=monthly_sales connector=superstore
SELECT
    strftime("Order Date", '%Y-%m') as month,
    SUM("Sales") as total_sales,
    SUM("Profit") as total_profit
FROM Orders
GROUP BY month
ORDER BY month
:::

<LineChart
    data={monthly_sales}
    x="month"
    y="total_sales,total_profit"
    title="Monthly Sales & Profit"
    subtitle="Revenue and profit over 2015"
    height="400"
    format="currency"
    decimals=0
/>

### 🤖 AI Commentary

The `<Ask>` component sends a query's result to an LLM and renders a plain-language
read-out inline — turning a chart into an explained insight.

<Ask
    data={monthly_sales}
    ask="Summarize the 2015 monthly sales and profit trend in 2-3 sentences. Call out the strongest and weakest months and any notable gap between sales and profit."
    label="What does the trend say?"
/>

---

## 👥 Customer Segments

:::query name=customer_segments connector=superstore
SELECT
    "Customer Segment" as segment,
    COUNT(DISTINCT "Customer ID") as customer_count,
    SUM("Sales") as total_sales
FROM Orders
GROUP BY "Customer Segment"
ORDER BY total_sales DESC
:::

<Grid cols=2>
<PieChart
    data={customer_segments}
    x="segment"
    y="customer_count"
    title="Customers by Segment"
    subtitle="Share of customers in each segment"
    format="number"
/>
<BarChart
    data={customer_segments}
    x="segment"
    y="total_sales"
    title="Sales by Segment"
    subtitle="Revenue generated per segment"
    color="#ec4899"
    format="currency"
    decimals=0
/>
</Grid>

---

## 🗺️ Regional Analysis

:::query name=regional_sales connector=superstore
SELECT
    "Region" as region,
    SUM("Sales") as total_sales,
    SUM("Profit") as total_profit
FROM Orders
GROUP BY "Region"
ORDER BY total_sales DESC
:::

<BarChart
    data={regional_sales}
    x="region"
    y="total_sales"
    title="Sales by Region"
    subtitle="Revenue distribution across regions"
    color="#06b6d4"
    format="currency"
    decimals=0
/>

---

## 🔁 Returns & Regional Ownership

The workbook has two more sheets — **Returns** and **Users** — and each becomes its
own SQL table. So you can **JOIN across sheets** exactly like a relational database.

### Returned Sales by Category

Joining `Orders` to the `Returns` sheet (on `Order ID`) reveals which categories come
back most by value.

:::query name=returns_by_category connector=superstore
SELECT
    o."Product Category" as category,
    COUNT(DISTINCT o."Order ID") as returned_orders,
    SUM(o."Sales") as returned_sales
FROM Orders o
JOIN (SELECT DISTINCT "Order ID" AS rid FROM Returns) r
  ON o."Order ID" = r.rid
GROUP BY o."Product Category"
ORDER BY returned_sales DESC
:::

<BarChart
    data={returns_by_category}
    x="category"
    y="returned_sales"
    title="Returned Sales by Category"
    subtitle="Value of returned orders, joined from the Returns sheet"
    color="#f59e0b"
    format="currency"
    decimals=0
/>

### Performance by Regional Manager

The `Users` sheet maps each region to its manager. Join it to the orders to see who
owns which numbers — including the region running at a loss.

:::query name=regional_managers connector=superstore
SELECT
    u."Region" as region,
    u."Manager" as manager,
    COUNT(DISTINCT o."Order ID") as orders,
    SUM(o."Sales") as total_sales,
    SUM(o."Profit") as total_profit
FROM Orders o
JOIN Users u ON o."Region" = u."Region"
GROUP BY u."Region", u."Manager"
ORDER BY total_sales DESC
:::

<Table
    data={regional_managers}
    title="Sales & Profit by Regional Manager"
    format="orders=number, total_sales=currency, total_profit=currency"
    decimals=0
/>

---

## 🔍 Interactive Explorer

Filter the orders below by category, region, and customer segment. The "All"
option in each dropdown clears that filter.

<Grid cols=3>
<Dropdown
    name="category"
    data={sales_by_category}
    column="category"
    label="Product Category"
/>
<Dropdown
    name="region"
    data={regional_sales}
    column="region"
    label="Region"
/>
<Dropdown
    name="segment"
    data={customer_segments}
    column="segment"
    label="Customer Segment"
/>
</Grid>

:::query name=filtered_orders connector=superstore
SELECT
    "Order ID" as order_id,
    "Order Date" as order_date,
    "Customer Name" as customer_name,
    "Customer Segment" as segment,
    "Product Category" as category,
    "Product Name" as product_name,
    "Region" as region,
    "Sales" as sales,
    "Profit" as profit
FROM Orders
WHERE ('${category}' = '' OR "Product Category" = '${category}')
  AND ('${region}' = '' OR "Region" = '${region}')
  AND ('${segment}' = '' OR "Customer Segment" = '${segment}')
ORDER BY "Sales" DESC
LIMIT 50
:::

<Table
    data={filtered_orders}
    title="Filtered Orders (top 50 by sales)"
    pagesize=10
    format="order_date=date, sales=currency, profit=currency"
    decimals=2
/>

---

## 📦 Top Products

:::query name=top_products connector=superstore
SELECT
    "Product Name" as product_name,
    "Product Category" as category,
    COUNT(*) as order_count,
    SUM("Sales") as total_sales,
    SUM("Profit") as total_profit
FROM Orders
GROUP BY "Product Name", "Product Category"
ORDER BY total_sales DESC
LIMIT 15
:::

<Table
    data={top_products}
    title="Top 15 Products by Sales"
    pagesize=15
    sortable=true
    format="order_count=number, total_sales=currency, total_profit=currency"
    decimals=0
/>

---

## 🎯 Key Insights

- **Category mix**: Technology and Office Supplies drive the bulk of revenue, but margins differ sharply between categories.
- **Customer segments**: Corporate and Home Office customers tend to place higher-value orders than Consumers.
- **Regional spread**: Sales are concentrated in a few high-performing regions — a cue for where to focus growth.
- **Profitability**: A high sales total doesn't guarantee profit — the margin chart shows where revenue actually converts to profit.

---

*🛍️ Super Store Sales 2015 · Built with [Dashdown](https://github.com/DirendAI/dashdown) and the Excel connector*
