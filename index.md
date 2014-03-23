# Groupoffice ActiveRecord Tutorial

## Introduction
The groupoffice ActiveRecord.php is a custom implementation of the [active record pattern][1] commonly
used for database access from web applications. The system can seem complex and daunting at first, but
as you get to know it you will find the time-saving features very helpful.

I may extend this to be a tutorial of the whole of groupoffice eventually.

This tutorial assumes you have a basic understanding of the groupoffice class autoloader and router. 
I give a quick recap below:

### Groupoffice basics recap
The php backend for groupoffice is divided into a module section and a non-module section. The module
section is the only place you will ever be adding code, by creating a custom module. Creating a custom
module involves creating a folder in the `modules` folder, and adding some boilerplate code.

There is a [groupoffice admin tool][2] for creating a skeleton module based on the notes module, or I
would recommend looking at the notes module or the [creating a model tutorial][3] on the groupoffice
wiki. You'll end up with something like this (assuming the module is called example):

    modules/
        ...
        example/
            ExampleModule.php
            model/
                < this is where we put our activerecords >
            controller/
            views/
                Extjs3/
            install/
            language/

## ActiveRecord Basics
### Creating the table
There is nothing groupoffice specific about creating the table - it is just a case of running the sql
to create the table (and then putting that sql in a file called `install.sql` in the install folder).

My usual workflow is to use phpmyadmin to construct the table structure, then code the module (which
will inevitably involve changing the table), and finally use the export feature of phpmyadmin to create
the `install.sql` file.

### Sub-classing
Groupoffice provides the class `GO_Base_Db_ActiveRecord` to represent a table in the database. To use a
table as an ActiveRecord, we must subclass this class:

```PHP
<?php
class GO_Example_Model_Thing extends GO_Base_Db_ActiveRecord {

    public static function model($className=__CLASS__) {
        return parent::model($className);
    }

    protected function tableName() {
        return 'example_thing';
    }
    
}
```

There are 2 parts to this class:

1. The `model` static function is boilerplate because in php, static functions aren't inherited by
   subclasses. It doesn't matter what it does, just cut and paste it.

2. The function `tableName` must return a string of the table name in the db.

Once you have this simple subclass, you can start using your model in code:

1. `$thing = new GO_Example_Model_Thing()` will create an object representing a new database record,
2. `$thing->foo = "bar"; $thing->baz = 2;` will set column `foo` to `"bar"` and `baz` to `2`.
3. `$thing->save();` will save the object as a new database record.

This is all the information you need to use a rudimentary model/table. There are lots of ways to extend
this functionality however.

### Model Controller
The vast majority of the time you don't need to create objects on the system directly. This is because
usually for each `GO_Example_Model_Thing`, there will be a `GO_Example_Controller_ThingController` to
handle CRUD requests for that model. So for example if in the `controllers` directory we create the
following file (`ThingController.php`)

```PHP
<?php
class GO_Example_Controller_ThingController extends 
      GO_Base_Controller_AbstractModelController {
    
    protected $model = "GO_Example_Model_Thing";
    
}
```

we get a set of routes to use in our AJAX requests from the browser.

NB There is a lot more to controllers - I'm not covering it here. To find out more check the
source/source documentation.

## Advanced
### Model Relations
In groupoffice, module relations are expressed by an overridable function `relations` in the
`ActiveRecord` class. Here is the contacts relations as an example:

```PHP
<?php
public function relations(){
    return array(
    'goUser' => array(
        'type' => self::BELONGS_TO,
        'model' => 'GO_Base_Model_User',
        'field' => 'go_user_id'
    ),
    'addressbook' => array(
        'type'=>self::BELONGS_TO,
        'model'=>'GO_Addressbook_Model_Addressbook',
        'field'=>'addressbook_id'
    ),
    'company' => array(
        'type' => self::BELONGS_TO,
        'model' => 'GO_Addressbook_Model_Company',
        'field' => 'company_id'
    ),
    'addresslists' => array(
        'type'=>self::MANY_MANY,
        'model'=>'GO_Addressbook_Model_Addresslist',
        'field'=>'contact_id',
        'linkModel' => 'GO_Addressbook_Model_AddresslistContact'
    ),
    'vcardProperties' => array(
        'type'=>self::HAS_MANY,
        'model'=>'GO_Addressbook_Model_ContactVcardProperty',
        'field'=>'contact_id',
        'delete'=> true
    ),
);
```
Once this function is created, you can get at related models through the keys in this array, for
example if `$contact` points to a contact on the database:
```php
<?php
$foo = $contact->addressbook 
// Now $foo points to the addressbook the contact belongs to 
echo $contact->addressbook->name
echo $foo->name
// Both output the name of the addressbook
```
Again in reality  you will never write this code - it is generally all taken care of by the
`AbstractModelController`.

The documentation does a better job of explaining the different options for relations than I can,
so check out the source of ActiveRecord! ;-)
### Other overrides
There are some other functions you can override in the `ActiveRecord` class:
#### getLocalizedName()
Return a string giving the name of the model. If writing a multilingual module this function should get
text from the current translation, e.g.
```php
<?php
protected function getLocalizedName() {
    return GO::t("thing", "example")
}
```
Again I don't want to go into translations here - see [TODO] for that.

#### aclField()
If you want to have access control on a per-record basis, you have to specify an integer field to use
for the acl id here. The field can either be in the model, or in one relation (so for a contact in the
addressbook module the addressbook contains
### Constructing queries

#### Filter by permissions
For example, if you wanted to only return those records where the user has write access, you would do
```php
<?php
$fp = GO_Base_Db_FindParams::newInstance()
	->permissionLevel(GO_Base_Model_Acl::WRITE_PERMISSION);
```

  [1]: http://en.wikipedia.org/wiki/Active_record_pattern "Active record pattern"
  [2]: https://github.com/derekdreery/goadmin "Groupoffice Admin"
  [3]: https://www.group-office.com/wiki/Creating_a_module "Groupoffice create model tutorial"
