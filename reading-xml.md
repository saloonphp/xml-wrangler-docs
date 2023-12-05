# 📖 Reading XML

Reading XML can be done by passing the XML string or file into the XML reader and using one of the many methods to search and find a specific element or value. You can also convert every element into an easily traversable array if you prefer.&#x20;

XML Wrangler provides methods to iterate through multiple elements while only keeping one element in memory at a time.

### Quick Start

Take the following XML:

```xml
<breakfast_menu>
  <food soldOut="false" bestSeller="true">
    <name>Belgian Waffles</name>
    <price>$5.95</price>
    <description>Two of our famous Belgian Waffles with plenty of real maple syrup</description>
    <calories>650</calories>
  </food>
  <food soldOut="false" bestSeller="false">
    <name>Strawberry Belgian Waffles</name>
    <price>$7.95</price>
    <description>Light Belgian waffles covered with strawberries and whipped cream</description>
    <calories>900</calories>
  </food>
  <food soldOut="false" bestSeller="true">
    <name>Berry-Berry Belgian Waffles</name>
    <price>$8.95</price>
    <description>Light Belgian waffles covered with an assortment of fresh berries and whipped cream</description>
    <calories>900</calories>
  </food>
</breakfast_menu>
```

You can load the XML from various data formats like string, file, stream or PSR-7 response.

```php
<?php

use Saloon\XmlWrangler\XmlReader;

$reader = XmlReader::fromString($xml);
```

After that, you can use various methods to read the XML. If you want to get an array of all of the elements and their values, you can use the `values` or `elements` methods. This is similar to PHP's `json_decode` method for JSON.

```php
$reader->values();

// ['breakfast_menu' => [['name' => '...'], ['name' => '...'], ['name' => '...']]

$reader->elements();

// ['breakfast_menu' => new Element([new Element('name'), new Element('name'), ...])]
```

&#x20;If you want to iterate through the XML in a memory-efficient way, you can use the `element` or `value` methods and define a dot-notation string to search for XML.

```php
$reader->value('food.0')->sole();

// ['name' => 'Belgian Waffles', 'price' => '$5.95', ...]

// Use the element method to get a simple Element DTO containing attributes and content

$reader->element('food.0')->sole(); 

// Element::class
```

XML Wrangler also supports using XPath.

```php
$reader->xpathValue('//food[@bestSeller="true"]/name')->get(); 

// ['Belgian Waffles', 'Berry-Berry Belgian Waffles']
```

### Reading XML Contents

