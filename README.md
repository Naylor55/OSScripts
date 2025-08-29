# OSScripts
操作系统脚本收集

## jar 包启动脚本

```shell

#!/bin/bash
# This is a shell script to manual start Jar service
source /etc/profile
JVM_XMS='256m'
JVM_XMX='2512m'
# 获取脚本所在目录
DIR=$( cd "$(dirname "${BASH_SOURCE[0]}")" && pwd);
echo "Shell_Directory:$DIR"
PRO_FILE=${DIR}/bootstrap.yml
echo "Pro File Path:${PRO_FILE}"
if [ "$2" != "" ]; then
    JVM_XMS=$2
fi
JAR_NAME=`find ${DIR} -name "*.jar"`
JAR_FILE="${DIR}/${JAR_NAME##*/}"
echo "Jar File Path:$JAR_FILE"
NAME=${JAR_NAME##*/}
NAME=${NAME%.*}
PID=`ps -ef | grep "$JAR_FILE" | grep java | grep -v grep | awk '{print $2}'`
echo "当前进程号为：$PID"
echo "---------------"
for pid in $PID
do
    sleep 2
    kill -9  $pid
    echo "killed [$pid]"
done
source /etc/profile
nohup java  -Xms$JVM_XMS -Xmx$JVM_XMX -Dspring.config.location=$PRO_FILE -jar $JAR_FILE --SERVER_NAME=$NAME >> ${DIR}/logs/nohup_$NAME.log 2>&1 &
echo "新的进程号为: $!"

```

## 直接内存配置

```shell

nohup java  -XX:MaxDirectMemorySize=2g -Xms$JVM_XMS -Xmx$JVM_XMX -Dspring.config.location=$PRO_FILE -jar $JAR_FILE --SERVER_NAME=$NAME >> ${DIR}/logs/nohup_$NAME.log 2>&1 &

```

## 启动kafka

```shell

cd    /usr/local/kafka_2.13-3.2.0/bin

./zookeeper-server-start.sh -daemon ../config/zookeeper.properties
./kafka-server-start.sh -daemon ../config/server.properties

```

# 数据库相关脚本



## 创建一个mysql库和表

```sql



SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;
CREATE database db2019;
USE db2019;

-- ----------------------------
-- Table structure for payment
-- ----------------------------
DROP TABLE IF EXISTS `payment`;
CREATE TABLE `payment`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `serial` varchar(200) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL COMMENT '支付流水号',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '支付表' ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of payment
-- ----------------------------
INSERT INTO `payment` VALUES (31, '尚硅谷111');
INSERT INTO `payment` VALUES (32, 'atguigu002');
INSERT INTO `payment` VALUES (34, 'atguigu002');
INSERT INTO `payment` VALUES (35, 'atguigu002');

SET FOREIGN_KEY_CHECKS = 1;

---seata
-- the table to store GlobalSession data
create database seata;
USE seata;
drop table if exists `global_table`;
create table `global_table` (
  `xid` varchar(128)  not null,
  `transaction_id` bigint,
  `status` tinyint not null,
  `application_id` varchar(32),
  `transaction_service_group` varchar(32),
  `transaction_name` varchar(128),
  `timeout` int,
  `begin_time` bigint,
  `application_data` varchar(2000),
  `gmt_create` datetime,
  `gmt_modified` datetime,
  primary key (`xid`),
  key `idx_gmt_modified_status` (`gmt_modified`, `status`),
  key `idx_transaction_id` (`transaction_id`)
);

-- the table to store BranchSession data
drop table if exists `branch_table`;
create table `branch_table` (
  `branch_id` bigint not null,
  `xid` varchar(128) not null,
  `transaction_id` bigint ,
  `resource_group_id` varchar(32),
  `resource_id` varchar(256) ,
  `lock_key` varchar(128) ,
  `branch_type` varchar(8) ,
  `status` tinyint,
  `client_id` varchar(64),
  `application_data` varchar(2000),
  `gmt_create` datetime,
  `gmt_modified` datetime,
  primary key (`branch_id`),
  key `idx_xid` (`xid`)
);

-- the table to store lock data
drop table if exists `lock_table`;
create table `lock_table` (
  `row_key` varchar(128) not null,
  `xid` varchar(96),
  `transaction_id` long ,
  `branch_id` long,
  `resource_id` varchar(256) ,
  `table_name` varchar(32) ,
  `pk` varchar(36) ,
  `gmt_create` datetime ,
  `gmt_modified` datetime,
  primary key(`row_key`)
);

---seata biz
create database seata_order;
USE seata_order;
CREATE TABLE `t_order`  (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `user_id` bigint(20) DEFAULT NULL COMMENT '用户id',
  `product_id` bigint(11) DEFAULT NULL COMMENT '产品id',
  `count` int(11) DEFAULT NULL COMMENT '数量',
  `money` decimal(11, 0) DEFAULT NULL COMMENT '金额',
  `status` int(1) DEFAULT NULL COMMENT '订单状态:  0:创建中 1:已完结',
  PRIMARY KEY (`int`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '订单表' ROW_FORMAT = Dynamic;

CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

create database seata_storage;
USE seata_storage;
DROP TABLE IF EXISTS `t_storage`;
CREATE TABLE `t_storage`  (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `product_id` bigint(11) DEFAULT NULL COMMENT '产品id',
  `total` int(11) DEFAULT NULL COMMENT '总库存',
  `used` int(11) DEFAULT NULL COMMENT '已用库存',
  `residue` int(11) DEFAULT NULL COMMENT '剩余库存',
  PRIMARY KEY (`int`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '库存' ROW_FORMAT = Dynamic;
INSERT INTO `t_storage` VALUES (1, 1, 100, 0, 100);

CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

CREATE database seata_account;
USE seata_account;
DROP TABLE IF EXISTS `t_account`;
CREATE TABLE `t_account`  (
  `id` bigint(11) NOT NULL COMMENT 'id',
  `user_id` bigint(11) DEFAULT NULL COMMENT '用户id',
  `total` decimal(10, 0) DEFAULT NULL COMMENT '总额度',
  `used` decimal(10, 0) DEFAULT NULL COMMENT '已用余额',
  `residue` decimal(10, 0) DEFAULT NULL COMMENT '剩余可用额度',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '账户表' ROW_FORMAT = Dynamic;

INSERT INTO `t_account` VALUES (1, 1, 1000, 0, 1000);

CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;


```


