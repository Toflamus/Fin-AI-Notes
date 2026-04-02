# SQL 语言教程 (SQL Tutorial)

> SQL 的学习本质是从**命令式编程**（怎么做）向**声明式编程**（要什么）的思维转换。
> 你只需描述"想要什么数据"，数据库引擎负责决定"怎么取"。

---

## 一、基础语法体系 (The Core Syntax)

所有 SQL 操作围绕三大类语句展开：**查询 (DQL)**、**操作 (DML)**、**定义 (DDL)**。

### 1.1 DQL — 数据查询

查询是 SQL 中最核心、使用最频繁的部分。

#### 基础查询与过滤

```sql
-- 查询所有列
SELECT * FROM employees;

-- 查询指定列，使用别名
SELECT 
    first_name AS 姓名,
    salary AS 薪资
FROM employees;

-- 去重
SELECT DISTINCT department FROM employees;

-- 条件过滤
SELECT * FROM employees
WHERE department = 'Finance'
  AND salary > 50000;

-- 常用运算符
WHERE age BETWEEN 25 AND 35        -- 范围
WHERE name LIKE '张%'               -- 模糊匹配（% 任意字符，_ 单个字符）
WHERE department IN ('Finance', 'IT') -- 集合匹配
WHERE manager_id IS NULL            -- 判空（注意：不能用 = NULL）
```

#### 排序与分页

```sql
-- 排序：ASC 升序（默认），DESC 降序
SELECT * FROM employees
ORDER BY salary DESC, hire_date ASC;

-- 分页：跳过前 10 条，取 5 条
SELECT * FROM employees
ORDER BY id
LIMIT 5 OFFSET 10;
```

### 1.2 DML — 数据操作（增/改/删）

```sql
-- 插入
INSERT INTO employees (first_name, department, salary)
VALUES ('张三', 'Finance', 60000);

-- 批量插入
INSERT INTO employees (first_name, department, salary)
VALUES 
    ('李四', 'IT', 70000),
    ('王五', 'HR', 55000);

-- 更新（务必加 WHERE，否则更新全表！）
UPDATE employees
SET salary = salary * 1.1
WHERE department = 'Finance';

-- 删除（同理，务必加 WHERE）
DELETE FROM employees
WHERE hire_date < '2020-01-01';
```

### 1.3 DDL — 数据定义（建表/改表/删表）

```sql
-- 建表
CREATE TABLE employees (
    id        INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    department VARCHAR(50),
    salary     DECIMAL(10, 2),
    hire_date  DATE DEFAULT CURRENT_DATE
);

-- 加列
ALTER TABLE employees ADD COLUMN email VARCHAR(100);

-- 删列
ALTER TABLE employees DROP COLUMN email;

-- 删表（不可逆！）
DROP TABLE employees;

-- 清空数据但保留表结构
TRUNCATE TABLE employees;
```

---

## 二、关系代数与逻辑处理 (Relational Logic)

这是 SQL 的灵魂，决定了处理复杂业务逻辑的能力。

### 2.1 聚合运算

```sql
-- 五大聚合函数
SELECT 
    department,
    COUNT(*)       AS 人数,
    SUM(salary)    AS 总薪资,
    AVG(salary)    AS 平均薪资,
    MAX(salary)    AS 最高薪资,
    MIN(salary)    AS 最低薪资
FROM employees
GROUP BY department;
```

**`HAVING` vs `WHERE` 的区别：**

- `WHERE` 在分组前过滤**行**
- `HAVING` 在分组后过滤**组**

```sql
-- 找出平均薪资超过 60000 的部门
SELECT department, AVG(salary) AS avg_sal
FROM employees
WHERE salary > 0           -- 先过滤掉无效数据
GROUP BY department
HAVING AVG(salary) > 60000; -- 再过滤分组结果
```

### 2.2 多表联接 (Joins)

Joins 是 SQL 处理关系型数据的核心能力。

```
表 A          表 B
┌───┬───┐    ┌───┬───┐
│ 1 │ a │    │ 1 │ x │
│ 2 │ b │    │ 3 │ y │
│ 3 │ c │    │ 4 │ z │
└───┴───┘    └───┴───┘
```

| Join 类型 | 结果 | 说明 |
|-----------|------|------|
| `INNER JOIN` | 1, 3 | 只保留两表都匹配的行 |
| `LEFT JOIN` | 1, 2, 3 | 保留左表全部，右表无匹配则填 NULL |
| `RIGHT JOIN` | 1, 3, 4 | 保留右表全部，左表无匹配则填 NULL |
| `FULL OUTER JOIN` | 1, 2, 3, 4 | 保留两表全部，无匹配处填 NULL |
| `CROSS JOIN` | 3×3=9 行 | 笛卡尔积，每一行与另一表所有行组合 |

