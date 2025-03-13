## **表结构**



我们将创建四个表结构，涵盖从基础到高级的各类问题。



 1. **Users** 表（用户信息）

    ```sql
    CREATE TABLE Users (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(100) NOT NULL,
        age INT,
        email VARCHAR(100) UNIQUE,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    );
    
    INSERT INTO Users (name, age, email) VALUES
    ('John Doe', 30, 'john@example.com'),
    ('Jane Smith', 25, 'jane@example.com'),
    ('Michael Brown', 35, 'michael@example.com'),
    ('Emily Davis', 28, 'emily@example.com'),
    ('Daniel Wilson', 22, 'daniel@example.com');
    ```

    

2. **Orders** 表（订单信息）

   ```sql
   CREATE TABLE Orders (
       order_id INT AUTO_INCREMENT PRIMARY KEY,
       user_id INT,
       product_name VARCHAR(100),
       amount DECIMAL(10, 2),
       order_date DATE,
       FOREIGN KEY (user_id) REFERENCES Users(id)
   );
   
   INSERT INTO Orders (user_id, product_name, amount, order_date) VALUES
   (1, 'Laptop', 1200.00, '2024-08-01'),
   (2, 'Smartphone', 800.00, '2024-08-02'),
   (1, 'Tablet', 300.00, '2024-08-03'),
   (3, 'Headphones', 150.00, '2024-08-04'),
   (4, 'Smartwatch', 200.00, '2024-08-05');
   ```

   

3. **Products** 表（产品信息）

   ```sql
   CREATE TABLE Products (
       product_id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(100),
       category VARCHAR(50),
       price DECIMAL(10, 2),
       stock INT
   );
   
   INSERT INTO Products (name, category, price, stock) VALUES
   ('Laptop', 'Electronics', 1200.00, 50),
   ('Smartphone', 'Electronics', 800.00, 100),
   ('Tablet', 'Electronics', 300.00, 75),
   ('Headphones', 'Accessories', 150.00, 200),
   ('Smartwatch', 'Accessories', 200.00, 150);
   ```

   

4. **OrderDetails** 表（订单详情）

   ```sql
   CREATE TABLE OrderDetails (
       detail_id INT AUTO_INCREMENT PRIMARY KEY,
       order_id INT,
       product_id INT,
       quantity INT,
       FOREIGN KEY (order_id) REFERENCES Orders(order_id),
       FOREIGN KEY (product_id) REFERENCES Products(product_id)
   );
   
   INSERT INTO OrderDetails (order_id, product_id, quantity) VALUES
   (1, 1, 1), -- Laptop
   (2, 2, 1), -- Smartphone
   (3, 3, 2), -- Tablet
   (4, 4, 3), -- Headphones
   (5, 5, 1); -- Smartwatch
   ```
   
   



## **题目和答案**

#### **基础部分**

1. **选择所有用户的信息。**

   ```sql
   SELECT * FROM Users;
   ```

2. **选择所有订单的订单号和金额。**

   ```sql
   SELECT order_id, amount FROM Orders;
   ```

3. **为用户的名称和电子邮件设置别名。**

   ```sql
   SELECT name AS user_name, email AS user_email FROM Users;
   ```

4. **选择所有不重复的产品类别。**

   ```sql
   SELECT DISTINCT category FROM Products;
   ```

5. **选择年龄大于25岁的用户。**

   ```sql
   SELECT * FROM Users WHERE age > 25;
   ```

6. **按价格从高到低排序选择所有产品。**

   ```sql
   SELECT * FROM Products ORDER BY price DESC;
   ```

7. **选择最新的5个订单。**

   ```sql
   SELECT * FROM Orders ORDER BY order_date DESC LIMIT 5;
   ```

8. **选择金额在50到150之间的订单。**

   ```sql
   SELECT * FROM Orders WHERE amount BETWEEN 50 AND 150;
   ```

9. **选择产品ID为1、3、5的所有订单详情。**

   ```sql
   SELECT * FROM OrderDetails WHERE product_id IN (1, 3, 5);
   ```

10. **选择名称中包含"phone"的所有产品。**

    ```sql
    SELECT * FROM Products WHERE name LIKE '%phone%';
    ```

#### **中级部分**

1. **统计用户表中的用户数量。**

   ```sql
   SELECT COUNT(*) FROM Users;
   ```

2. **计算所有订单的总金额。**

   ```sql
   SELECT SUM(amount) FROM Orders;
   ```

3. **计算产品表中所有产品的平均价格。**

   ```sql
   SELECT AVG(price) FROM Products;
   ```

4. **找到库存中数量最多和最少的产品。**

   ```sql
   SELECT MAX(stock), MIN(stock) FROM Products;
   ```

