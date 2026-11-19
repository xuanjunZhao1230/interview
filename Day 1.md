# 测试开发实习生面试记录——Day 1

## 一、SQL 数据库

### 1. 统计每个用户的登录次数

**面试题：**

查询每个用户的登录次数，并按照登录次数从高到低排序。

**正确答案：**

```sql
SELECT
    user_id,
    COUNT(*) AS login_count
FROM userdata
GROUP BY user_id
ORDER BY login_count DESC;
```

**知识点：**

- `COUNT(*)`：统计记录数量。
- `GROUP BY`：按照用户 ID 分组。
- `ORDER BY DESC`：按照登录次数降序排列。

### 2. 查询平均成绩大于 70 的学生

**面试题：**

查询每个学生的平均成绩，只显示平均成绩大于 70 的学生。

**正确答案：**

```sql
SELECT
    student_id,
    AVG(score) AS average
FROM studentdata
GROUP BY student_id
HAVING AVG(score) > 70;
```

**知识点：**

- `AVG()`：计算平均值。
- `GROUP BY`：按照学生 ID 分组。
- `HAVING`：对分组后的统计结果进行筛选。

### 3. 查询每个部门的最高工资

**面试题：**

查询每个部门的最高工资。

**正确答案：**

```sql
SELECT
    department,
    MAX(salary) AS max_salary
FROM employees
GROUP BY department;
```

**知识点：**

- `MAX()`：获取最大值。
- `GROUP BY`：按照部门分组。
- 每个部门都需要单独计算最高工资，不能使用 `LIMIT 1` 代替。

---

## 二、Day 1 重点复习清单

| SQL 知识点 | 作用 |
|---|---|
| COUNT() | 统计数量 |
| SUM() | 求和 |
| AVG() | 计算平均值 |
| MAX() | 获取最大值 |
| MIN() | 获取最小值 |
| WHERE | 分组前筛选数据 |
| GROUP BY | 数据分组 |
| HAVING | 分组后筛选数据 |
| ORDER BY | 排序 |
| LIMIT | 限制返回记录数量 |

## 三、需要记住的核心区别

**WHERE 和 HAVING：**

- WHERE：筛选原始数据。
- HAVING：筛选分组后的统计结果。

**GROUP BY 和 ORDER BY：**

- GROUP BY：按照指定字段分组。
- ORDER BY：按照指定字段排序。

**今日复习目标：**

1. 熟练掌握五种聚合函数。
2. 理解 GROUP BY 的使用场景。
3. 熟练区分 WHERE 和 HAVING。
4. 能够独立完成简单的分组统计 SQL 题。
