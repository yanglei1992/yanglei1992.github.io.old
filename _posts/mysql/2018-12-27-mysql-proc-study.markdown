---
layout: post
title:  "mysql储存过程"
date:   2018-12-27 19:21:39 +0800
categories: mysql
tag: mysql
---

* content
{:toc}


存储过程(Stored Procedure)：

　　一组可编程的函数，是为了完成特定功能的SQL语句集，经编译创建并保存在数据库中，用户可通过指定存储过程的名字并给定参数(需要时)来调用执行。

1. 创建与删除储存过程
2. 调用储存过程
3. 存储过程体
4. 语句块标签
5. 储存过程的参数
6. 我的使用
7. 参考资料

----------


## 1. 创建与删除储存过程  ##



- 删除存储过程：


>     DROP PROCEDURE IF EXISTS proc_name

- 创建存储过程：以下是来自官方API的定义

	
> 
>     CREATE PROCEDURE sp_name ([proc_parameter[,...]]) 
>      [characteristic ...] routine_body
>      
>     proc_parameter:
>     [ IN | OUT | INOUT ] param_name type
>     
>     characteristic:
>     LANGUAGE SQL
>       | [NOT] DETERMINISTIC
>       | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
>       | SQL SECURITY { DEFINER | INVOKER }
>       | COMMENT 'string'
>      
>     routine_body:
>     Valid SQL procedure statement or statements
>     [begin_label:] BEGIN
>     　　[statement_list]
>     　　　　……
>     END [end_label]
    

## 2. 调用储存过程 ##


## 3. 存储过程体 ##


## 4. 语句块标签 ##


## 5. 储存过程的参数 ##


## 6. 我的使用 ##

  >   -- 生成图像
>     DROP PROCEDURE IF EXISTS `proc_add_image`;
>     DELIMITER ;;
>     CREATE DEFINER=`pcs`@`localhost` PROCEDURE `proc_add_image`(IN clro_id_in int, IN insert_time_in VARCHAR(50))
>     BEGIN
>     -- Routine body goes here...
>     	DECLARE devi_puid_tmp varchar(50);
>     	DECLARE clde_channel_num_tmp int(11);
>     	DECLARE image_id_tmp int(11);  -- 生成图片的ID
>     
>     -- 遍历数据结束标志
>     DECLARE done INT DEFAULT FALSE;
>     -- 游标
>     DECLARE clde_data CURSOR FOR select devi_puid,clde_channel_num from t_classroom_device where clro_id_in = t_classroom_device.clro_id;
>     -- 将结束标志绑定到游标
>     DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
>     -- 打开游标
>     OPEN  clde_data; 
>     -- 遍历
>     read_loop: LOOP
>     -- 取值 取多个字段
>     FETCH  NEXT from clde_data INTO devi_puid_tmp, clde_channel_num_tmp;
>     IF done THEN
>     LEAVE read_loop;
>      END IF;
>     -- 你自己想做的操作
>       select devi_puid_tmp;
>       select clde_channel_num_tmp;
>     INSERT INTO `t_image` (`insert_time`, `update_time`, `recycle_sign`, `area_code`, `devi_puid`, `cach_num`, `imag_shot_time`, `imag_source`, `imag_format_type`, `imag_storage_path`, `imag_width`, `imag_height`, `imag_file_size`) VALUES 
>       (NULL, NULL, 0, NULL, devi_puid_tmp, clde_channel_num_tmp, insert_time_in, NULL, NULL, 'img/smallDia.png', NULL, NULL, NULL);
>     				
>       select max(id) from t_image into image_id_tmp;
>     
>       call proc_add_image_people_count(image_id_tmp);
>     
>     END LOOP;
>     CLOSE clde_data;
>     
>     END
>     ;;
>     DELIMITER ;
>     
>     
>     -- 生成图像数据，一张图片两个数据
>     DROP PROCEDURE IF EXISTS `proc_add_image_people_count`;
>     DELIMITER ;;
>     CREATE DEFINER=`pcs`@`localhost` PROCEDURE `proc_add_image_people_count`(IN imag_id_in int)
>     BEGIN
>     
>     		DECLARE all_student int(11) default 0;
>     		DECLARE pre_student int(11) default 0;
>     -- 实到人数(至少到50人)
>     		select (RAND()*20 + 50) into all_student;
>     -- 前排人数
>     		select RAND()*10 into pre_student;
>     		INSERT INTO `t_image_people_count` (`insert_time`, `update_time`, `recycle_sign`, `area_code`, `imag_id`, `imde_id`, `dezo_polygons`, `ipco_type`, `imag_people_count`) VALUES
>     				(NULL, NULL, 0, 'kedacom', imag_id_in, imag_id_in, NULL, 1, all_student),
>     				(NULL, NULL, 0, 'kedacom', imag_id_in, imag_id_in, NULL, 2, pre_student);
>     
>     
>     END
>     ;;
>     DELIMITER ;
>     
>     -- 生成一节课的图片数据
>     DROP PROCEDURE IF EXISTS `proc_add_one_course_image`;
>     DELIMITER ;;
>     CREATE DEFINER=`pcs`@`localhost` PROCEDURE `proc_add_one_course_image`(IN clro_id_in int, IN cour_begin_time_in VARCHAR(50))
>     BEGIN
>     		DECLARE cour_begin_time_tmp VARCHAR(50);
>     		DECLARE i int;
>     		set i=0;
>     		while i<12 do
>     				-- 每四分钟一张，生成12张图片
>     				select FROM_UNIXTIME(UNIX_TIMESTAMP(cour_begin_time_in) + 240*i,'%Y-%m-%d %H:%i:%s') into cour_begin_time_tmp;
>     				call proc_add_image(clro_id_in,cour_begin_time_tmp);
>     				set i=i+1;
>     		end while;
>     
>     END
>     ;;
>     DELIMITER ;
    
    

## 7、参考资料 ##

[MySQL 5.1参考手册](http://tool.oschina.net/apidocs/apidoc?api=mysql-5.1-zh)

[https://www.cnblogs.com/geaozhang/p/6797357.html](https://www.cnblogs.com/geaozhang/p/6797357.html)

