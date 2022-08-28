## MySQL存储过程

### 0.环境说明：

| 软件     | 版本 |
| -------- | ---- |
| mysql    | 5.6  |
| HeidiSQL |      |

### 1.使用说明

​		存储过程时数据库的一个重要的对象，可以封装SQL语句集，可以用来完成一些较复杂的业务逻辑，并且可以入参出参(类似于java中的方法的书写)。

​		创建时会预先编译后保存，用户后续的调用都不需要再次编译。

```java
// 把editUser类比成一个存储过程
public void editUser(User user,String username){
    String a = "nihao";
    user.setUsername(username);
}
main(){
    User user = new User();
	editUser(user,"张三");
    user.getUseranme();   //java基础还记得不
}
```

​	大家可能会思考，用sql处理业务逻辑还要重新学，我用java来处理逻辑（比如循环判断、循环查询等）不行吗？那么，为什么还要用存储过程处理业务逻辑呢？

```
优点：
	在生产环境下，可以通过直接修改存储过程的方式修改业务逻辑（或bug），而不用重启服务器。
	执行速度快，存储过程经过编译之后会比单独一条一条执行要快。
	减少网络传输流量。
	方便优化。
缺点：
	过程化编程，复杂业务处理的维护成本高。
	调试不便
	不同数据库之间可移植性差。-- 不同数据库语法不一致！
```



### 2.准备：

> 数据库参阅资料中的sql脚本；
>
> ```sql
> delimiter $$ --声明结束符
> ```



### 3.语法

```
-- 官方参考网址
https://dev.mysql.com/doc/refman/5.6/en/sql-statements.html
https://dev.mysql.com/doc/refman/5.6/en/sql-compound-statements.html
```

#### 3.0 语法结构

```sql
CREATE
    [DEFINER = user]
	PROCEDURE sp_name ([proc_parameter[,...]])
    [characteristic ...] routine_body
    
-- proc_parameter参数部分，可以如下书写：
	[ IN | OUT | INOUT ] param_name type
	-- type类型可以是MySQL支持的所有类型
	
-- routine_body(程序体)部分，可以书写合法的SQL语句 BEGIN ... END
```

简单演示：

```sql
-- 声明结束符。因为MySQL默认使用‘;’作为结束符，而在存储过程中，会使用‘;’作为一段语句的结束，导致‘;’使用冲突
delimiter $$

CREATE PROCEDURE hello_procedure ()
BEGIN
	SELECT 'hello procedure';
END $$

call hello_procedure();
```



#### 3.1 变量及赋值

```
类比一下java中的局部变量和成员变量的声明和使用
```

##### 局部变量：

用户自定义，在begin/end块中有效

```sql
语法：
声明变量 declare var_name type [default var_value];
举例：declare nickname varchar(32);
```

```sql
-- set赋值
create procedure sp_var01()
begin
	declare nickname varchar(32) default 'unkown';
	set nickname = 'ZS';
	-- set nickname := 'SF';
	select nickname;
end$$
```

```sql
-- into赋值
delimiter $$
create procedure sp_var_into()
begin
	declare emp_name varchar(32) default 'unkown' ;
	declare emp_no int default 0;
	select e.empno,e.ename into emp_no,emp_name from emp e where e.empno = 7839;
	select emp_no,emp_name;
end$$
```



##### 用户变量：

用户自定义，当前会话（连接）有效。类比java的成员变量

```
语法： 
@var_name
不需要提前声明，使用即声明
```

```sql
-- 赋值
delimiter $$
create procedure sp_var02()
begin
	set @nickname = 'ZS';
	-- set nickname := 'SF';
end$$
call sp_var02() $$
select @nickname$$  --可以看到结果
```



##### 会话变量：

由系统提供，当前会话（连接）有效

```
语法：
@@session.var_name
```

```sql
show session variables; -- 查看会话变量
select @@session.unique_checks; -- 查看某会话变量
set @@session.unique_checks = 0; --修改会话变量
```



##### 全局变量：

由系统提供，整个mysql服务器有效

```
语法：
@@global.var_name
```

举例

```sql
-- 查看全局变量中变量名有char的记录
show global variables like '%char%'; 

-- 查看全局变量character_set_client的值
select @@global.character_set_client; 
```



#### 3.2 入参出参

