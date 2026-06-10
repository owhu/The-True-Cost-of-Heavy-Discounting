# The True Cost of Heavy Discounting: A Retail Profitability Optimization Case Study

A case study investigating the financial impact of discounting strategies on profit margins. This project translates raw transaction data into insights by pairing SQL and Python with an interactive, narrative-driven Tableau dashboard.

**[Interact with the Live Tableau Dashboard Here!](https://public.tableau.com/app/profile/oliver.hu5770/viz/RetailProfitabilityLeakageTheTrueCostofHeavyDiscounting/Dashboard1?publish=yes)**

Unmonitored price cuts can quietly erode profitability. This analysis evaluates over $2 million in transaction data to identify poor performing products, establish clear markdown safety thresholds, and isolate regional financial losses answering the following questions:

1. What is the scale of the problem?
2. Where and why is it happening?
3. Which products are responsible?

## 1. What is the scale of the problem?

I constructed a Financial Waterfall Chart utilizing a Gantt mark configuration with negative sizing logic to display the exact dollar impact from markdowns.

**Insight:** The business generates a $383K profit from normal sales, but aggressive markdowns over 50% wipe out nearly $100K in value, pulling final net profit down to $286K.

## 2. Where and why is it happening?

Using a scatter plot and segmented horizontal bar charts, I analyzed the average transaction discounts against net margins to find the tipping point where promotional volume turns unprofitable.

**Insight:** Profitability remains stable up to a 20% markdown threshold. Profitability collapses when the average discounts exceed 20%.

## 3. Which products are responsible?
## Benchmarking via SQL
To accurately measure profit loss, we cannot simply look at negative profit numbers. We must compare a product's underperformance during a heavy sale against its own baseline when sold at a normal price point.

I wrote the following query using SQL to calculate a benchmark for every unique product (defined as orders with a discount less than 10%) and isolate the top 10 worst financial bleeders where promotional discounts were more than 50%.
```
WITH global_stats AS (
    SELECT 
        "Product Name", 
        AVG(Profit) AS avg_normal_profit
    FROM data_table
    WHERE Discount <= 0.10
    GROUP BY "Product Name"
)
SELECT 
    main."Product Name",
    main.Category,
    COUNT(main."Order ID") AS total_discounted_orders,
    ROUND(AVG(main.Discount), 2) AS avg_heavy_discount,
    ROUND(SUM(main.Profit), 2) AS total_financial_loss,
    ROUND(global_stats.avg_normal_profit, 2) AS benchmark_normal_profit
FROM data_table AS main
JOIN global_stats ON main."Product Name" = global_stats."Product Name"
WHERE main.Discount >= 0.50
  AND main.Profit < 0
GROUP BY main."Product Name", main.Category, global_stats.avg_normal_profit
ORDER BY total_financial_loss ASC
LIMIT 10;
```

The above SQL query produced the following table displaying the losses when sold at a heavy discount and the profit when sold at a normal price point:

<img width="1084" height="472" alt="Screenshot 2026-06-10 at 12 34 42 PM" src="https://github.com/user-attachments/assets/276842d7-6c0e-436a-9a9d-bf89f1d31e50" />

The dashboard features an interactive map linked via action filters to a Top 10 Financial Bleeders table, a scatterplot showing the relationship between average discount and profit, and a Financial Waterfall Chart. Clicking on high-loss states updates the 3 visuals, giving a list of exact items causing margin drainage and the financial cost of heavy discounting.

[Here is the final dashboard architecture:](https://public.tableau.com/app/profile/oliver.hu5770/viz/RetailProfitabilityLeakageTheTrueCostofHeavyDiscounting/Dashboard1?publish=yes)

</a><img width="1643" height="822" alt="Screenshot 2026-06-10 at 1 34 35 PM" src="https://github.com/user-attachments/assets/0fb69829-be64-42ff-9a45-89275da10273" />


