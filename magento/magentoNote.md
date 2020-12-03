
### Magento2 初始化 

1、 初始化 sql
```
    SET FOREIGN_KEY_CHECKS=0;
    UPDATE `store` SET store_id = 0 WHERE code='admin';
    UPDATE `store_group` SET group_id = 0 WHERE name='Default';
    UPDATE `store_website` SET website_id = 0 WHERE code='admin';
    UPDATE `customer_group` SET customer_group_id = 0 WHERE customer_group_code='NOT LOGGED IN';
    SET FOREIGN_KEY_CHECKS=1;
    
```
2、 修改 path 网址
- **core_config_data** 数据表 搜索 path=url 更改大概位置为第2、3、795、797 处的value值--更改为为配置的虚拟主机地址


### windows 路径兼容

- 2.3.0 在windows 下，文件处理BUG:在 vendor/magento/framework/View/Element/Template/File/Validator.php 的 138 行，将：
- `$realPath = $this->fileDriver->getRealPath($path);`
- 更改为：
- `$realPath = str_replace('\\', '/', $this->fileDriver->getRealPath($path));`


### 其他问题

- 后台登录如果一直跳转到登录页，查看数据表配置项：`core_config_data` 中 web/cookie/cookie_domain 的后缀是否与你当前配置的网站的后缀一致

- 报错下面信息，可以尝试运行：`php bin/magento setup:upgrade`
```
exception(s):
    Exception #0 (Magento\Framework\Exception\ValidatorException):
    Invalid template file: 'D:/project/sw/vendor/magento/module-backend/view/adminhtml/templates/page/js/require_js.phtml' in module: 'Magento_Backend' block's name: 'require.js'
```

- 报错下面信息，在后台修改当前时区或者在数据表 `core_config_data` 中 修改 general/locale/timezone 的值
``` 
Exception #0 (Exception): DateTimeZone::__construct(): Unknown or bad timezone ()
```

- 报错下面信息，解决方法：pub/index.php 注释掉：32-62行
```
Fatal error: Uncaught GeoIp2\Exception\AddressNotFoundException: The address 127.0.0.1 is not in the database. in phar://D:/project/breitling/pub/geoip2.phar/src/Database/Reader.php:244 Stack trace: #0 phar://D:/project/breitling/pub/geoip2.phar/src/Database/Reader.php(215): GeoIp2\Database\Reader->getRecord('City', 'City', '127.0.0.1') #1 phar://D:/project/breitling/pub/geoip2.phar/src/Database/Reader.php(71): GeoIp2\Database\Reader->modelFor('City', 'City', '127.0.0.1') #2 D:\project\breitling\pub\index.php(53): GeoIp2\Database\Reader->city('127.0.0.1') #3 {main} thrown in phar://D:/project/breitling/pub/geoip2.phar/src/Database/Reader.php on line 244
```

- 如果加载 js、css 文件失败，在下面文件的 `isPathInDirectories` 方法中
D:\project\magento2.3\vendor\magento\framework\View\Element\Template\File\Validator.php
增加：
```
$realPath = str_replace('\\', '/', $this->fileDriver->getRealPath($path));
```

- 如果代码添加商品属性，在后台不能点击编辑，查看添加商品的时候是否添加 `attribute_set_id`，添加即可解决

- 如果添加或者更新商品的时候在后台不是所有的店铺都可以查看，在运行脚本的时候设置店铺的 `store_id`

- 默认添加所有的产品信息后，后台所有店铺都可见。可以在脚本运行代码之前添加下面的代码：
```
$state->setAreaCode('frontend');
$storeManager = $objectManager->get('Magento\Store\Model\StoreManagerInterface');
$storeManager->setCurrentStore(0);
```