```sql
-- 语法
in | out | inout param_name type
```

举例

```sql
-- IN类型演示
delimiter $$
create procedure sp_param01(in age int)
begin
	set @user_age = age;
end$$
call sp_param01(10) $$
select @user_age$$
```

```sql
-- OUT类型，只负责输出！
-- 需求：输出传入的地址字符串对应的部门编号。
delimiter $$

create procedure sp_param02(in loc varchar(64),out dept_no int(11))
begin
	select d.deptno into dept_no from dept d where d.loc = loc;
	--此处强调，要么表起别名，要么入参名不与字段名一致
end$$
delimiter ;

--测试
set @dept_no = 100;
call sp_param01('DALLAS',@dept_no);
select @dept_no;
```

```sql
-- INOUT类型 
delimiter $$
create procedure sp_param03(inout name varchar)
begin
	set name = concat('hello' ,name);
end$$
delimiter ;

set @user_name = '小明';
call sp_param03(@user_name);
select @user_name;
```

#### 3.3 流程控制-判断

```
官网说明
https://dev.mysql.com/doc/refman/5.6/en/flow-control-statements.html
```

###### `if`

```
-- 语法
IF search_condition THEN statement_list
    [ELSEIF search_condition THEN statement_list] ...
    [ELSE statement_list]
END IF
```

举例：

```sql
-- 前置知识点：timestampdiff(unit,exp1,exp2) 取差值exp2-exp1差值，单位是unit
select timestampdiff(year,e.hiredate,now()) from emp e where e.empno = '7499';
```

```sql
-- 需求：入职年限<=38是新手 >38并且<=40老员工 >40元老
delimiter $$
create procedure sp_hire_if()
begin
	declare result varchar(32);
	if timestampdiff(year,'2001-01-01',now()) > 40 
		then set result = '元老';
	elseif timestampdiff(year,'2001-01-01',now()) > 38
		then set result = '老员工';
	else 
		set result = '新手';
	end if;
	select result;
end$$
delimiter ;
```

###### `case`

此语法是不仅可以用在存储过程，查询语句也可以用！

```sql
-- 语法一（类比java的switch）：
CASE case_value
    WHEN when_value THEN statement_list
    [WHEN when_value THEN statement_list] ...
    [ELSE statement_list]
END CASE
-- 语法二：
CASE
    WHEN search_condition THEN statement_list
    [WHEN search_condition THEN statement_list] ...
    [ELSE statement_list]
END CASE
```

举例：

```sql
-- 需求：入职年限年龄<=38是新手 >38并 <=40老员工 >40元老
delimiter $$
create procedure sp_hire_case()
begin
	declare result varchar(32);
	declare message varchar(64);
	case
    when timestampdiff(year,'2001-01-01',now()) > 40 
		then 
			set result = '元老';
			set message = '老爷爷';
	when timestampdiff(year,'2001-01-01',now()) > 38
		then 
			set result = '老员工';
			set message = '油腻中年人';
	else 
		set result = '新手';
		set message = '萌新';
	end case;
	select result;
end$$
delimiter ;
```



#### 3.4 流程控制-循环

###### `loop`

```sql
-- 语法
[begin_label:] LOOP
    statement_list
END LOOP [end_label]

```

举例

> 需要说明，loop是死循环，需要手动退出循环，我们可以使用`leave`来退出。
>
> 可以把leave看成我们java中的break；与之对应的，就有`iterate`（继续循环）——类比java的continue

```sql
--需求：循环打印1到10
-- leave控制循环的退出
delimiter $$
create procedure sp_flow_loop()
begin
	declare c_index int default 1;
	declare result_str  varchar(256) default '1';
	cnt:loop
	
		if c_index >= 10
		then leave cnt;
		end if;

		set c_index = c_index + 1;
		set result_str = concat(result_str,',',c_index);
		
	end loop cnt;
	
	select result_str;
end$$

-- iterate + leave控制循环
delimiter $$
create procedure sp_flow_loop02()
begin
	declare c_index int default 1;
	declare result_str  varchar(256) default '1';
	cnt:loop

		set c_index = c_index + 1;
		set result_str = concat(result_str,',',c_index);
		if c_index < 10 then 
			iterate cnt; 
		end if;
		-- 下面这句话能否执行到？什么时候执行到？ 当c_index < 10为false时执行
		leave cnt;
		
	end loop cnt;
	select result_str;
	
end$$
```



