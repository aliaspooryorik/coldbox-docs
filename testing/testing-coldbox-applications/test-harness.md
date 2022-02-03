# Test Harness

Every ColdBox application template comes with a nice test harness inside of a `tests` folder.

![](../../.gitbook/assets/testharness.png)

Here is a breakdown of what it contains:

* `Application.cfc` - A unique application file for your test harness. It should mimic exactly the one in your root application folder
* `resources` - Some basic testing resources or any of your own testing resources
* `results` - Where automated results are archived
* `runner.cfm` - The HTML runner for your test bundles
* `specs` - Where you will be writing your testing bundle specs for integration testing, unit testing and module testing.
* `test.xml` - Your ANT runner

## Application.cfc

The `Application.cfc` for your tests is extremly important as it should mimic your applications real `Application.cfc`.

```javascript
/**
********************************************************************************
Copyright 2005-2007 ColdBox Framework by Luis Majano and Ortus Solutions, Corp
www.ortussolutions.com
********************************************************************************
*/
component{

	// APPLICATION CFC PROPERTIES
	this.name 				= "ColdBoxTestingSuite" & hash(getCurrentTemplatePath());
	this.sessionManagement 	= true;
	this.sessionTimeout 	= createTimeSpan( 0, 0, 15, 0 );
	this.applicationTimeout = createTimeSpan( 0, 0, 15, 0 );
	this.setClientCookies 	= true;

	// Create testing mapping
	this.mappings[ "/tests" ] = getDirectoryFromPath( getCurrentTemplatePath() );
	// Map back to its root
	rootPath = REReplaceNoCase( this.mappings[ "/tests" ], "tests(\\|/)", "" );
	this.mappings["/root"]   = rootPath;

	public void function onRequestEnd() {

		if( !isNull( application.cbController ) ){
			application.cbController.getLoaderService().processShutdown();
		}

		structDelete( application, "cbController" );
		structDelete( application, "wirebox" );
	}
}
```

Please note that we provide already a mapping to your root application via `/root` and a mapping to the tests themselves via `/tests` .  We would recommend you add any ORM specs here or any other mappings here as well.

{% hint style="success" %}
**Tip:** Make sure all the same settings and configs from your root `Application.cfc` are replicated in your tests `Application.cfc`
{% endhint %}
