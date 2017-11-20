## 检索
从Products表中检索一个名为prod_name的列


```
SELECT prod_name
FROM Products;
```

从Products表中检索下面的列：prod_id, prod_name, prod_price


```
SELECT prod_id, prod_name, prod_price
FROM Products;
```

检索所有列

```
SELECT *
FROM Products;
```

检索vend_id,只返回不同的值

```
SELECT DISTINCT vend_id
FROM Products;
```

###### 注释
分为行内注释 --，单行注释 #，多行注释 /* ... */。

## 排序检索数据

检索prod_name，并以prod_name排序

```
SELECT prod_name
FROM Products
ORDER BY prod_name;
```

检索 prod_id,prod_price,prod_name,以 prod_price, prod_name 排序

```
SELECT prod_id, prod_price, prod_name
FROM Products
ORDER BY prod_price, prod_name;
```
检索 prod_id, prod_price, prod_name, 以 prod_price 降序排列

```
SELECT prod_id, prod_price, prod_name
FROM Products
ORDER BY prod_price DESC;
```

## 过滤数据

从products表中检索两个列 prod_name, prod_price，只返回prod_price值为3.49的行

```
SELECT prod_name, prod_price
FROM Products
WHERE prod_price = 3.49;
```

检索 prod_name, prod_price, 列出所有价格小于10美元的产品。

```
SELECT prod_name, prod_price
FROM Products
WHERE prod_price < 10;
```

检索 vend_id, prod_name, 列出不是供应商DLL01制造的产品。
```
SELECT vend_id, prod_name
FROM Products
WHERE vend_id <> 'DLL01';
```

检索 prod_name, prod_price , 价格在5美元和10美元之间的所有产品。

```
SELECT prod_name, prod_price
FROM Products
WHERE prod_price BETWEEN  5 AND 10;
```

1. 检查空值，检索 prod_name ，列出 prod_price 为空的行。 2. 从 CUSTOMERS 中检索 cust_name ，列出 cust_email 为空的行。

```
SELECT prod_name
FROM Products
WHERE prod_price IS NULL;

SELECT cust_name
FROM CUSTOMERS
WHERE cust_email IS NULL;
```

从 Products 表中检索 prod_id, prod_price, prod_name ， vend_id 为 DLL01 并且 prod_price <= 4。

```
SELECT prod_id, prod_price, prod_name
FROM Products
WHERE vend_id = 'DLL01' AND  prod_price <= 4;
```

从 Products 表中检索 prod_id, prod_price, prod_name ， vend_id 为 DLL01 并且 prod_price <= 4 ，并按价格排序。

```
SELECT prod_id, prod_price, prod_name
FROM Products
WHERE vend_id = 'DLL01' AND  prod_price <= 4
ORDER BY prod_price;
```

从 Products 表中检索 prod_name, prod_price ，vend_id 为 DLL01 或者 BRS01 。

```
SELECT prod_name, prod_price
FROM Products
WHERE vend_id = 'DLL01' OR vend_id = ‘BRS01’;
```

从 Products 表中检索 prod_name, prod_price , 列出价格为10美元及以上，且由DLL01或BRS01制造的所有产品。

```
SELECT prod_name, prod_price
FROM Products
WHERE (vend_id = 'DLL01' OR vend_id = 'BRS01')
AND  prod_price >= 10;
```

从 Products 表中检索 prod_name, prod_price , 用 IN 操作符指定提供商为 DLL01 或 BRS01 ，按照 prod_price 排序 。

```
SELECT prod_name, prod_price
FROM Products
WHERE vend_id IN  ( 'DLL01', 'BRS01' )
ORDER BY prod_name;
```

从 Products 表中检索 prod_name, 列出除DLL01之外的所有供应商制造的产品。

```
SELECT prod_name
FROM Products
WHERE NOT vend_id = 'DLL01'
ORDER BY prod_name;
```

## 通配符过滤

从 Products 表中检索 prod_id,prod_name, 列出 1.Fish起头的产品; 2.包含文本bean bag的产品; 

```
SELECT prod_id, prod_name
FROM Products
WHERE prod_name LIKE 'Fish%';

SELECT prod_id, prod_name
FROM Products
WHERE prod_name LIKE '%bean bag%';
```

## 计算字段

从 Vendors 表中获取 vend_name,vend_country,  拼接成如下形式 vend_name(vend_country), 例如 Bear Emporium(USA), 新字段名称为 vend_title。

```
SELECT Concat (vend_name, ' (', vend_country , ')')
AS  vend_title
FROM Vendors
ORDER BY vend_name;
```

1. 从 OrderItems 表中获取 prod_id, quantity, item_price ，条件 order_num = 20008 。 2. 检索上述列并计算总价 expanded_price 。
```
SELECT prod_id, quantity , item_price
FROM OrderItems
WHERE order_num = 20008;

SELECT prod_id,
quantity ,
item_price,
quantity * item_price AS expanded_price
FROM OrderItems
WHERE order_num = 20008;
```

## 函数

从 Vendors 表中获取 vend_name ,并且输出大写的版本,按名称排序。