###### `repeat`

![1586956047152](MySql存储过程.assets/1586956047152.png)

```sql
[begin_label:] REPEAT
    statement_list
UNTIL search_condition	-- 直到…为止，才退出循环
END REPEAT [end_label]
```

```sql
-- 需求：循环打印1到10
delimiter $$
create procedure sp_flow_repeat()
begin
	declare c_index int default 1;
	-- 收集结果字符串
	declare result_str varchar(256) default '1';
	count_lab:repeat
		set c_index = c_index + 1;
		set result_str = concat(result_str,',',c_index);
		until c_index >= 10
	end repeat count_lab;
	select result_str;
end$$
```



###### `while`

```
类比java的while(){}
```

```sql
[begin_label:] WHILE search_condition DO
    statement_list
END WHILE [end_label]
```

```sql
-- 需求：循环打印1到10
delimiter $$
create procedure sp_flow_while()
begin
	declare c_index int default 1;
	-- 收集结果字符串
	declare result_str varchar(256) default '1';
	while c_index < 10 do
		set c_index = c_index + 1;
		set result_str = concat(result_str,',',c_index);
	end while;
	select result_str;
end$$
```

#### 3.5 流程控制-退出、继续循环

###### `leave` 

```
类比java的breake
```

```sql
-- 退出 LEAVE can be used within BEGIN ... END or loop constructs (LOOP, REPEAT, WHILE).
LEAVE label
```

###### `iterate`

```
类比java的continue
```

```sql
-- 继续循环 ITERATE can appear only within LOOP, REPEAT, and WHILE statements
ITERATE label
```

#### 3.6 游标

用游标得到某一个结果集，逐行处理数据。

```
类比jdbc的ResultSet
```

```sql
-- 声明语法
DECLARE cursor_name CURSOR FOR select_statement
-- 打开语法
OPEN cursor_name
-- 取值语法
FETCH cursor_name INTO var_name [, var_name] ...
-- 关闭语法
CLOSE cursor_name
```

```sql
-- 需求：按照部门名称查询员工，通过select查看员工的编号、姓名、薪资。（注意，此处仅仅演示游标用法）
delimiter $$
create procedure sp_create_table02(in dept_name varchar(32))
begin
	declare e_no int;
	declare e_name varchar(32);
	declare e_sal decimal(7,2);
	
	declare lp_flag boolean default true;
	
	declare emp_cursor cursor for 
		select e.empno,e.ename,e.sal
		from emp e,dept d
		where e.deptno = d.deptno and d.dname = dept_name;
		
	-- handler 句柄
	declare continue handler for NOT FOUND set lp_flag = false;
		
	open emp_cursor;
	
	emp_loop:loop
		fetch emp_cursor into e_no,e_name,e_sal;
		
		if lp_flag then
			select e_no,e_name,e_sal;
		else
			leave emp_loop;
		end if;
		
	end loop emp_loop;
	set @end_falg = 'exit_flag';
	close emp_cursor;
end$$

call sp_create_table02('RESEARCH');
```

> 特别注意：
>
> 在语法中，变量声明、游标声明、handler声明是必须按照先后顺序书写的，否则创建存储过程出错。

#### 3.7 存储过程中的handler

```sql
DECLARE handler_action HANDLER
    FOR condition_value [, condition_value] ...
    statement

handler_action: {
    CONTINUE
  | EXIT
  | UNDO
}

condition_value: {
    mysql_error_code
  | SQLSTATE [VALUE] sqlstate_value
  | condition_name
  | SQLWARNING
  | NOT FOUND
  | SQLEXCEPTION
}


CONTINUE: Execution of the current program continues.
EXIT: Execution terminates for the BEGIN ... END compound statement in which the handler is declared. This is true even if the condition occurs in an inner block.


SQLWARNING: Shorthand for the class of SQLSTATE values that begin with '01'.
NOT FOUND: Shorthand for the class of SQLSTATE values that begin with '02'.
SQLEXCEPTION: Shorthand for the class of SQLSTATE values that do not begin with '00', '01', or '02'.
```

