File structure 
✅ - Required Files for CRUD Operations

```bash
app/code/LandingPage
└── Form
    ├── Api
    │   ├── Data
    │   │   └── FormDataInterface.php ✅
    │   └── FormDataRepositoryInterface.php ✅
    ├── Block
    │   └── Index.php
    ├── Controller
    │   └── Index
    │       ├── Index.php 💾
    │       └── Post.php ✅
    ├── Helper
    │   └── Data.php (isModuleEnabled())
    ├── Model
    │   ├── FormData.php ✅
    │   ├── FormDataRepository.php ✅
    │   └── ResourceModel
    │       ├── FormData.php ✅
    │       └── FormData
    │           └── Collection.php ✅
    ├── etc
    │   ├── adminhtml
    │   │   └── system.xml 💾
    │   ├── db_schema.xml 💾
    │   ├── di.xml ✅
    │   ├── frontend
    │   │   └── routes.xml 💾
    │   └── module.xml 💾
    ├── registration.php 💾
    └── view
        └── frontend
            ├── layout
            │   └── landingpage_index_index.xml 💾
            └── templates
                ├── content
                │   └── content.phtml
                ├── form
                │   └── form.phtml ✅
                └── no-form
                    ├── disabled.phtml
                    └── session.phtml
```


Content 'app/code/LandingPage/Form/Api/Data/FormDataInterface.php'

```php
<?php

namespace LandingPage\Form\Api\Data;

interface FormDataInterface
{
    /**
     * ID
     *
     * @return int|null
     */
    public function getId();

    /**
     * Set ID
     *
     * @param int $id
     * @return FormDataInterface
     */
    public function setId($id);

    /**
     * Customer Id
     *
     * @return int|null
     */
    public function getCustomerId();

    /**
     * Set Customer Id
     *
     * @param int $customerId
     * @return FormDataInterface
     */
    public function setCustomerId($customerId);

    /**
     * Comment
     *
     * @return string|null
     */
    public function getComment();

    /**
     * Set Customer Id
     *
     * @param string $comment
     * @return FormDataInterface
     */
    public function setComment($comment);
}
```


Content 'app/code/LandingPage/Form/Api/FormDataRepositoryInterface.php'

```php
<?php
namespace LandingPage\Form\Api;

use LandingPage\Form\Api\Data\FormDataInterface;

interface FormDataRepositoryInterface
{
    /**
     * Save form data.
     *
     * @param FormDataInterface $formData
     * @return FormDataInterface
     */
    public function save(FormDataInterface $formData);

    /**
     * Get form data by ID.
     *
     * @param int $id
     * @return FormDataInterface
     */
    public function getById($id);

    /**
     * Delete form data.
     *
     * @param FormDataInterface $formData
     * @return bool
     */
    public function delete(FormDataInterface $formData);
}
```

Content 'app/code/LandingPage/Form/Controller/Index/Post.php'

```php
<?php
namespace LandingPage\Form\Controller\Index;

use Magento\Framework\App\Action\Context;
use Magento\Framework\App\Action\Action;
use Magento\Framework\Controller\ResultFactory;
use Psr\Log\LoggerInterface;
use LandingPage\Form\Api\FormDataRepositoryInterface;
use LandingPage\Form\Api\Data\FormDataInterfaceFactory;
use Magento\Customer\Api\CustomerRepositoryInterface;

class Post extends Action
{
    protected $formDataRepository;
    protected $formDataFactory;
    protected $logger;
    protected $customerRepository;

    public function __construct(
        Context $context,
        FormDataRepositoryInterface $formDataRepository,
        FormDataInterfaceFactory $formDataFactory,
        CustomerRepositoryInterface $customerRepository,
        LoggerInterface $logger
    ) {
        parent::__construct($context);
        $this->formDataRepository = $formDataRepository;
        $this->formDataFactory = $formDataFactory;
        $this->customerRepository = $customerRepository;
        $this->logger = $logger;
    }

    public function execute()
    {
        $postData = $this->getRequest()->getPostValue();

        if (!empty($postData)) {
            try {
                $customer = $this->getCustomer($postData['email']);
                if ($customer) {
                    $customerId = $customer->getId();

                    $formData = $this->formDataFactory->create();
                    $formData->setCustomerId($customerId);
                    $formData->setComment($postData['comment']);

                    $this->formDataRepository->save($formData);
                    $this->messageManager->addSuccessMessage(__('Form data saved successfully.'));
                } else {
                    $this->messageManager->addErrorMessage(__('Customer not found.'));
                }
            } catch (\Exception $e) {
                $this->logger->error($e->getMessage());
                $this->messageManager->addErrorMessage(__('An error occurred while saving the form data.'));
            }
        }

        $resultRedirect = $this->resultFactory->create(ResultFactory::TYPE_REDIRECT);
        return $resultRedirect->setPath('*/*/');
    }

    protected function getCustomer($email)
    {
        try {
            return $this->customerRepository->get($email);
        } catch (\Exception $e) {
            $this->logger->error($e->getMessage());
            return null;
        }
    }
}
```