The XML reader can accept a variety of input types. You can use an XML string, file, or provide a resource. You can also read the XML directly from a PSR response (like from [Guzzle](https://github.com/guzzle/guzzle)) or a [Saloon](https://github.com/saloonphp/saloon) response.

```php
<?php

use Saloon\XmlWrangler\XmlReader;

$reader = XmlReader::fromString('<?xml version="1.0" encoding="utf-8"?><breakfast_menu>...');
$reader = XmlReader::fromFile('path/to/file.xml');
$reader = XmlReader::fromStream(fopen('path/to/file.xml', 'rb');
$reader = XmlReader::fromPsrResponse($response);
$reader = XmlReader::fromSaloonResponse($response);
```

{% hint style="info" %}
Due to limitations of the underlying PHP XMLReader class, the `fromStream`, `fromPsrResponse` and `fromSaloon` methods will create a temporary file on your machine/server which will be automatically removed when the reader is destructed. You will need to ensure that you have enough storage on your machine to use these methods.
{% endhint %}

### **Converting Everything Into An Array**

You can use the `elements` and `values` methods to convert the whole XML document into an array. If you would like an array of values, use the `values` method - but if you need to access attributes on the elements, the `elements` method will return an array of `Element` DTOs.

```php
$reader = XmlReader::fromString(...);

$elements = $reader->elements(); 

// Array of `Element::class` DTOs

$values = $reader->values(); 

// Array of values.
```

### Reading Specific Values

You can use the `value` method to get a specific element's value. You can use dot notation to search for child elements. You can also use whole numbers to find specific positions of multiple elements. This method searches through the whole XML body in a memory-efficient way.

This method will return a `LazyQuery` class which has different methods on to retrieve the data.

```php
$reader = XmlReader::fromString('
    <?xml version="1.0" encoding="utf-8"?>
    <person>
        <name>Sammyjo20</name>
        <favourite-songs>
            <song>Luke Combs - When It Rains It Pours</song>
            <song>Sam Ryder - SPACE MAN</song>
            <song>London Symfony Orchestra - Starfield Suite</song>
        </favourite-songs>
    </person>
');

$reader->value('person.name')->sole();

// 'Sammyjo20'

$reader->value('song')->get(); 

// ['Luke Combs - When It Rains It Pours', 'Sam Ryder - SPACE MAN', ...]

$reader->value('song.2')->sole(); 

// 'London Symfony Orchestra - Starfield Suite'
```

### Reading Specific Elements

You can use the `element` method to search for a specific element. This method will provide a `Element` class which contains the value and attributes. You can use dot notation to search for child elements. You can also use whole numbers to find specific positions of multiple elements. This method searches through the whole XML body in a memory-efficient way.

This method will return a `LazyQuery` class which has different methods to retrieve the data.

```php
$reader = XmlReader::fromString('
    <?xml version="1.0" encoding="utf-8"?>
    <person>
        <name>Sammyjo20</name>
        <favourite-songs>
            <song>Luke Combs - When It Rains It Pours</song>
            <song>Sam Ryder - SPACE MAN</song>
            <song>London Symfony Orchestra - Starfield Suite</song>
        </favourite-songs>
    </person>
');

$reader->element('name')->sole(); 

// Element('Sammyjo20')

$reader->element('song')->get(); 

// [Element('Luke Combs - When It Rains It Pours'), Element('Sam Ryder - SPACE MAN'), ...]

$reader->element('song.2')->sole(); 

// Element('London Symfony Orchestra - Starfield Suite')
```

### Removing Namespaces & Prefixes

Sometimes it's easier to traverse an XML document when you don't have to worry about namespaces and prefixes to elements. If you would like to remove them you can use the `removeNamespaces()` method on the reader.

```php
$reader = XmlReader::fromString(...);

$reader->removeNamespaces();
```

### Searching Specific Attributes

Sometimes you might want to search for a specific element or value where the element contains a specific attribute. You can do this by providing a second argument to the `value` or `element` method. This will search the last element for the attributes and will return if they match.

```php
$reader = XmlReader::fromString('
    <?xml version="1.0" encoding="utf-8"?>
    <person>
        <name>Sammyjo20</name>
        <favourite-songs>
            <song>Luke Combs - When It Rains It Pours</song>
            <song>Sam Ryder - SPACE MAN</song>
            <song recent="true">London Symfony Orchestra - Starfield Suite</song>
        </favourite-songs>
    </person>
');

$reader->element('song', ['recent' => 'true'])->sole();

// Element('London Symfony Orchestra - Starfield Suite')

$reader->value('song', ['recent' => 'true'])->sole(); 

// 'London Symfony Orchestra - Starfield Suite'
```

### XPath

XPath is a fantastic way to search through XML. With one string, you can search for a specific element, with specific attributes or indexes. If you are interested in learning XPath, you can [click here for a useful cheatsheet](https://devhints.io/xpath).

**Reading Specific Values via XPath**

You can use the `xpathValue` method to find a specific element's value with an XPath query. This method will return a `Query` class which has different methods to retrieve the data.

```php
<?php
$reader = XmlReader::fromString(...);

$reader->xpathValue('//person/favourite-songs/song[3]')->sole(); 

//  'London Symfony Orchestra - Starfield Suite'
```

**Reading Specific Elements via XPath**

You can use the `xpathElement` method to find a specific element with an XPath query. This method will return a `Query` class which has different methods to retrieve the data.

```php
<?php

$reader = XmlReader::fromString(...);

$reader->xpathElement('//person/favourite-songs/song[3]')->sole(); 

//  Element('London Symfony Orchestra - Starfield Suite')
```

> **Warning** Due to limitations with XPath - the above methods used to query with XPath are not memory safe and may not be suitable for large XML documents.

**XPath and un-prefixed namespaces**

You might find yourself with an XML document that contains an un-prefixed `xmlns` attribute - like this:

```xml
<container xmlns="http://example.com/xml-wrangler/person" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" />
```

When this happens, XML Wrangler will automatically remove these un-prefixed namespaces to improve compatibility. If you would like to keep these namespaces, you can use `setXpathNamespaceMap` to map each un-prefixed XML namespace.

```php
$reader = XmlReader::fromString(...);
$reader->setXpathNamespaceMap([
    'root' => 'http://example.com/xml-wrangler/person',
]);

$reader->xpathValue('//root:person/root:favourite-songs/root:song[3]')->sole();
```

### Query / LazyQuery Methods

The `element`, `value`, `xpathElement` and `xpathValue` methods all return an instance of `Query`. This is a special class which you can use to control the response value.

{% hint style="info" %}
If you are using the `element` methods you will receive `Element` objects, but if you want just the values of the XML elements, you should use the `value` methods.
{% endhint %}

#### First

The first method will return the first element found by a given search and will return `null` if the element could not be found.

{% tabs %}
{% tab title="Element" %}
```php
$reader->element('name')->first(); // Element('Belgian Waffles')
```
{% endtab %}

{% tab title="Value" %}
```php
$reader->value('name')->first(); // 'Belgian Waffles'
```
{% endtab %}
{% endtabs %}

#### FirstOrFail

The `firstOrFail` method is similar to `first` but will throw an exception if no elements can be found.

{% tabs %}
{% tab title="Element" %}
```php
$reader->element('name')->firstOrFail(); // Element('Belgian Waffles');
```
{% endtab %}

{% tab title="Value" %}
```php
$reader->value('name')->firstOrFail(); // 'Belgian Waffles'
```
{% endtab %}
{% endtabs %}

#### Sole

The `sole` method is the same as `firstOrFail` but will also throw an exception if more than one element is found, ensuring you have the "sole" item.

{% tabs %}
{% tab title="Element" %}
```php
$reader->element('food.0.name')->sole(); // Element('Belgian Waffles');
```
{% endtab %}

{% tab title="Value" %}
```php
$reader->value('food.0.name')->sole(); // 'Belgian Waffles'
```
{% endtab %}
{% endtabs %}

#### Get

The `get` method will return an array of elements or values.

{% tabs %}
{% tab title="Element" %}
```php
$reader->element('name')->get(); 

// [
//    Element('Belgian Waffles'),
//    Element('Strawberry Belgian Waffles'),
//    Element('Berry-Berry Belgian Waffles'),
// ];
```
{% endtab %}

{% tab title="Value" %}
```php
$reader->value('name')->get(); 

// [
//    'Belgian Waffles',
//    'Strawberry Belgian Waffles',
//    'Berry-Berry Belgian Waff,
// ];
```
{% endtab %}
{% endtabs %}

#### Collect

The `collect` method will return a Laravel Collection of elements or values.

{% tabs %}
{% tab title="Element" %}
```php
$reader->element('name')->collect(); 

// new Collection([
//    Element('Belgian Waffles'),
//    Element('Strawberry Belgian Waffles'),
//    Element('Berry-Berry Belgian Waffles'),
// ]);
```
{% endtab %}

{% tab title="Value" %}
```php
$reader->value('name')->collect(); 

// new Collection([
//    'Belgian Waffles',
//    'Strawberry Belgian Waffles',
//    'Berry-Berry Belgian Waffles',
// ]);
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
The `illuminate/collections` package must be installed for this to work.
{% endhint %}

#### Lazy

The `lazy` method will return a Generator which can be iterated over when dealing with large XML files.

{% tabs %}
{% tab title="Element" %}
```php
$results = $reader->element('name')->lazy();

foreach($results as $result) {
    // Element('Belgian Waffles')
}
```
{% endtab %}

{% tab title="Value" %}
```php
$results = $reader->value('name')->lazy();

foreach($results as $result) {
    // 'Belgian Waffles'
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
The `lazy` method is only available when using the `element` or `value` methods. It's not available for the `xpath` methods.
{% endhint %}

#### LazyCollect

Similar to the `lazy` method, the `collectLazy` method will return a Laravel Lazy Collection.&#x20;

{% tabs %}
{% tab title="Element" %}
```php
$collection = $reader->element('name')->collectLazy(); 

$results = $collection->map(...)->filter(...)->sort(...);

foreach($results as $result) {
    // Element('Belgian Waffles')
}
```
{% endtab %}

{% tab title="Value" %}
```php
$collection = $reader->value('name')->collectLazy(); 

$results = $collection->map(...)->filter(...)->sort(...);

foreach($results as $result) {
    // 'Belgian Waffles'
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
The `illuminate/collections` package must be installed for this to work.
{% endhint %}