```sql
-- 各种写法：
	DECLARE exit HANDLER FOR SQLSTATE '42S01' set @res_table = 'EXISTS';
	DECLARE continue HANDLER FOR 1050 set @res_table = 'EXISTS';
	DECLARE continue HANDLER FOR not found set @res_table = 'EXISTS';
```



### 4.练习

——大家注意，存储过程的业务过程在java代码中一般也可以实现，我们下面的需求是为了练习存储过程

#### 4.1 利用存储过程更新数据

```
为某部门(需指定)的人员涨薪100;如果是公司总裁，则不涨薪。
```

```sql
delimiter //
create procedure high_sal(in dept_name varchar(32))
begin
	declare e_no int;
	declare e_name varchar(32);
	declare e_sal decimal(7,2);
	
	declare lp_flag boolean default true;
	
	declare emp_cursor cursor for 
		select e.empno,e.ename,e.sal
		from emp e,dept d
		where e.deptno = d.deptno and d.dname = dept_name;
		
	-- handler 句柄
	declare continue handler for NOT FOUND set lp_flag = false;
		
	open emp_cursor;
	
	emp_loop:loop
		fetch emp_cursor into e_no,e_name,e_sal;
		
		if lp_flag then
			if e_name = 'king' then 
				iterate emp_loop;
			else 
				update emp e set e.sal = e.sal + 100 where e.empno = e_no;
			end if;
		else
			leave emp_loop;
		end if;
		
	end loop emp_loop;
	set @end_falg = 'exit_flag';
	close emp_cursor;
end //


call high_sal('ACCOUNTING');

```



#### 4.2 循环创建表

```
创建下个月的每天对应的表comp_2020_04_01、comp_2020_04_02、...
（模拟）需求描述：
我们需要用某个表记录很多数据，比如记录某某用户的搜索、购买行为(注意，此处是假设用数据库保存)，当每天记录较多时，如果把所有数据都记录到一张表中太庞大，需要分表，我们的要求是，每天一张表，存当天的统计数据，就要求提前生产这些表——每月月底创建下一个月每天的表！
```

```sql
-- 知识点 预处理 prepare语句from后使用局部变量会报错 
-- https://dev.mysql.com/doc/refman/5.6/en/sql-prepared-statements.html
PREPARE stmt_name FROM preparable_stmt
EXECUTE stmt_name [USING @var_name [, @var_name] ...]
{DEALLOCATE | DROP} PREPARE stmt_name
-- 知识点 时间的处理
-- EXTRACT(unit FROM date)截取时间的指定位置值
-- DATE_ADD(date,INTERVAL expr unit)  日期运算
-- LAST_DAY(date)  获取日期的最后一天
-- YEAR(date) 返回日期中的年
-- MONTH(date) 返回日期的月
-- DAYOFMONTH(date) 返回日
```

```sql
-- 思路：循环构建表名 comp_2020_05_01 到 comp_2020_05_31；并执行create语句。
delimiter //
create procedure sp_create_table()
begin
	declare next_year int;
	declare next_month int;
	declare next_month_day int;
		
	declare next_month_str char(2);
	declare next_month_day_str char(2);
	
	-- 处理每天的表名
	declare table_name_str char(10);
	
	declare t_index int default 1;
	-- declare create_table_sql varchar(200);
	
	-- 获取下个月的年份
	set next_year = year(date_add(now(),INTERVAL 1 month));
	-- 获取下个月是几月 
	set next_month = month(date_add(now(),INTERVAL 1 month));
	-- 下个月最后一天是几号
	set next_month_day = dayofmonth(LAST_DAY(date_add(now(),INTERVAL 1 month)));
	
	if next_month < 10
		then set next_month_str = concat('0',next_month);
	else
		set next_month_str = concat('',next_month);
	end if;
	
	
	while t_index <= next_month_day do
		
		if (t_index < 10)
			then set next_month_day_str = concat('0',t_index);
		else
			set next_month_day_str = concat('',t_index);
		end if;
		
		-- 2020_05_01
		set table_name_str = concat(next_year,'_',next_month_str,'_',next_month_day_str);
		-- 拼接create sql语句
		set @create_table_sql = concat(
					'create table comp_',
					table_name_str,
					'(`grade` INT(11) NULL,`losal` INT(11) NULL,`hisal` INT(11) NULL) COLLATE=\'utf8_general_ci\' ENGINE=InnoDB');
		-- FROM后面不能使用局部变量！
		prepare create_table_stmt FROM @create_table_sql;
		execute create_table_stmt;
		DEALLOCATE prepare create_table_stmt;
		
		set t_index = t_index + 1;
		
	end while;	
end//

call sp_create_table()
```



