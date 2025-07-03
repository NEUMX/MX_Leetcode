# SQL算法

---

### 一、高频SQL50题（基础版）
#### 题目分类
1. **查询**：1-5
2. **连接**：6-14
3. **聚合函数**：15-22
4. **排序和分组**：23-29
5. **高级查询和连接**：30-36
6. **子查询**：37-43
7. **高级字符串函数/正则表达式/子句**：44-50

#### 题目及带注释的答案
1. **1757. 可回收且低脂的产品**

```sql
-- 目的：查找既是低脂又是可回收的产品编号
-- WHERE条件：low_fats='Y'表示低脂，recyclable='Y'表示可回收
SELECT product_id
FROM Products 
WHERE low_fats = 'Y' AND recyclable = 'Y';
```

2. **584. 寻找用户推荐人**

```sql
-- 目的：找出没有被ID=2的用户推荐的客户姓名
-- WHERE条件：referee_id != 2排除ID=2推荐的客户，IS NULL包括未填写推荐人的客户
SELECT name
FROM Customer
WHERE referee_id != 2 OR referee_id IS NULL;
```

3. **595. 大的国家**

```sql
-- 目的：查找面积>=300万或人口>=2500万的国家
-- WHERE条件：使用OR连接两个条件，满足任一条件即返回
SELECT name, population, area
FROM World
WHERE area >= 3000000 OR population >= 25000000;
```

4. **1148. 文章浏览 I**

```sql
-- 目的：找出浏览自己文章的作者ID
-- DISTINCT：去重重复的author_id
-- WHERE条件：author_id=viewer_id表示同一人
-- ORDER BY：按ID升序排列
SELECT DISTINCT author_id AS id
FROM Views
WHERE author_id = viewer_id
ORDER BY id;
```

5. **1683. 无效的推文**

```sql
-- 目的：找出内容长度超过15个字符的推文ID
-- LENGTH(content)：计算内容长度
-- WHERE条件：筛选出长度>15的推文
SELECT tweet_id
FROM Tweets
WHERE LENGTH(content) > 15;
```

6. **1378. 使用唯一标识码替换员工ID**

```sql
-- 目的：将员工ID替换为唯一标识码并返回姓名
-- LEFT JOIN：保留所有员工，即使没有唯一标识码（unique_id可能为NULL）
-- ON条件：通过ID关联两表
SELECT unique_id, name
FROM Employees 
LEFT JOIN EmployeeUNI ON EmployeeUNI.id = Employees.id;
```

7. **1068. 产品销售分析 I**

```sql
-- 目的：返回产品名称、销售年份和价格
-- JOIN：通过product_id连接Sales和Product表
SELECT product_name, year, price
FROM Sales 
JOIN Product ON Sales.product_id = Product.product_id;
```

8. **1581. 进店却未进行过交易的顾客**

```sql
-- 目的：统计每个顾客未交易的访问次数
-- LEFT JOIN：保留所有访问记录，即使没有交易（transaction_id为NULL）
-- WHERE条件：筛选未交易的记录
-- GROUP BY：按顾客ID分组，COUNT统计次数
SELECT customer_id, COUNT(*) AS count_no_trans
FROM Visits v 
LEFT JOIN Transactions t ON v.visit_id = t.visit_id
WHERE transaction_id IS NULL
GROUP BY customer_id;
```

9. **197. 上升的温度**

```sql
-- 目的：找出比前一天温度高的日期ID
-- JOIN：自连接Weather表，w2日期比w1晚一天
-- WHERE条件：w2温度高于w1温度
SELECT w2.id AS id
FROM Weather w1 
JOIN Weather w2 ON w1.recordDate = w2.recordDate - INTERVAL 1 DAY
WHERE w1.Temperature < w2.Temperature;
```

10. **1661. 每台机器的进程平均运行时间**
    - **方法1（运行时间17%）**

```sql
-- 目的：计算每台机器的平均进程运行时间
-- 子查询：分别筛选start和end活动
-- JOIN：通过machine_id和process_id匹配开始和结束时间
-- AVG：计算时间差平均值，ROUND保留3位小数
SELECT start.machine_id, ROUND(AVG(end.timestamp - start.timestamp), 3) AS processing_time
FROM 
    (SELECT * FROM Activity WHERE activity_type = 'start') AS start
JOIN 
    (SELECT * FROM Activity WHERE activity_type = 'end') AS end
ON start.machine_id = end.machine_id AND start.process_id = end.process_id
GROUP BY start.machine_id;
```

    - **方法2（运行时间45%）**

```sql
-- 目的：同上，使用另一种方式计算平均时间
-- CASE WHEN：start时间取负值，end时间取正值
-- SUM和COUNT：计算总时间差并平均，乘2调整结果
SELECT machine_id, 
       ROUND((2 * SUM(timestamp * (CASE WHEN activity_type = 'start' THEN -1 ELSE 1 END))) / COUNT(activity_type), 3) AS processing_time
FROM Activity
GROUP BY machine_id;
```

11. **577. 员工奖金**

```sql
-- 目的：找出奖金<1000或无奖金的员工
-- LEFT JOIN：保留所有员工，即使没有奖金记录
-- WHERE条件：筛选奖金<1000或为空的记录
SELECT name, bonus
FROM Employee 
LEFT JOIN Bonus ON Employee.empId = Bonus.empId
WHERE bonus < 1000 OR bonus IS NULL;
```

12. **1280. 学生们参加各科测试的次数**

```sql
-- 目的：统计每个学生每科的考试次数，未参加记为0
-- 子查询a：生成学生和科目所有组合
-- 子查询b：统计实际考试次数
-- LEFT JOIN：匹配考试记录，未匹配的用IFNULL填0
SELECT a.student_id, a.student_name, a.subject_name, IFNULL(attended_exams, 0) AS attended_exams
FROM 
    (SELECT * FROM Subjects JOIN Students) a
LEFT JOIN 
    (SELECT *, COUNT(e.student_id) AS attended_exams
     FROM Examinations e
     GROUP BY e.student_id, e.subject_name) b
ON a.student_id = b.student_id AND a.subject_name = b.subject_name
ORDER BY a.student_id, a.subject_name;
```

13. **570. 至少有5名直接下属的经理**

```sql
-- 目的：找出至少有5名下属的经理姓名
-- LEFT JOIN：通过managerId关联员工和经理
-- GROUP BY和HAVING：统计每个经理的下属人数，筛选>=5的
SELECT e2.name AS name
FROM Employee e2
LEFT JOIN Employee e1 ON e1.managerId = e2.id
GROUP BY e2.id
HAVING COUNT(*) >= 5;
```

14. **1934. 确认率**

