# ✏ Writing XML

Writing XML is as simple as defining a PHP array and using keys and values to define elements. When you need to define elements with a few more characteristics like attributes or namespaces you can use the `Element` DTO to define more advanced elements.

### Quick Start

{% tabs %}
{% tab title="Definition" %}
```php
<?php

use Saloon\XmlWrangler\Data\Element;
use Saloon\XmlWrangler\XmlWriter;

$writer = new XmlWriter;

$xml = $writer->write('breakfast_menu', [
    'food' => [
        [
            'name' => 'Belgian Waffles',
            'price' => '$5.95',
            'description' => 'Two of our famous Belgian Waffles with plenty of real maple syrup',
            'calories' => '650',
        ],
        [
            'name' => 'Strawberry Belgian Waffles',
            'price' => '$7.95',
            'description' => 'Light Belgian waffles covered with strawberries and whipped cream',
            'calories' => '900',
        ],

        // You can also use the Element class if you need to define elements with
        // namespaces or with attributes.

        Element::make([
            'name' => 'Berry-Berry Belgian Waffles',
            'price' => '$8.95',
            'description' => 'Light Belgian waffles covered with an assortment of fresh berries and whipped cream',
            'calories' => '900',
        ])->setAttributes(['bestSeller' => 'true']),
    ],
]);
```
{% endtab %}

{% tab title="Result" %}
```xml
<?xml version="1.0" encoding="utf-8"?>
<breakfast_menu>
  <food>
    <name>Belgian Waffles</name>
    <price>$5.95</price>
    <description>Two of our famous Belgian Waffles with plenty of real maple syrup</description>
    <calories>650</calories>
  </food>
  <food>
    <name>Strawberry Belgian Waffles</name>
    <price>$7.95</price>
    <description>Light Belgian waffles covered with strawberries and whipped cream</description>
    <calories>900</calories>
  </food>
  <food bestSeller="true">
    <name>Berry-Berry Belgian Waffles</name>
    <price>$8.95</price>
    <description>Light Belgian waffles covered with an assortment of fresh berries and whipped cream</description>
    <calories>900</calories>
  </food>
</breakfast_menu>
```
{% endtab %}
{% endtabs %}

### **Basic Usage**

The most basic usage of the reader is to use string keys for the element names and values for the values of the element. The writer accepts infinitely nested arrays for nested elements.

```php
use Saloon\XmlWrangler\XmlWriter;

$xml = XmlWriter::make()->write('root', [
    'name' => 'Sam',
    'twitter' => '@carre_sam',
    'facts' => [
        'favourite-song' => 'Luke Combs - When It Rains It Pours'
    ],
]);
```

The above code will be converted into the following XML

```xml
<?xml version="1.0" encoding="utf-8"?>
<root>
  <name>Sam</name>
  <twitter>@carre_sam</twitter>
  <facts>
    <favourite-song>Luke Combs - When It Rains It Pours</favourite-song>
  </facts>
</root>
```

### **Using the Element DTO**

When writing XML, you will often need to define attributes and namespaces on your elements. You can use the `Element` class in the array of XML to add an element with an attribute or namespace. You can mix the `Element` class with other arrays and string values.

```php
use Saloon\XmlWrangler\XmlWriter;
use Saloon\XmlWrangler\Data\Element;

$xml = XmlWriter::make()->write('root', [
    'name' => 'Sam',
    'twitter' => Element::make('@carre_sam')->addAttribute('url', 'https://twitter.com/@carre_sam'),
    'facts' => [
        'favourite-song' => 'Luke Combs - When It Rains It Pours'
    ],
    'soap:custom-namespace' => Element::make()->addNamespace('soap', 'http://www.w3.org/2003/05/soap-envelope'),
]);
```

This will result in the following XML