5. **按产品类别分组并统计每个类别的产品数量。**

   ```sql
   SELECT category, COUNT(*) FROM Products GROUP BY category;
   ```

6. **筛选出产品数量多于10个的类别。**

   ```sql
   SELECT category, COUNT(*) FROM Products GROUP BY category HAVING COUNT(*) > 10;
   ```

7. **连接用户表和订单表，选择用户的名称和对应的订单金额。**

   ```sql
   SELECT Users.name, Orders.amount FROM Users 
   INNER JOIN Orders ON Users.id = Orders.user_id;
   ```

8. **使用`LEFT JOIN`连接产品表和订单详情表，选择所有产品及其对应的订单数量（即使没有订单也要显示产品信息）。**

   ```sql
   SELECT Products.name, OrderDetails.quantity FROM Products 
   LEFT JOIN OrderDetails ON Products.product_id = OrderDetails.product_id;
   ```

9. **使用`RIGHT JOIN`连接用户表和订单表，选择所有订单及其对应的用户名（即使用户没有下单也要显示订单信息）。**

   ```sql
   SELECT Orders.order_id, Users.name FROM Orders 
   RIGHT JOIN Users ON Orders.user_id = Users.id;
   ```

10. **使用`FULL OUTER JOIN`连接订单表和订单详情表，选择所有订单及其详情（如果数据库支持）。**

    ```sql
    SELECT Orders.order_id, OrderDetails.quantity FROM Orders 
    FULL OUTER JOIN OrderDetails ON Orders.order_id = OrderDetails.order_id;
    ```

#### **进阶部分**

1. **在`JOIN`语句中为用户和订单表设置别名并查询。**

   ```sql
   SELECT u.name, o.amount FROM Users AS u 
   JOIN Orders AS o ON u.id = o.user_id;
   ```

2. **使用子查询查找订单金额最大的订单。**

   ```sql
   SELECT * FROM Orders 
   WHERE amount = (SELECT MAX(amount) FROM Orders);
   ```

3. **在`WHERE`子句中使用子查询，选择下过订单的用户。**

   ```sql
   SELECT * FROM Users 
   WHERE id IN (SELECT user_id FROM Orders);
   ```

4. **在`FROM`子句中使用子查询，选择最近的5个订单。**

   ```sql
   SELECT * FROM (SELECT * FROM Orders ORDER BY order_date DESC LIMIT 5) AS recent_orders;
   ```

5. **使用`UNION`合并两个查询结果，显示产品和订单金额（假设订单金额也可以被视为产品的价格）。**

   ```sql
   SELECT name AS item, price FROM Products 
   UNION 
   SELECT product_name AS item, amount FROM Orders;
   ```

6. **使用`UNION`查询两个表中的数据，并去除重复结果。**

   ```sql
   SELECT name FROM Users 
   UNION 
   SELECT name FROM Products;
   ```

7. **创建一个视图，显示所有用户的名称及其对应的订单金额。**

   ```sql
   CREATE VIEW UserOrders AS 
   SELECT Users.name, Orders.amount FROM Users 
   JOIN Orders ON Users.id = Orders.user_id;
   ```

8. **更新视图中某个用户的订单金额（假设允许直接更新视图）。**

   ```sql
   UPDATE UserOrders SET amount = 150 WHERE name = 'John Doe';
   ```

9. **使用`ALTER`语句为`Products`表添加一个新的列`description`。**

   ```sql
   ALTER TABLE Products ADD description VARCHAR(255);
   ```

10. **删除`Orders`表中的所有数据，但保留表结构。**

    ```sql
    DELETE FROM Orders;
    ```

#### **高级部分**

1. **使用事务将用户表和订单表的操作组合在一起。**

   ```sql
   START TRANSACTION;
   INSERT INTO Users (name, age, email) VALUES ('Jane Doe', 28, 'jane@example.com');
   INSERT INTO Orders (user_id, product_name, amount, order_date) 
   VALUES (LAST_INSERT_ID(), 'Laptop', 1200, '2024-08-20');
   COMMIT;
   ```

2. **在插入数据后使用`ROLLBACK`撤销对订单表的更改。**

   ```sql
   START TRANSACTION;
   INSERT INTO Orders (user_id, product_name, amount, order_date) 
   VALUES (1, 'Smartphone', 800, '2024-08-20');
   ROLLBACK;
   ```

3. **在插入数据后使用`COMMIT`提交对订单表的更改。**

   ```sql
   START TRANSACTION;
   INSERT INTO Orders (user_id, product_name, amount, order_date) 
   VALUES (1, 'Smartphone', 800, '2024-08-20');
   COMMIT;
   ```

