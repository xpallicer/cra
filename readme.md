# Common Resource Administration   

*CloudHotelier API-first App development standard*


Author: Xavier Pallicer Majó
Date: September 2017


**Index:**

- Basics
- CRA API definition
  - Requests
  - Responses
- Common resource administration  
  - Get Item
  - Save Item
  - Get Items
  - Update Items
  - Delete Items
  - Reorder Items
  - Tree Items
- CHAdmin implementation
  - YAML definitions for API resources
  - CHAdmin Item
  - CHAdmin List
  - Joins 
  - XRef
  - Access check
  - The user object
- Getting started
  - Internal operation
  - Admin Interface
  - REST API


## Basics

**CRA** is a API definition for app desing and development standarization. 

The current document describes the API definition, and our specific implementation via **CHAdmin** helpers, an api-first (AMVC) app design and development tools built on top of **Joomla CMS**.

## CRA API definition 

### Requests:

API requests must have `resource`, `task` and `data` and may need `username`, `password` and `uid` (unique identifier for checks and logs).

```yaml
---
# required:
resource: article
task: get
data: {}  

# optional:
username: username
password: password
uid: 5f673fd7-f20e-4d09-9c59-e718249fbcb0    
```

### Responses:

API will always respond with `status` and `data`. Status can only be *success* or *error*. Data will include the response details or be *null*.

```yaml
---
status: success
data: {}
```

*Get* responses will always respond with `resource` and `type`.

```yaml
---
# request
resource: article
task: get
data: {id: 123456}

---
# response
status: success
resource: article
type: item
data: {item: {id: 123456, title: Item Title}}
``` 

Successful responses for tasks that *manipulate* resources will always respond a `message` with information of the result of the action performed.

```yaml
---
# request
resource: articles
task: delete
data: {ids: [123456]}

---
# response
status: success
message: 1 article deleted
data: null
``` 

Errors always include `code` and `message` with information of the problem occurred.

```yaml
---
# request
resource: article
task: get
data: {id: 000000} 

---
# response
status: error
code: NOT_FOUND
message: Article not found
data: {}
```

## Common Resources Administration

**CRA** provides standard operation for the most common app design approaches. 

You can extend this basic operation with custom resources and tasks.

The common implementation includes 2 types of resources, *item* and *list*, associated with the following tasks:

- **`item`:** `get`, `save`
- **`list`:** `get`, `update`, `delete`, `reorder`, `tree`

### Correspondence

As item and list types are corresponded to each other, **CRA** API will respond with the corresponding type resource name. 

**Get Item Example:**

```yaml
---
#request
resource: article
task: get
data: {id: 123456}

---
# response
status: success
resource: article
type: item
list: articles # corresponding list name
data: {item: {id: 123456, title: Item Title}}
```

**Get List Example:**

```yaml
---
# request 
resource: articles
task: get
data: {}

---
# response:
status: success
resource: articles
type: list
item: article # corresponding item name
data: {list: [{id: 123456, title: Item Title}]}
```

### Example: Get Item

Fetch the information of a item by providing it's *id* within the *data* parameters.

```yaml
---
# request
resource: article
task: get
data: {id: 123456}

---
# success response
status: success
resource: article
type: item
list: articles
data: {item: {id: 123456, title: Item Title}}

---
# error response
status: error
code: NOT_FOUND
message: Article not found
data: null
```

### Example: Save Item

Same task for New / Update / Duplicate. If you don't provide a item *id*, a **new item** will be created. On success, CRA will return the full item.


```yaml
---
# request
resource: article
task: save
data: {id: 123456, title: Updated Item Title}

# success response
---
status: success
message: Item saved
data: {item: {id: 123456, title: Updated Item Title}}

# error response
---
status: error
code: INVALID_DATA
message: Validation errors found
data: {errors: [{title: Required field}]}
```

### Example: Get List

Get a list of articles by providing the query details in the *data* parameters.