## 模拟一个慢查询

```sql

-- 单位 s
SELECT sleep(300) from uav_expand  

```



## 查询某表占用物理存储空间

```sql


SELECT
    table_schema AS '数据库',
    table_name AS '表名',
    table_rows AS '记录数',
    TRUNCATE ( data_length / 1024 / 1024, 2 ) AS '数据容量(MB)',
    TRUNCATE ( index_length / 1024 / 1024, 2 ) AS '索引容量(MB)'
FROM
    information_schema.TABLES
WHERE
    table_schema = 'smart-bams'
    AND table_name = 'label_type';


```



## mysqldump命令


### 备份

```shell

-- 备份单个schema
mysqldump -h 127.0.0.1 -u root -pJ48Yxxx   smart-ar-159 >/home/cml/b2.sql

-- 备份单个表
mysqldump -h 127.0.0.1 -u root -pJ48Yxxx smart-ar  ddd_expand > /home/db_backup/20250528_ddd_expand.sql



-- 备份多个schema

mysqldump -h 127.0.0.1 -u root -pJ48Yxxx  -B smart-ar smart-ba >/home/cml/b3.sql

```


* mysqldump：本机mysql服务自带的程序
* -h：mysql所在的centos，即mysql服务所在的ip
* -u：mysql的登录账号
* -p：mysql的登录密码，p和密码中间不可有空格
* -B：数据库schema 名称，多个使用空格隔开
* smart-ar-159：需要备份的数据库名字
* >/home/cml/b2.sql：保存位置


远程备份：备份远程服务器中的mysql，即mysql在 192.168.1.159 上面，在192.168.1.128上面备份159上mysql中的数据库

```sql

mysqldump -h 192.168.1.159 -u root -pJ48Yxxx  -B smart-ar-159   bams-159 >/home/cml/128-a.sql

```


### 还原

```shell

-- 本机还原
mysql    -h  127.0.0.1    -u  root      -pJ48Yxxx     -B   smart-bams-159  >/home/cml/bams-159.sql


-- 远程还原
mysql    -h  192.168.1.159    -u  root      -pJ48Yxxx     -B  smart-ar-159   smart-bams-159 >/home/cml/128-a.sql

```

## 删除重复数据仅保留一个

```sql

DELETE t1 FROM hkcy_tag_info t1 
JOIN hkcy_tag_info t2 ON t1.tag_logic_id = t2.tag_logic_id AND t1.id > t2.id; -- 假设表中有自增主键id，保留id最小的记录

```


## 死锁分析

```sql

-- 查看死锁
show OPEN TABLES where In_use > 0;

-- 查看正在运行的进程
show processlist;

-- 杀死进程
kill   123456

```


## 常用DDL