```sql
-- 目的：计算每个用户的确认率（确认次数/总次数）
-- LEFT JOIN：保留所有注册用户，即使无确认记录
-- SUM和COUNT：计算确认次数占比，ROUND保留2位小数
SELECT s.user_id, ROUND(SUM(IF(action = 'confirmed', 1, 0)) / COUNT(*), 2) AS confirmation_rate
FROM Signups s
LEFT JOIN Confirmations c ON s.user_id = c.user_id
GROUP BY s.user_id;
```

15. **620. 有趣的电影**

```sql
-- 目的：找出ID为奇数且描述不是“boring”的电影
-- WHERE条件：筛选非boring且ID为奇数
-- ORDER BY：按评分降序排列
SELECT *
FROM cinema
WHERE description != 'boring' AND id % 2 != 0
ORDER BY rating DESC;
```

16. **1251. 平均售价**

```sql
-- 目的：计算每个产品的平均售价，未售出记为0
-- LEFT JOIN：匹配价格和销售单位，日期需在范围内
-- IFNULL：未售出时返回0
-- ROUND：保留2位小数
SELECT p.product_id, IFNULL(ROUND((SUM(price * units) / SUM(units)), 2), 0) AS average_price
FROM Prices p 
LEFT JOIN UnitsSold u ON p.product_id = u.product_id
AND u.purchase_date BETWEEN p.start_date AND p.end_date
GROUP BY p.product_id;
```

17. **1075. 项目员工 I**

```sql
-- 目的：计算每个项目的员工平均经验年限
-- LEFT JOIN：匹配项目和员工信息
-- AVG和ROUND：计算平均值并保留2位小数
SELECT project_id, ROUND(AVG(experience_years), 2) AS average_years
FROM Project p 
LEFT JOIN Employee e ON p.employee_id = e.employee_id
GROUP BY project_id;
```

18. **1633. 各赛事的用户注册率**

```sql
-- 目的：计算每个赛事的注册用户百分比
-- LEFT JOIN：匹配注册和用户信息
-- 子查询：计算总用户数作为分母
-- ROUND：保留2位小数，乘100转为百分比
SELECT contest_id, ROUND(COUNT(contest_id) / (SELECT COUNT(*) FROM Users) * 100, 2) AS percentage
FROM Register r 
LEFT JOIN Users u ON r.user_id = u.user_id
GROUP BY contest_id
ORDER BY percentage DESC, contest_id;
```

19. **1211. 查询结果的质量和占比**

```sql
-- 目的：计算查询的质量（评分/位置）和低质量占比（评分<3）
-- AVG：计算平均质量
-- SUM和CASE：计算低质量查询的百分比
-- HAVING：排除空查询名称
SELECT query_name, 
       ROUND(AVG(rating / position), 2) AS quality,
       ROUND((100 * SUM(CASE WHEN rating < 3 THEN 1 ELSE 0 END) / COUNT(*)), 2) AS poor_query_percentage
FROM Queries
GROUP BY query_name
HAVING query_name IS NOT NULL;
```

20. **1193. 每月交易 I**

```sql
-- 目的：统计每月的交易总数、批准数和金额
-- LEFT(trans_date, 7)：提取年月
-- CASE WHEN：分别统计批准的交易和金额
-- GROUP BY：按月和国家分组
SELECT LEFT(trans_date, 7) AS month,
       country, 
       COUNT(*) AS trans_count,
       SUM(CASE WHEN state = 'approved' THEN 1 ELSE 0 END) AS approved_count,
       SUM(amount) AS trans_total_amount,
       SUM((CASE WHEN state = 'approved' THEN 1 ELSE 0 END) * amount) AS approved_total_amount
FROM Transactions
GROUP BY month, country;
```

21. **1174. 即时食物配送 II**

```sql
-- 目的：计算首次订单中即时配送的百分比
-- 子查询：找出每个顾客的首次订单日期和偏好配送日期
-- CASE WHEN：统计即时配送订单数
-- ROUND：计算百分比，保留2位小数
SELECT ROUND((SUM(CASE WHEN customer_pref_delivery_date = order_date THEN 1 ELSE 0 END) * 100 / COUNT(*)), 2) AS immediate_percentage
FROM 
    (SELECT customer_id, MIN(order_date) AS order_date, MIN(customer_pref_delivery_date) AS customer_pref_delivery_date
     FROM Delivery
     GROUP BY customer_id) AS first_order;
```

22. **550. 游戏玩法分析 IV**

```sql
-- 目的：计算连续两天登录的玩家比例
-- 子查询a1：找出每个玩家的首次登录日期
-- JOIN：匹配次日登录记录
-- COUNT和子查询：计算比例，保留2位小数
SELECT ROUND(COUNT(*) / (SELECT COUNT(DISTINCT player_id) FROM Activity), 2) AS fraction
FROM 
    ((SELECT player_id, MIN(event_date) AS event_date
      FROM Activity 
      GROUP BY player_id) AS a1
JOIN Activity a2
ON a1.player_id = a2.player_id AND a1.event_date = a2.event_date - INTERVAL 1 DAY);
```

23. **2356. 每位教师所教授的科目种类的数量**

```sql
-- 目的：统计每位教师教授的独特科目数
-- COUNT(DISTINCT)：去重计算科目数量
-- GROUP BY：按教师ID分组
SELECT teacher_id, COUNT(DISTINCT subject_id) AS cnt
FROM Teacher
GROUP BY teacher_id;
```

24. **1141. 查询近30天活跃用户数**

```sql
-- 目的：统计近30天每天的活跃用户数
-- COUNT(DISTINCT)：去重计算用户数
-- HAVING：限定日期范围为2019-07-27前30天
SELECT activity_date AS day, COUNT(DISTINCT user_id) AS active_users
FROM Activity
GROUP BY activity_date
HAVING activity_date BETWEEN ("2019-07-27" - INTERVAL 29 DAY) AND "2019-07-27";
```

25. **1084. 销售分析 III**

```sql
-- 目的：找出只在2019年Q1销售的产品
-- LEFT JOIN：匹配销售和产品信息
-- HAVING：确保所有销售都在Q1（SUM=COUNT）
SELECT s.product_id, product_name
FROM Sales s 
LEFT JOIN Product p ON s.product_id = p.product_id
GROUP BY s.product_id
HAVING SUM(s.sale_date BETWEEN "2019-01-01" AND "2019-03-31") = COUNT(*);
```

26. **596. 超过5名学生的课**

```sql
-- 目的：找出学生数>=5的班级
-- GROUP BY：按班级分组
-- HAVING：筛选学生数>=5的班级
SELECT class
FROM Courses
GROUP BY class
HAVING COUNT(*) >= 5;
```

