# Revenue Optimization Analysis for Multi-City Hospitality Chain

## Business Problem 
This multi-city hotel chain was facing revenue decline and decreasing profit margins over the past few years. In some cities, many hotel rooms were not occupied, which resulted in a low occupancy rate. Their overall booking efficiency was also not increasing.

Management wanted to improve occupancy rate, increase profit margins, and increase the occupancy rate. However, they did not clearly understand the main reasons why this causes.

---

## Key Business Questions

1) Is pricing aligned properly with demand across room categories?
2) Which cities and properties are underperforming?
3) Is revenue dropping because of low occupancy or high cancellations?
4) How much revenue is lost due to cancellations and no-shows?
5) Is OTA commission affecting overall profitability?
6) Are weekday bookings weaker compared to weekend bookings?



---



##  How I Solved the Business Problem

To address revenue decline and low occupancy challenges, I followed a structured end-to-end analytics workflow:

- Collected booking, revenue, customer rating, and inventory data across multiple cities.
- Cleaned raw datasets by:
  - Handling missing/null values  
  - Standardizing data types and formats  
  - Detecting and treating outliers using statistical methods (IQR, Z-Score)
- **STAR SCHEMA DATA MODEL**
  - Used to connect all the tables with fact table (center table) that storing like booking and revenue. The dimension tables are lied on center table they kept remianing details like city, date kind of information.
  - **Why STAR SCHEMA**
    - That simplifies relationships, improves performances foraggregations and rankings, and makes analysis easier and faster.
- Performed Exploratory Data Analysis (EDA) to uncover trends in occupancy, cancellations, pricing, and platform performance.
- Created new key measures to evaluate revenue performance, booking efficiency, pricing strategy, and week-over-week growth.
- Designed interactive dashboards to compare city-level, property-level, and platform-level performance.

---

# Key Metrics & Formulas

## Core Performance Metrics

### 1. Revenue  
Total realized revenue from bookings.
```DAX
Revenue = SUM(fact_bookings[revenue_realized])
```

### 2. Total Bookings  
Total number of bookings recorded.
```DAX
Total Bookings = COUNT(fact_bookings[booking_id])
```

### 3. Total Capacity  
Total available rooms across all hotels.
```DAX
Total Capacity = SUM(fact_aggregated_bookings[capacity])
```

### 4. Total Successful Bookings  
Total successfully occupied rooms.
```DAX
Total Successful Bookings = SUM(fact_aggregated_bookings[successful_bookings])
```

### 5. Occupancy %  
Percentage of rooms occupied out of total capacity.
```DAX
Occupancy % = DIVIDE([Total Successful Bookings], [Total Capacity], 0)
```

### 6. Average Rating  
Average customer rating.
```DAX
Average Rating = AVERAGE(fact_bookings[ratings_given])
```

### 7. No of Days  
Total number of days in selected period.
```DAX
No of Days = DATEDIFF(MIN(dim_date[date]), MAX(dim_date[date]), DAY) + 1
```

### 8. Total Cancelled Bookings
```DAX
Total Cancelled Bookings =
CALCULATE([Total Bookings], fact_bookings[booking_status] = "Cancelled")
```

### 9. Cancellation %
```DAX
Cancellation % =
DIVIDE([Total Cancelled Bookings], [Total Bookings])
```

### 10. Total Checked Out
```DAX
Total Checked Out =
CALCULATE([Total Bookings], fact_bookings[booking_status] = "Checked Out")
```

### 11. Total No Show Bookings
```DAX
Total No Show Bookings =
CALCULATE([Total Bookings], fact_bookings[booking_status] = "No Show")
```

### 12. No Show Rate %
```DAX
No Show Rate % =
DIVIDE([Total No Show Bookings], [Total Bookings])
```

### 13. Realisation %
Percentage of bookings successfully completed.
```DAX
Realisation % =
1 - ([Cancellation %] + [No Show Rate %])
```
### 14. Booking % by Platform
Percentage contribution of each booking platform (OTA vs Direct).
```DAX
Booking % by Platform =
DIVIDE(
    [Total Bookings],
    CALCULATE([Total Bookings], ALL(fact_bookings[booking_platform]))
) * 100
```

### 15. Booking % by Room Class
Contribution of each room category.
```DAX
Booking % by Room Class =
DIVIDE(
    [Total Bookings],
    CALCULATE([Total Bookings], ALL(dim_rooms[room_class]))
) * 100
```




### 16. ADR (Average Daily Rate)
Average revenue earned per booked room.
```DAX
ADR = DIVIDE([Revenue], [Total Bookings], 0)
```

### 17. RevPAR (Revenue Per Available Room)
Revenue generated per available room.
```DAX
RevPAR = DIVIDE([Revenue], [Total Capacity])
```

### 18. DBRN (Daily Booked Room Nights)
Average rooms booked per day.
```DAX
DBRN = DIVIDE([Total Bookings], [No of Days])
```

### 19. DSRN (Daily Sellable Room Nights)
Average rooms available for sale per day.
```DAX
DSRN = DIVIDE([Total Capacity], [No of Days])
```

### 20. DURN (Daily Utilized Room Nights)
Average rooms successfully utilized per day.
```DAX
DURN = DIVIDE([Total Checked Out], [No of Days])
```

