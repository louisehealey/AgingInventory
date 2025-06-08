# Aging Inventory
This Power BI report analyzes aging inventory by categorizing total cost and stock quantities into time-based buckets. These buckets allow users to track stock movement, identify slow-moving items, and optimize inventory turnover to improve efficiency and reduce waste.

Since most components are not serialized, tracking individual part movement is a challenge. To estimate how long stock has remained at the facility, this report leverages Purchase Order Transactions and assumes a FIFO (First-In, First-Out) inventory management approach.
### Total Cost Report
<p align="center">
  <img src="https://raw.githubusercontent.com/louisehealey/AgingInventory/main/AgingInventoryDashboard-TotalCost.png"
      >
</p>

### Total Quantity Report
<p align="center">
  <img src="https://raw.githubusercontent.com/louisehealey/AgingInventory/main/AgingInventoryDashboard-TotalQuantity.png"
      >
</p>


## Data Model Overview

This data model integrates multiple tables to effectively manage and analyze inventory aging. 

- **`PART_MANAGER`**: Contains detailed attributes of each part including description, commodity and unit cost.
- **`INVENTORY_TRANSACTION`**: Log of inventory transactions used for tracking stock age by individual transactions.
- **`STOCK_STATUS`**: Provides an overview of total `ON_HAND_QTY` for each part, ensuring visibility into current inventory levels.
- **`AGING_INVENTORY_MATRIX`**: A Matrix table that consolidates the data in the above tables to a single table that summarizes the data. 

<p align="center">
  <img src="https://raw.githubusercontent.com/louisehealey/AgingInventory/main/AgingInventoryDataModel.png"
      >
</p>

## How to Calculate the Time Based Buckets?
The calculated column below determines how old each transaction is based on it's `ACTION DATE/TIME`
```
DateBucket =
SWITCH(
    TRUE(),
    DATEDIFF([ACTION DATE/TIME], TODAY(), DAY) < 30, 1,
    DATEDIFF([ACTION DATE/TIME], TODAY(), DAY) >= 30 && DATEDIFF([ACTION DATE/TIME], TODAY(), DAY) < 60, 2,
    DATEDIFF([ACTION DATE/TIME], TODAY(), DAY) >= 60 && DATEDIFF([ACTION DATE/TIME], TODAY(), DAY) < 90, 3,
    DATEDIFF([ACTION DATE/TIME], TODAY(), DAY) >= 90 && DATEDIFF([ACTION DATE/TIME], TODAY(), DAY) < 180, 4,
    DATEDIFF([ACTION DATE/TIME], TODAY(), DAY) >= 180 && DATEDIFF([ACTION DATE/TIME], TODAY(), DAY) < 365, 5,
    DATEDIFF([ACTION DATE/TIME], TODAY(), DAY) >= 365,6
)
```
## Calculating Aging Inventory

Starting with the current `ON_HAND_QTY` of a specified part, the most recent purchase order (`PO`) transaction quantity is subtracted. The transaction is then classified into a `Date bucket` based on its corresponding `ACTION DATE/TIME` value. This process continues until the total `ON_HAND_QTY` is accounted for.

### Example Calculation

In this example, the `Total_Qty_At_PO_Date` for the latest `ACTION_DATE/TIME` aligns with the `ON_HAND_QTY` recorded in the `STOCK_STATUS` table for `PART_ID`: **PHN1041**.

1. The `PO_Quantity` is deducted from the starting on-hand quantity, producing the `Previous Qty` value: <br>
      **1404** (ON_HAND_QTY)- **20**(PO_Quantity)= **1384** (Previous_Qty)
2. The next PO transaction is subtracted from the `Previous Qty`, continuing the process of inventory depletion.

If today were **May 30, 2025**, and **PHN1041** had a starting on-hand quantity of **1,404**, the table below illustrates how the data model calculates inventory age based on PO transactions.

