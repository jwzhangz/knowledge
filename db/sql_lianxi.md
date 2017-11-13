## 检索
###### 从Products表中检索一个名为prod_name的列

```
SELECT prod_name
FROM Product s;
```

###### 从Products表中检索下面的列：prod_id, prod_name, prod_price

```
SELECT prod_id, prod_name, prod_price
FROM Product s;
```

###### 检索所有列
```
SELECT *
FROM Product s;
```

###### 检索vend_id,只返回不同的值
```
SELECT DISTINCT vend_id
FROM Product s;
```

###### 注释
分为行内注释 --，单行注释 #，多行注释 /* ... */。

## 排序检索数据

###### 检索prod_name，并以prod_name排序
```
SELECT prod_name
FROM Products
ORDER BY prod_name;
```

###### 检索 prod_id,prod_price,prod_name,以 prod_price, prod_name 排序
```
SELECT prod_id, prod_price, prod_name
FROM Products
ORDER BY prod_price, prod_name;
```
###### 检索 prod_id, prod_price, prod_name, 以 prod_price 降序排列
```
SELECT prod_id, prod_price, prod_name
FROM Products
ORDER BY prod_price DESC;
```

## 过滤数据