```sql

-- 新增字段
ALTER TABLE poi_info ADD `label_type_logic_id` varchar(100) NULL COMMENT '标签类型ID' after tag_small_type ;


-- 删除字段
ALTER TABLE hkcy_tag_info  DROP  COLUMN  `type` ;

-- 修改字段
-- value字段原来的长度为45，太短了，修改成了100
ALTER TABLE `bams_dict` MODIFY COLUMN `value` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NULL DEFAULT NULL COMMENT '数据值';

-- 修改字段数据类型
ALTER TABLE `tag_info` MODIFY COLUMN `theme_id` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '标签主题id';

-- 新增索引
-- add  INDEX 新增普通索引
-- auth_field_4_index 索引名称，自己随便写
-- auth_field_4 列名
alter table hkcy_camera_equipment_info add INDEX  auth_field_4_index (auth_field_4);

-- 删除索引
DROP INDEX ad_logic_id_index ON hkcy_camera_equipment_info;


```


## 慢查询监控开关


```sql

show variables like '%slow_query_log%'


SET GLOBAL slow_query_log = 'OFF';

```


## 通过sql 导出数据字典


### 单张表

```sql

    SELECT
    t.TABLE_NAME AS 表名,
    t.COLUMN_NAME AS 字段名,
    t.COLUMN_TYPE AS 数据类型,
CASE
        IFNULL( t.COLUMN_DEFAULT, 'Null' )
        WHEN '' THEN
        '空字符串'
        WHEN 'Null' THEN
        'NULL' ELSE t.COLUMN_DEFAULT
    END AS 默认值,
CASE
        t.IS_NULLABLE
        WHEN 'YES' THEN
        '是' ELSE '否'
    END AS 是否允许为空,
    t.COLUMN_COMMENT AS 字段说明
FROM
    information_schema.COLUMNS t
WHERE
    t.TABLE_SCHEMA = 'data-center'
    AND t.table_name = 'registered_residence_info'

```


### 表名和注释

```sql

SELECT table_name, table_comment
FROM information_schema.tables
WHERE table_schema = 'smart-ar'

```


## 展示表字段

```sql

SHOW COLUMNS FROM poi_info

```


## 存储过程嵌套循环

两个循环嵌套，外层循环查询所有的pid ， 内层循环根据pid查询id，并维护一个index字段用来更新排序字段。

排序规则：同级按照reginname递增