```sql
-- 典型用法：查员工及其部门名称
SELECT e.first_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;

-- LEFT JOIN：保留所有员工，即使没有分配部门
SELECT e.first_name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;

-- 自联接：查每个员工的直属经理
SELECT 
    e.first_name AS 员工,
    m.first_name AS 经理
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### 2.3 集合运算

```sql
-- 并集（自动去重）
SELECT city FROM customers
UNION
SELECT city FROM suppliers;

-- 并集（保留重复）
SELECT city FROM customers
UNION ALL
SELECT city FROM suppliers;

-- 交集
SELECT city FROM customers
INTERSECT
SELECT city FROM suppliers;

-- 差集
SELECT city FROM customers
EXCEPT
SELECT city FROM suppliers;
```

### 2.4 条件逻辑 — CASE WHEN

SQL 中的 if-else：

```sql
SELECT 
    first_name,
    salary,
    CASE
        WHEN salary >= 100000 THEN '高薪'
        WHEN salary >= 60000  THEN '中等'
        ELSE '待提升'
    END AS 薪资等级
FROM employees;

-- 搭配聚合：统计各部门的高薪人数
SELECT 
    department,
    COUNT(CASE WHEN salary >= 100000 THEN 1 END) AS 高薪人数,
    COUNT(*) AS 总人数
FROM employees
GROUP BY department;
```

---

## 三、高级分析特性 (Advanced Analytics)

### 3.1 子查询 (Subqueries)

#### 独立子查询

子查询独立执行，结果传给外部查询：

```sql
-- 找出薪资高于全公司平均的员工
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- IN：找出有订单的客户
SELECT * FROM customers
WHERE id IN (SELECT DISTINCT customer_id FROM orders);
```

#### 相关子查询

子查询依赖外部查询的当前行，逐行执行：

```sql
-- EXISTS：找出有订单的客户（通常比 IN 更高效）
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.id
);

-- 找出每个部门薪资最高的员工
SELECT * FROM employees e1
WHERE salary = (
    SELECT MAX(salary) FROM employees e2
    WHERE e2.department = e1.department
);
```

### 3.2 CTE — 公用表表达式

用 `WITH` 定义临时命名结果集，提升可读性：

```sql
-- 基本用法
WITH dept_stats AS (
    SELECT 
        department,
        AVG(salary) AS avg_salary,
        COUNT(*) AS headcount
    FROM employees
    GROUP BY department
)
SELECT * FROM dept_stats
WHERE avg_salary > 60000;

-- 多个 CTE 串联
WITH 
active_orders AS (
    SELECT * FROM orders WHERE status = 'active'
),
order_summary AS (
    SELECT 
        customer_id,
        SUM(amount) AS total_amount
    FROM active_orders
    GROUP BY customer_id
)
SELECT c.name, os.total_amount
FROM customers c
JOIN order_summary os ON c.id = os.customer_id;

-- 递归 CTE：生成组织架构树
WITH RECURSIVE org_tree AS (
    -- 锚点：顶层 CEO
    SELECT id, first_name, manager_id, 0 AS level
    FROM employees WHERE manager_id IS NULL
    
    UNION ALL
    
    -- 递归：逐层展开下属
    SELECT e.id, e.first_name, e.manager_id, t.level + 1
    FROM employees e
    JOIN org_tree t ON e.manager_id = t.id
)
SELECT * FROM org_tree ORDER BY level;
```

### 3.3 窗口函数 (Window Functions) ★

窗口函数是分析师的核心技能，它能在**不折叠行**的情况下做聚合/排序计算。

语法结构：`函数() OVER (PARTITION BY ... ORDER BY ... ROWS/RANGE ...)`

#### 排名函数

```sql
SELECT 
    first_name,
    department,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS 序号,
    RANK()       OVER (PARTITION BY department ORDER BY salary DESC) AS 排名,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS 密集排名
FROM employees;
```

三者区别（salary: 100, 100, 90）：

| 函数 | 结果 | 说明 |
|------|------|------|
| `ROW_NUMBER` | 1, 2, 3 | 严格递增，无并列 |
| `RANK` | 1, 1, 3 | 并列后跳号 |
| `DENSE_RANK` | 1, 1, 2 | 并列后不跳号 |

#### 窗口聚合

```sql
SELECT 
    first_name,
    department,
    salary,
    -- 部门内平均薪资（不折叠行）
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    -- 累计求和
    SUM(salary) OVER (ORDER BY hire_date ROWS UNBOUNDED PRECEDING) AS running_total
