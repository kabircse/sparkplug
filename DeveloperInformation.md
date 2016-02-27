# Introduction #

While usage notes are sufficient for most people, I know that a lot of developers like to hack around, so I'll try to make it easier for you.  Templates are at the very bottom of the page, if that is what you're looking for.


---


# 2 in 1 #

With the release of Version 0.3, the two files that were SparkPlug and Scaffold were consolidated to make a more compact version.

The implementation only changed in that you now need to run a function after loading the library.  I know CodeIgniter can pass multiple parameters, but you have to use an array, and I think this:
```
$this->load->library('sparkPlug', 'test');
$this->sparkplug->scaffold();
```
looks better than this:
```
$this->load->library('sparkPlug', array('test', 'scaffold'));
```

It also makes the source code a lot more readable.  If you decide to only use one portion of the library, you can still get the old functionality by adding `$this->scaffold()` or `this->generate()` at the end of the constructor.


---


# Structure #

Functions are listed as they appear in the file. The headers Dynamic and Generate are represented as big starry boxes in the source.  Refer to the source code comments for more information.

## Class ##
  * `SparkPlug() - Constructor`
  * `scaffold() - Acts as the front-controller for the dynamic page`
  * `generate() - Section specific Constructor for the generation process`
### Dynamic ###
  * `_processRequest() - resolves the dynamic url and routes posts`
  * Database Functions:
    * `_db_insert - takes all post variables (except for action and submit) and creates a new row`
    * `_db_edit - same as insert, but updates an existing row`
    * `_db_delete - deletes a database row based on id`
  * `_dynamic($action) - gets the requested page and calls the proper display function | defaults to the list view`
  * View Functions:
    * `_delete() - calls the _db_delete function | here just to have a url for the delete link`
    * `_list() - the main page, shows a tabular view of all the data in the scaffolded table with links to various actions`
    * `_show() - shows a row based on the id in the url`
    * `_insert() - shows a page with a form to add a new row`
    * `_edit() - shows row's contents in a table for easy changes`
      * `_insertMarkup($field) - generates a form field for the insert view based on the meta-type of the field`
      * `_editMarkup($field, $data) - same as insertMarkup, but populates the field with the given data`
  * Helper Functions:
    * `_delete_link($id) - creates a delete link for the id`
    * `_edit_link($id) - creates an edit link for the id`
    * `_show_link($id) - creates a show link for the id`
    * `_insert_link() - creates a link to the new-entry page`
    * `_back_link() - creates a link to the main list view`
    * `_header() - echos a basic html header`
    * `_footer() - closes any tags opened by the header`
### Generate ###
  * `_generate() - figures out the paths, and creates the files`
  * Model Functions:
    * `_generate_model() - evalutes the template, fixes the tags and returns the final text`
  * View Functions:
    * `_generate_views() - grabs the view templates, replaces tags and returns all the views in an array`
  * Controller Functions:
    * `_generate_controller() - takes the controller template, replaces the tags and returns the text`
  * Helper Functions:
    * `_fix_indent - helper to make sure that tags with multiple lines are indented properly`
    * `_form_fields($action) - entry point for form generation | form type depends on $action`
      * `_getMarkup($field) - Creates a form field based on the meta-type`
      * `_getEditMarkup($field) - Same as getMarkup, but adds existing data`
  * Templates - more detail below:
    * `_controller_text() - returns the controller template`
    * `_model_text() - returns the model template`
    * `_index_view() - returns index view`
    * `_list_view() - returns list view`
    * `_show_view() - returns show view`
    * `_edit_view() - returns edit view`
    * `_new_view() - returns new view`


---


# Templates #

SparkPlug provides for basic templating of the generated files before you run the generation.  This is useful if you use it frequently (more than once).  Changing code to fit your style once is annoying, but doing it multiple times is plain tedious.

This is the part of the library that will probably see the most change in the next weeks.  It's probably also the part that you don't want changing a lot.  It's an unfortunate coincidence, but it can't be helped.  I definitely want them to be more flexible than they currently are.

### Why are they indented funny ###
To make sure that the generated files look clean, I needed indentation to be from the left edge of the source file.  It doesn't look as clean, but it ensures that you get what you see.

### What about my own tags? ###

If you want different values for your tags you will need to look in the `_generate_` function of the view type.  Custom tags with a better place for definitions is a feature I'm planning.

## Controller ##
| {model} | Replaced by the model name, use it for loading and calling models |
|:--------|:------------------------------------------------------------------|
| {ucf\_controller} | Controller name in upper case for the class name, constructor, etc |
| {controller} | the controller as it is written in the url for anchors, redirects, etc |
| {view\_folder} | name of the generated view folder, default is the same as {controller} |

**Default:**