```sql
-- 删除已有的存储过程
DROP PROCEDURE IF EXISTS update_order_data;




-- 定义更新数据表存储过程
CREATE PROCEDURE update_order_data()
BEGIN
   -- 定义存储过程变量
     DECLARE _pid BIGINT(20);
     DECLARE _id BIGINT(20);
     -- 游标索引
     DECLARE _index BIGINT(20)  DEFAULT 0;      
     -- 变量  1   
     DECLARE  _one int DEFAULT 1 ;
     -- 退出循环标识
     DECLARE stopCur INT DEFAULT 0;
     DECLARE stopCur2 INT DEFAULT 0;
     
     DECLARE c  CURSOR FOR (SELECT parent_id  from  bams_region_copy1 where has_del=0   GROUP BY parent_id ORDER BY  parent_id asc  ) ;
     DECLARE CONTINUE HANDLER FOR NOT FOUND SET stopCur2 = null;
     OPEN c ;
     FETCH c into _pid ;
                -- pid的循环
            WHILE( stopCur2 IS NOT NULL) DO        
                            insert into  p_log SELECT CONCAT('pid=',_pid );
                                BEGIN                                             
                                     -- 定义游标(更新指定部分数据)
                                     DECLARE cur CURSOR FOR (SELECT id  from  bams_region_copy1 where has_del=0 and  parent_id = _pid   ORDER BY  region_name ASC);
                                     -- 定义游标结束,当遍历完成时，将stopCur设置为null ,也可以写成 DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET stopCur = null;
                                     DECLARE CONTINUE HANDLER FOR NOT FOUND SET stopCur = null;
--                                              insert into  p_log SELECT CONCAT('进入=','begin' ) ;
                                     OPEN cur;   
                                     FETCH cur INTO _id;    
--                                                     insert into  p_log SELECT CONCAT('cur=','打开了，id=',_id ) ;
--                                                     insert into  p_log SELECT CONCAT('stopCur=',stopCur ) ;
                                         -- 数据治理循环            
                                             WHILE( stopCur IS NOT NULL) DO            
                                                        insert into  p_log SELECT CONCAT('id=',_id ) ;
                                                     set  _index =  _index + _one ;      
                                                        insert into  p_log SELECT CONCAT('index=',_index );
                                                     UPDATE bams_region_copy1 SET order_num=_index WHERE id = _id ;      
--                                                         insert into  p_log SELECT CONCAT('update=','wanbi' );
                                                     FETCH cur INTO _id;      
                                             END WHILE;   
                                     CLOSE cur;
                                             set stopCur = 0 ;
                                     set _index=0;                                                
                                         
                              END;        
                        FETCH c INTO _pid;             
        END WHILE;    
    CLOSE c ;        
END


-- 执行存储过程
CALL update_order_data;


-- 删除 已有的 存储过程
DROP PROCEDURE  update_order_data;


--
-- show processlist
--
-- kill 56529
--
--
-- SELECT * from p_log
--
-- delete from  p_log


改造为根据regionName 字段首字排序

-- 删除已有的存储过程
DROP PROCEDURE IF EXISTS update_order_data_for_region;




-- 定义更新数据表存储过程
CREATE PROCEDURE update_order_data_for_region()
BEGIN
   -- 定义存储过程变量
     DECLARE _pid BIGINT(20);
   DECLARE _id BIGINT(20);
     -- 游标索引
     DECLARE _index BIGINT(20)  DEFAULT 0;      
     -- 变量  1   
     DECLARE  _one int DEFAULT 1 ;
     -- 退出循环标识
   DECLARE stopCur INT DEFAULT 0;
     DECLARE stopCur2 INT DEFAULT 0;
     
     DECLARE c  CURSOR FOR (SELECT parent_id  from  bams_region where has_del=0   GROUP BY parent_id ORDER BY  parent_id asc  ) ;
     DECLARE CONTINUE HANDLER FOR NOT FOUND SET stopCur2 = null;
     OPEN c ;
     FETCH c into _pid ;
                -- pid的循环
            WHILE( stopCur2 IS NOT NULL) DO        
                            -- insert into  p_log SELECT CONCAT('pid=',_pid );
                                BEGIN                                             
                                     -- 定义游标(更新指定部分数据)
                                     DECLARE cur CURSOR FOR (SELECT id  from  bams_region where has_del=0 and  parent_id = _pid   ORDER BY  CONVERT ( region_name USING gbk ) COLLATE gbk_chinese_ci ASC);
                                     -- 定义游标结束,当遍历完成时，将stopCur设置为null ,也可以写成 DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET stopCur = null;
                                     DECLARE CONTINUE HANDLER FOR NOT FOUND SET stopCur = null;
--                                              insert into  p_log SELECT CONCAT('进入=','begin' ) ;
                                     OPEN cur;   
                                     FETCH cur INTO _id;    
--                                                     insert into  p_log SELECT CONCAT('cur=','打开了，id=',_id ) ;
--                                                     insert into  p_log SELECT CONCAT('stopCur=',stopCur ) ;
                                         -- 数据治理循环            
                                             WHILE( stopCur IS NOT NULL) DO            
                                                    --     insert into  p_log SELECT CONCAT('id=',_id ) ;
                                                     set  _index =  _index + _one ;      
                                                    --     insert into  p_log SELECT CONCAT('index=',_index );
                                                     UPDATE bams_region SET order_num=_index WHERE id = _id ;      
--                                                         insert into  p_log SELECT CONCAT('update=','wanbi' );
                                                     FETCH cur INTO _id;      
                                             END WHILE;   
                                     CLOSE cur;
                                             set stopCur = 0 ;
                                     set _index=0;                                                
                                         
                              END;        
                        FETCH c INTO _pid;             
        END WHILE;    
    CLOSE c ;        
END




-- 执行存储过程
CALL update_order_data_for_region;


-- 删除 已有的 存储过程
DROP PROCEDURE  update_order_data_for_region;



--
-- show processlist
--
-- kill 56529
--
--
-- SELECT * from p_log
--
-- delete from  p_log


-- SELECT region_id , region_name , parent_id ,  order_num   from  bams_region where has_del=0   ORDER BY  parent_id asc  , order_num asc

```


## 存储过程根据主表更新子表

```sql




-- 删除已有的存储过程
DROP PROCEDURE IF EXISTS update_data;
-- 定义更新数据表存储过程
CREATE PROCEDURE update_data()
BEGIN
   -- 定义存储过程变量
   DECLARE _id BIGINT(20);
   DECLARE stopCur INT DEFAULT 0;
   -- 定义游标(更新指定部分数据)
   DECLARE cur CURSOR FOR (SELECT id FROM hkcy_label_camera );
   -- 定义游标结束,当遍历完成时，将stopCur设置为null ,也可以写成 DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET stopCur = null;
   DECLARE CONTINUE HANDLER FOR NOT FOUND SET stopCur = null;
   
   -- 开游标
   OPEN cur;
   -- 游标向下走一步，将查询出来的两个值赋给定义的两个变量
   FETCH cur INTO _id;
      -- 循环体
      WHILE( stopCur IS NOT NULL) DO
      -- 更新对应关系表数据
      UPDATE hkcy_label_camera SET draw_type=1 WHERE id = _id ;
      -- 游标向下走一步
      FETCH cur INTO _id;      
      END WHILE;
   -- 关闭游标
   CLOSE cur;
END
-- 执行存储过程
CALL update_data;
-- 删除 已有的 存储过程
DROP PROCEDURE  update_data;


```