- 卸载集成环境，重新配置项目的时候，报错init....等信息，查看 php.ini 中是否开启 init 扩展
- 配置域名后，访问域名重定向，查看数据表 `core_config_data` 中的域名链接是否为完整路径(如 http://)

- 本地环境后台js、image等路径问题
   - 首先修改下面的代码
```
D:\project\breitling\app\etc\di.xml
D:\project\breitling\vendor\magento\magento2-base\app\etc\di.xml
D:\project\breitling\vendor\magento\module-developer\etc\di.xml
将<item name="view_preprocessed" xsi:type="object">Magento\Framework\App\View\Asset\MaterializationStrategy\Symlink</item>
<item name="default" xsi:type="object">Magento\Framework\App\View\Asset\MaterializationStrategy\Copy</item>
里面的 Symlink 改为 Copy
```
   - 然后执行下述命令
```
php bin/magento cache:clean
    php bin/magento cache:flush
    remove everything(not .htacess) from pub, var, generated
    php bin/magento setup:upgrade
    php bin/magento setup:static-content:deploy -f
    php bin/magento setup:static-content:deploy en_AU --exclude-theme Magento/luma --exclude-theme Magento/blank -f
```



### 常用代码

```

// 获取objectManager 
$objectManager = \Magento\Framework\App\ObjectManager::getInstance();

$storeManager = $objectManager->get('\Magento\Store\Model\StoreManagerInterface');

// 获取产品信息
$product = $objectManager->get(\Magento\Catalog\Api\ProductRepositoryInterface::class)->getById($productId);

// DataObject是所有Model的基类 
$row = new \Magento\Framework\DataObject();
$row->setData($key, $value);
$row->getData($key);
$row->hasData($key);
$row->unsetData();
$row->toXml();
$row->toJson();
$row->debug();

// 购物车相关类引用 获购物车相关信息 
$cart = \Magento\Checkout\Model\Cart::getInstance;
$this->quote = $cart->getQuote();
// e.q
$items = $this->quote->getAllVisibleItems();
$totalItems = $this->quote->getItemsCount();
$grandTotal = $this->quote->getGrandTotal();
$shippingAddress = $this->quote->getShippingAddress();

// cache save 
$this->cache = $context->getCache();
$this->cache->save(\Zend_Json::encode($data), self::CACHE_PREFIX . $cacheKey, [], $lifeTime);

// cache load 
$jsonStr = $this->cache->load(self::CACHE_PREFIX . $cacheKey);
if (strlen($jsonStr)) {
    $this->cachedData = \Zend_Json::decode($jsonStr);
}

// Session 
$session = $objectManager->get('Magento\Framework\Session\Storage');
$session->setXxxx($value);
$session->getXxxx();
$session->hasXxxx();
$session->unsXxxx();

// 结果集过滤  参数 
/*
["from" => $fromValue, "to" => $toValue]
["eq" => $equalValue]
["neq" => $notEqualValue]
["like" => $likeValue]
["in" => [$inValues]]
["nin" => [$notInValues]]
["notnull" => $valueIsNotNull]
["null" => $valueIsNull]
["moreq" => $moreOrEqualValue]
["gt" => $greaterValue]
["lt" => $lessValue]
["gteq" => $greaterOrEqualValue]    
["lteq" => $lessOrEqualValue]
["finset" => $valueInSet]
*/
$collection->addFieldToFilter('status', array('eq' => 'pending'));

// 判断是否首页 
// in action
if ($this->_request->getRouteName() == 'cms' && $this->_request->getActionName() == 'index') {
    // is home
}

/** Registry 用于内部传递临时值 */
/* @var \Magento\Framework\Registry $coreRegistry */
$coreRegistry = $this->_objectManager->get('Magento\Framework\Registry');
/** 存储值 */
$coreRegistry->register('current_category', $category);
/** 提取值 */
$coreRegistry->registry('current_category');

/** 获取当前店铺对象 */
$store = $objectManager->get('Magento\Store\Model\StoreManagerInterface')->getStore();

/** 提示信息 */
// \Magento\Framework\App\Action\Action::execute()
$this->messageManager->addSuccessMessage(__('You deleted the event.'));
$this->messageManager->addErrorMessage(__('You deleted the event.'));
return $this->_redirect($returnUrl);
log(support_report . log);
\Magento\Framework\App\ObjectManager::getInstance()
    ->get('\Psr\Log\LoggerInterface')->addCritical('notice message', [
        'order_id' => $order_id,
        'item_id'  => $item_id
        // ...
    ]);
    
/** 获取当前页面 URL */
$currentUrl = $objectManager->get('Magento\Framework\UrlInterface')->getCurrentUrl();

/** 获取指定路由 URL */
$url = $objectManager->get('Magento\Store\Model\StoreManagerInterface')->getStore()->getUrl('xxx/xxx/xxx');

/** block里的URL */
// 首页地址
$block->getBaseUrl();

/** 指定 route 地址 */
$block->getUrl('[module]/[controller]/[action]');

/** 指定的静态文件地址 */
$block->getViewFileUrl('Magento_Checkout::cvv.png');
$block->getViewFileUrl('images/loader-2.gif');

/** 获取所有website的地址 */
$storeManager = $objectManager->get('Magento\Store\Model\StoreManagerInterface');
foreach ($storeManager->getWebsites() as $website) {
    $store_id = $storeManager->getGroup($website->getDefaultGroupId())->getDefaultStoreId();
    $storeManager->getStore($store_id)->getBaseUrl();
}

/** 获取module下的文件 */
/** @var \Magento\Framework\Module\Dir\Reader $reader */
$reader = $objectManager->get('Magento\Framework\Module\Dir\Reader');
$reader->getModuleDir('etc', 'Infinity_Project') . '/di.xml';

/** 获取资源路径 */
/* @var \Magento\Framework\View\Asset\Repository $asset */
$asset = $this->_objectManager->get('\Magento\Framework\View\Asset\Repository');
$asset->createAsset('Vendor_Module::js/script.js')->getPath();

/** 文件操作 */
/* @var \Magento\Framework\Filesystem $fileSystem */
$fileSystem = $this->_objectManager->get('Magento\Framework\Filesystem');
$fileSystem->getDirectoryWrite('tmp')->copyFile($from, $to);
$fileSystem->getDirectoryWrite('tmp')->delete($file);
$fileSystem->getDirectoryWrite('tmp')->create($file);

/** 文件上传 */
// 上传到media
$request = $this->getRequest();
if ($request->isPost()) {
    try {
        /* @var $uploader \Magento\MediaStorage\Model\File\Uploader */
        $uploader = $this->_objectManager->create(
            'Magento\MediaStorage\Model\File\Uploader',
            ['fileId' => 'personal_photos']
        );
        /* @var $filesystem \Magento\Framework\Filesystem */
        $filesystem = $this->_objectManager->get('Magento\Framework\Filesystem');
        $dir        = $filesystem->getDirectoryRead(\Magento\Framework\App\Filesystem\DirectoryList::UPLOAD)->getAbsolutePath();
        $fileName   = time() . '.' . $uploader->getFileExtension();
        $uploader->save($dir, $fileName);
        $fileUrl = 'pub/media/upload/' . $fileName;
    } catch (Exception $e) {
        // 未上传
    }
}

/** 上传到tmp */
/* @var \Magento\Framework\App\Filesystem\DirectoryList $directory */
$directory = $this->_objectManager->get('Magento\Framework\App\Filesystem\DirectoryList');
/* @var \Magento\Framework\File\Uploader $uploader */
$uploader = $this->_objectManager->create('Magento\Framework\File\Uploader', array('fileId' => 'file1'));
$uploader->setAllowedExtensions(array('csv'));
$uploader->setAllowRenameFiles(true);
$uploader->setFilesDispersion(true);
$result = $uploader->save($directory->getPath($directory::TMP));
$directory->getPath($directory::TMP) . $result['file'];

/** 产品缩略图 */
$imageHelper  = $objectManager->get('Magento\Catalog\Helper\Image');
$productImage = $imageHelper->init($product, 'category_page_list')
    ->constrainOnly(FALSE)
    ->keepAspectRatio(TRUE)
    ->keepFrame(FALSE)
    ->resize(400)
    ->getUrl();
    
/** 缩略图 */
$imageFactory = $objectManager->get('Magento\Framework\Image\Factory');
$imageAdapter = $imageFactory->create($path);
$imageAdapter->resize($width, $height);
$imageAdapter->save($savePath);

/** 产品属性 */
/* @var \Magento\Catalog\Model\ProductRepository $product */
$product = $objectManager->create('Magento\Catalog\Model\ProductRepository')->getById($id);

/** print price */
/* @var \Magento\Framework\Pricing\PriceCurrencyInterface $priceCurrency */
$priceCurrency = $objectManager->get('Magento\Framework\Pricing\PriceCurrencyInterface');
$priceCurrency->format($product->getData('price'));

/** print option value */
$product->getResource()->getAttribute('color')->getDefaultFrontendLabel();
$product->getAttributeText('color');

/** all options */
$this->helper('Magento\Catalog\Helper\Output')->productAttribute($product, $product->getData('color'), 'color');
$product->getResource()->getAttribute('color')->getSource()->getAllOptions();

/** save attribute */
$product->setWeight(1.99)->getResource()->saveAttribute($product, 'weight');

/** 库存 */
$stockItem  = $this->stockRegistry->getStockItem($productId, $product->getStore()->getWebsiteId());
$minimumQty = $stockItem->getMinSaleQty();

/** Configurable Product 获取父级产品ID */
$parentIds = $this->objectManager->get('Magento\ConfigurableProduct\Model\Product\Type\Configurable')
    ->getParentIdsByChild($productId);
$parentId  = array_shift($parentIds);

/** Configurable Product 获取子级产品ID */
$childIds = $this->objectManager->get('Magento\ConfigurableProduct\Model\Product\Type\Configurable')
    ->getChildrenIds($parentId);
    
/** 获取Configurable Product的子产品Collection */
$collection = $this->objectManager->get('Magento\ConfigurableProduct\Model\Product\Type\Configurable')
    ->getUsedProducts($product);
    
/** 判断是否Configurable Product */
if ($_product->getTypeId() == \Magento\ConfigurableProduct\Model\Product\Type\Configurable::TYPE_CODE);

/** 购物车中的所有产品 */
/* @var \Magento\Checkout\Model\Session $checkoutSession */
$checkoutSession = $objectManager->get('Magento\Checkout\Model\Session');
foreach ($checkoutSession->getQuote()->getItems() as $item) {
    /* @var \Magento\Quote\Model\Quote\Item $item */
    echo $item->getProduct()->getName() . '<br/>';
}

/** 获取类型配置 */
$eavConfig->getAttribute('catalog_product', 'price');
$eavConfig->getEntityType('catalog_product');

/** 获取 EAV 属性所有可选项 */
/* @var $objectManager \Magento\Framework\App\ObjectManager */
$eavConfig = $objectManager->get('\Magento\Eav\Model\Config');
$options   = $eavConfig->getAttribute('[entity_type]', '[attribute_code]')
    ->getFrontend()->getSelectOptions();
    
// 或者
/* @var $objectManager \Magento\Framework\App\ObjectManager */
$options = $objectManager->create('Magento\Eav\Model\Attribute')
    ->load('[attribute_code]', 'attribute_code')->getSource()
    ->getAllOptions(false);

/** 获取config.xml与system.xml里的参数 */
$this->_scopeConfig = $this->_objectManager->create('Magento\Framework\App\Config\ScopeConfigInterface');

/** 语言代码 */
$this->_scopeConfig->getValue('general/locale/code', \Magento\Store\Model\ScopeInterface::SCOPE_STORE, $storeId);

/** 客户唯一属性验证 */
if ($customer instanceof \Magento\Customer\Model\Customer) {
    /* @var \Magento\Customer\Model\Attribute $attribute */
    foreach ($customer->getAttributes() as $attribute) {
        if ($attribute->getIsUnique()) {
            if (!$attribute->getEntity()->checkAttributeUniqueValue($attribute, $customer)) {
                $label = $attribute->getFrontend()->getLabel();
                throw new \Magento\Framework\Exception\LocalizedException(
                    __('The value of attribute "%1" must be unique.', $label)
                );
            }
        }
    }
}

/** 读取design view.xml */
/*
<vars module = "Vendor_Module" >
    <var name = "var1" > value1</var>
</vars >
*/
/* @var \Magento\Framework\Config\View $viewConfig */
$viewConfig = $objectManager->get('Magento\Framework\Config\View');
$viewConfig->getVarValue('Vendor_Module', 'var1');

/** 获取邮件模板 */
// 虽然叫邮件模板，但也可以用于需要后台编辑模板的程序
// template id, 通常在email_templates.xml定义。如果是在后台加的email template，需要换成template的记录ID
$identifier = 'contact_email_email_template';
/* @var \Magento\Framework\Mail\TemplateInterface $templateFactory */
$templateFactory = $this->_objectManager->create(
    'Magento\Framework\Mail\TemplateInterface',
    ['data' => ['template_id' => $identifier]]
);

/** 模板变量，取决于phtml或后台email template的内容 */
$dataObject = new \Magento\Framework\DataObject();
$dataObject->setData('name', 'william');

/** 决定模板变量取值区域，例如像{{layout}}这样的标签，如果取不到值可以试试把area设为frontend */
$templateFactory->setOptions([
    'area'  => \Magento\Backend\App\Area\FrontNameResolver::AREA_CODE,
    'store' => \Magento\Store\Model\Store::DEFAULT_STORE_ID
]);
$templateFactory->setVars(['data' => $dataObject]);
return $templateFactory->processTemplate();

/** 内容返回 */
$this->resultFactory = $this->_objectManager->create('Magento\Framework\Controller\Result\RawFactory');
/* @var \Magento\Framework\Controller\Result\Raw $result */
$result = $this->resultFactory->create();
$result->setContents('hello world');
return $result;

$this->resultFactory = $this->_objectManager->create('Magento\Framework\Controller\Result\JsonFactory');
/* @var \Magento\Framework\Controller\Result\Json $result */
$result = $this->resultFactory->create();
$result->setData(['message' => 'hellog world']);
return $result;

/** HTTP文件 */
$this->_fileFactory = $this->_objectManager->create('Magento\Framework\App\Response\Http\FileFactory');
$this->_fileFactory->create(
    'invoice' . $date . '.pdf',
    $pdf->render(),
    DirectoryList::VAR_DIR,
    'application/pdf'
);

/** 切换货币与语言 */
$currencyCode = 'GBP';
/* @var \Magento\Store\Model\Store $store */
$store = $this->_objectManager->get('Magento\Store\Model\Store');
$store->setCurrentCurrencyCode($currencyCode);

$storeCode = 'uk';
/* @var \Magento\Store\Api\StoreRepositoryInterface $storeRepository */
$storeRepository = $this->_objectManager->get('Magento\Store\Api\StoreRepositoryInterface');
/* @var \Magento\Store\Model\StoreManagerInterface $storeManager */
$storeManager = $this->_objectManager->get('Magento\Store\Model\StoreManagerInterface');
/* @var \Magento\Store\Api\StoreCookieManagerInterface $storeCookieManager */
$storeCookieManager = $this->_objectManager->get('Magento\Store\Api\StoreCookieManagerInterface');
/* @var \Magento\Framework\App\Http\Context $httpContext */
$httpContext      = $this->_objectManager->get('Magento\Framework\App\Http\Context');
$defaultStoreView = $storeManager->getDefaultStoreView();
$store            = $storeRepository->getActiveStoreByCode($storeCode);
$httpContext->setValue(\Magento\Store\Model\Store::ENTITY, $store->getCode(), $defaultStoreView->getCode());
$storeCookieManager->setStoreCookie($store);

$this->getResponse()->setRedirect($this->_redirect->getRedirectUrl());

/** Profiler */
\Magento\Framework\Profiler::start(
    'CONFIGURABLE:' . __METHOD__,
    ['group' => 'CONFIGURABLE', 'method' => __METHOD__]
);
\Magento\Framework\Profiler::stop('CONFIGURABLE:' . __METHOD__);

/** HTML表单元素 */
// 日历控件
$block->getLayout()->createBlock('Magento\Framework\View\Element\Html\Date')
    ->setName('date')
    ->setId('date')
    ->setClass('date')
    ->setDateFormat('M/d/yy')
    ->setImage($block->getViewFileUrl('Magento_Theme::calendar.png'))
    ->setExtraParams('data-validate="{required:true}"')
    ->toHtml();

/** select控件 */
$block->getLayout()->createBlock('Magento\Framework\View\Element\Html\Select')
    ->setName('sel1')
    ->setId('sel1')
    ->setClass('select')
    ->addOption('value', 'label')
    ->setValue($default)
    ->setExtraParams('data-validate="{required:true}"')
    ->toHtml();

/**
 * 查找文件
 * find / -name 'flag.frm'
 */

//\Magento\Sales\Model\OrderFactory $orderFactory
/**@var $order \Magento\Sales\Model\Order */
$order = $this->_orderFactory->create()->loadByIncrementId($params['order_bn']);

/**
 * @var \Magento\Sales\Model\ResourceModel\Order\CollectionFactory
 */
$this->_relatedOrders = $this->_orderCollectionFactory->create()->addFieldToSelect(
    '*'
)->addFieldToFilter(
    'customer_id',
    (int)$this->_customerSession->getCustomerId()
)->addFieldToFilter(
    'status',
    ['in' => $this->_orderConfig->getVisibleOnFrontStatuses()]
)->setOrder(
    'created_at',
    'desc'
);




```




