```xml
<?xml version="1.0" encoding="utf-8"?>
<root>
  <name>Sam</name>
  <twitter url="https://twitter.com/@carre_sam">@carre_sam</twitter>
  <facts>
    <favourite-song>Luke Combs - When It Rains It Pours</favourite-song>
  </facts>
  <soap:custom-namespace xmlns:soap="http://www.w3.org/2003/05/soap-envelope"/>
</root>
```

### **Arrays Of Values**

You will often need to define an array of elements. You can do this by simply providing an array of values or element classes.

```php
use Saloon\XmlWrangler\XmlWriter;
use Saloon\XmlWrangler\Data\Element;

$xml = XmlWriter::make()->write('root', [
    'name' => 'Luke Combs',
    'songs' => [
        'song' => [
            'Fast Car',
            'The Kind Of Love We Make',
            'Beautiful Crazy',
            Element::make('She Got The Best Of Me')->addAttribute('hit', 'true'),
        ],
    ],
]);
```

This will result in the following XML

```xml
<?xml version="1.0" encoding="utf-8"?>
<root>
  <name>Luke Combs</name>
  <songs>
    <song>Fast Car</song>
    <song>The Kind Of Love We Make</song>
    <song>Beautiful Crazy</song>
    <song hit="true">She Got The Best Of Me</song>
  </songs>
</root>
```

### **Customising the root element**

Sometimes you may need to change the name of the root element. This can be customised as the first argument of the `write` method.

```php
$xml = XmlWriter::make()->write('custom-root', [...])
```

If you would like to add attributes and namespaces to the root element you can use a `RootElement` class here too.

```php
use Saloon\XmlWrangler\Data\RootElement;

$rootElement = RootElement::make('root')->addNamespace('soap', 'http://www.w3.org/2003/05/soap-envelope');

$xml = XmlWriter::make()->write($rootElement, [...])
```

### **CDATA Element**

If you need to add a CDATA tag you can use the `CDATA` class.

```php
use Saloon\XmlWrangler\Data\CDATA;use Saloon\XmlWrangler\XmlWriter;
use Saloon\XmlWrangler\Data\Element;

$xml = XmlWriter::make()->write('root', [
    'name' => 'Sam',
    'custom' => CDATA::make('Here is some CDATA content!'),
]);
```

This will result in the following XML

```xml
<?xml version="1.0" encoding="utf-8"?>
<root>
  <name>Sam</name>
  <custom><![CDATA[Here is some CDATA content!]]></custom>
</root>
```

### **Customising XML encoding, version, and standalone**

The default XML encoding is `utf-8`, the default version of XML is `1.0`, and the default standalone is `null` (XML parsers interpret no standalone attribute the same as `false`). If you would like to customise this you can with the `setXmlEncoding`, `setXmlVersion`, and `setXmlStandalone` methods on the writer.

```php
use Saloon\XmlWrangler\XmlWriter;

$writer = new XmlWriter();

$writer->setXmlEncoding('ISO-8859-1');
$writer->setXmlVersion('2.0');
$writer->setXmlStandalone(true);

// $writer->write(...);
```

Which results in the XML declaration `<?xml version="2.0" encoding="ISO-8859-1" standalone="yes"?>`.

### **Adding custom "Processing Instructions" to the XML**

You can add a custom "Processing Instruction" to the XML by using the `addProcessingInstruction` method.

```php
use Saloon\XmlWrangler\XmlWriter;

$writer = new XmlWriter();
$writer->addProcessingInstruction('xml-stylesheet', 'type="text/xsl" href="base.xsl"');

$xml = $writer->write('root', ['name' => 'Sam']);
```

This will result in the following XML

```xml
<?xml version="1.0" encoding="utf-8"?>
<?xml-stylesheet type="text/xsl" href="base.xsl"?>
<root>
  <name>Sam</name>
</root>
```

### **Minification**

By default the XML written is not minified. You can provide the third argument to the `write` method to minify the XML.

```php
use Saloon\XmlWrangler\XmlWriter;

$xml = XmlWriter::make()->write('root', [...], minified: true);
```