FROM employees;
```

#### 偏移函数 — LAG / LEAD

```sql
-- 计算环比增长率（对比上月）
SELECT 
    month,
    revenue,
    LAG(revenue, 1) OVER (ORDER BY month) AS 上月收入,
    (revenue - LAG(revenue, 1) OVER (ORDER BY month)) 
        / LAG(revenue, 1) OVER (ORDER BY month) * 100 AS 环比增长率
FROM monthly_revenue;

-- LEAD：取下一行的值
LEAD(revenue, 1) OVER (ORDER BY month) AS 下月收入
```

#### 窗口帧 (Frame) 详解

```sql
-- 移动平均（当前行及前 2 行）
AVG(salary) OVER (
    ORDER BY hire_date 
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
) AS moving_avg_3

-- 常用帧定义
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW  -- 从头到当前行
ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING           -- 前后各一行
ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING   -- 当前行到末尾
```

---

## 四、数据库对象与约束 (Schema & Constraints)

### 4.1 完整性约束

```sql
CREATE TABLE orders (
    id          INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    amount      DECIMAL(10,2) CHECK (amount > 0),
    email       VARCHAR(100) UNIQUE,
    
    -- 外键：关联到 customers 表
    FOREIGN KEY (customer_id) REFERENCES customers(id)
        ON DELETE CASCADE      -- 客户删除时，订单也删除
        ON UPDATE SET NULL     -- 客户 ID 更新时，订单中置 NULL
);
```

| 约束 | 作用 |
|------|------|
| `PRIMARY KEY` | 唯一标识每一行，不允许 NULL |
| `FOREIGN KEY` | 建立表间引用关系 |
| `UNIQUE` | 列值不重复（允许 NULL） |
| `NOT NULL` | 不允许空值 |
| `CHECK` | 自定义条件约束 |
| `DEFAULT` | 未赋值时的默认值 |

### 4.2 索引 (Indexing)

索引是影响查询性能的关键。核心数据结构是 **B-Tree**（平衡树）。

```sql
-- 创建索引
CREATE INDEX idx_emp_dept ON employees(department);

-- 复合索引（遵循最左前缀原则）
CREATE INDEX idx_emp_dept_sal ON employees(department, salary);

-- 唯一索引
CREATE UNIQUE INDEX idx_emp_email ON employees(email);

-- 查看索引
SHOW INDEX FROM employees;
```

**何时加索引：**
- `WHERE` 中频繁过滤的列
- `JOIN` 条件中的列
- `ORDER BY` / `GROUP BY` 的列

**索引代价：** 写入变慢（插入/更新需维护索引），占用额外存储。

### 4.3 视图 (Views)

视图是保存的查询，行为像虚拟表：

```sql
-- 创建视图
CREATE VIEW high_earners AS
SELECT first_name, department, salary
FROM employees
WHERE salary > 100000;

-- 使用视图（和普通表一样查询）
SELECT * FROM high_earners WHERE department = 'IT';

-- 删除视图
DROP VIEW high_earners;
```

### 4.4 事务控制 (TCL)

事务保证一组操作的 **ACID** 特性：

| 特性 | 含义 |
|------|------|
| **A**tomicity（原子性） | 全部成功或全部回滚 |
| **C**onsistency（一致性） | 事务前后数据一致 |
| **I**solation（隔离性） | 并发事务互不干扰 |
| **D**urability（持久性） | 提交后永久保存 |

```sql
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
UPDATE accounts SET balance = balance + 1000 WHERE id = 2;

-- 确认没问题
COMMIT;

-- 出错时回滚
-- ROLLBACK;
```

---

## 五、底层原理与优化 (Performance & Internals)

### 5.1 SQL 的实际执行顺序

SQL 的执行顺序**不是**书写顺序！

```
书写顺序                    实际执行顺序
─────────                  ─────────────
SELECT        ←─── 5      FROM          ←─── 1
FROM          ←─── 1      JOIN / ON     ←─── 2
WHERE         ←─── 2      WHERE         ←─── 3
GROUP BY      ←─── 3      GROUP BY      ←─── 4
HAVING        ←─── 4      HAVING        ←─── 5
ORDER BY      ←─── 6      SELECT        ←─── 6
LIMIT         ←─── 7      ORDER BY      ←─── 7
                           LIMIT         ←─── 8
```

> 这解释了为什么 `WHERE` 中不能用 `SELECT` 里定义的别名——因为 `SELECT` 在 `WHERE` 之后才执行。

### 5.2 执行计划 (EXPLAIN)

```sql
-- 查看查询的执行计划
EXPLAIN SELECT * FROM employees WHERE department = 'Finance';

