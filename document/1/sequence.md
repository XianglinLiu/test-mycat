```
DROP TABLE IF EXISTS MYCAT_SEQUENCE;
CREATE TABLE MYCAT_SEQUENCE (
NAME VARCHAR (50) NOT NULL,
current_value INT NOT NULL,
increment INT NOT NULL DEFAULT 100,
PRIMARY KEY (NAME)
) ENGINE = INNODB ;


INSERT INTO MYCAT_SEQUENCE(NAME,current_value,increment) VALUES ('GLOBAL', 100000, 100);

DROP FUNCTION IF EXISTS `mycat_seq_currval`;
DELIMITER ;;
CREATE FUNCTION `mycat_seq_currval`(seq_name VARCHAR(50)) 
RETURNS VARCHAR(64) CHARSET utf8
    DETERMINISTIC
BEGIN DECLARE retval VARCHAR(64);
        SET retval="-999999999,null";  
        SELECT CONCAT(CAST(current_value AS CHAR),",",CAST(increment AS CHAR) ) INTO retval 
          FROM MYCAT_SEQUENCE WHERE NAME = seq_name;  
        RETURN retval ; 
END
;;
DELIMITER ;

DROP FUNCTION IF EXISTS `mycat_seq_nextval`;
DELIMITER ;;
CREATE FUNCTION `mycat_seq_nextval`(seq_name VARCHAR(50)) RETURNS VARCHAR(64)
 CHARSET utf8
    DETERMINISTIC
BEGIN UPDATE MYCAT_SEQUENCE  
                 SET current_value = current_value + increment 
                  WHERE NAME = seq_name;  
         RETURN mycat_seq_currval(seq_name);  
END
;;
DELIMITER ;


DROP FUNCTION IF EXISTS `mycat_seq_setval`;
DELIMITER ;;
CREATE FUNCTION `mycat_seq_setval`(seq_name VARCHAR(50), VALUE INTEGER) 
RETURNS VARCHAR(64) CHARSET utf8
    DETERMINISTIC
BEGIN UPDATE MYCAT_SEQUENCE  
                   SET current_value = VALUE  
                   WHERE NAME = seq_name;  
         RETURN mycat_seq_currval(seq_name);  
END
;;
DELIMITER ;

```

```
#设定TEST3表的增长方式为 步进为1 ，
insert into MYCAT_SEQUENCE (name,current_value,increment) values ('MEMBER',0,100);
```