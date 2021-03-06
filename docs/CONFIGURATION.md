FlexiForm Configuration
=======================

Most configuration is accomplished through the CMS -- however you can further 
tailor behavior through subclassing (protected properties, getters, and setters)
and [YAML Configuration](http://doc.silverstripe.org/framework/en/topics/configuration).

This approach allots flexibility and enables different strategies to accomplish 
behavioral needs.


### Limiting Field Types

The choice of fields types can be defined per form. Here's a couple examples. 


* Strategy 1: Using [YAML Configuration](http://doc.silverstripe.org/framework/en/topics/configuration)

```yaml
---
FormPage:
  flexiform_field_types:
    - FlexiFormTextField
    - FlexiFormDropdownField
```

* Strategy 2: Overload  the **$flexiform_field_types** property

```php
class FormPage extends Page {

  private static $extensions = array(
    'FlexiFormExtension'
  );
    
  private static $flexiform_field_types = array(
    'FlexiFormTextField',
    'FlexiFormDropdownField'
  );

}
```

* Strategy 3: Use the **setFlexiFormFieldTypes** setter method 

```php
class FormPage extends Page {

  private static $extensions = array(
    'FlexiFormExtension'
  );

  public function getCMSFields()
  {
    // make configuration changes _BEFORE_ calling parent getCMSFields...
    $allowed_types = array(
      'FlexiFormTextField',
      'FlexiFormDropdownField'
    );
    $this->setFlexiFormFieldTypes($allowed_types);
    
    $fields = parent::getCMSFields();
    
    return $fields;
   }
}
```

### Changing the Tab FlexiForm appears in

By default, flexiform will add a Form to "Root.Form". You can change it a couple of ways;

* Strategy 1: Using [YAML Configuration](http://doc.silverstripe.org/framework/en/topics/configuration)

```yaml
---

# Global Change
FlexiFormExtension:
  flexiform_tab: Root.Addresses
  
# Class Specific
FormPage:
  flexiform_tab: Root.Main
  flexiform_insertBefore: Metadata
```

* Strategy 2: Overload the **setFlexiFormTab** property

```php
class FormPage extends Page {

  private static $extensions = array(
    'FlexiFormExtension'
  );
  
  private static $flexiform_tab = 'Root.Main';
  private static $flexiform_insertBefore = 'Metadata';

}
```

* Strategy 3: Use the **setFlexiFormTab** setter method 

```php
class RegistrationForm extends DataObject {

  public function getCMSFields()
  {
    $this->setFlexiFormTab('Root.Registration');
    
    $fields = parent::getCMSFields();
    
    return $fields;
  }

}
```
 
### Automatically adding fields to a form

Newly created forms can be programmatically initialized with a set of fields by
providing Initial Field definitions. 

Initial Field definitions are nearly the same 
as [Custom Fields](#Custom-Fields), except that you must provide a `Type` and property.
Additionally, you may provide extra values (such as `Prompt`). 

Flexiform will attempt to link the field of matching `Name` and `Type`. If
no matching field is found, a new field will be created from the definiton and linked.


* Strategy 1: Overload the **$flexiform_initial_fields** property

```php
class AuthorChoiceForm extends DataObject {

  private static $extensions = array(
      'FlexiFormExtension'
  );

  private static $flexiform_initial_fields = array(
      array(
          'Type' => 'FlexiFormTextField',
          'Name' => 'Name',
          'Prompt' => 'Your Name'
      ),
      array(
          'Type' => 'FlexiFormEmailField',
          'Name' => 'Email',
          'Prompt' => 'Your Email'
      )
  );

}
```

* Strategy 2: Use the **setFlexiFormInitialFields** setter method 

```php
class Event extends SiteTree {

  private static $extensions = array(
      'FlexiFormExtension'
  );
  
  public function getCMSFields()
  {
      // make configuration changes _BEFORE_ calling parent getCMSFields...
      $this->setFlexiFormTab('Root.Registration');
      $this->setFlexiFormInitialFields(
        array(
            array(
                'Type' => 'FlexiFormTextField',
                'Name' => 'Name',
                'Prompt' => 'Your Name'
            ),
            array(
                'Type' => 'FlexiFormEmailField',
                'Name' => 'Email',
                'Prompt' => 'Your Email'
            )
        ));
      
      $fields = parent::getCMSFields();
      
      return $fields;
  }

}
```

* Strategy 3: Using [YAML Configuration](http://doc.silverstripe.org/framework/en/topics/configuration

```yaml
---
Event:
  flexiform_initial_fields:
    - Name: Name
      Type: FlexiFormTextField
      Prompt: Your XX
      
    - Name: Email
      Type: FlexiFormEmailField
      Prompt: Your YY
```


  
Custom Fields
-------------

New custom fields can be created by subclassing `FlexiFormField` types.
* If your field has selectable options (like a Dropdown), extend the `FlexiFormOptionField` type.
* If your field does not have selectable options (like an Email), extend the `FlexiFormField` type.
* Alternatively, extend the [existing type](https://github.com/briceburg/silverstripe-flexiforms/tree/master/code/model/fieldtypes) that best matches your behavior.


TODO: frontend field documentation &c.

### Programmatically adding fields

The Environment Builder (/dev/build) is used to automatically create fields
specifying a **$required_field_definitions** property. Fields can 
share a FieldName providing they're of a different type.

* Strategy 1: Overload the **$required_field_definitions** property

First, create your Custom Field with a valid **$field_definitions** property.
```php
<?php
class FlexiAuthorField extends FlexiFormDropdownField
{
  protected $field_description = 'Author Preference Dropdown';
  protected $field_label = 'Author';

  private static $required_field_definitions = array(
    array(
      'Name' => 'Author',
      'EmptyString' => 'Other',
      'Options' => array(
        'Balzac' => 'Honoré de Balzac',
        'Dumas' => 'Alexandre Dumas',
        'Flaubert' => 'Gustave Flaubert',
        'Hugo' => 'Victor Hugo',
        'Verne' => 'Jules Verne',
        'Voltaire' => 'Voltaire'
      )
    )
  );

}
```


* Strategy 2: Using [YAML Configuration](http://doc.silverstripe.org/framework/en/topics/configuration), **especially useful for creating fields from built-in field types**

```yaml
---
FlexiFormTextField:
  required_field_definitions: 
    - Name: FirstName
    - Name: LastName 
    
FlexiFormDropdownField: 
  required_field_definitions: 
    - Name: Author
      Options: 
        Balzac: Honoré de Balzac
        Dumas: Alexandre Dumas
        Flaubert: Gustave Flaubert
        Hugo: Victor Hugo
        Verne: Jules Verne
        Voltaire: Voltaire

# Alt Syntax    
FlexiFormDropdownField:
  required_field_definitions:
    - { Name: Preference, Options: { Eastern: Abacus, Western: Calculator } }
``` 

Be sure trigger the Environment Builder (e.g. by visiting /dev/build) after making changes.

### Readonly fields

By default, all fields are editable. The CMS administrator can change the 
FieldName, list of selectable options, &c. This flexibility may be 
unwanted in certain situations. 

For instance, pretend you have a Custom Form that references a field named 
'Email' in  _$initial_flexi_fields_. Soon after, the admin renamed the 'Email' 
field to 'WorkEmail'. Now, whenever the Custom Form is created, a Validation 
Exception will occur complaining that 'Email' is not found. 

Readonly fields are a great way to protect against this. They differ from
normal fields as follows;
* **Readonly fields have their FieldName, FieldDefaultValue, and Options locked**
* **Readonly fields are marked with an asterix (*) when searched for**
* **Readonly fields must be created programmatically**
* **Readonly field names must be unique, regardless of underlying field type**

```php
class FlexiAuthorField extends FlexiFormDropdownField
{
  protected $required_field_definitions = array(
    array(
      'Name' => 'Author',
      'Readonly' => true,
      ...
}
```

or

```yaml
---
FlexiAuthorField:
  required_field_definitions: 
    - Name: Author
      Readonly: true
```