27. **1729. 求关注者的数量**

```sql
-- 目的：统计每个用户的关注者数量
-- COUNT：计算关注者数
-- ORDER BY：按用户ID升序排列
SELECT user_id, COUNT(*) AS followers_count
FROM Followers
GROUP BY user_id
ORDER BY user_id;
```

28. **619. 只出现一次的最大数字**

```sql
-- 目的：找出只出现一次的最大数字
-- 子查询：筛选只出现一次的数字
-- MAX：取最大值
SELECT MAX(num) AS num
FROM 
    (SELECT num
     FROM MyNumbers
     GROUP BY num
     HAVING COUNT(*) = 1) num1;
```

29. **1045. 买下所有产品的客户**

```sql
-- 目的：找出购买了所有产品的客户
-- COUNT(DISTINCT)：统计购买的不同产品数
-- 子查询：获取产品总数
-- HAVING：确保购买产品数等于总数
SELECT customer_id
FROM Customer
GROUP BY customer_id
HAVING COUNT(DISTINCT product_key) = (SELECT COUNT(*) FROM Product);
```

30. **1731. 每位经理的下属员工数量**

```sql
-- 目的：统计每个经理的下属数量和平均年龄
-- LEFT JOIN：通过reports_to关联经理和员工
-- COUNT和AVG：计算下属数和平均年龄
-- HAVING：排除空经理ID
SELECT e2.employee_id, e2.name, COUNT(*) AS reports_count, ROUND(AVG(e1.age), 0) AS average_age
FROM Employees e1 
LEFT JOIN Employees e2 ON e1.reports_to = e2.employee_id
GROUP BY e2.employee_id
HAVING e2.employee_id IS NOT NULL
ORDER BY employee_id;
```

31. **1789. 员工的直属部门**

```sql
-- 目的：返回员工的直属部门（主部门或唯一部门）
-- UNION：合并主部门和仅有一个部门的员工
-- 第一部分：筛选primary_flag='Y'的记录
-- 第二部分：筛选只有一个部门的员工
(SELECT employee_id, department_id
 FROM Employee
 WHERE primary_flag = 'Y')
UNION
(SELECT employee_id, department_id
 FROM Employee
 GROUP BY employee_id
 HAVING COUNT(*) = 1)
ORDER BY employee_id;
```

32. **610. 判断三角形**

```sql
-- 目的：判断三边是否能构成三角形
-- CASE WHEN：检查三角形条件（两边之和大于第三边）
-- 返回：Yes或No
SELECT *,
       (CASE WHEN (x + y > z AND x + z > y AND z + y > x) THEN "Yes" ELSE "No" END) AS triangle
FROM Triangle;
```

33. **180. 连续出现的数字**

```sql
-- 目的：找出连续出现至少三次的数字
-- JOIN：自连接三次，检查ID连续递增且数字相同
-- DISTINCT：去重结果
SELECT DISTINCT L1.num AS ConsecutiveNums
FROM Logs L1
JOIN Logs L2 ON L1.id = L2.id - 1
JOIN Logs L3 ON L2.id = L3.id - 1
WHERE L1.num = L2.num AND L2.num = L3.num;
```

34. **1164. 指定日期的产品价格**

```sql
-- 目的：获取截至2019-08-16的最新产品价格，未更新用10
-- 第一部分：子查询找出每个产品在指定日期前的最新价格
-- 第二部分：未更新价格的产品返回默认值10
SELECT product_id, new_price AS price
FROM Products
WHERE (product_id, change_date) IN
    (SELECT product_id, MAX(change_date)
     FROM Products
     WHERE change_date <= "2019-08-16"
     GROUP BY product_id)
UNION
SELECT product_id, 10 AS price
FROM Products
WHERE product_id NOT IN (SELECT product_id FROM Products WHERE change_date <= "2019-08-16");
```

35. **1204. 最后一个能进入巴士的人 ****⭐**

```sql
-- 目的：找出最后一个能上车的乘客（总重量<=1000）
-- JOIN：计算每个乘客之前的所有重量总和
-- HAVING：筛选重量和<=1000的记录
-- ORDER BY和LIMIT：取最后一个
SELECT q1.person_name
FROM Queue q1
JOIN Queue q2 ON q1.turn >= q2.turn
GROUP BY q1.person_id
HAVING SUM(q2.weight) <= 1000
ORDER BY q1.turn DESC LIMIT 1;
```

36. **1907. 按分类统计薪水**

```sql
-- 目的：按薪资分类（低、中、高）统计账户数
-- UNION：合并三种分类的结果
-- WHERE：分别筛选不同薪资范围
SELECT "Low Salary" AS category, COUNT(*) AS accounts_count
FROM Accounts
WHERE income < 20000
UNION
SELECT "Average Salary" AS category, COUNT(*) AS accounts_count
FROM Accounts
WHERE income BETWEEN 20000 AND 50000
UNION
SELECT "High Salary" AS category, COUNT(*) AS accounts_count
FROM Accounts
WHERE income > 50000;
```

37. **1978. 上级经理已离职的公司员工**

```sql
-- 目的：找出薪资<30000且经理已离职的员工
-- 子查询：排除仍在职的经理ID
-- ORDER BY：按员工ID升序排列
SELECT employee_id
FROM Employees
WHERE salary < 30000 AND manager_id NOT IN (SELECT employee_id FROM Employees)
ORDER BY employee_id;
```

38. **626. 换座位**

```sql
-- 目的：交换相邻座位的学生，奇数最后一个不变
-- CASE WHEN：奇数ID+1，偶数ID-1，最后一个奇数ID不变
-- ORDER BY：按ID升序排列
SELECT
    (CASE WHEN id % 2 != 0 AND id != (SELECT COUNT(*) FROM Seat) THEN id + 1
          WHEN id % 2 = 0 THEN id - 1
          ELSE id END) AS id, student
FROM Seat
ORDER BY id;
```

39. **1341. 电影评分**

```sql
-- 目的：找出评分次数最多的用户和2020年2月评分最高的电影
-- 第一部分：统计用户评分次数，取最多者
-- 第二部分：计算2月电影平均评分，取最高者
-- UNION ALL：合并两部分结果
(SELECT name AS results
 FROM Users u
 JOIN MovieRating r1 ON u.user_id = r1.user_id 
 GROUP BY u.user_id
 ORDER BY COUNT(*) DESC, name LIMIT 1)
UNION ALL
(SELECT title AS results
 FROM Movies m
 JOIN MovieRating r2 ON m.movie_id = r2.movie_id AND LEFT(r2.created_at, 7) = "2020-02"
 GROUP BY r2.movie_id 
 ORDER BY AVG(rating) DESC, title LIMIT 1);
```

