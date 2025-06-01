# Aging Inventory
This Power BI report analyzes aging inventory by categorizing total cost and stock quantities into time-based buckets.These buckets allow users to track stock movement, identify slow-moving items, and optimize inventory turnover to improve efficiency and reduce waste.

Since most components are not serialized, tracking individual part movement is a challenge. To estimate how long stock has remained at the facility, this report leverages Purchase Order Transactions and assumes a FIFO (First-In, First-Out) inventory management approach.
| ACTION DATE/TIME  | PART_ID  | PO_ID   | Previous Qty | Quantity | Total Qty At PO Date |
|------------------|---------|--------|--------------|----------|----------------------|
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
