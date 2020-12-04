
### 相关命令

- `php bin/magento`  查看所有命令
- `php bin/magento --version`  查看magento版本
- `php bin/magento cache:clean`  清除缓存
- `php bin/magento setup:upgrade`  升级数据库表结构与数据
- `php bin/magento setup:di:compile`  生成自动类文件
- `php bin/magento deploy:mode:show`  查看当前模式
- `php bin/magento deploy:mode:set developer` 设为开发模式
- `php bin/magento index:reindex`  刷新索引
- `php bin/magento cache:status`  查看缓存是否开启的状态
- `php bin/magento cache:clean`  清空缓存
- `php bin/magento cache:flush`  刷新缓存
- `php bin/magento sampledata:deploy`  安装Sample Data 数据
- `php bin/magento sampledata:remove`  移除Sample Data 数据
- `php bin/magento d1m:regenerate:producturl`  重新生成产品链接
- `php bin/magento setup:static-content:deploy -f --theme=Magento/backend zh_Hans_CN`
- `php bin/magento setup:static-content:deploy -f --theme=Magento/backend en_US`
- `php bin/magento setup:static-content:deploy -f`
- `php bin/magento setup:static-content:deploy -f zh_Hans_CN`  如果未生成相关 css/js 文件，则在管理员下运行git命令窗口运行



### 相关数据表

- 库存相关数据表：
   - inventory_source_item 库存表
   - inventory_reservation 库存占用表
- 更改订单前缀：
   - sales_sequence_meta
   - sales_sequence_profile
- token 表 : 
   - oauth_token
- 清除库存相关数据表：
   - inventory_reservation  预约库存
   - cataloginventory_stock_item 商品库存
- store_website
- url_rewrite 路由重写 (url_rewrite 与 cms_page 数据表可以更改网站 title)
- core_config_data 后台配置信息表
- quote_address 购物车地址信息
- quote_item 购物车产品信息
- quote 购物车
- quote_payment 购物车支付
- quote_shipping_rate 运费
- customer_address_entity 用户地址信息
- customer_entity 用户信息表
- directory_country_district 县/区信息表
- directory_country_city 市信息表
- directory_country_region 省信息表
- catalog_product_entity 产品信息表
- sales_order 销售订单（包含发票信息）
- sales_order_item 订单详情
- setup_module 模块版本信息表
- review 评论
- review_detail 评论详情
- sms_code 发送手机验证码
- sms_log 发送验证码日志
- sms_template 发送短信模板
- sales_coupon 订单使用优惠券
- salesrule 优惠券信息
- rating 评分
- catalog_category_entity 分类表



### 发货方式

- flat rate - 统一价格/均一税率
- freee shipping - 免运费
- table rates - 运费表/税率表
- magento shipping - magento 发货
- ups - 联合包裹
- usps - 美国邮政
- fedex - 联邦快递
- dhl - 中外运敦豪/敦豪中外运
- SF Express - 顺丰速运



### 支付方式

- paypal - 贝宝（全球最大的在线支付平台）
- klarna - Klarna Bank AB（科拉纳公司），通常被称为Klarna，是一家瑞典银行，提供在线金融服务，如在线店面支付解决方案，直接支付，购买后付款等
- Zero Subtotal Checkout -  零元结账零金额结帐
- cash on delivery payment - 货到付款
- bank transfer payment - 银行转账付款
- check/money order - 支票/汇票
- purchase order - 订购单、采购单
- cybersource - Cyber​​Source是一家电子商务信用卡支付系统管理公司。客户处理在线支付，简化在线欺诈管理并简化支付安全性
- worldpay - 世界支付
- eway
- authorize.net
- authorize.net direct post（deprecated）
- wexin mobile payment(native) - 微信移动支付（本地）
- wexin pc payment(native) - 微信pc支付（本地）
- alipay payment method - 支付宝支付
- applet payment - 小程序支付


### 相关知识点

- `event.xml` 文件中的 `event` 事件，如果以 `controller_action_predispatch_ `开头，则控制器自动触发监听事件
- `sales_order` 表添加字段属性，一定在 `quote` 表也添加，因为信息会先保存在 `quote` 表中，然后转存到 `sales_order` 表
- `setup` 模块：
   - `upgradeSchema.php` 主要是添加/更改表结构的
   - `upgradeData.php` 主要是添加/更改 `eav` 属性的

- 底层锁定订单：在底层 `magento/module-checout/controller/onepage/success` 中修改
```
    $session->setLastSuccessQuoteId('250');
    $session->setLastQuoteId('250');
    $session->setLastOrderId('8405');
    $session->setLastRealOrder('TEST000000201');
```

- 取消 magento 后台强制修改密码的设置：
`stores` - `configuration` - `admin` - `security` - `password change` (将 forced 改成 recommended)

- 后台设置密码校验：
`configuration` - `customers` - `customer configuration` - `passwords options`

- 添加到主表的字段必须添加到  `eav_attribute` 表中，添加的 `eav_attribute` 中的字段不一定需要添加到主表

- 获取当前域名的url ： `$this->storeManager->getstore()->getBaseUrl();`

- 获取结算的总价格：`$quote->getGrandTotal();（包含运费，优惠券等）`

- 获取服务器时间：
`$this->timezone->date($_order->getCreatedAt(), null)->format('Y.m.d H:i:s');`

- magento2 配置最大加入购物车数量：
   - 在 `admin` - `catalog` - `product` 商品里点击 `Quantity` 下的 `Advanced Inventory` 设置单个商品
   - 在 `admin` - `store` - `configuration` - `catalog` - `inventory` - `product stock option` 中设置默认值
   - 
- 在后台 `Marketing` - `cart price rules` 可以配置优惠券信息、购物车价格规则、选择了不同支付渠道，给出不同的优惠结算价格

- 编辑购物车逻辑：先把原来的 `item_id` 删除，然后再重新增加一条（无法在原来的 `item_id` 基础上做修改，因为有可能不同的属性价格不一致）

- 在 magento 后台 - `stores` - `order status` 可以配置 订单状态

- 在代码 `app/etc/config.php` 可以开启模块，但是尽量使用命令行开启

- 在后台增加 magento 产品属性值：`admin` - `stores` - `product` 中设置

- 在后台清除缓存：`admin` - `stores` - `cache management`

- 更改产品属性的编辑框：
`admin` - `stores` - `product` - `properties` - `catelog input type for store owner` (`text editor` 可以显示编辑域 `text area`)

- 代码 `Setup` 目录中，添加属性的时候，如果想显示文本域的编辑框，在编写升级脚本时添加下面的代码即可
```
    'wysiwyg_enabled' => true,
    'is_html_allowed_on_front' => true,
```

- 获取media目录路径：`$url = $this->getUrl(\Magento\Framework\UrlInterface::URL_TYPE_MEDIA);`

- 在设置商品的库存和可售库存都为0的时候：
   - 如果设置 `Quantity` 为0，但是 `Default Stock` 不为0时
       - 在 `Quantity` 值下方点击 `Advanced Inventory`，勾选 `Manage Stock` 下方的 `Use Config Setting`

- xml文件中，在前端直接输出产品属性值：
```
    $this->helper('Magento\Catalog\Helper\Output')->productAttribute($block->getProduct(), 
    $block->getProduct()->getPdpProductDetail(), 'pdp_product_detail');
```




