40. **1321. 餐馆营业额变化增长 ****⭐⭐**

```sql
-- 目的：计算每天及前7天的总营业额和平均额
-- 子查询a：获取所有唯一日期
-- LEFT JOIN：匹配前7天的营业额数据
-- WHERE：从第7天开始统计
-- GROUP BY：按日期分组
SELECT a.visited_on, SUM(c.amount) AS amount, ROUND((SUM(c.amount)) / 7, 2) AS average_amount
FROM 
    (SELECT DISTINCT visited_on FROM Customer) AS a
LEFT JOIN Customer c
ON (c.visited_on >= a.visited_on - INTERVAL 6 DAY) AND (c.visited_on <= a.visited_on)
WHERE a.visited_on >= (SELECT MIN(visited_on) FROM Customer) + 6
GROUP BY a.visited_on
ORDER BY a.visited_on;
```

41. **602. 好友申请 II：谁有最多的好友**

```sql
-- 目的：找出好友最多的用户
-- 子查询：合并请求者和接受者的ID
-- UNION ALL：保留所有记录
-- ORDER BY和LIMIT：取好友数最多的用户
SELECT a.id, COUNT(*) AS num
FROM 
    (SELECT requester_id AS id FROM RequestAccepted r1
     UNION ALL
     SELECT accepter_id AS id FROM RequestAccepted r2) AS a
GROUP BY id
ORDER BY num DESC LIMIT 1;
```

42. **585. 2016年的投资**

```sql
-- 目的：计算满足条件的2016年投资总额
-- 条件1：tiv_2015在多条记录中重复
-- 条件2：lat和lon组合唯一
-- ROUND：保留2位小数
SELECT ROUND(SUM(tiv_2016), 2) AS tiv_2016
FROM Insurance
WHERE tiv_2015 IN
    (SELECT tiv_2015 FROM Insurance 
     GROUP BY tiv_2015 HAVING COUNT(*) > 1)
AND CONCAT(lat, lon) IN 
    (SELECT CONCAT(lat, lon) 
     FROM Insurance 
     GROUP BY CONCAT(lat, lon) 
     HAVING COUNT(*) = 1);
```

43. **185. 部门工资前三高的所有员工 ****⭐⭐⭐**

```sql
-- 目的：找出每个部门工资前三的员工
-- 子查询：比较同一部门内工资高于当前员工的记录数
-- HAVING：筛选工资排名<=2（即前三）
-- LEFT JOIN：关联部门信息
SELECT d.name AS Department, e.name AS Employee, e.salary
FROM Employee e 
LEFT JOIN Department d ON e.departmentId = d.id
WHERE e.id IN
    (SELECT e1.id
     FROM Employee e1 
     LEFT JOIN Employee e2 ON e1.departmentId = e2.departmentId AND e1.salary < e2.salary
     GROUP BY e1.id
     HAVING COUNT(DISTINCT e2.salary) <= 2);
```

44. **1667. 修复表中的名字**

```sql
-- 目的：将姓名首字母大写，其余小写
-- CONCAT：拼接首字母大写和其余小写部分
-- UPPER和LOWER：转换大小写
SELECT user_id, CONCAT(UPPER(LEFT(name, 1)), LOWER(SUBSTRING(name, 2))) AS name
FROM Users
ORDER BY user_id;
```

45. **1527. 患某种疾病的患者**

```sql
-- 目的：找出患有DIAB1疾病的患者
-- LIKE：匹配条件以DIAB1开头或包含" DIAB1"
SELECT *
FROM Patients
WHERE conditions LIKE "DIAB1%" OR conditions LIKE "% DIAB1%";
```

46. **196. 删除重复的电子邮箱**

```sql
-- 目的：删除重复邮箱中ID较大的记录
-- 子查询：保留每个邮箱的最小ID
-- DELETE：删除不在子查询中的记录
DELETE FROM Person
WHERE id NOT IN
    (SELECT id FROM (SELECT MIN(id) AS id
     FROM Person
     GROUP BY email) AS a);
```

47. **176. 第二高的薪水**

```sql
-- 目的：找出第二高的薪水，无则返回NULL
-- DISTINCT：去重薪水
-- LIMIT 1 OFFSET 1：取第二高
-- IFNULL：无第二高时返回NULL
SELECT IFNULL(
    (SELECT DISTINCT salary
     FROM Employee
     ORDER BY salary DESC
     LIMIT 1 OFFSET 1), NULL) AS SecondHighestSalary;
```

48. **1484. 按日期分组销售产品**

```sql
-- 目的：按日期统计销售产品数和产品列表
-- COUNT(DISTINCT)：统计不同产品数
-- GROUP_CONCAT：合并产品名，用逗号分隔
SELECT sell_date, COUNT(DISTINCT product) AS num_sold,
       GROUP_CONCAT(DISTINCT product ORDER BY product SEPARATOR ',') AS products
FROM Activities
GROUP BY sell_date
ORDER BY sell_date;
```

49. **1327. 列出指定时间段内所有的下单产品**

```sql
-- 目的：统计2020年2月订单量>=100的产品
-- JOIN：匹配产品和订单信息
-- LEFT：提取年月
-- HAVING：筛选订单量>=100
SELECT product_name, SUM(unit) AS unit
FROM Products p 
JOIN Orders o ON p.product_id = o.product_id AND LEFT(o.order_date, 7) = "2020-02"
GROUP BY product_name
HAVING SUM(unit) >= 100;
```

50. **1517. 查找拥有有效邮箱的用户**

```sql
-- 目的：找出符合邮箱格式的用户
-- REGEXP：匹配以字母开头，后接字母数字或特定字符，最后为@leetcode.com
SELECT user_id, name, mail
FROM Users
WHERE mail REGEXP '^[a-zA-Z][a-zA-Z0-9_.-]*\\@leetcode\\.com$';
```

---

### 二、高频SQL50题（进阶版）
#### 题目分类
1. **查询**：1-5
2. **连接**：6-11
3. **聚合函数**：12-19
4. **排序和分组**：20-26
5. **高级查询和连接**：27-35
6. **子查询**：36-43
7. **高级字符串函数/正则表达式/子句**：44-50

#### 题目及带注释的答案
1. **1821. 寻找今年具有正收入的客户**

```sql
-- 目的：找出2021年收入大于0的客户ID
-- WHERE：筛选年份为2021且收入>0
SELECT customer_id
FROM Customers
WHERE year = 2021 AND revenue > 0;
```

2. **183. 从不订购的客户**