Content 'app/code/LandingPage/Form/Helper/Data.php'


```php
<?php
namespace LandingPage\Form\Helper;

use Magento\Framework\App\Helper\AbstractHelper;
use Magento\Framework\App\Helper\Context;
use Magento\Framework\App\Config\ScopeConfigInterface;
use Magento\Store\Model\ScopeInterface;

class Data extends AbstractHelper
{
    const XML_PATH_MODULE_ENABLED = 'landingpage_form/general/enable_module';

    protected $scopeConfig;

    public function __construct(
        Context $context,
        ScopeConfigInterface $scopeConfig
    ) {
        parent::__construct($context);
        $this->scopeConfig = $scopeConfig;
    }

    /**
     * Check if the module is enabled
     *
     * @return bool
     */
    public function isModuleEnabled()
    {
        return $this->scopeConfig->isSetFlag(self::XML_PATH_MODULE_ENABLED,ScopeInterface::SCOPE_STORE);
    }
}
```

Content: 'app/code/LandingPage/Form/Model/FormData.php'

```php
<?php
// send FormData to AbstractModel
namespace LandingPage\Form\Model;

use Magento\Framework\Model\AbstractModel;
use  LandingPage\Form\Api\Data\FormDataInterface;
use Magento\Framework\DataObject\IdentityInterface;


class FormData extends AbstractModel implements FormDataInterface, IdentityInterface
{
    protected function _construct()
    {
        $this->_init(\LandingPage\Form\Model\ResourceModel\FormData::class);
    }

    /**
     * Get identities
     *
     * @return array
     */
    public function getIdentities() : array
    {
        return [self::CACHE_TAG . '_' . $this->getId(), self::CACHE_TAG . '_' . $this->getIdentifier()];
    }


    /**
     * ID
     *
     * @return int
     */
    public function getId() : int
    {
        return (bool)$this->getData('id');
    }

    /**
     * Set ID
     *
     * @param int $id
     * @return FormDataInterface
     */
    public function setId($id) : FormDataInterface
    {
        return $this->setData('id', $id);
    }

     /**
     * Customer ID
     *
     * @return int
     */
    public function getCustomerId() : int
    {
        return (bool)$this->getData('customer_id');
    }

    /**
     * Set Customer ID
     *
     * @param int $customerId
     * @return FormDataInterface
     */
    public function setCustomerId($customerId) : FormDataInterface
    {
        return $this->setData('customer_id', $customerId);
    }

     /**
     * Comment
     *
     * @return string
     */
    public function getComment() : string
    {
        return $this->getData('comment');
    }

    /**
     * Set Customer ID
     *
     * @param string $comment
     * @return FormDataInterface
     */
    public function setComment($comment) : FormDataInterface
    {
        return $this->setData('comment', $comment);
    }

}
```

Content 'app/code/LandingPage/Form/Model/FormDataRepository.php'

