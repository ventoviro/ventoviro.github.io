---
layout: documentation.twig
title: Form Definition

---

## Use `AbstractFieldDefinition`

Windwalker provides a simple interface to define form field. Just create a class extends to `Windwalker\Core\Form\AbstractFieldDefinition`
and add field type by `$this->{type}('{name}')`.

``` php
<?php

use Windwalker\Core\Form\AbstractFieldDefinition;
use Windwalker\Form\Form;

class EditDefinition extends AbstractFieldDefinition
{
	protected function doDefine(Form $form)
	{
		$this->fieldset('basic', function (Form $form)
		{
		    $this->hidden('id');

			$this->text('title')->label('Title')
				->setClass('input-large')
				->set('placeholder', 'Please enter title')
				->required();

			$this->textarea('content')->label('Content')
				->set('rows', 15)
				->required();
		});
	}
}
```

Then register this class to `Form`:

``` php
use Windwalker\Form\Form;

$form = new Form;
$form->defineFormFields(new EditDefinition);

echo $form->renderFields('basic');
```

The output:

``` php
<div id="input-id-control" class="hidden-field ">
    <label id="input-id-label" for="input-id"></label>
    <input type="hidden" name="id" id="input-id" />
</div>
<div id="input-title-control" class="text-field ">
    <label id="input-title-label" for="input-title"><span class="windwalker-input-required-hint">*</span> Title</label>
    <input type="text" name="title" id="input-title" class="input-large" placeholder="Please enter title" value="" required />
</div>
<div id="input-content-control" class="textarea-field ">
    <label id="input-content-label" for="input-content"><span class="windwalker-input-required-hint">*</span> Content</label>
    <textarea name="content" id="input-content" required rows="15"></textarea>
</div>
```

### Custom Field Class

You can also use `add()` method to add custom field class:

``` php
use Windwalker\Form\Field\AbstractField;

class MyField extends AbstractField
{
    // ...
}
```

``` php
// in AbstractFieldDefinition

$this->add('field_name', new MyField);
```

There has three methods: `fieldset()`, `group()` and `wrap()` are alias to methods with same name in `Form`, See [Form Builder](form-builder.html)

### Available Fields

- `text($name = null, $label = null)`
- `button($name = null, $label = null)`
- `checkbox($name = null, $label = null)`
- `checkboxes($name = null, $label = null)`
- `customHtml($name = null, $label = null)`
- `email($name = null, $label = null)`
- `hidden($name = null, $label = null)`
- `list($name = null, $label = null)`
- `password($name = null, $label = null)`
- `radio($name = null, $label = null)`
- `spacer($name = null, $label = null)`
- `textarea($name = null, $label = null)`
- `timezone($name = null, $label = null)`

## Add Custom Methods by Traits

If you have a lot of custom fields, you can register them to `AbstractFieldDefinition`.

First, you must put all fields in one folder with same namespace, and all field classes' name should follows `{Name}Field` rule.
And create a `trait` with boot method.

``` php
/**
 * The MyFieldTrait class.
 *
 * @method  \MyPackage\Field\FooField foo($name = null, $label = null)
 * @method  \MyPackage\Field\BarField bar($name = null, $label = null)
 * @method  \MyPackage\Field\BazField baz($name = null, $label = null)
 */
trait MyFieldTrait
{
	public function bootMyFieldTrait()
	{
		$this->addNamespace('MyPackage\Field');
	}
}
```

Use this trait in your definition class, then you are able to use `foo()` to get `MyPackage\Field\FooField`:

``` php
class EditDefinition extends AbstractFieldDefinition
{
	use MyFieldTrait;

    protected function doDefine(Form $form)
    {
        // ...

        $this->foo('field_name')
            ->label('Foo');
    }
}
```

If your field naming convention not follows default rule, you can add a method to create it.

``` php
trait MyFieldTrait
{
	// ...

	public function sakura($name = null, $label = null)
	{
		return new MyFieldSakura($name, $label);
	}
}
```