### 21. Revenue WoW Change %
```DAX
Revenue WoW Change % =
VAR selv =
    IF(HASONEFILTER(dim_date[wn]),
       SELECTEDVALUE(dim_date[wn]),
       MAX(dim_date[wn]))
VAR revcw =
    CALCULATE([Revenue], dim_date[wn] = selv)
VAR revpw =
    CALCULATE([Revenue],
        FILTER(ALL(dim_date), dim_date[wn] = selv - 1))
RETURN
DIVIDE(revcw, revpw, 0) - 1
```
### 22. Occupancy WoW Change %

```DAX
Occupancy WoW Change % =
VAR selv =
    IF(
        HASONEFILTER(dim_date[wn]),
        SELECTEDVALUE(dim_date[wn]),
        MAX(dim_date[wn])
    )
VAR curr_week =
    CALCULATE([Occupancy %], dim_date[wn] = selv)
VAR prev_week =
    CALCULATE(
        [Occupancy %],
        FILTER(ALL(dim_date), dim_date[wn] = selv - 1)
    )
RETURN
DIVIDE(curr_week, prev_week, 0) - 1
```



### 23. ADR WoW Change %

```DAX
ADR WoW Change % =
VAR selv =
    IF(
        HASONEFILTER(dim_date[wn]),
        SELECTEDVALUE(dim_date[wn]),
        MAX(dim_date[wn])
    )
VAR curr_week =
    CALCULATE([ADR], dim_date[wn] = selv)
VAR prev_week =
    CALCULATE(
        [ADR],
        FILTER(ALL(dim_date), dim_date[wn] = selv - 1)
    )
RETURN
DIVIDE(curr_week, prev_week, 0) - 1
```



### 24. RevPAR WoW Change %

```DAX
RevPAR WoW Change % =
VAR selv =
    IF(
        HASONEFILTER(dim_date[wn]),
        SELECTEDVALUE(dim_date[wn]),
        MAX(dim_date[wn])
    )
VAR curr_week =
    CALCULATE([RevPAR], dim_date[wn] = selv)
VAR prev_week =
    CALCULATE(
        [RevPAR],
        FILTER(ALL(dim_date), dim_date[wn] = selv - 1)
    )
RETURN
DIVIDE(curr_week, prev_week, 0) - 1
```



### 25. Realisation WoW Change %

```DAX
Realisation WoW Change % =
VAR selv =
    IF(
        HASONEFILTER(dim_date[wn]),
        SELECTEDVALUE(dim_date[wn]),
        MAX(dim_date[wn])
    )
VAR curr_week =
    CALCULATE([Realisation %], dim_date[wn] = selv)
VAR prev_week =
    CALCULATE(
        [Realisation %],
        FILTER(ALL(dim_date), dim_date[wn] = selv - 1)
    )
RETURN
DIVIDE(curr_week, prev_week, 0) - 1
```


### 26. DSRN WoW Change %

```DAX
DSRN WoW Change % =
VAR selv =
    IF(
        HASONEFILTER(dim_date[wn]),
        SELECTEDVALUE(dim_date[wn]),
        MAX(dim_date[wn])
    )
VAR curr_week =
    CALCULATE([DSRN], dim_date[wn] = selv)
VAR prev_week =
    CALCULATE(
        [DSRN],
        FILTER(ALL(dim_date), dim_date[wn] = selv - 1)
    )
RETURN
DIVIDE(curr_week, prev_week, 0) - 1
```


---

# Business Impact
With help of these kpi's i can able to:

- Identify underperforming cities and properties
- Measure revenue loss due to cancellations and no-shows
- Compare OTA vs Direct booking profitability
- Analyze weekday vs weekend demand trends
- Evaluate pricing effectiveness using ADR & RevPAR
- Track weekly performance trends using WoW growth metrics

---
# Recommendations

Based on my analysis, I suggest the following improvements for the business:

## 1. Use Dynamic Pricing
- Reduce room prices during weekdays because demand is low.
- Increase prices during weekends where occupancy is already high.
- Adjust pricing based on demand and room category performance.

## 2. Improve Booking Channel Strategy
- Reduce dependency on high-commission OTA platforms.
- Encourage more direct bookings through website offers and loyalty programs.
- Focus not only on booking volume but also on profitability.

## 3. Reduce Cancellations and No-Shows
- Introduce partial advance payment to reduce last-minute cancellations.
- Send reminder notifications before check-in date.
- Provide rescheduling options instead of full cancellation.

## 4. Improve Underperforming Cities and Properties
- Run marketing campaigns in cities with low occupancy.
- Review pricing strategy in underperforming hotels.
- Improve service quality based on customer ratings and feedback.

## 5. Increase Weekday Bookings
- Launch weekday special offers and discounts.
- Target business customers for weekday stays.
- Provide long-stay offers to increase room utilization.

## 6. Increase Online Visibility
- Improve digital marketing and SEO performance.
- Promote hotels more actively on social media.
- Improve property listing quality on booking platforms.

## 7. Monitor KPIs Regularly
- Track Occupancy %, ADR, RevPAR, and Realisation % regularly.
- Monitor week-over-week performance changes.
- Review city-level and property-level performance monthly.

## Tools Used

- Power BI
- DAX
- Power Query

## Dashboard Link 
https://app.powerbi.com/groups/me/reports/d4af6be0-a1d9-4c71-93f1-9cba3e437f56/ReportSectionce2063a216d8e001051e?experience=power-bi