```sql
-- 目的：找出没有订单的客户姓名
-- NOT IN：排除有订单的客户ID
SELECT name AS Customers
FROM Customers
WHERE id NOT IN (SELECT customerId FROM Orders);
```

3. **1873. 计算特殊奖金**

```sql
-- 目的：为员工ID为奇数且姓名不以M开头的员工发放奖金
-- CASE WHEN：满足条件时返回原薪资，否则返回0
-- ORDER BY：按员工ID排序
SELECT employee_id, 
       salary * (CASE WHEN employee_id % 2 != 0 AND LEFT(name, 1) != "M" THEN 1 ELSE 0 END) AS bonus
FROM Employees
ORDER BY employee_id;
```

4. **1398. 购买了产品 A 和产品 B 却没有购买产品 C 的顾客**
    - **方法1**

```sql
-- 目的：找出买了A和B但没买C的顾客
-- 子查询1：筛选买了A或B且产品数=2的顾客
-- 子查询2：排除买了C的顾客
SELECT customer_id, customer_name
FROM Customers
WHERE customer_id IN
    (SELECT customer_id
     FROM Orders
     WHERE product_name IN ("A", "B")
     GROUP BY customer_id
     HAVING COUNT(DISTINCT product_name) = 2)
AND customer_id NOT IN
    (SELECT customer_id
     FROM Orders
     WHERE product_name = "C"
     GROUP BY customer_id);
```

    - **方法2**

```sql
-- 目的：同上，使用SUM优化
-- SUM：A和B的乘积>0表示都买了，C=0表示没买
SELECT customer_id, customer_name
FROM Customers
WHERE customer_id IN
    (SELECT customer_id
     FROM Orders
     GROUP BY customer_id
     HAVING SUM(product_name = "A") * SUM(product_name = "B") > 0
     AND SUM(product_name = "C") = 0);
```

5. **1112. 每位学生的最高成绩**

```sql
-- 目的：找出每个学生的最高成绩及其课程ID
-- 子查询：获取每个学生的最高成绩
-- MIN(course_id)：取成绩相同时的最小课程ID
SELECT student_id, MIN(course_id) AS course_id, grade
FROM Enrollments
WHERE (student_id, grade) IN
    (SELECT student_id, MAX(grade) AS grade
     FROM Enrollments 
     GROUP BY student_id)
GROUP BY student_id
ORDER BY student_id, course_id;
```

6. **175. 组合两个表**

```sql
-- 目的：合并Person和Address表，保留所有人员信息
-- LEFT JOIN：即使没有地址信息也保留人员记录
SELECT firstName, lastName, city, state
FROM Person p 
LEFT JOIN Address a ON p.PersonId = a.personId;
```

7. **1607. 没有卖出的卖家**

```sql
-- 目的：找出2020年没有销售记录的卖家
-- NOT IN：排除有2020年订单的卖家
-- LEFT：提取年份
SELECT seller_name
FROM Seller s
WHERE seller_id NOT IN 
    (SELECT seller_id
     FROM Orders 
     WHERE LEFT(sale_date, 4) = "2020")
ORDER BY seller_name;
```

8. **1407. 排名靠前的旅行者**

```sql
-- 目的：统计每个用户的总旅行距离
-- LEFT JOIN：保留所有用户，即使没有旅行记录
-- IFNULL：无记录时返回0
-- ORDER BY：按距离降序和姓名升序排列
SELECT name, IFNULL(SUM(r.distance), 0) AS travelled_distance
FROM Users u
LEFT JOIN Rides r ON u.id = r.user_id
GROUP BY u.id
ORDER BY travelled_distance DESC, name;
```

9. **607. 销售员**

```sql
-- 目的：找出未与“RED”公司合作的销售员
-- NOT IN：排除与RED公司有订单的销售员
-- JOIN：匹配订单和公司信息
SELECT name
FROM SalesPerson
WHERE sales_id NOT IN
    (SELECT sales_id
     FROM Orders o 
     JOIN Company c ON o.com_id = c.com_id AND c.name = "RED");
```

10. **1440. 计算布尔表达式的值**

```sql
-- 目的：根据操作符计算表达式的布尔值
-- LEFT JOIN：匹配左右操作数的值
-- CASE WHEN：根据操作符和值比较返回true/false
SELECT e.*,
       (CASE WHEN operator = ">" AND v1.value > v2.value THEN "true"
             WHEN operator = "<" AND v1.value < v2.value THEN "true"
             WHEN operator = "=" AND v1.value = v2.value THEN "true"
             ELSE "false" END) AS value
FROM Expressions e
LEFT JOIN Variables v1 ON v1.name = e.left_operand
LEFT JOIN Variables v2 ON v2.name = e.right_operand;
```

11. **1212. 查询球队积分**

```sql
-- 目的：计算每支球队的积分
-- CASE WHEN：胜3分，平1分，负0分
-- RIGHT JOIN：保留所有球队，即使无比赛
-- ORDER BY：按积分降序和球队ID升序
SELECT t.team_id, t.team_name,
       SUM(CASE WHEN m.host_team = t.team_id AND host_goals > guest_goals THEN 3
                WHEN m.guest_team = t.team_id AND host_goals < guest_goals THEN 3
                WHEN host_goals = guest_goals THEN 1
                ELSE 0 END) AS num_points
FROM Matches m
RIGHT JOIN Teams t ON m.host_team = t.team_id OR m.guest_team = t.team_id
GROUP BY t.team_id
ORDER BY num_points DESC, team_id;
```

12. **1890. 2020年最后一次登录**

```sql
-- 目的：找出2020年每个用户的最后登录时间
-- LEFT：提取年份
-- MAX：取最后登录时间
SELECT user_id, MAX(time_stamp) AS last_stamp
FROM Logins
WHERE LEFT(time_stamp, 4) = '2020'
GROUP BY user_id;
```

13. **511. 游戏玩法分析 I**

```sql
-- 目的：找出每个玩家的首次登录日期
-- MIN：取最早的事件日期
SELECT player_id, MIN(event_date) AS first_login
FROM Activity
GROUP BY player_id;
```

14. **1571. 仓库经理**

```sql
-- 目的：计算每个仓库的总体积
-- LEFT JOIN：匹配仓库和产品信息
-- SUM：计算体积（单位数*长*宽*高）
SELECT name AS warehouse_name, SUM(units * Width * Length * Height) AS volume
FROM Warehouse w 
LEFT JOIN Products p ON w.product_id = p.product_id
GROUP BY w.name;
```

15. **586. 订单最多的客户**

```sql
-- 目的：找出下单最多的客户
-- GROUP BY：按客户分组
-- ORDER BY和LIMIT：取订单数最多的客户
SELECT customer_number
FROM Orders
GROUP BY customer_number
ORDER BY COUNT(customer_number) DESC LIMIT 1;
```

