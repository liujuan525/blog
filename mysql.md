


- 添加外键 
```
    ALTER TABLE `social_login_correlation` ADD CONSTRAINT `customer_entity_entity_id_social_login_correlation_customer_id` FOREIGN KEY (`customer_id`) REFERENCES `customer_entity` (`entity_id`) ON DELETE CASCADE ;
```

- 添加索引
```
ALTER TABLE `social_login_correlation`
ADD INDEX `SOCIAL_LOGIN_CORRELATION_CUSTOMER_ID` (`customer_id`) USING BTREE ;
```

- 添加无符号
```
ALTER TABLE `social_login_correlation`
MODIFY COLUMN `customer_id`  int(11) UNSIGNED NOT NULL COMMENT 'Customer Id' AFTER `id`;
```












