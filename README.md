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
    │       ├── Index.php
    │       └── Post.php ✅
    ├── Helper
    │   └── Data.php ✅
    ├── Model
    │   ├── FormData.php ✅
    │   ├── FormDataRepository.php ✅
    │   └── ResourceModel
    │       ├── FormData.php ✅
    │       └── FormData
    │           └── Collection.php ✅
    ├── etc
    │   ├── adminhtml
    │   │   └── system.xml
    │   ├── db_schema.xml
    │   ├── di.xml ✅
    │   ├── frontend
    │   │   └── routes.xml
    │   └── module.xml
    ├── registration.php
    └── view
        └── frontend
            ├── layout
            │   └── landingpage_index_index.xml
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

class FormDataRepository implements FormDataRepositoryInterface
{
    protected $resource;

    public function __construct(FormDataResource $resource)
    {
        $this->resource = $resource;
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
        $formData = $this->formDataFactory->create();
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








