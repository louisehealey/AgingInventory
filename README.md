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
- **`INVENTORY_TRANSACTION`**: Log of inventory transactions, used tohe tracking and  of stock age by invidual transactions.
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

In the following example, the `Total Qty At PO Date` for the maximum `ACTION DATE/TIME` matches the `ON_HAND_QTY` recorded in the `STOCK_STATUS` table for `PART_ID`: **PHN1041**. The `PO_Quantity` is then deducted, yielding the `Previous Qty` value: 1404 - 20 = 1348. The next PO transaction is then subtracted from the `Previous Qty`, continuing the process of inventory depletion.

| ACTION DATE/TIME  | PART_ID  | PO_ID   | Previous Qty | PO_Quantity | Total_Qty_At_PO_Date |
|------------------|---------|--------|--------------|----------|----------------------------|
| 5/8/2025 4:12   | PHN1041 | 1406373 | 1384        | 20       | 1404                 |
| 5/8/2025 2:20   | PHN1041 | 1406373 | 1364        | 20       | 1384                 |
| 5/5/2025 3:28   | PHN1041 | 1407370 | 1354        | 10       | 1364                 |
| 5/5/2025 3:11   | PHN1041 | 1407370 | 1339        | 15       | 1354                 |
| 5/5/2025 2:34   | PHN1041 | 1407370 | 1324        | 15       | 1339                 |
| 5/5/2025 1:52   | PHN1041 | 1407370 | 1314        | 10       | 1324                 |
| 5/5/2025 1:39   | PHN1041 | 1407370 | 1299        | 15       | 1314                 |
| 5/5/2025 1:16   | PHN1041 | 1407370 | 1284        | 15       | 1299                 |
| 5/5/2025 0:46   | PHN1041 | 1407370 | 1269        | 15       | 1284                 |
| 4/17/2025 5:04  | PHN1041 | 1407199 | 1254        | 15       | 1269                 |
| 4/17/2025 4:50  | PHN1041 | 1407199 | 1244        | 10       | 1254                 |
| 4/17/2025 4:26  | PHN1041 | 1407199 | 1234        | 10       | 1244                 |
| 4/17/2025 4:12  | PHN1041 | 1407199 | 1224        | 10       | 1234                 |
| 4/16/2025 3:47  | PHN1041 | 1407346 | 1207        | 17       | 1224                 |
| 4/16/2025 3:28  | PHN1041 | 1407346 | 1192        | 15       | 1207                 |
| 4/1/2025 3:11   | PHN1041 | 1407346 | 1186        | 6        | 1192                 |
| 3/29/2025 2:34  | PHN1041 | 1407346 | 1046        | 140      | 1186                 |
| 3/27/2025 19:47 | PHN1041 | 1407346 | 1031        | 15       | 1046                 |

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
            - 'INVENTORY_TRANSACTION'[PO_Quantity] 
```
### Previous_Qty (Calculated Column)
```
Previous_Qty = 
IF(
    ('INVENTORY_TRANSACTION'[Total_Qty_At_PO_Date]-'INVENTORY_TRANSACTION'[PO_Quantity]) < 0,
        0,
        'INVENTORY_TRANSACTION'[Total_Qty_At_PO_Date]-'INVENTORY_TRANSACTION'[PO_Quantity] )
```
