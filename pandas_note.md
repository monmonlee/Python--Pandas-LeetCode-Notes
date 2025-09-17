# Python pandas 語法筆記

## 索引

- [如何條件篩選](#filtering)
- [如何合併查找兩張表](#merge(),isin(),set())
---

## `如何條件篩選`

**範例：**
```python
# 題目：
# A country is big if:
# it has an area of at least three million (i.e., 3000000 km2), or it has a population of at least twenty-five million (i.e., 25000000).
# Write a solution to find the name, population, and area of the big countries.


#解答

import pandas as pd 
def find_big_country():
    df = world[(world["area"] >= 3000000)|(world["population"] >= 25000000)]
    return df[["name", "population", "area"]]

```

**架構介紹：**
- pandas的過濾：`df = world[(world["area"] >= 3000000])`
    - 條件需要用`()`包起來
    - `|` or, `&` and, `~`not
- 返回`[[]]`和`[]`的差別
    - `[]` 只會回傳 series 類型的資料，單一欄位且一維
    - `[[]]` 回傳 dataframe 類型的資料，不限欄位（一欄也可）、二維
    - 所以題目要求返回df的三個欄位，所以是 `df[['name', 'area', 'population']]`

---

## `如何合併查找兩張表`
- 題目：有 customers（顧客總表）& orders（訂單） 兩張表，現在要找出「沒有訂購紀錄」的顧客「姓名」
- 解答：有三種作法

**範例一：merge()**
```python
def find_ids():
    merged = customers.merge(
        orders,
        left_on = "id",
        right_on = "customerId",
        how = "left")
        # merge() → 左表.merge(右表, 左表的key, 右表的key, join方式)

    result = merged[merged['customerId'].isna()] # 找出 customerId 是 NaN 的列
    return result[['name']]
```
- merge()最像sql
- 找「沒有訂購紀錄」的方法：合併後，如果沒有存在於訂購表，customerId會是NaN，因此透過`.isna()`就可以過濾出要的答案
- 結構：合併兩張表 → 找到兩張表中為nan的欄位 → 印出這些欄位的姓名

**範例二：.isin()**
```python
def find_ids():

    all_orders_id = orders['customerId'].unique()
    no_orders = customers[~customers['id'].isin(all_orders_id)]
    return no_orders[['name']]
```
- 結構重點：利用邏輯反轉
- 結構：找出訂單行為的唯一值 → 找到總表中「非」在「訂單名單」中的人 → 返回姓名 

**範例三：set()**
```python
def find_ids():
    
    all_customers = set(customer['id'])
    order_customers = set(orders['customerId'])
    no_orders = all_customers - order_customers
    results = customer[customer['id'].isin(no_orders)]
    return results[['name']]

```
- 結構重點：利用set的特性（數學的集、唯一值）
- 結構：兩張表的id都轉換成set → 找到非交集的一群 → 在原表.isin()找到要的一群人

**結論**
- `set()`最快、中數據最適合用`.isin()`
- 只有 `merge()` 可以處理多欄位關聯，且最直觀（適合邏輯複雜）
- `.isin()`可以接受的資料型態
    - list, numpy array, series, set, another dataframe column
---


