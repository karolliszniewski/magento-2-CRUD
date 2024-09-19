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
    │   ├── di.xml
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



