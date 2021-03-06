Metabox
=======

As the name suggests, the `Metabox` class helps you build custom WordPress metabox. A metabox is a UI container for your custom fields (post metadata).

Before digging into the `Metabox` documentation, make sure to read the [Field class guide](http://framework.themosis.com/docs/field/).

- Basic usage
	- Set a custom id attribute
	- Custom post type metabox
	- Page metabox
- Sanitize metabox fields
- Retrieve data
- Customize the metabox
- Send data to your metabox

Basic usage
-----------

In order to create a metabox for your post, page or custom post type, you have to call the `Metabox::make()` method.

```php
Metabox::make($title, $postType, $options = [], $view = null)->set($fields);
```

* **$title**: _string_. The display title of the metabox.
* **$postType**: _string_. The post type slug to link the metabox with.
* **$options**: _array_. An array of options. You can set the `context`, `priority` and `id` properties of the metabox.
* **$view**: _IRenderable_. A view file. Allow you to customize the UI of your metabox.

Here is an example of a basic metabox, containing one text field, that is displayed inside your posts edit screen:

```php
Metabox::make('Infos', 'post')->set([
	Field::text('author')
]);
```
The code above will render a metabox with a title of _Infos_ and a single custom text field.

> A metabox is not registered/displayed until you call its `set()` method.

```php
Metabox::make('Infos', 'post')->set();
```

### Set a custom ID attribute

In some scenarios, you might need to be able to specify an ID to your metabox in order to fetch it for JS scripts or for styling. In order to set the id attribute for your metabox, pass it an `id` option like so:

```php
Metabox::make('Infos', 'post', [
    'id' => 'my-custom-id'
])->set();
```

### Custom post type metabox

Here is an example of a metabox attached to a custom post type with a slug of `books`:

```php
// Let's create our custom post type
$books = PostType::make('books', 'Books', 'Book')->set();

// Attach our metabox to our custom post type
Metabox::make('Informations', $books->get('name'))->set();
```

The code above is using the PostType `get()` method in order to fetch the custom post type name. Check the [PostType guide](http://framework.themosis.com/docs/posttype/) for more information.

### Page metabox

In this example, a metabox is displayed on all pages edit screen with some custom fields:

```php
Metabox::make('Informations', 'page')->set([
	Field::text('name'),
	Field::text('phone'),
	Field::textarea('address')
]);
```

> Note: Make sure to always prefix your custom fields name so they don't conflict with the [WordPress reserved terms](https://codex.wordpress.org/Reserved_Terms). We don't prefix names in the documentation (boouuuh...) but you should in your project.

Sanitize/validate metabox fields
--------------------------------

The Metabox API contains an instance of the Validator class which gives you a method to validate/sanitize the custom fields attach to your metabox.

In order to validate/sanitize your fields, use the `validate()` method like so:

```php
$metabox = Metabox::make('A title', 'page')->set([
	Field::text('email'),
	Field::text('name'),
	Field::text('phone'),
	Field::textarea('address'),
	Field::infinite('team', [
		Field::text('name'),
		Field::text('age')
	])
]);

// Let's validate our custom fields
$metabox->validate([
	'email'		=> ['email'],
	'name'		=> ['textfield', 'min:3', 'alpha'],
	'phone'		=> ['num', 'max:25'],
	'address'	=> ['textarea'],
	'team'		=> [
		'name'		=> ['textfield', 'alpha', 'min:3', 'max:50'],
		'age'		=> ['num']
	]
]);
```
> Note how the `infinite` field is validated.

If validation passes, the value entered is registered. In case the validation fails, it returns an empty string value.

The validate method works more like a "sanitizer". This means that currently if an end-user is updating its post and that the value entered in your custom field is not valid, the post is still updated.

> Tip: If you want to make required custom fields, currently we suggest to pass a `required` attribute to your custom field. This will avoid the update of the post until a value is defined inside your custom field. A better required API is planned and will come in a future release of the framework.

Check the [validation guide](http://framework.themosis.com/docs/validation/) for more information about the validation rules.

Retrieve data
-------------

In order to retrieve the custom fields data, you can use the core function `get_post_meta()` or use the `Meta` class. Please refer to the [Meta guide](http://framework.themosis.com/docs/meta/).

Customize the metabox
---------------------

You can customize the look and feel and the behaviour of your metabox by defining a custom view (and why not view composers...).

You can pass a custom view to your metabox using the 4th argument of the `Metabox::make()` method like so:

```php
// Code below written inside the 'admin/metabox.php' file.
// File stored inside the 'views/metabox/custom.scout.php'.
$view = View::make('metabox.custom');

Metabox::make('Properties', 'post', ['priority' => 'high'], $view);
```

Inside the view file of your metabox you have access to "special" variables by default:

- **$__fields**: Gives you an array of registered fields with your metabox.
- **$__metabox**: Gives you access to your metabox instance.
- **$__post**: Gives you access to the current post instance object (WP_Post).

This allows you to customize as you want the look of your metabox. By also using `View::composer()` method, you might also perform specific actions when the metabox is rendered.

In case you needed to customize the metabox output but still need to output the core custom fields, simply add this code snippet inside your metabox view:

```php
<!-- Default Themosis metabox view -->
<table class="form-table themosis-metabox">
    <tbody>
        @each('_themosisMetaboxRow', $__fields, 'field')
    </tbody>
</table>
```

Send data to your metabox
-------------------------

Each time you create a metabox, a view file is attached to it. If you need to customize the metabox and need to use more data, you can pass data to your metabox view by using the `with` method like so:

```php
$metabox = Metabox::make('Infos', 'post')->set();

// Send a custom data to the metabox view
$metabox->with('key', 'value');
```

Of course, this methods is only useful if you specify a custom view for your metabox. You can also pass an array of `key/value` pairs to the `with` method.

Next
----

* [Meta guide](http://framework.themosis.com/docs/meta/)
* [Taxonomy guide](http://framework.themosis.com/docs/taxonomy/)
* [Page guide](http://framework.themosis.com/docs/page/)
