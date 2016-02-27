# Usage #

## SparkPlug.php v0.4 ##

Remove the version info from the file name (rename to SparkPlug.php) then add the file to your application/controllers folder.

You can then access the scaffolding feature at:
http://yourhost/SparkPlug/scaffolding/{table_name}

For example, to scaffold a table named 'users':
http://yourhost/SparkPlug/scaffolding/users

If instead of scaffolding you want to generate the actual files:
http://yourhost/SparkPlug/generate/users

**DO NOT include this file when pushing your website to a production server or any server that can be accessed publicly.  It allows almost unrestricted access to your database by ANYONE.**

## SparkPlug.php v0.31 ##

Add the file to your application/libraries folder - I renamed mine to SparkPlug for simplicity.

In the controller/function you want to scaffold load the class and pass the table name as a parameter:

```
<?php
class Scaffold_test extends Controller {

	function Scaffold_test() {
		parent::Controller();
	}

	function index() {
		$this->load->library('sparkPlug', 'test');
		$this->sparkplug->scaffold();
                /* OR */
		$this->sparkplug->generate();
	}
}
```

Use **either** scaffold or generate.  Do **not** use both.

  1. `$this->sparkplug->scaffold();` is similar to the native CI scaffolding.
  1. `$this->sparkplug->generate();` generates code to provide the same features as scaffold() did.

Scaffold will present you with all the functions generate gives you, but it does not generate any code.  Use scaffold for quick changes in a table that does not require full blown access.

Using Generate the current controller will be overwritten with any code you need.


In this case, going to the index function in your browser will run the scaffolding.  There is **NO** warning, yet.  It just does it's thing, no questions asked.  Read Caution statement at the bottom of the page.


# Caution - when using SparkPlug #

  * Existing files that share a name with the generated version (such as the controller you're calling the scaffold from) will be **replaced**, any existing code will be lost.
  * I highly suggest you make sure the paths the script uses are correct.
  * I am **NOT** responsible for any lost data.