| ACTION_DATE/TIME  | PART_ID  | PO_ID   | PO_LINE | Previous_Qty | PO_Quantity | Total_Qty_At_PO_Date | Date_Bucket |
|------------------|---------|--------|--------|--------------|------------|----------------------|-------------|
| 5/8/2025 4:12   | PHN1041 | 1406373 | 1      | 1384        | 20         | 1404                 | 1           |
| 5/8/2025 2:20   | PHN1041 | 1406373 | 2      | 1364        | 20         | 1384                 | 1           |
| 5/5/2025 3:28   | PHN1041 | 1407370 | 1      | 1354        | 10         | 1364                 | 1           |
| 5/5/2025 3:11   | PHN1041 | 1407370 | 2      | 1339        | 15         | 1354                 | 1           |
| 5/5/2025 2:34   | PHN1041 | 1407370 | 3      | 1324        | 15         | 1339                 | 1           |
| 5/5/2025 1:52   | PHN1041 | 1407370 | 4      | 1314        | 10         | 1324                 | 1           |
| 5/5/2025 1:39   | PHN1041 | 1407370 | 5      | 1299        | 15         | 1314                 | 1           |
| 5/5/2025 1:16   | PHN1041 | 1407370 | 6      | 1284        | 15         | 1299                 | 1           |
| 5/5/2025 0:46   | PHN1041 | 1407370 | 7      | 1269        | 15         | 1284                 | 1           |
| 4/17/2025 5:04  | PHN1041 | 1407199 | 1      | 1254        | 15         | 1269                 | 2           |
| 4/17/2025 4:50  | PHN1041 | 1407199 | 2      | 1244        | 10         | 1254                 | 2           |
| 4/17/2025 4:26  | PHN1041 | 1407199 | 3      | 1234        | 10         | 1244                 | 2           |
| 4/17/2025 4:12  | PHN1041 | 1407199 | 4      | 1224        | 10         | 1234                 | 2           |
| 4/16/2025 3:47  | PHN1041 | 1407346 | 1      | 1207        | 17         | 1224                 | 2           |
| 4/16/2025 3:28  | PHN1041 | 1407346 | 2      | 1192        | 15         | 1207                 | 2           |
| 4/1/2025 3:11   | PHN1041 | 1407346 | 3      | 1186        | 6          | 1192                 | 3           |
| 3/29/2025 2:34  | PHN1041 | 1407346 | 4      | 1046        | 140        | 1186                 | 4           |
| 3/27/2025 19:47 | PHN1041 | 1407346 | 5      | 1031        | 15         | 1046                 | 4           |


### Total_Qty_At_PO_Date (Calculated Column)
```
Total_Qty_At_PO_Date = 
VAR CurrentDate = [ACTION DATE/TIME]
VAR PartID = [PART_ID]

VAR StartingValue =
    CALCULATE(
        SUM('STOCK_STATUS'[ON_HAND_QTY]),
        ALLEXCEPT('STOCK_STATUS', 'STOCK_STATUS'[PART_ID]),
        'INVENTORY_TRANSACTION'[ACTION DATE/TIME] <= CurrentDate
    )

VAR CumulativeTotal =
    StartingValue +
    CALCULATE(
        SUMX(
            FILTER(
                ALL('INVENTORY_TRANSACTION'),
                'INVENTORY_TRANSACTION'[PART_ID] = PartID &&
                 'INVENTORY_TRANSACTION'[ACTION DATE/TIME]  > CurrentDate
            ),
            - 'INVENTORY_TRANSACTION'[PO_Quantity]  // Subtract QUANTITY (PO Qty)
        )
    )

RETURN
    CumulativeTotal
```
### Previous_Qty (Calculated Column)
```
Previous_Qty = 
IF(
    ('INVENTORY_TRANSACTION'[Total_Qty_At_PO_Date]-'INVENTORY_TRANSACTION'[PO_Quantity]) < 0,
        0,
        'INVENTORY_TRANSACTION'[Total_Qty_At_PO_Date]-'INVENTORY_TRANSACTION'[PO_Quantity] )
```
## Creating the 'AGING_INVENTORY_MATRIX' table
**1.**Starts with establishing a new table that pulls the 'PART_ID' and 'ON_HAND_QTY 'fields from the 'STOCK_STATUS' table. The 'STOCK_STATUS' provides a unique list of all Part IDs and their associated inventory levels, making it the best table to summarize the data.
```
AGING_INVENTORY_MATRIX= 
    SELECTCOLUMNS(T
        "PART_ID", STOCK_STATUS[PART_ID],
        "ON_HAND_QTY", STOCK_STATUS[ON_HAND_QTY]
    )
```
**2.**  Retrieve the Standard Unit Cost for each Part ID using a straightforward 'LOOKUPVALUE' function, ensuring blank values are handled effectively.
```
Unit Cost =
VAR UC =
    LOOKUPVALUE('PART_MANAGER'[Unit Cost], 'PART_MANAGER'[Part ID], 'AGING_INVENTORY_MATRIX'[Part ID])
RETURN
    IF(ISBLANK(UC), 0, UC)
```
**3.** Establish a calculated column that calculates the 'PO_Quantity' for the first date bucket by Part ID
```
DateBucket1_QTY =

IF(
    ISBLANK(
        SUMX(
            FILTER(
                'INVENTORY_TRANSACTION',
                'INVENTORY_TRANSACTION'[DateBucket] = 1 && 'AGING_INVENTORY_MATRIX'[PART_ID] = 'INVENTORY_TRANSACTION'[PART_ID] && 'INVENTORY_TRANSACTION'[PO_Quantity]>0
            ),
            'INVENTORY_TRANSACTION'[PO_Quantity]
        )
    ),
    0,
    SUMX(
        FILTER(
            'INVENTORY_TRANSACTION',
            'INVENTORY_TRANSACTION'[DateBucket] = 1 && 'AGING_INVENTORY_MATRIX'[PART_ID] = 'INVENTORY_TRANSACTION'[PART_ID] && 'INVENTORY_TRANSACTION'[PO_Quantity]>0
        ),
        'INVENTORY_TRANSACTION'[PO_Quantity]
    )
)
```
This Same process is repeated for creating each of the 6 'DateBucket#_QTY' columns

**4.**

