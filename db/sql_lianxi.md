## 检索
从Products表中检索一个名为prod_name的列


```
SELECT prod_name
FROM Product s;
```

从Products表中检索下面的列：prod_id, prod_name, prod_price


```
SELECT prod_id, prod_name, prod_price
FROM Product s;
```

检索所有列

```
SELECT *
FROM Product s;
```

检索vend_id,只返回不同的值

```
SELECT DISTINCT vend_id
FROM Product s;
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
WHERE prod_price IS  NULL;

SELECT cust_name
FROM CUSTOMERS
WHERE cust_email IS  NULL;
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