```php
<?php
namespace LandingPage\Form\Model;

use LandingPage\Form\Api\FormDataRepositoryInterface;
use LandingPage\Form\Api\Data\FormDataInterface;
use LandingPage\Form\Model\ResourceModel\FormData as FormDataResource;
use Magento\Framework\Exception\CouldNotSaveException;
use LandingPage\Form\Api\Data\FormDataInterfaceFactory; // Dodaj import

class FormDataRepository implements FormDataRepositoryInterface
{
    protected $resource;
    protected $formDataFactory; // Dodaj właściwość dla fabryki

    public function __construct(FormDataResource $resource, FormDataInterfaceFactory $formDataFactory) // Dodaj fabrykę jako argument
    {
        $this->resource = $resource;
        $this->formDataFactory = $formDataFactory; // Przypisz fabrykę do właściwości
    }

    public function save(FormDataInterface $formData)
    {
        try {
            $result = $this->resource->save($formData);
        } catch (\Exception $exception) {
            throw new CouldNotSaveException(__($exception->getMessage()));
        }
        return $formData;
    }

    public function getById($id)
    {
        $formData = $this->formDataFactory->create(); // Użyj fabryki
        $this->resource->load($formData, $id);
        return $formData;
    }

    public function delete(FormDataInterface $formData)
    {
        try {
            $this->resource->delete($formData);
        } catch (\Exception $exception) {
            throw new CouldNotDeleteException(__($exception->getMessage()));
        }
        return true;
    }
}
```


Content 'app/code/LandingPage/Form/Model/ResourceModel/FormData.php'

```php
<?php
namespace LandingPage\Form\Model\ResourceModel;

use Magento\Framework\Model\ResourceModel\Db\AbstractDb;
use Magento\Framework\Model\AbstractModel;

use Magento\Framework\EntityManager\EntityManager;
use Magento\Framework\EntityManager\MetadataPool;
use Magento\Framework\Model\ResourceModel\Db\Context;
use Magento\Store\Model\Store;
use Magento\Store\Model\StoreManagerInterface;

class FormData extends AbstractDb
{

    /**
     * @param Context $context
     * @param StoreManagerInterface $storeManager
     * @param EntityManager $entityManager
     * @param MetadataPool $metadataPool
     * @param string $connectionName
     */
    public function __construct(
        Context $context,
        protected EntityManager $entityManager,
        $connectionName = null
    ) {
        parent::__construct($context, $connectionName);
    }

    /**
     * Define main table and primary key
     */
    protected function _construct()
    {
        // Table name and primary key field
        $this->_init('landingpage_form', 'id');
    }

        /**
     * Save an object.
     *
     * @param AbstractModel $object
     * @return $this
     * @throws \Exception
     */
    public function save(AbstractModel $object) : self
    {
        $this->entityManager->save($object);
        return $this;
    }
}
```


Content: 'app/code/LandingPage/Form/Model/ResourceModel/FormData/Collection.php'

```php
<?php
namespace LandingPage\Form\Model\ResourceModel\FormData;

use Magento\Framework\Model\ResourceModel\Db\Collection\AbstractCollection;
use LandingPage\Form\Model\FormData;
use LandingPage\Form\Model\ResourceModel\FormData as FormDataResource;

class Collection extends AbstractCollection{
    protected function _construct(){
        $this->_int(FormData::class,FormDataResource::class);
    }
}
```


Content 'app/code/LandingPage/Form/etc/di.xml'

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <preference for="LandingPage\Form\Api\FormDataRepositoryInterface" type="LandingPage\Form\Model\FormDataRepository" />
    <preference for="LandingPage\Form\Api\FormDataInterface" type="LandingPage\Form\Model\FormData" />
    <preference for="LandingPage\Form\Api\Data\FormDataInterface" type="LandingPage\Form\Model\FormData" />

    <type name="Magento\Framework\EntityManager\MetadataPool">
        <arguments>
            <argument name="metadata" xsi:type="array">
                <item name="LandingPage\Form\Api\Data\FormDataInterface" xsi:type="array">
                    <item name="entityTableName" xsi:type="string">landingpage_form</item>
                    <item name="identifierField" xsi:type="string">id</item>
                </item>
            </argument>
        </arguments>
    </type>