4. **创建一个存储过程，用于根据用户ID查询该用户的所有订单。**

   ```sql
   CREATE PROCEDURE GetUserOrders(IN userId INT)
   BEGIN
       SELECT * FROM Orders WHERE user_id = userId;
   END;
   ```

5. **在存储过程中使用输入和输出参数，返回用户的订单总金额。**

   ```sql
   CREATE PROCEDURE GetUserTotalAmount(IN userId INT, OUT totalAmount DECIMAL(10,2))
   BEGIN
       SELECT SUM(amount) INTO totalAmount FROM Orders WHERE user_id = userId;
   END;
   ```

6. **创建一个触发器，当插入新订单时自动更新产品的库存数量。**

   ```sql
   CREATE TRIGGER UpdateStock AFTER INSERT ON OrderDetails
   FOR EACH ROW
   BEGIN
       UPDATE Products SET stock = stock - NEW.quantity WHERE product_id = NEW.product_id;
   END;
   ```

7. **使用触发器在数据插入或更新时自动执行操作，防止库存数量为负。**

   ```sql
   CREATE TRIGGER PreventNegativeStock BEFORE UPDATE ON Products
   FOR EACH ROW
   BEGIN
       IF NEW.stock < 0 THEN
           SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Stock cannot be negative';
       END IF;
   END;
   ```

8. **通过添加索引优化查询以提高性能。**

   ```sql
   CREATE INDEX idx_user_id ON Orders(user_id);
   ```

9. **使用索引提高查询速度，例如在`Orders`表中为`order_date`列添加索引。**

   ```sql
   CREATE INDEX idx_order_date ON Orders(order_date);
   ```

10. **分析查询执行计划，检查优化效果。**

    ```sql
    EXPLAIN SELECT * FROM Orders WHERE user_id = 1;
    ```

#### **挑战部分**

1. **使用分区优化大型表的查询，例如按`order_date`进行分区。**

   ```sql
   ALTER TABLE Orders PARTITION BY RANGE (YEAR(order_date)) (
       PARTITION p0 VALUES LESS THAN (2022),
       PARTITION p1 VALUES LESS THAN (2023),
       PARTITION p2 VALUES LESS THAN (2024),
       PARTITION p3 VALUES LESS THAN MAXVALUE
   );
   ```

2. **使用正则表达式查询名称以"A"开头并以"n"结尾的用户。**

   ```sql
   SELECT * FROM Users WHERE name REGEXP '^A.*n$';
   ```

3. **在MySQL中使用递归查询查找所有产品的类别层级（假设有层级关系）。**

   ```sql
   WITH RECURSIVE CategoryHierarchy AS (
       SELECT product_id, name, category
       FROM Products
       WHERE parent_category IS NULL
       UNION ALL
       SELECT p.product_id, p.name, p.category
       FROM Products p
       JOIN CategoryHierarchy ch ON p.parent_category = ch.category
   )
   SELECT * FROM CategoryHierarchy;
   ```

4. **处理并发和锁机制，防止同时修改同一产品的库存。**

   ```sql
   START TRANSACTION;
   SELECT stock FROM Products WHERE product_id = 1 FOR UPDATE;
   UPDATE Products SET stock = stock - 1 WHERE product_id = 1;
   COMMIT;
   ```

5. **管理MySQL用户权限，授予用户`db_admin`对数据库的完全访问权限。**

   ```sql
   GRANT ALL PRIVILEGES ON my_database.* TO 'db_admin'@'localhost';
   FLUSH PRIVILEGES;
   ```

6. **使用MySQL的`JSON`数据类型存储用户的附加信息，并查询某个用户的`address`字段。**

   ```sql
   SELECT JSON_EXTRACT(additional_info, '$.address') FROM Users WHERE id = 1;
   ```

7. **使用全文检索查找包含特定关键词的订单产品名称。**

   ```sql
   CREATE FULLTEXT INDEX ft_index ON Orders(product_name);
   SELECT * FROM Orders WHERE MATCH(product_name) AGAINST('Laptop');
   ```

8. **备份和恢复MySQL数据库。**

   ```sql
   mysqldump -u root -p my_database > backup.sql;
   mysql -u root -p my_database < backup.sql;
   ```

9. **在`Orders`表中设置外键约束，确保`user_id`的值存在于`Users`表中。**

   ```sql
   ALTER TABLE Orders ADD CONSTRAINT fk_user_id 
   FOREIGN KEY (user_id) REFERENCES Users(id);
   ```

10. **处理数据库中的死锁，通过设置锁等待超时时间减少死锁发生的可能性。**

    ```sql
    SET innodb_lock_wait_timeout = 50;
    ```