```
'<?php
class {ucf_controller} extends Controller {

	function {ucf_controller}() {
		parent::Controller();
		
		$this->load->database();
		$this->load->model(\'{model}\');
		$this->load->helper(\'url\');
		$this->load->helper(\'form\');
		$this->load->library(\'session\');
	}
		
	function index() {
		redirect(\'{controller}/show_list\');
	}

	function show_list() {
		$data[\'results\'] = $this->{model}->get_all();
		$this->load->view(\'{view_folder}/list\', $data);
	}

	function show($id) {
		$data[\'result\'] = $this->{model}->get($id);
		$this->load->view(\'{view_folder}/show\', $data);		
	}

	function new_entry() {
		$this->load->view(\'{view_folder}/new\');
	}

	function create() {
		$this->{model}->insert();
		
		$this->session->set_flashdata(\'msg\', \'Entry Created\');
		redirect(\'{controller}/show_list\');
	}

	function edit($id) {
		$res = $this->{model}->get($id);
		$data[\'result\'] = $res[0];
		$this->load->view(\'{view_folder}/edit\', $data);
	}

	function update() {
		$this->{model}->update();
		
		$this->session->set_flashdata(\'msg\', \'Entry Updated\');
		redirect(\'{controller}/show_list\');
	}

	function delete($id) {
		$this->{model}->delete($id);
		
		$this->session->set_flashdata(\'msg\', \'Entry Deleted\');
		redirect(\'{controller}/show_list\');
	}
}'
```

## Model ##
| {model\_name} | Upper case model name for use in the class name and constructor |
|:--------------|:----------------------------------------------------------------|
| {table}       | Name of the scaffolded table                                    |
| {variables}   | Variables initializations                                       |
| {set\_variables\_from\_post} | All the variables replaced by their $_POST counterparts_|

**Default:**

```
'<?php
class {model_name} extends Model {
	{variables}

	function {model_name}() {
		parent::Model();
	}

	function insert() {
		{set_variables_from_post}

		$this->db->insert(\'{table}\', $this);
	}

	function get($id) {
		$query = $this->db->get_where(\'{table}\', array(\'id\' => $id));
		return $query->result_array();
	}
	
	function get_all() {
		$query = $this->db->get(\'{table}\');
		return $query->result_array();
	}
	
	function get_field_data() {
		return $this->db->field_data(\'{table}\');
	}

	function update() {
		{set_variables_from_post}

		$this->db->update(\'{table}\', $this, array(\'id\' => $_POST[\'id\']));
	}

	function delete($id) {
		$this->db->delete(\'{table}\', array(\'id\' => $id));
	}
}'
```

## Views ##
| {controller} | Controller name for links, etc |
|:-------------|:-------------------------------|
| {form\_fields\_create} | Empty form entry fields for all table fields |
| {form\_fields\_update} | Same as {form\_fields\_create}, but with existing data passed to view |

**Default - Index:**

```
'You should not see this after scaffolding - index controller redirect by default.'
```

**Default - List:**

```
'<p style="color: green"><?= $this->session->flashdata(\'msg\') ?></p>

<h1>List</h1>

<table>
	<tr>
	<? foreach(array_keys($results[0]) as $key): ?>
		<th><?= ucfirst($key) ?></th>
	<? endforeach; ?>
	</tr>

<? foreach ($results as $row): ?>
	<tr>
	<? foreach ($row as $field_value): ?>
		<td><?= $field_value ?></td>
	<? endforeach; ?>
		<td> <?= anchor("{controller}/show/".$row[\'id\'], \'View\') ?></td>
		<td> <?= anchor("{controller}/edit/".$row[\'id\'], \'Edit\') ?></td>
		<td> <?= anchor("{controller}/delete/".$row[\'id\'], \'Delete\') ?></td>
	</tr>
<? endforeach; ?>
</table>
<?= anchor("{controller}/new_entry", "New") ?>'
```

**Default - Show:**

```
'<h1>Show</h1>

<? foreach ($result[0] as $field_name => $field_value): ?>
<p>
	<b><?= ucfirst($field_name) ?>:</b> <?= $field_value ?>
</p>
<? endforeach; ?>
<?= anchor("{controller}/show_list", "Back") ?>'
```

**Default - Edit:**

```
'<h1>Edit</h1>

<?= form_open(\'{controller}/update\') ?>
{form_fields_update}
<p>
	<?= form_submit(\'submit\', \'Update\') ?>
</p>
<?= form_close() ?>
<?= anchor("{controller}/show_list", "Back") ?>'
```

**Default - New:**

```
'<h1>New</h1>

<?= form_open(\'{controller}/create\') ?>
{form_fields_create}
<p>
	<?= form_submit(\'submit\', \'Create\') ?>
</p>
<?= form_close() ?>
<?= anchor("{controller}/show_list", "Back") ?>'
```