</config>
```

Content: 'app/code/LandingPage/Form/view/frontend/templates/form/form.phtml'


```phtml
<div style="width:100%;display:flex;justify-content:center;align-items:center;" class="account_container-inner">
    <form style="width:50%;" action="<?= $block->getUrl('landingpage/index/post') ?>" class="form" method="post" id="user-info-form">
        <?= $block->getBlockHtml('formkey') ?>
        <fieldset class="fieldset">
            <div class="field">
                <label class="label label--required" for="name"><span><?= __("Name")?></span></label>
                <input name="name" class="input" required id="name" type="text" title="name" value="<?=$block->getCustomerName()?>" disabled require>
            </div>      
             
            <div class="field">
            <label class="label label--required" for="username"><span><?= __("Email")?></span></label>
                <input name="email" class="input" required id="email" type="email" title="Email" value="<?=$block->getCustomerEmail()?>" disabled>
            </div>
            <label class="label" for="username"><span><?= __("Comment")?></span></label>
            <div class="field">
            
                <textarea name="comment" class="input" id="comment" title="Comment" > </textarea>
            </div>
            <!-- Hidden fields to send username and email -->
            <input type="hidden" name="name" value="<?=$block->getCustomerName()?>">
            <input type="hidden" name="email" value="<?=$block->getCustomerEmail()?>">

            <div class="actions">
                <button type="button" class="action-primary" name="add-to-cart">
                    <span><?= __("Add to Cart") ?></span>    
                </button>
                <button class="action-secondary" type="submit">
                    <span><?= __("Read More") ?></span>
                </button>
            </div>
        </fieldset>
    </form>
</div>
```

### Summary:

AbstractModel - Interact with database.
IdentityInterface - Cache tag

What is this doing - replace standard magento crud to EntityManager and implements this by using interfaces 


Summary:
- '/Api/Data/FormDataInterface.php' - Defines the interface for FormData.
- '/Api/FormDataRepositoryInterface.php' - Defines the interface for the FormDataRepository.
- '/Model/FormData.php' - Implements FormDataInterface and IdentityInterface (cache identity is used in getIdentities).
- '/Controller/Index/Post.php' - A dedicated file for handling POST requests in the controller.
- '/Model/FormData.php' - implements FormDataInterface and IdentityInterface , extends AbstractMode.
- '/Model/ResourceModel/FormData.php' - 
- '/Model/FormDataRepository.php' - 

```bash
app/code/LandingPage/Form/Api/FormDataRepositoryInterface.php
    |
    ▼
app/code/LandingPage/Form/Model/FormDataRepository.php
    |
    ▼
app/code/LandingPage/Form/Api/Data/FormDataInterface.php <------------------------------> app/code/LandingPage/Form/Model/FormData.php
                                        (Implements)

    |
    ▼
app/code/LandingPage/Form/Model/ResourceModel/FormData.php (Database Operations: save, load, delete)

```

```bash
app/code/LandingPage
└── Form
    ├── Api
    │   ├── Data
    │   │   └── FormDataInterface.php 🧩
    │   └── FormDataRepositoryInterface.php 🧩
    ├── Controller
    │   └── Index
    │       └── Post.php ✅
    ├── Model
    │   ├── FormData.php ✅
    │   ├── FormDataRepository.php 🗂️
    │   └── ResourceModel
    │       ├── FormData.php 🗂️
    │       └── FormData
    │           └── Collection.php 🗂️  (send Model\FormData and ResourceModel\FormData to AbstractCollection) typo but is working ❌
    ├── etc
    │   ├── di.xml 🧰 (Maps interfaces to their corresponding classes. It also defines the database table structure, specifying the table name and primary key column. )
```



rest of the files:


'app/code/LandingPage/Form/Controller/Index/Index.php'

```php
<?php
namespace LandingPage\Form\Controller\Index;

use Magento\Framework\App\Action\Action;
use Magento\Framework\App\Action\Context;
use Magento\Framework\View\Result\PageFactory;

class Index extends Action{
    protected $_pageFactory;

    public function __construct(Context $context, 
    PageFactory $pageFactory){
        $this->_pageFactory = $pageFactory;
        parent::__construct($context);
    }

    public function execute(){
        return $this->_pageFactory->create();
    }
}
```


file 'app/code/LandingPage/Form/Helper/Data.php'

```php
<?php
namespace LandingPage\Form\Helper;

use Magento\Framework\App\Helper\AbstractHelper;
use Magento\Framework\App\Helper\Context;
use Magento\Framework\App\Config\ScopeConfigInterface;
use Magento\Store\Model\ScopeInterface;

class Data extends AbstractHelper
{
    const XML_PATH_MODULE_ENABLED = 'landingpage_form/general/enable_module';

    protected $scopeConfig;

    public function __construct(
        Context $context,
        ScopeConfigInterface $scopeConfig
    ) {
        parent::__construct($context);
        $this->scopeConfig = $scopeConfig;
    }