```
SELECT vend_name, UPPER(vend_name) AS vend_name_upcase
FROM Vendors
ORDER BY vend_name;
```

从 Orders 表中获取 order_num ，只获取2012年的记录

```
SELECT order_num
FROM Orders
WHERE YEAR(order_date) = 2012;
```

## 汇总数据

AVG()返回Products表中所有产品的平均价格。

```
SELECT AVG (prod_price) AS avg_price
FROM Products;
```

返回Products表中所有供应商(vend_id)为 DLL01 的产品的平均价格

```
SELECT AVG(prod_price) AS  avg_price
FROM Products
WHERE vend_id = 'DLL01';
```

统计表 Customers 中行的总数。包括NULL。

```
SELECT COUNT(*) AS num_cust
FROM Customers;
```

统计表 Customers 中行的总数。只统计具有电子邮件地址(cust_email)的客户。

```
SELECT COUNT(cust_email) AS num_cust
FROM Customers;
```

统计表 Products 中 prod_price 列中的最大值。

```
SELECT MAX(prod_price) AS max_price
FROM Products;
```

表 OrderItems 包含订单中实际的物品，每个物品有相应的数量。检索order_num = 20005的所订购物品的总数（所有quantity值之和）

```
SELECT SUM(quantity ) AS items_ordered
FROM OrderItems
WHERE order_num = 20005;
```

表 OrderItems 中，合计每项物品的总价 item_price * quantity，得出订单号 order_num = 20005 的每个物品的总金额。

```
SELECT item_price* quantity AS  total_price
FROM OrderItems
WHERE order_num = 20005;
```

表 OrderItems 中，合计每项物品的总价 item_price * quantity，得出订单号 order_num = 20005 总的金额。

```
SELECT SUM(item_price* quantity ) AS  total_price
FROM OrderItems
WHERE order_num = 20005;
```

统计表 Products 中供应商 vend_id = DLL01 提供的产品的平均价格，只统计不同的价格。

```
SELECT AVG (DISTINCT prod_price) AS avg_price
FROM Products
WHERE vend_id = 'DLL01';
```

统计表 Products 中所有产品数量 num_items ，最低价格 price_min ，最高价格 price_max ，平均价格 price_avg 。

```
SELECT COUNT(*) AS num_items,
MIN(prod_price) AS price_min,
MAX(prod_price) AS price_max ,
AVG(prod_price) AS price_avg
FROM Products;
```

## 分组数据

检索 vend_id 和每个供应商提供的产品数量。

```
SELECT vend_id, COUNT(*) AS num_prods
FROM Products
GROUP BY vend_id;
```

从 Orders 表中检索 cust_id, 每个 cust_id 的订单数量。

```
SELECT vend_id, COUNT(*) AS num_prods
FROM Products
GROUP BY vend_id;
```

从 Orders 表中检索 cust_id, 订单数量大于等于2的 cust_id 的订单数量。

```
SELECT cust_id, COUNT(*) AS orders
FROM Orders
GROUPBY cust_id
HAVING  COUNT(*) >= 2;
```

从表 Products 中，列出具有两个以上产品且其价格大于等于4的供应商。

```
SELECT vend_id, COUNT(*) AS num_prods
FROM Products
WHERE prod_price >= 4
GROUP BY vend_id
HAVING COUNT(*) >= 2;
```

从表 OrderItems 中按订单号（order_num列）分组数据,返回每个订单中的物品数目 items，只返回三个或更多物品的订单。按照 items,order_num 排序输出。

```
SELECT order_num, COUNT(*) AS items
FROM OrderItems
GROUP BY order_num
HAVING  COUNT(*) >= 3
ORDER BY items,order_num;
```

## 子查询

先从 OrderItems 中检索产品id(prod_id) 为 RGAN01 的订单号(order_num)，通过订单号在表 Orders 中检索用户 cust_id。使用子查询。

```
SELECT cust_id
FROM Orders
WHERE order_num IN  (SELECT order_num
    FROM OrderItems
    WHERE prod_id = 'RGAN01');
```

从 Customers 表中检索顾客列表；对于检索出的每个顾客，统计其在 Orders 表中的订单数目。 检索列 cust_name , cust_state ,  订单数量。

```
SELECT cust_name,
  cust_state,
  (SELECT COUNT(*)
  FROM Orders
  WHERE Orders.cust_id = Customers.cust_id) AS orders
FROM Customers
ORDER BY cust_name;
```

## 联结表

使用联结，检索 Vendors ， Products 表中的列 vend_name, prod_name, prod_price 。

```
SELECT vend_name, prod_name, prod_price
FROM Vendors, Products
WHERE Vendors.vend_id = Products.vend_id;
```

使用内联结完成上面习题。

```
SELECT vend_name, prod_name, prod_price
FROM Vendors INNER JOIN Products
ON Vendors.vend_id = Products.vend_id;
```

检索 prod_name, vend_name, prod_price, quantity ，order_num = 20007 

```
SELECT prod_name, vend_name, prod_price, quantity
FROM OrderItems, Products, Vendors
WHERE Products.vend_id = Vendors.vend_id
AND OrderItems.prod_id = Products.prod_id
AND order_num = 20007;
```
