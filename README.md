# BetterMysqlTestDemo
mysql 优化，压力测试脚本。

适用人群：想做mysql优化的朋友可以看看，写了一个存储过程来随机生成一个400万条数据的表，然后进行相关优化策略实战。



1.建表
总共建了三张表，脚本分别如下：
CREATE TABLE dept( /*部门表*/
deptno MEDIUMINT   UNSIGNED  NOT NULL  DEFAULT 0,  /*编号*/
dname VARCHAR(20)  NOT NULL  DEFAULT "", /*名称*/
loc VARCHAR(13) NOT NULL DEFAULT "" /*地点*/
) ENGINE=MyISAM DEFAULT CHARSET=utf8 ;


CREATE TABLE emp
(empno  MEDIUMINT UNSIGNED  NOT NULL  DEFAULT 0, /*编号*/
ename VARCHAR(20) NOT NULL DEFAULT "", /*名字*/
job VARCHAR(9) NOT NULL DEFAULT "",/*工作*/
mgr MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,/*上级编号*/
hiredate DATE NOT NULL,/*入职时间*/
sal DECIMAL(7,2)  NOT NULL,/*薪水*/
comm DECIMAL(7,2) NOT NULL,/*红利*/
deptno MEDIUMINT UNSIGNED NOT NULL DEFAULT 0 /*部门编号*/
)ENGINE=MyISAM DEFAULT CHARSET=utf8 ;


CREATE TABLE salgrade
(
grade MEDIUMINT UNSIGNED NOT NULL DEFAULT 0,
losal DECIMAL(17,2)  NOT NULL,
hisal DECIMAL(17,2)  NOT NULL
)ENGINE=MyISAM DEFAULT CHARSET=utf8;


还提供一些测试数据：

INSERT INTO salgrade VALUES (1,700,1200);
INSERT INTO salgrade VALUES (2,1201,1400);
INSERT INTO salgrade VALUES (3,1401,2000);
INSERT INTO salgrade VALUES (4,2001,3000);
INSERT INTO salgrade VALUES (5,3001,9999);
2.两个随机函数的书写
//先要修改截止标志符
delimiter $$

//执行下面函数
create function rand_string(n INT) 
returns varchar(255) #该函数会返回一个字符串
begin 
#chars_str定义一个变量 chars_str,类型是 varchar(100),默认值'abcdefghijklmnopqrstuvwxyzABCDEFJHIJKLMNOPQRSTUVWXYZ';
 declare chars_str varchar(100) default
   'abcdefghijklmnopqrstuvwxyzABCDEFJHIJKLMNOPQRSTUVWXYZ';
 declare return_str varchar(255) default '';
 declare i int default 0;
 while i < n do 
   set return_str =concat(return_str,substring(chars_str,floor(1+rand()*52),1));
   set i = i + 1;
   end while;
  return return_str;
  end $$
//测试函数是否成功
select rand_string(6) from dual$$

//另一个函数

create function rand_num()
returns int(5)
begin
 declare i int default 0;
 set i =floor(10+rand()*500); 
return i;
end $$

3.存储过程的书写

create procedure insert_emp(in start int(10),in max_num int(10))
begin
declare i int default 0; 
#set autocommit =0 把autocommit设置成0
 set autocommit = 0;  
 repeat
 set i = i + 1;
 insert into emp values ((start+i) ,rand_string(6),'SALESMAN',0001,curdate(),2000,400,rand_num());
  until i = max_num
 end repeat;
   commit;
 end $$

//调用刚刚写好的函数, 1800000条记录,从100001号开始

call insert_emp(100001,4000000);

//等待几分钟就可以了，此时可以看看插入的数据条数

mysql> delimiter ;
mysql> select count(*) from emp;
+----------+
| count(*) |
+----------+
|  4000000 |
+----------+
1 row in set (0.00 sec)


#**************************************************************
#  向dept表中插入记录

delimiter $$
drop procedure insert_dept $$


create procedure insert_dept(in start int(10),in max_num int(10))
begin
declare i int default 0; 
 set autocommit = 0;  
 repeat
 set i = i + 1;
 insert into dept values ((start+i) ,rand_string(10),rand_string(8));
  until i = max_num
 end repeat;
   commit;
 end $$


delimiter ;
call insert_dept(100,10);

//出现以下提示就成功啦！
mysql> delimiter $$
mysql> create procedure insert_dept(in start int(10),in max_num int(10))
    -> begin
    -> declare i int default 0; 
    ->  set autocommit = 0;  
    ->  repeat
    ->  set i = i + 1;
    ->  insert into dept values ((start+i) ,rand_string(10),rand_string(8));
    ->   until i = max_num
    ->  end repeat;
    ->    commit;
    ->  end $$
Query OK, 0 rows affected (0.01 sec)

mysql> delimiter ;
mysql> call insert_dept(100,10);
Query OK, 0 rows affected (0.01 sec)

4.到这里就结束了，具体优化方法可以参考博文哦！！！


参考博客地址：www.wustzoujing.github.io

参考mysql标签或分类的博文。