16. **1741. 查找每个员工花费的总时间**

```sql
-- 目的：统计每天每个员工的总工作时间
-- SUM：计算出勤时间差
-- GROUP BY：按日期和员工ID分组
SELECT event_day AS day, emp_id, SUM(out_time - in_time) AS total_time
FROM Employees
GROUP BY event_day, emp_id;
```

17. **1173. 即时食物配送 I**

```sql
-- 目的：计算即时配送订单的百分比
-- CASE WHEN：统计即时配送订单数
-- ROUND：计算百分比，保留2位小数
SELECT ROUND(100 * SUM(CASE WHEN order_date = customer_pref_delivery_date THEN 1 ELSE 0 END) / COUNT(delivery_id), 2) AS immediate_percentage
FROM Delivery;
```

18. **1445. 苹果和桔子**

```sql
-- 目的：计算每天苹果和橘子的销售差额
-- CASE WHEN：苹果为正，橘子为负
-- SUM：计算净差额
SELECT sale_date, SUM(CASE WHEN fruit = "apples" THEN sold_num ELSE -sold_num END) AS diff 
FROM Sales
GROUP BY sale_date;
```

19. **1699. 两人之间的通话次数**

```sql
-- 目的：统计两人间的通话次数和总时长
-- CASE WHEN：确保person1<=>person2顺序一致
-- COUNT和SUM：计算通话次数和时长
SELECT 
    (CASE WHEN from_id < to_id THEN from_id ELSE to_id END) AS person1,
    (CASE WHEN from_id < to_id THEN to_id ELSE from_id END) AS person2,
    COUNT(*) AS call_count,
    SUM(duration) AS total_duration
FROM Calls
GROUP BY person1, person2;
```

20. **1587. 银行账户概要 II**

```sql
-- 目的：统计余额>10000的账户信息
-- LEFT JOIN：匹配交易和用户信息
-- HAVING：筛选余额大于10000的账户
SELECT u.name AS NAME, SUM(t.amount) AS BALANCE
FROM Transactions t 
LEFT JOIN Users u ON t.account = u.account
GROUP BY t.account
HAVING SUM(amount) > 10000;
```

21. **182. 查找重复的电子邮箱**

```sql
-- 目的：找出重复的邮箱地址
-- GROUP BY：按邮箱分组
-- HAVING：筛选出现次数>=2的邮箱
SELECT email AS Email
FROM Person
GROUP BY email
HAVING COUNT(*) >= 2;
```

22. **1050. 合作过至少三次的演员和导演**

```sql
-- 目的：找出合作>=3次的演员和导演组合
-- GROUP BY：按演员和导演分组
-- HAVING：筛选合作次数>=3
SELECT actor_id, director_id
FROM ActorDirector
GROUP BY actor_id, director_id
HAVING COUNT(*) >= 3;
```

23. **1511. 消费者下单频率**

```sql
-- 目的：找出2020年6月和7月订单金额均>=100的顾客
-- JOIN：关联订单、产品和客户表
-- CASE WHEN：分别统计6月和7月金额
-- HAVING：筛选满足条件的顾客
SELECT o.customer_id, c.name
FROM Orders o 
JOIN Product p ON o.product_id = p.product_id
JOIN Customers c ON o.customer_id = c.customer_id
GROUP BY customer_id
HAVING SUM(CASE WHEN LEFT(o.order_date, 7) = "2020-06" THEN quantity * price ELSE 0 END) >= 100
AND SUM(CASE WHEN LEFT(o.order_date, 7) = "2020-07" THEN quantity * price ELSE 0 END) >= 100;
```

24. **1693. 每天的领导和合伙人**

```sql
-- 目的：统计每天的独特领导和合伙人数量
-- COUNT(DISTINCT)：去重计算领导和合伙人
-- GROUP BY：按日期和品牌分组
SELECT date_id, make_name, COUNT(DISTINCT lead_id) AS unique_leads, COUNT(DISTINCT partner_id) AS unique_partners
FROM DailySales
GROUP BY date_id, make_name;
```

25. **1495. 上月播放的儿童适宜电影**

```sql
-- 目的：找出2020年6月播放的儿童电影
-- LEFT JOIN：匹配内容和节目信息
-- WHERE：筛选儿童内容且为电影，限定6月
-- DISTINCT：去重电影标题
SELECT DISTINCT title
FROM Content c 
LEFT JOIN TVProgram t ON c.content_id = t.content_id 
WHERE c.Kids_content = 'Y' AND LEFT(t.program_date, 7) = "2020-06" AND content_type = "Movies";
```

26. **1501. 可以放心投资的国家**

```sql
-- 目的：找出通话平均时长高于全局平均的国家
-- JOIN：关联人员、国家和通话信息
-- HAVING：筛选平均时长大于全局平均值
SELECT co.name AS country
FROM Person p 
JOIN Country co ON LEFT(p.phone_number, 3) = co.country_code
JOIN Calls ca ON p.id = ca.caller_id OR p.id = ca.callee_id
GROUP BY co.country_code
HAVING AVG(duration) > (SELECT AVG(duration) FROM Calls);
```

27. **603. 连续空余座位**

```sql
-- 目的：找出连续的空余座位
-- JOIN：自连接，检查相邻座位
-- WHERE：确保两个座位都空闲
-- DISTINCT：去重座位ID
SELECT DISTINCT c1.seat_id
FROM Cinema c1 
JOIN Cinema c2 ON ABS(c1.seat_id - c2.seat_id) = 1
WHERE c1.free = 1 AND c2.free = 1
ORDER BY c1.seat_id;
```

28. **1795. 每个产品在不同商店的价格**

```sql
-- 目的：列出产品在每个商店的价格
-- UNION ALL：合并三个商店的价格记录
-- WHERE：排除空价格记录
SELECT product_id, "store1" AS store, store1 AS price
FROM Products
WHERE store1 IS NOT NULL
UNION ALL
SELECT product_id, "store2" AS store, store2 AS price
FROM Products
WHERE store2 IS NOT NULL
UNION ALL
SELECT product_id, "store3" AS store, store3 AS price
FROM Products
WHERE store3 IS NOT NULL;
```

29. **613. 直线上的最近距离**

```sql
-- 目的：计算直线上两点的最短距离
-- JOIN：自连接，排除相同点
-- MIN：取最小距离
SELECT MIN(ABS(p1.x - p2.x)) AS shortest
FROM Point p1 
JOIN Point p2 ON p1.x != p2.x;
```

30. **1965. 丢失信息的雇员**

