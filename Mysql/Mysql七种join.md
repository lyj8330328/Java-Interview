# 一、准备工作

## 1.1 建表

```mysql
CREATE TABLE tbl_dept(
	id INT(11) NOT NULL AUTO_INCREMENT,
	deptName VARCHAR(30),
	locAdd VARCHAR(40),
	PRIMARY KEY(id)
)ENGINE=INNODB CHARSET = utf8;

CREATE TABLE tbl_emp(
	id INT(11) NOT NULL AUTO_INCREMENT,
	NAME VARCHAR(30),
	deptId INT(11),
	PRIMARY KEY(id),
	FOREIGN KEY tbl_emp(deptId) REFERENCES tbl_dept(id)
)ENGINE=INNODB CHARSET = utf8;
```

## 1.2 插入数据

```mysql
INSERT INTO tbl_dept(deptName,locAdd) VALUES('RD',11);
INSERT INTO tbl_dept(deptName,locAdd) VALUES('HR',12);
INSERT INTO tbl_dept(deptName,locAdd) VALUES('MK',13);
INSERT INTO tbl_dept(deptName,locAdd) VALUES('MIS',14);
INSERT INTO tbl_dept(deptName,locAdd) VALUES('FD',15);

INSERT INTO tbl_emp(NAME,deptId) VALUES('z3',1);
INSERT INTO tbl_emp(NAME,deptId) VALUES('z4',1);
INSERT INTO tbl_emp(NAME,deptId) VALUES('z5',1);

INSERT INTO tbl_emp(NAME,deptId) VALUES('w5',2);
INSERT INTO tbl_emp(NAME,deptId) VALUES('w6',2);

INSERT INTO tbl_emp(NAME,deptId) VALUES('s7',3);

INSERT INTO tbl_emp(NAME,deptId) VALUES('s8',4);
```

# 二、连接

## *2.1 内连接

![](http://mycsdnblog.work/201919042008-J.png)

```mysql
SELECT
	* 
FROM
	tbl_dept a
	INNER JOIN tbl_emp b ON a.id = b.deptId;
```

或者

```mysql
SELECT
	* 
FROM
	tbl_dept,
	tbl_emp 
WHERE
	tbl_dept.id = tbl_emp.deptId;
```

结果：

![](http://mycsdnblog.work/201919042016-a.png)

## 2.2 外连接

### 2.2.1 左外连接

![](http://mycsdnblog.work/201919042021-X.png)

```mysql
SELECT
	* 
FROM
	tbl_dept a
	LEFT JOIN tbl_emp b ON a.id = b.deptId;
```

或者

```mysql
SELECT
	* 
FROM
	tbl_dept a
	LEFT OUTER JOIN tbl_emp b ON a.id = b.deptId;
```

结果：

![](http://mycsdnblog.work/201919042026-a.png)

### 2.2.2 右外连接

![](http://mycsdnblog.work/201919042031-Y.png)

```mysql
SELECT
	* 
FROM
	tbl_dept a
	RIGHT JOIN tbl_emp b ON a.id = b.deptId;
```

或者：

```mysql
SELECT
	* 
FROM
	tbl_dept a
	RIGHT OUTER JOIN tbl_emp b ON a.id = b.deptId;
```

结果：

![](http://mycsdnblog.work/201919042033-m.png)

### 2.2.3 全连接

![](http://mycsdnblog.work/201919042042-i.png)

mysql不支持，用其他方案代替

## 2.3 自连接

自连接，连接的两个表都是同一个表，同样可以由内连接，外连接各种组合方式，按实际应用去组合

```mysql
SELECT
	* 
FROM
	tbl_dept a
	LEFT JOIN tbl_dept b ON a.id = b.id;
```

结果：

![](http://mycsdnblog.work/201919042055-f.png)

## 2.4 笛卡儿积

A x B

```mysql
SELECT
	* 
FROM
	tbl_dept
	CROSS JOIN tbl_emp;
```

结果：

| id   | deptName | locAdd | id(1) | NAME | deptId |
| ---- | -------- | ------ | ----- | ---- | :----- |
| 1    | RD       | 11     | 1     | z3   | 1      |
| 2    | HR       | 12     | 1     | z3   | 1      |
| 3    | MK       | 13     | 1     | z3   | 1      |
| 4    | MIS      | 14     | 1     | z3   | 1      |
| 5    | FD       | 15     | 1     | z3   | 1      |
| 1    | RD       | 11     | 2     | z4   | 1      |
| 2    | HR       | 12     | 2     | z4   | 1      |
| 3    | MK       | 13     | 2     | z4   | 1      |
| 4    | MIS      | 14     | 2     | z4   | 1      |
| 5    | FD       | 15     | 2     | z4   | 1      |
| 1    | RD       | 11     | 3     | z5   | 1      |
| 2    | HR       | 12     | 3     | z5   | 1      |
| 3    | MK       | 13     | 3     | z5   | 1      |
| 4    | MIS      | 14     | 3     | z5   | 1      |
| 5    | FD       | 15     | 3     | z5   | 1      |
| 1    | RD       | 11     | 4     | w5   | 2      |
| 2    | HR       | 12     | 4     | w5   | 2      |
| 3    | MK       | 13     | 4     | w5   | 2      |
| 4    | MIS      | 14     | 4     | w5   | 2      |
| 5    | FD       | 15     | 4     | w5   | 2      |
| 1    | RD       | 11     | 5     | w6   | 2      |
| 2    | HR       | 12     | 5     | w6   | 2      |
| 3    | MK       | 13     | 5     | w6   | 2      |
| 4    | MIS      | 14     | 5     | w6   | 2      |
| 5    | FD       | 15     | 5     | w6   | 2      |
| 1    | RD       | 11     | 6     | s7   | 3      |
| 2    | HR       | 12     | 6     | s7   | 3      |
| 3    | MK       | 13     | 6     | s7   | 3      |
| 4    | MIS      | 14     | 6     | s7   | 3      |
| 5    | FD       | 15     | 6     | s7   | 3      |
| 1    | RD       | 11     | 7     | s8   | 4      |
| 2    | HR       | 12     | 7     | s8   | 4      |
| 3    | MK       | 13     | 7     | s8   | 4      |
| 4    | MIS      | 14     | 7     | s8   | 4      |
| 5    | FD       | 15     | 7     | s8   | 4      |