```yaml
---
# request
resource: articles
task: get
data:
  limit: 5
  start: 0
  order: title
  direction: asc
  filters: 
    search: 'Item'
  
---
# response
status: success
resource: articles
type: list
item: article
data:
  limit: 5
  start: 0
  order: title
  direction: asc
  filters: 
    search: 'Item'
  total: 8
  end: 5
  list:
    - {id: 123455, title: Item 1 Title}
    - {id: 123456, title: Item 1 Title}
    - {id: 123457, title: Item 1 Title}
    - {id: 123458, title: Item 1 Title}
    - {id: 123459, title: Item 1 Title}
```

Indexed results: **CRA** lists have `id` field as *index* by default. The reason for this is to avoid the need of looping through the entire list to fetch a specific item programatically. 

For pagination purposes, CRA will respond with `total` and `end` on evrey list request.

### Example: Update List

Update one or multiple items, this task updates the values of specified fields.

```yaml
---
# request
resource: articles
task: update
data: 
  ids: [123456, 123457]
  fields: {status: 1, published: '2017-09-11'}
 
---
# response
status: success
message: 2 items updated
```

### Example: Delete List

Delete one or multiple items.

```yaml
---
# request
resource: articles
task: delete
data: {ids: [123456, 123457]}

---
# response
status: success
message: 2 items deleted
```

### Example: Reorder List

Reorder and ordered list of items.

```yaml
---
# request
resource: articles
task: reorder
data: 
  ids: [123456, 123457]
  direction: asc

---
# response
status: success
message: 2 items reordered
```

### Example: Tree List

Reorganize a nested tree set of items.

```yaml
---
# request
resource: articles
task: tree
data: 
  tree:
    - id: 1
    - id: 2
    - id: 3
      children:
        - id: 4
        - id: 5
    - id: 6

---
# response
status: success
message: 6 items reorganized 
```

## CHAdmin helpers

**CHAdmin** is a *api-first* development helpers for Joomla that implements CRA api definition.

The idea behind this concept is to add another layer to the Joomla MVC, the API layer. We colud call it *AMVC*. Api, Model, View, Controller.

The new API layer, allows the app developer to have a **unique interface** for 3 common operations: 

- Internal operation, component level: easily build lists, forms and front-end views. The classic user-managed oriented app. `LOGGED USER -> MVC -> API`
- Internal operation, supracomponent level: integration between diferents components or apps. With this implementation, apps can easily administer other apps resources. `APP1 -> APP2 API`
- External operation: remote JSON API administration. `JSON ENDPOINT -> API`


### YAML standard for API design

CRA resources have a data model that is defined using YAML files. The library will use this definitions to automatically perform the tasks.

Yaml is a powerful yet easy to write and read that is great for API design. CHAdmin uses this format extensively, specially useful for CRA api definitions.

### CHAdmin Item

`/components/com_mycomponent/app/models/article.yml`

```yaml
---
name: article
type: item
list: articles
table: mycomponent_articles

fields:
  title: {fied: true, filter: string, validate: required|min:2}
  category_id: {field: cat_id, filter: int, validate: required}  
  body: {validate: required|min:50}  
 
settings: 
  status: 1
  created: 1
  created_by: 1
  modified: 1
  modified_by: 1
```

CHAdmin Yaml item definitions provide information of the resource and also how it's implemented at a data level, including field validation and database details.

- `table`: the name of the table that stores the resource data.
- `fields` definition including: 
  - `filter`:  that should be used when retriving the value
  - `validate`: validation that should be applied to the value
  - `field`: how to store the value: 
    - A) `true`: in a table field with the same name
    - B) *field name*: i a table field named as stated
    - C) `false` or not present: json_encoded in the `data` field of the table

This would be the MySQL table for the resource above:

```sql
CREATE TABLE `prefix_mycomponent_articles` (
  `id` int(10) AUTO_INCREMENT,
  `title` varchar(255),
  `cat_id`  int(10),
  `data` mediumtext,
  `status` tinyint(1),
  `created` datetime,
  `created_by` datetime,
  `modified` datetime,
  `modified_by` datetime,
  PRIMARY KEY (`id`),
  KEY `cat_id` (`cat_id`),
  KEY `status` (`status`)
);
```