    /**
     * Check if the module is enabled
     *
     * @return bool
     */
    public function isModuleEnabled()
    {
        return $this->scopeConfig->isSetFlag(self::XML_PATH_MODULE_ENABLED,ScopeInterface::SCOPE_STORE);
    }
}
```

file: 'app/code/LandingPage/Form/etc/adminhtml/system.xml'
```xml
<?xml version="1.0"?>
<!-- app/code/LandingPage/Form/etc/adminhtml/system.xml -->
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Config/etc/system_file.xsd">
    <system>
        <section id="landingpage_form" translate="label" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="1">
            <label>LandingPage Form Configuration</label>
            <tab>general</tab>
            <resource>LandingPage_Form::config</resource>
            <group id="general" translate="label" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="1">
                <label>General Settings</label>
                <field id="enable_module" translate="label comment" type="select" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="1">
                    <label>Enable Module</label>
                    <comment>Choose whether to enable or disable the module.</comment>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
     
                </field>
            </group>
        </section>
    </system>
</config>
```

file 'app/code/LandingPage/Form/etc/db_schema.xml'

```xml
<?xml version="1.0"?>
<schema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:framework:Setup/Declaration/Schema/etc/schema.xsd">

    <table name="landingpage_form" resource="default" engine="innodb" comment="Landing Page Form Table">
        <column xsi:type="int" name="id" nullable="false" identity="true" unsigned="true" comment="Form ID"/>
        <column xsi:type="int" name="customer_id" nullable="false" comment="Customer ID"/>
        <column xsi:type="varchar" name="comment" nullable="true" length="255" comment="Comment"/>
        <constraint xsi:type="primary" referenceId="PRIMARY">
            <column name="id"/>
        </constraint>
    </table>

</schema>
```


file 'app/code/LandingPage/Form/etc/frontend/routes.xml'

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="standard">
        <route id="landingpage" frontName="landingpage">
            <module name="LandingPage_Form"/>
        </route>
    </router>
</config>
```


file 'app/code/LandingPage/Form/etc/module.xml'

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
    <module name="LandingPage_Form" setup_version="1.0.0"/>
</config>
```


file: 'app/code/LandingPage/Form/registration.php'

```php
<?php

use Magento\Framework\Component\ComponentRegistrar;

ComponentRegistrar::register(
    ComponentRegistrar::MODULE,
    
     // The name of the module we're registering
    'LandingPage_Form',
    __DIR__
);


```

file 'app/code/LandingPage/Form/view/frontend/layout/landingpage_index_index.xml'

```xml
<?php

use Magento\Framework\Component\ComponentRegistrar;

ComponentRegistrar::register(
    ComponentRegistrar::MODULE,
    
     // The name of the module we're registering
    'LandingPage_Form',
    __DIR__
);
```

file 'app/code/LandingPage/Form/view/frontend/templates/content/content.phtml'

```phtml
<!-- app/code/LandingPage/Form/view/frontend/templates/content.phtml -->

<?php
/** @var LandingPage\Form\Block\Index $block */

?>
<div class="landing-page-content">
    <?php   if($block->isModuleEnabled() && $block->isEnglishStore()):  ?>
        <?php if($block->isCustomerLoggerIn()): ?>
            <?= $block->getChildHtml('form_block_child_form'); ?>
        <?php else: ?>
            <?= $block->getChildHtml('form_block_child_session'); ?>
        <?php endif; ?>



    <?php else: ?> 
        <?= $block->getChildHtml('form_block_child_disabled'); ?>   
    <?php endif; ?>
</div>
```

file: 'app/code/LandingPage/Form/view/frontend/templates/no-form/disabled.phtml'

```phtml

<!-- app/code/LandingPage/Form/view/frontend/templates/guest.phtml -->

<?php
/** @var LandingPage\Form\Block\Index $block */

?>

<div  style="display:flex;justify-content:center;align-items:center;"class="product-hero__content" x-data="{ seeMore: false }">

    <h2 class="product-hero__title">
        <!-- Add Product Name -->
        <?= __("The module is disabled or not supported on this store.") ?>      </h2>

```

file 'app/code/LandingPage/Form/view/frontend/templates/no-form/session.phtml'

```phtml
<a style="display:flex;justify-content:center;align-items:center;" href="<?php echo $block->getLoginUrl(); ?>" >
                <button style="width:50%;" type="form-actions-submit" class="action-primary" name="send">
                        <span><?php echo __('Please login'); ?></span>
                </button>
            </a>
```












