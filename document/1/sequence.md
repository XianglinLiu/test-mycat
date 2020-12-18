```1.创建MYCAT_SEQUENCE表

**注意：MYCAT_SEQUENCE必须大写**
```mysql
DROP TABLE IF EXISTS MYCAT_SEQUENCE;
```
```
#insert into member(`id`,`name`) values(1,'panglongxia');

CREATE TABLE MYCAT_SEQUENCE (name VARCHAR(50) NOT NULL,current_value INT NOT NULL,increment INT NOT NULL DEFAULT 100, PRIMARY KEY(name)) ENGINE=InnoDB;
```

 其中：
 – name sequence名称  
 – current_value 当前value
 – increment 增长步长! 可理解为mycat在数据库中一次读取多少个sequence. 当这些用完后, 下次再从数据库中读取.


2.插入sequence记录

```mysql
INSERT INTO MYCAT_SEQUENCE(name,current_value,increment) VALUES ('MEMBER', 1, 100);
```

3.创建存储函数。

**注意：必须在同一个数据库中创建，在本例中，是db2。一共要创建三个。**

- 获取当前sequence的值（返回当前值,增量）
```
DROP FUNCTION IF EXISTS `mycat_seq_currval`;
DELIMITER ;;
CREATE DEFINER=`root`@`%` FUNCTION `mycat_seq_currval`(seq_name VARCHAR(50)) RETURNS varchar(64) CHARSET latin1
DETERMINISTIC
BEGIN
DECLARE retval VARCHAR(64);
SET retval="-999999999,null";
SELECT concat(CAST(current_value AS CHAR),",",CAST(increment AS CHAR) ) INTO retval FROM MYCAT_SEQUENCE WHERE name = seq_name;
RETURN retval ;
END
;;
DELIMITER ;
```
- 设置sequence值

```
DROP FUNCTION
IF
	EXISTS `mycat_seq_nextval`;

DELIMITER;;
CREATE DEFINER = `root` @`%` FUNCTION `mycat_seq_nextval` ( seq_name VARCHAR ( 50 ) ) RETURNS VARCHAR ( 64 ) CHARSET latin1 DETERMINISTIC BEGIN
	UPDATE MYCAT_SEQUENCE 
	SET current_value = current_value + increment 
	WHERE
		NAME = seq_name;
	RETURN mycat_seq_currval ( seq_name );
	
END;;

DELIMITER;
```

- 获取下一个sequence值

```
DROP FUNCTION
IF
	EXISTS `mycat_seq_setval`;

DELIMITER;;
CREATE DEFINER = `root` @`%` FUNCTION `mycat_seq_setval` ( seq_name VARCHAR ( 50 ), VALUE INTEGER ) RETURNS VARCHAR ( 64 ) CHARSET latin1 DETERMINISTIC BEGIN
	UPDATE MYCAT_SEQUENCE 
	SET current_value = 
	VALUE
		
	WHERE
		NAME = seq_name;
	RETURN mycat_seq_currval ( seq_name );
	
END;;

DELIMITER;　
```
至此，数据库方面的准备工作已结束完毕。

```