#### 4.3 其他场景：

```
“为用户添加购物积分，并更新到用户的总积分表中”等需要对多张表进行CRUD操作的业务。
而且内部可以使用事务命令。
```



### 5.其他

#### 5.1 characteristic

```
characteristic:
    COMMENT 'string'
  | LANGUAGE SQL
  | [NOT] DETERMINISTIC
  | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
  | SQL SECURITY { DEFINER | INVOKER }
```

其中，SQL SECURITY的含义如下：

```
	MySQL存储过程是通过指定SQL SECURITY子句指定执行存储过程的实际用户；
	如果SQL SECURITY子句指定为DEFINER(定义者)，存储过程将使用存储过程的DEFINER执行存储过程，验证调用存储
	过程的用户是否具有存储过程的execute权限和DEFINER用户是否具有存储过程引用的相关对象(存储过程中的表等对象)的权限；
	如果SQL SECURITY子句指定为INVOKER(调用者)，那么MySQL将使用当前调用存储过程的用户执行此过程，并验证用户是否具有存储过程的execute权限和存储过程引用的相关对象的权限；
	如果不显示的指定SQL SECURITY子句，MySQL默认将以DEFINER执行存储过程。
```

#### 5.2 死循环处理

```sql
-- 如有死循环处理，可以通过下面的命令查看并结束
show processlist;
kill id;
```

#### 5.3 可以在select语句中写case

```http
https://dev.mysql.com/doc/refman/5.6/en/control-flow-functions.html
```

```sql
select 
	case
		when timestampdiff(year,e.hiredate,now()) <= 38 then '新手'
		when timestampdiff(year,e.hiredate,now()) <= 40 then '老员工'
		else '元老'
	end hir_loc,
	e.*
from emp e;
```

#### 5.4 临时表

```sql
create temporary table 表名(
　　字段名 类型 [约束],
　　name varchar(20) 
)Engine=InnoDB default charset utf8;

-- 需求：按照部门名称查询员工，通过select查看员工的编号、姓名、薪资。（注意，此处仅仅演示游标用法）
delimiter $$
create procedure sp_create_table02(in dept_name varchar(32))
begin
	declare emp_no int;
	declare emp_name varchar(32);
	declare emp_sal decimal(7,2);
	declare exit_flag int default 0;
	
	declare emp_cursor cursor for
		select e.empno,e.ename,e.sal
		from emp e inner join dept d on e.deptno = d.deptno where d.dname = dept_name;
	
	declare continue handler for not found set exit_flag = 1;
	
	-- 创建临时表收集数据
	CREATE temporary TABLE `temp_table_emp` (
		`empno` INT(11) NOT NULL COMMENT '员工编号',
		`ename` VARCHAR(32) NULL COMMENT '员工姓名' COLLATE 'utf8_general_ci',
		`sal` DECIMAL(7,2) NOT NULL DEFAULT '0.00' COMMENT '薪资',
		PRIMARY KEY (`empno`) USING BTREE
	)
	COLLATE='utf8_general_ci'
	ENGINE=InnoDB;	
	
	open emp_cursor;
	
	c_loop:loop
		fetch emp_cursor into emp_no,emp_name,emp_sal;
		
		
		if exit_flag != 1 then
			insert into temp_table_emp values(emp_no,emp_name,emp_sal); 
		else
			leave c_loop;
		end if;
		
	end loop c_loop;
	
	select * from temp_table_emp;
	
	select @sex_res; -- 仅仅是看一下会不会执行到
	close emp_cursor;
	
end$$

call sp_create_table02('RESEARCH');
```



#### 5.5 oracle存储过程

请参阅B站视频

```http
https://www.bilibili.com/video/BV1a7411Z7BN
```



#### 5.6 复制表和数据

```sql
CREATE TABLE dept SELECT * FROM procedure_demo.dept;
CREATE TABLE emp SELECT * FROM procedure_demo.emp;
CREATE TABLE salgrade SELECT * FROM procedure_demo.salgrade;
```