```sql
-- 目的：找出员工表或薪资表中缺失信息的员工ID
-- UNION：合并两表中缺失的记录
-- NOT IN：检查另一表中不存在的ID
SELECT employee_id
FROM Employees
WHERE employee_id NOT IN (SELECT employee_id FROM Salaries)
UNION
SELECT employee_id
FROM Salaries
WHERE employee_id NOT IN (SELECT employee_id FROM Employees)
ORDER BY employee_id;
```

31. **1264. 页面推荐**

```sql
-- 目的：为用户1推荐朋友喜欢的页面（用户1未点赞）
-- 子查询：获取用户1的朋友ID
-- NOT IN：排除用户1已点赞的页面
-- DISTINCT：去重推荐页面
SELECT DISTINCT page_id AS recommended_page
FROM Likes
WHERE user_id IN (
    SELECT user2_id AS user_id
    FROM Friendship
    WHERE user1_id = "1"
    UNION
    SELECT user1_id AS user_id
    FROM Friendship
    WHERE user2_id = "1")
AND page_id NOT IN (SELECT page_id FROM Likes WHERE user_id = 1);
```

32. **608. 树节点**

```sql
-- 目的：判断树节点的类型（根、内、叶）
-- CASE WHEN：根节点无父节点，叶节点无子节点，其余为内节点
SELECT id, 
       (CASE WHEN p_id IS NULL THEN "Root"
             WHEN id NOT IN (SELECT p_id FROM Tree WHERE p_id IS NOT NULL) THEN "Leaf"
             ELSE "Inner" END) AS type
FROM Tree;
```

33. **534. 游戏玩法分析 III**

```sql
-- 目的：计算每个玩家每天的累计游戏次数
-- LEFT JOIN：匹配同一玩家的所有日期
-- SUM：累计截至当前日期的游戏次数
SELECT a2.player_id, a2.event_date, SUM(a1.games_played) AS games_played_so_far
FROM Activity a1 
LEFT JOIN Activity a2 ON a1.player_id = a2.player_id AND a2.event_date >= a1.event_date
GROUP BY a2.event_date, a2.player_id;
```

34. **1783. 大满贯数量**

```sql
-- 目的：统计每个玩家的四大满贯次数
-- SUM：计算每个赛事中玩家的胜利次数
-- HAVING：排除无满贯的玩家
SELECT p.player_id, p.player_name, 
       SUM(c.Wimbledon = p.player_id) + SUM(c.Fr_open = p.player_id) + 
       SUM(c.US_open = p.player_id) + SUM(c.Au_open = p.player_id) AS grand_slams_count
FROM Players p, Championships c
GROUP BY p.player_id
HAVING grand_slams_count > 0;
```

35. **1747. 应该被禁止的 Leetflex 账户**

```sql
-- 目的：找出登录时间重叠的账户
-- 子查询：按账户和登录时间编号
-- LEAD：获取下一次登录时间
-- WHERE：检查首次登录与下次登录时间重叠
SELECT account_id
FROM (
    SELECT account_id, login, logout, LEAD(login, 1) OVER() AS ll, nums
    FROM (
        SELECT *, ROW_NUMBER() OVER (PARTITION BY account_id ORDER BY login) AS nums
        FROM loginfo) aaa) bbb
WHERE nums = 1 AND ll BETWEEN login AND logout;
```

36. **1350. 院系无效的学生**

```sql
-- 目的：找出所属院系无效的学生
-- NOT IN：排除存在于Departments中的院系ID
SELECT id, name
FROM Students s 
WHERE s.department_id NOT IN (SELECT id FROM Departments);
```

37. **1303. 求团队人数**

```sql
-- 目的：统计每个员工所在团队的人数
-- LEFT JOIN：自连接，按团队ID匹配
-- COUNT：计算团队人数
SELECT e1.employee_id, COUNT(*) AS team_size
FROM Employee e1 
LEFT JOIN Employee e2 ON e1.team_id = e2.team_id
GROUP BY e1.employee_id;
```

38. **512. 游戏玩法分析 II**

```sql
-- 目的：找出每个玩家首次登录的设备ID
-- 子查询：获取首次登录日期
-- WHERE：匹配首次登录记录
SELECT player_id, device_id
FROM Activity
WHERE (player_id, event_date) IN (
    SELECT player_id, MIN(event_date)
    FROM Activity
    GROUP BY player_id);
```

39. **184. 部门工资最高的员工**

```sql
-- 目的：找出每个部门工资最高的员工
-- 子查询：获取每个部门的最大工资
-- LEFT JOIN：关联部门信息
SELECT d.name AS Department, e.name AS Employee, e.salary
FROM Employee e 
LEFT JOIN Department d ON e.departmentId = d.id 
WHERE (e.departmentId, e.salary) IN (
    SELECT departmentId, MAX(salary)
    FROM Employee  
    GROUP BY departmentId);
```

40. **1549. 每件商品的最新订单**

```sql
-- 目的：找出每件商品的最新订单信息
-- 子查询：获取每个商品的最新订单日期
-- LEFT JOIN：关联产品信息
-- ORDER BY：按产品名、ID和日期排序
SELECT p.product_name, o.product_id, o.order_id, o.order_date
FROM Orders o 
LEFT JOIN Products p ON o.product_id = p.product_id
WHERE (o.product_id, o.order_date) IN (
    SELECT product_id, MAX(order_date)
    FROM Orders
    GROUP BY product_id)
ORDER BY p.product_name, o.product_id, o.order_id;
```

41. **1532. 最近的三笔订单**

```sql
-- 目的：找出每个客户最近三笔订单
-- LEFT JOIN：自连接，比较订单日期
-- HAVING：限制每笔订单前的订单数<=3
-- ORDER BY：按客户名、ID和日期降序
SELECT c.name AS customer_name, c.customer_id, o2.order_id, o2.order_date
FROM Orders o1 
LEFT JOIN Orders o2 ON o1.customer_id = o2.customer_id AND o1.order_date >= o2.order_date 
LEFT JOIN Customers c ON o1.customer_id = c.customer_id
GROUP BY o2.order_id
HAVING COUNT(o1.order_date) <= 3
ORDER BY c.name, c.customer_id, o2.order_date DESC;
```

42. **1831. 每天的最大交易**

```sql
-- 目的：找出每天的最大交易ID
-- 子查询：获取每天的最大交易金额
-- WHERE：匹配最大金额的交易
-- ORDER BY：按交易ID排序
SELECT transaction_id
FROM Transactions
WHERE (day, amount) IN (
    SELECT day, MAX(amount)
    FROM Transactions
    GROUP BY day)
ORDER BY transaction_id;
```

43. **1077. 项目员工 III**

