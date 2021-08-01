#索引

-----------------------------------------------------------


mysql 索引是否失效问题

    版本5.7 
    tablein`id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    `logic_id` int(11) unsigned NOT NULL COMMENT '机器人的逻辑id',
    `wechat_username` varchar(32) CHARACTER SET utf8 NOT NULL DEFAULT '' COMMENT '微信id',
    `specialized` tinyint(3) unsigned DEFAULT NULL COMMENT '角色1 小助手 2 Master 3 Slave 4 个人号 5 苦工号 6 直播号 8 销售号 10潜行号',
    `status` tinyint(3) unsigned NOT NULL COMMENT '状态 0 可用, 1 不可用',
    `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (`id`),
    KEY `idx_logic_id` (`logic_id`),
    KEY `idx_username` (`wechat_username`),
    KEY `idx_specialized` (`specialized`)
    
    ad_device_status_info	0	PRIMARY	        1	id	        A	    404356	NULL	NULL		BTREE		
    ad_device_status_info	1	idx_logic_id	1	logic_id	A	    1598	NULL	NULL		BTREE		
    ad_device_status_info	1	idx_username	1	wechat_username	A	3546	NULL	NULL		BTREE		
    ad_device_status_info	1	idx_specialized	1	specialized	A	    36	NULL	NULL	    YES	BTREE		

    1.
    select * from ad_device_status_info  where   logic_id > 30000;
    >3000 不走索引; <1000 走索引 另外测试了几个值 并不固定；以前是按区分数占总数量的30% 现在考虑更多因素 这个值不固定了
    != 不走索引 
    select logic_id ....... 无论是什么范围（> < !=...) 都走索引 因为不用回表 但是需要回表 就看范围大小了
    比如 select * from  --- where id != < > 也走索引 

    2.
	select * from ad_device_status_info  where  logic_id < 1000 and wechat_username like "%sfd%";
    虽然后面的wechat_username like "%fjwe% 会失效 但是前面的logic_id生效了
    
    3.带函数 类型转换的不走索引 例如 id +1 = 10; cast(id)
    
     