`id` and `data` are required fields for all **CHAdmin** based tables, while  `status`, `created`, `modified`, `created_by` and `modified_by`, are *magic* fields wich may be required, depending on the resource *settings*. 


### CHAdmin List

`/components/com_mycomponent/app/models/articles.yml`

```yaml
---
name: articles
type: list
item: article
table: mycomponent_articles

fields:
  id: {order: id}
  title: {order: order}
  created: {order: created}
  
filters:    
  status: {filter: int}
  search: {fields: [title], filter: string}
    
settings:    
  order: id
  direction: asc
  limit: 20
```

Yaml list definitions provide information of the list and how it can be retrieved, manipulated, filtered or ordered.

- `table`: the name of the table that stores the resource data.
- `fields` definition including: 
  - `order`: can be used if the results can be ordered by this field
- `filters`: filters that can be appliead to get task
  - default behaviour is *name = value*, but they can be overriden. `status`and `search` are a special type of filters. 
  - `search`: will receive a string and will perform a **LIKE** on the defined fields. If the value is a number, a **ID = VALUE** search will be performed. If there's no fields value, a **title LIKE %VALUE%** search will be performed. By default, search on strings are case insensitive.
  - `status`: status filter will receive a string, if the string is empty, it will perform a **status < 2 ** search, that is, all items except the trashed (satuts=2). If it receives a integer, it will perform a **status = value** search. Status values are 1 = published, 0 = unpublished, 2 = trashed.
- `settings`: the default values of the get task. Default order field, default direction and limit. If no values are provieded, default order is ID, direction is ASC, and limit is 20.

**IMPORTANT NOTE**: lists can only operate directly on database fields, so if you need a field to be filtered, indexed, searchbale or updatable, make sure that you store it in a custom field in the database, not in the json_encoded `data`.

### Joins

Get info from external table. Can be used in both models, items and lists.

```yaml
---
fields:
  title: {}
  category: 
    external: 
      table: mycomponent_categories
      field: title

joins:
  - table: mycomponent_categories
    base_key: category_id  
    
    # optional: id, left by default    
    key: id 
    type: left
```

### Cross Reference

XRef on Item: will store the values in the xref table on save. 

```yaml
---
fields:
  title: {}
  tags: {filter: array}

xref:
  tags: 
    table: mycomponent_articles_tags
    base_key: article_id
    table_key: tag_id 
    
    # optional: id by default
    base_field: id 
```

XRef on List: will delete the xref values on delete. 

```yaml
---
fields:
  title: {}

xref:
  tags:
    table: mycomponent_articles_tags
    key: article_id
  
    # optional: id by default
    base_field: id 
```


## Internal operation

In your code, you can use the API helper like this:

```php
$request = (object)[
  'resource' => 'articles',
  'task' => 'get',
  'data' => (object)['id' => 123456]
];
$response = (new CHAdminApi('mycomponent', $request))->process();
```

The method above will look for the corresponding resource helper, that will be loaded with this priority:

1. `/components/com_mycomponent/api/articles.php` 
2. `/libraries/chadmin/api/articles.php` 
2. `/components/com_mycomponent/api/list.php` 
4. `/libraries/chadmin/api/list.php` 

This means you can easily override or extend the default list helper like this:

`/components/com_mycomponent/api/articles.php` 

```php
/**
 * Example overriding default checkAccess
 */
class myComponentApiArticles extends CHAdminApiList
{

    protected function queryPreparePost()
    {
        // extend $this->query with advanced joins etc.  
    }

    protected function filterSpecialField()
    {
        // attach your behaviour to $this->query 
    }     
}
```  

## REST JSON API  

Components implementing **CHAdmin Library** are automatically provided with an *REST JSON api endpoint* that can be used to administer the app resources remotely. 

