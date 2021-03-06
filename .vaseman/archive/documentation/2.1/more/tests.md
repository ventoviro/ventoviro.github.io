---
layout: documentation.twig
title: Tests

---

## Use Unit Test

Windwalker includes [PHPUnit](https://phpunit.de/) as default unit test system.

To start a unit test, please rename `/phpunit.xml.dist` to `/phpunit.xml` and edit the content:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit bootstrap="test/bootstrap.php" colors="false"
	convertErrorsToExceptions="true"
	convertNoticesToExceptions="true"
	convertWarningsToExceptions="true"
	strict="true"
	syntaxCheck="true"
	>
	<php>
		<ini name="error_reporting" value="32767" />

		<const name="WINDWALKER_CORE_TEST_DB_DSN_MYSQL"      value="host=localhost;dbname=windwalker_test;user=root;pass=;prefix=ww_" />
		<!--<const name="WINDWALKER_CORE_TEST_DB_DSN_POSTGRESQL" value="host=localhost;dbname=windwalker_test;user=root;pass=;prefix=ww_" />-->
		<!--<const name="WINDWALKER_CORE_TEST_DB_DSN_ORACLE"     value="host=localhost;port=5432;dbname=windwalker_test;user=root;pass=;prefix=ww_" />-->
		<!--<const name="WINDWALKER_CORE_TEST_DB_DSN_MSSQL"      value="host=localhost;port=1521;dbname=windwalker_test;user=root;pass=;prefix=ww_" />-->
		<!--<const name="WINDWALKER_CORE_TEST_DB_DSN_SQLITE"     value="host=localhost;dbname=windwalker_test;user=root;pass=;prefix=ww_" />-->
	</php>
	<testsuites>
		<testsuite name="Starter">
			<directory>src/*/Test</directory>
		</testsuite>
	</testsuites>
</phpunit>
```

Change the `testsuite` to:

``` xml
<testsuites>
	<testsuite name="Flower">
		<directory>src/Flower/Test</directory>
	</testsuite>
</testsuites>
```

Then we can write all test case in `/src/Flower/Test`.

More about how to writing unit test, please see: [PHPUnit Tutorial](https://phpunit.de/getting-started.html)

## Test Cases

Windwlaker Provides some useful TestCase to extends asserts.

### `AbstractBaseTestCase`

``` php
use Windwalker\Test\TestCase\AbstractBaseTestCase;

class ModelTest extends AbstractBaseTestCase
{
	public function testFoo()
	{
	    // Minify string to one line and remove extra spaces.
		$this->assertStringDataEquals('string data', "string  \n  data");

        // Compare string with LF or CRLF
		$this->assertStringSafeEquals("string \r\ndata", "string \ndata");

        // Match exception type
		$this->assertExpectedException(function()
		{
			throw new \RuntimeException;
		}, new \RuntimeException /* Or use `RuntimeException` string */);
	}
}
```

### `AbstractDomTestCase`

``` php
class ModelTest extends AbstractDomTestCase
{

	public function testFoo()
	{
		$dom = <<<DOM
<root>
	<node foo="bar" />
</root>
DOM;

        // Will minify all string and compare them.
		$this->assertDomStringEqualsDomString('<root><node foo="bar" /></root>', $dom);

        // Will normalize all string and compare them
		$this->assertDomFormatEquals('<root><node foo="bar" /></root>', $dom);

		$html = <<<HTML
<div>
	<input type="text" value="" />
</div>
HTML;

        // Will normalize all string and compare them
		$this->assertHtmlFormatEquals('<div><input type="text" value="" /></div>', $html);
	}
}
```