-- MySQL 详细版
EXPLAIN ANALYZE SELECT ...;
```

关注点：
- **type**: `ALL`（全表扫描，最慢） → `index` → `range` → `ref` → `const`（最快）
- **rows**: 预估扫描行数
- **Extra**: `Using index`（覆盖索引）、`Using filesort`（需额外排序）

### 5.3 常见优化技巧

```sql
-- ❌ 索引失效：对列使用函数
WHERE YEAR(hire_date) = 2024

-- ✅ Sargable 写法：保持列的原始形态
WHERE hire_date >= '2024-01-01' AND hire_date < '2025-01-01'

-- ❌ 前导通配符导致索引失效
WHERE name LIKE '%三'

-- ✅ 后缀通配符可利用索引
WHERE name LIKE '张%'

-- ❌ SELECT * 取所有列（浪费 I/O）
SELECT * FROM employees;

-- ✅ 只取需要的列
SELECT first_name, salary FROM employees;
```

### 5.4 存储过程与触发器（了解即可）

```sql
-- 存储过程：封装一段可复用的 SQL 逻辑
CREATE PROCEDURE raise_salary(IN dept VARCHAR(50), IN pct DECIMAL(5,2))
BEGIN
    UPDATE employees
    SET salary = salary * (1 + pct / 100)
    WHERE department = dept;
END;

-- 调用
CALL raise_salary('Finance', 10);

-- 触发器：在 INSERT/UPDATE/DELETE 时自动执行
CREATE TRIGGER log_salary_change
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    INSERT INTO salary_log (emp_id, old_salary, new_salary, changed_at)
    VALUES (OLD.id, OLD.salary, NEW.salary, NOW());
END;
```

---

## 六、避坑指南

### 6.1 NULL 的三值逻辑

SQL 中 `NULL` 表示 **Unknown**，不是 0，不是空字符串。

```sql
-- ❌ 永远不会返回结果！
SELECT * FROM employees WHERE bonus = NULL;

-- ✅ 正确写法
SELECT * FROM employees WHERE bonus IS NULL;

-- NULL 参与运算的结果仍是 NULL
SELECT 100 + NULL;  -- 结果: NULL

-- 聚合函数自动忽略 NULL
SELECT AVG(bonus) FROM employees;  -- 只对非 NULL 值求平均
```

### 6.2 GROUP BY 的陷阱

```sql
-- ❌ 错误：first_name 既不在 GROUP BY 中，也不在聚合函数中
SELECT first_name, department, AVG(salary)
FROM employees
GROUP BY department;

-- ✅ 正确
SELECT department, AVG(salary) FROM employees GROUP BY department;
```

### 6.3 集合思维 vs 循环思维

```sql
-- ❌ 循环思维（用游标逐行处理，慢）
DECLARE cur CURSOR FOR SELECT id, salary FROM employees;
-- ...逐行 FETCH 并 UPDATE

-- ✅ 集合思维（一条语句搞定，快）
UPDATE employees SET salary = salary * 1.1 WHERE department = 'IT';
```

---

## 七、Python 集成

### 7.1 Pandas + SQL

```python
import pandas as pd
import sqlite3

conn = sqlite3.connect('my_database.db')

# 读取 SQL 查询结果到 DataFrame
df = pd.read_sql('SELECT * FROM employees WHERE salary > 50000', conn)

# 将 DataFrame 写回数据库
df.to_sql('employees_backup', conn, if_exists='replace', index=False)
```

### 7.2 SQLAlchemy (ORM)

```python
from sqlalchemy import create_engine, text

engine = create_engine('sqlite:///my_database.db')

# 原生 SQL
with engine.connect() as conn:
    result = conn.execute(text("SELECT * FROM employees"))
    for row in result:
        print(row)

# 搭配 Pandas
df = pd.read_sql('SELECT * FROM employees', engine)
```

---

## 速查表

| 需求 | 语法 |
|------|------|
| 去重 | `SELECT DISTINCT` |
| 别名 | `AS` |
| 模糊匹配 | `LIKE '%pattern%'` |
| 范围 | `BETWEEN a AND b` |
| 空值判断 | `IS NULL` / `IS NOT NULL` |
| 条件逻辑 | `CASE WHEN ... THEN ... ELSE ... END` |
| 分组过滤 | `GROUP BY ... HAVING ...` |
| 排名 | `ROW_NUMBER() / RANK() / DENSE_RANK() OVER(...)` |
| 前后行 | `LAG() / LEAD() OVER(...)` |
| 临时表 | `WITH cte AS (...)` |
| 判断存在 | `EXISTS (subquery)` |
| 事务 | `BEGIN; ... COMMIT; / ROLLBACK;` |
| 执行计划 | `EXPLAIN SELECT ...` |
