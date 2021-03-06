---

layout: rad.twig
title: Form Dependent

---

## List Dependent

Phoenix includes a simple `list-dependent` JS library to help you load `<select>` list from ajax value:

First, prepare two select list:

```php
// #input-item-country
$this->list('country')
    ->label('Country')
    ->option('-- Select Country --', '')
    ->option('USA', 1)
    ->option('Russia', 2);

// #input-item-city
$this->list('city')
    ->label('City');
```

Set dependency script:

```php
PhoenixScript::listDependent(
    '#input-item-city', 
    '#input-item-country',
    CoreRouter::route('flower@ajax_city')
);
```

And you must have a route and a controller to return list data:

```yaml
ajax_city:
    pattern: /ajax/cities
    controller: Cities
```

Controller:

```php
namespace Asuka\Flower\Controller\Cities;

use Windwalker\Core\Controller\AbstractController;
use Windwalker\Core\Controller\Traits\JsonApiTrait;

class GetController extends AbstractController
{
	use JsonApiTrait;

	protected function doExecute()
	{
		$value = $this->input->get('value');
		$data = [
			['title' => '-- Select City --']
		];
		
		foreach (CityMapper::find(['country_id' => $value]) => $city)
		{
		    $data[] = [
		        'id' => $city->id,
		        'title' => $city->title
		    ];
		}

		return $data;
	}
}
```

The result is:

![Imgur](https://i.imgur.com/HlMnft4.gif)

### Use Array as Source

You can also set an array as your source instead call ajax:

```php
<?
PhoenixScript::listDependent(
    '#input-item-city', 
    '#input-item-country',
    [
        1 => [
            ['id' => 1, 'title' => 'New York'],
            ['id' => 2, 'title' => 'DC'],
        ],
        2 => [
            ['id' => 3, 'title' => 'Moscow'],
            ['id' => 4, 'title' => 'Saint Petersburg'],
        ]
    ]
);
```

If you want to set a default selection with empty value, use `__EMPTY__` as key name, now the option with value "" 
will get `-- Select A Country First --` to notice users.:

```php
PhoenixScript::listDependent(
    '#input-item-city', 
    '#input-item-country',
    [
        '__EMPTY__' => [
            ['id' => '', 'title' => '-- Select A Country First --']
        ],
        1 => [
            ['id' => 1, 'title' => 'New York'],
            ['id' => 2, 'title' => 'DC'],
        ],
        2 => [
            ['id' => 3, 'title' => 'Moscow'],
            ['id' => 4, 'title' => 'Saint Petersburg'],
        ]
    ]
);
```

### Options

```php
PhoenixScript::listDependent(
	'#input-item-city', 
	'#input-item-country', 
	CoreRouter::route('flower@ajax_city'), 
	[
        'default_value' => null, // Set default value.
        'initial_load' => true, // Load list first after page initial?
        'ajax' => [
            'value_field' => 'value' // Change request value field
        ],
        'text_field' => 'title', // Which field in response should add to option text
        'value_field' => 'id', // Which field in response should add to option value
        'first_option' => null, // Auto add first option to all result: [id: ..., title: ...]
        'empty_mark' => '__EMPTY__', // If you have an option's with "" value, it will get list by key: `__EMPTY__` from your source.
        'hooks' => [
            'before_request' => '\\function () { ... }',
            'after_request' => '\\function () { ... }',
        ]
    ]
);
```

### Hooks

You can add your logic before and after process:

```php
JQueryScript::ui(['effect']);
PhoenixScript::listDependent('#input-item-city', '#input-item-country', CoreRouter::route('flower@ajax_city'), 
    [
        'hooks' => [
            'before_request' => "\\function (value, ele, depEle) {
                ele.attr('disabled', true);
            }",
            'after_request' => "\\function (value, ele, depEle) {
                ele.attr('disabled', false);
                ele.effect('highlight');
            }"
        ]
    ]
);
```

![Imgur](https://i.imgur.com/U9PDX52.gif)

## Fields Show On (dependsOn)

Assume we have these fields, add `showon` to field config and set the target field name with value.

```php
$this->list('use_ftp')
    ->label('Use FTP')
    ->option('Yes', 1)
    ->option('No', 0);

$this->text('ftp_user')
    ->label('User')
    ->set('showon', ['use_ftp' => '1']);

$this->password('ftp_pass')
    ->label('Pass')
    ->set('showon', ['use_ftp' => '1']);
```

Now, user and pass field will show if `use_ftp` field is `1`, otherwise will hide.

![Imgur](https://i.imgur.com/rwv911F.gif)

### Use Multiple Conditions

Add multiple values will be `AND` condition:

```php
    ->set('showon', ['value1' => '1', 'value2' => 'foo']);
```

If you want to use `OR` conditions, you must implement this logic in JS manually:

```js
$('#orTest .subject').dependsOn({
	'#orTest input[name="xtraCheese"]': {
		checked: true
	}
}, {
	hide: false
}).or({
	'#orTest select': {
		not: ['select']
	}
});
```

See [dependsOn: Advanced Usage](http://dstreet.github.io/dependsOn/)

### Use Other Qualifiers

By default, Phoenix only use `values` as qualifier, `dependsOn.js` has many other qualifier, we can add it by PHP:

```php
JQueryScript::dependsOn('#emailTest .subject', [
	'#emailTest input[type="checkbox"]' => [
	    'checked' => true
	],
	'#emailTest .email' => [
	    'email' => true
	]
]);
```

Available Qualifiers:

| Qualifier | Description                                                                                                                              | Type     |
|-----------|------------------------------------------------------------------------------------------------------------------------------------------|----------|
| enabled   | If true, then dependency must not have the "disabled" attribute.                                                                       | Boolean  |
| checked   | If true, then dependency must not have the "checked" attribute. Used for checkboxes and radio   buttons.                               | Boolean  |
| values    | Dependency value must equal one of the provided values.                                                                                | Array    |
| not       | Dependency value must not equal any of the provided values.                                                                            | Array    |
| match     | Dependency value must match the regular expression.                                                                                    | RegEx    |
| contains  | One of the provided values must be contained in an array of dependency values. Used for select fields with the   "multiple" attribute. | Array    |
| email     | If true, dependency value must match an email address.                                                                                 | Boolean  |
| url       | If true, Dependency value must match an URL.                                                                                           | Boolean  |
| Custom    | Custom function which return true or false.                                                                                            | Function |

See [jQuery dependsOn.js](http://dstreet.github.io/dependsOn/)