*Note that access is not allowed by default by CHAdmin unless you override checkAccess method as described above.* 

Example of command line curl request:

```
curl -H "Content-Type: application/json" -X POST -d '
{
  "user": "appuser",
  "password": "userPass",
  "resource": "article",
  "task": "get",
  "data": {"id": 123456}
}
' https://my-site-or-app.com/mycomponent/api
```

## Building your app admin interface

**CHAdmin** provides a set of tools for easy develop admin interfaces in your Joomla front-end. 

*Note we don't use Joomla back-end or core-components because we don't like the users to sign-up at the joomla back-end, as it's ugly and slow, and it has tons of features that we don't want/need to display to our users.*

Load the **CHAdmin** library and **chadmin** template as follows.

```php
// load CHAdmin library
JLoader::import('chadmin', JPATH_LIBRARIES . '/chadmin');

// force chadmin template
JFactory::getApplication()->setTemplate('chadmin');
```

## Access check

CHAdmin provides a basic but flexible user access. There are 4 default level user types:

- 0: Super Administrator
- 1: Administrator
- 2: Manager
- 3: Editor

**Super User:** Can do everything including tasks than can affect system integrity like uploading or manipulating files.

**Administrator:** Can do everything but cannot compromise the system integrity. Can view and create *Administrator*, *Manager* and *Editor* users.

**Manager**: Can do everything but only to the data that owns. Cannot see users or data from other Managers. This means that every Manager can create it's own "space" within the system, with it's own users and data, separated from other Managers data. 

**Editor:** Can do everything to it's own data. Cannot create other users. Must have a parent *Manager* user.

User levels can be extended by the apps to fit custom needs.

### Access tables and logs


```yaml
---
# chadmin_apps
id
component
title

# chadmin_users
id
type
parent_id
first_name
last_name
email
phone

# chadmin_users_apps
user_id
app_id

# chadmin_users_access
user_id
app_id
resource
resource_id

# chadmin_user_logs
id
created
user_id
app_id
resource
task
resource_id
request
response

# chadmin_user_tokens
id
created
user_id
app_id
resource
task
resource_id
request
response
```

### The user object

CHAdmin helper loads an user object with contact information and 

```yaml
---
# Manager user
id: 123456
parent_id: 0
type: 2
first_name: John
last_name: Smith
email: john@smith.com
phone: 0034 0123456789
  
# users under supervision 
children_users_ids: [123457, 123458] 
  
# access granted apps
mycomponent:
  id: 123456
  title: My Component
  
  # resources owned by the user or the supervised users 
  articles: [123456, 123457, 123458]

---
# Editor user
id: 123457
parent_id: 123456
level: 3
first_name: Bob
last_name: Marley
email: bob@marley.com
phone: 0034 0123456799
  
# access granted apps
mycomponent: 
  id: 101
  title: My Component Title
      
  # resources owned by the user
  articles: [123456, 123457]
```

In the data model you can define access check like this:

```yaml
---
name: article
type: item
list: articles
access_level: 2 # optional, minimum user level, default 2
access:
  resource: articles 
  table: articles # optional, default is the resource table
  field: id # optional, default is id
``` 
On save a new article, the resource id is stored in the users_access table. If the user has a parent, the parent access is also stored.

1) If user level is Editor:


```yaml
---
# Store 1 row for current user
user_id: current_user_id
app_id: current_app_id
resource: articles
resource_id: 123456

# Store 1 row for current user parent
user_id: current_user_parent_id
app_id: current_app_id
resource: articles
resource_id: 123456
```

When requesting that resource ID, CHAdmin will lookup if the resource is present in the user->mycomponent->articles array. If not, access will be forbidden.

In the case of dependent items, access check can be defined as follows:

```yaml
---
name: image
type: item
list: images
access:
  resource: article
  table: mycomponent_images
  field: article_id

``` 
On save a new image, the value stored and the access check will be performed on the articles resource, and the key to store will be article_id.

Access can be defined in the CHAdmin Users view, wich will be based load the user_access definitions.