```sql
-- 目的：找出每个项目经验最丰富的员工
-- 子查询：获取每个项目的最大经验年限
-- LEFT JOIN：关联员工信息
SELECT p.project_id, p.employee_id
FROM Project p 
LEFT JOIN Employee e ON p.employee_id = e.employee_id
WHERE (p.project_id, e.experience_years) IN (
    SELECT p.project_id, MAX(experience_years)
    FROM Project p 
    LEFT JOIN Employee e ON p.employee_id = e.employee_id
    GROUP BY p.project_id);
```

44. **1285. 找到连续区间的开始和结束数字**

```sql
-- 目的：找出连续日志ID区间的起始和结束
-- 子查询：用ROW_NUMBER计算差值，相同diff表示连续
-- MIN和MAX：取每个连续区间的起始和结束ID
SELECT MIN(log_id) AS start_id, MAX(log_id) AS end_id
FROM (
    SELECT log_id, log_id - ROW_NUMBER() OVER() AS diff
    FROM logs) AS t
GROUP BY diff;
```

45. **1596. 每位顾客最经常订购的商品**

```sql
-- 目的：找出每个顾客最常订购的商品
-- 子查询：用RANK排序订购次数
-- JOIN：关联产品信息
-- WHERE：取排名第一的商品
SELECT o.customer_id, o.product_id, p.product_name
FROM (
    SELECT customer_id, product_id,
           RANK() OVER (PARTITION BY customer_id ORDER BY COUNT(product_id) DESC) AS rnk 
    FROM Orders
    GROUP BY customer_id, product_id) o
JOIN Products p ON o.product_id = p.product_id
WHERE rnk = 1;
```

46. **1709. 访问日期之间最大的空档期**

```sql
-- 目的：计算每个用户访问日期的最大间隔
-- 子查询：用LEAD获取下次访问日期，默认末尾为2021-1-1
-- DATEDIFF：计算日期差
-- MAX：取最大间隔
SELECT user_id, MAX(DATEDIFF(next_day, visit_date)) AS biggest_window
FROM (
    SELECT user_id, visit_date, 
           LEAD(visit_date, 1, '2021-1-1') OVER (PARTITION BY user_id ORDER BY visit_date) AS next_day
    FROM UserVisits) AS tmp
GROUP BY user_id
ORDER BY user_id;
```

47. **1270. 向公司 CEO 汇报工作的所有人**
    - **方法1**

```sql
-- 目的：找出向CEO（ID=1）直接或间接汇报的员工
-- UNION ALL：递归合并三层汇报关系
-- DISTINCT：去重员工ID
-- WHERE：排除CEO本人
SELECT DISTINCT employee_id
FROM (
    SELECT employee_id FROM Employees WHERE manager_id = 1
    UNION ALL
    SELECT employee_id FROM Employees
    WHERE manager_id IN (SELECT employee_id FROM Employees WHERE manager_id = 1)
    UNION ALL
    SELECT employee_id FROM Employees
    WHERE manager_id IN (
        SELECT employee_id FROM Employees
        WHERE manager_id IN (SELECT employee_id FROM Employees WHERE manager_id = 1))) e
WHERE employee_id != 1;
```

    - **方法2**

```sql
-- 目的：同上，使用JOIN实现三层汇报
-- JOIN：连接三层经理关系
-- WHERE：确保顶层经理为CEO且排除CEO本人
SELECT e1.employee_id
FROM Employees e1
JOIN Employees e2 ON e1.manager_id = e2.employee_id
JOIN Employees e3 ON e2.manager_id = e3.employee_id
WHERE e1.employee_id != 1 AND e3.manager_id = 1;
```

48. **1412. 查找成绩处于中游的学生**

```sql
-- 目的：找出考试成绩既非最高也非最低的学生
-- DENSE_RANK：计算每场考试的成绩排名
-- HAVING：排除最高（d_rank=1）和最低（a_rank=1）的学生
SELECT tmp.student_id, s.student_name
FROM (
    SELECT *,
           IF(DENSE_RANK() OVER (PARTITION BY exam_id ORDER BY score DESC) = 1, 1, 0) AS d_rank,
           IF(DENSE_RANK() OVER (PARTITION BY exam_id ORDER BY score) = 1, 1, 0) AS a_rank
    FROM Exam) tmp
LEFT JOIN Student s ON tmp.student_id = s.student_id
GROUP BY tmp.student_id
HAVING SUM(d_rank) = 0 AND SUM(a_rank) = 0
ORDER BY tmp.student_id;
```

49. **1767. 寻找没有被执行的任务对 ****⭐⭐⭐**

```sql
-- 目的：找出未执行的任务和子任务组合
-- WITH RECURSIVE：递归生成所有任务和子任务对
-- LEFT JOIN：匹配已执行记录，未匹配的即未执行
WITH RECURSIVE table1 AS (
    SELECT task_id, subtasks_count AS subtask_id 
    FROM Tasks
    UNION ALL
    SELECT task_id, subtask_id - 1 
    FROM table1 
    WHERE subtask_id > 1)
SELECT task_id, subtask_id
FROM table1
LEFT JOIN Executed E USING(task_id, subtask_id)
WHERE E.task_id IS NULL;
```

50. **1225. 报告系统状态的连续日期 ****⭐⭐⭐**

```sql
-- 目的：报告2019年内连续的系统状态区间
-- 子查询：合并失败和成功日期，计算日期差
-- ROW_NUMBER：生成连续分组依据
-- GROUP BY：按状态和差值分组取起止日期
SELECT type AS period_state, MIN(date) AS start_date, MAX(date) AS end_date
FROM (
    SELECT type, date, SUBDATE(date, ROW_NUMBER() OVER (PARTITION BY type ORDER BY date)) AS diff
    FROM (
        SELECT "failed" AS type, fail_date AS date FROM Failed
        UNION ALL
        SELECT "succeeded" AS type, success_date AS date FROM Succeeded) tmp1) tmp2
WHERE date BETWEEN "2019-01-01" AND "2019-12-31"
GROUP BY type, diff
ORDER BY start_date;
```

---

### 总结
本文整理了LeetCode高频SQL题目，共100题（基础版50题+进阶版50题），涵盖查询、连接、聚合函数、排序分组、高级查询、子查询、高级字符串函数等类型。每道题均提供带详细注释的SQL代码，解释查询逻辑，帮助理解。难度较高的题目标有⭐，适合SQL学习者和面试准备者使用。

---

希望这份带注释的整理符合您的需求！如果需要进一步调整，请随时告知。



> 更新: 2025-03-04 11:05:25  
> 原文: <https://www.yuque.com/neumx/ko4psh/gz9uyzsf8rw00gb9>