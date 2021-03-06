# unique_with Validator Rule For Laravel 4.1

[![Build Status](https://travis-ci.org/felixkiss/uniquewith-validator.png?branch=master)](https://travis-ci.org/felixkiss/uniquewith-validator)

This package contains a variant of the `validateUnique` rule for Laravel 4, that allows for validation of multi-column UNIQUE indexes. It also allows for the inclusion of an id parameter in the validation rule string to ignore a particular row in your data store when running these rules. A typical use case is that you want to exclude the current row from validation when performing an update operation.

## Installation

Install the package through [Composer](http://getcomposer.org).

In your `composer.json` file:

```json
{
    "require": {
        "laravel/framework": "4.1.*",
        // ...
        "kylenoland/uniquewith-validator": "dev-master"
    },
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/kylenoland/uniquewith-validator"
        }
    ]
}
```

Run `composer install` or `composer update` to install the package.

Add the following to your `providers` array in `config/app.php`:

```php
'providers' => array(
    // ...

    'Kylenoland\UniqueWithValidator\UniqueWithValidatorServiceProvider',
),
```

## Usage

Use it like any `Validator` rule:

```php
$rules = array(
    '<field1>' => 'unique_with:<table>,<field2>[,<field3>...]',
);
```

See the [Validation documentation](http://laravel.com/docs/validation) of Laravel 4.

### Specify different column names in the database

If your input field names are different from the corresponding database columns,
you can specify the column names explicitly.

e.g. your input contains a field 'last_name', but the column in your database is called 'sur_name':
```php
$rules = array(
    'first_name' => 'unique_with:users, middle_name, last_name = sur_name',
);
```


## Example

Pretend you have a `users` table in your database plus `User` model like this:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;

class CreateUsersTable extends Migration {

    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('users', function(Blueprint $table) {
            $table->increments('id');

            $table->timestamps();

            $table->string('first_name');
            $table->string('last_name');

            $table->unique(array('first_name', 'last_name'));
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('users');
    }

}
```

```php
<?php

class User extends Eloquent { }
```

Now you can validate a given `first_name`, `last_name` combination with something like this:

```php
Route::post('test', function() {
    $rules = array(
        'first_name' => 'required|unique_with:users,last_name',
        'last_name' => 'required',
    );

    $validator = Validator::make(Input::all(), $rules);

    if($validator->fails())
    {
        return Redirect::back()->withErrors($validator);
    }

    $user = new User;
    $user->first_name = Input::get('first_name');
    $user->last_name = Input::get('last_name');
    $user->save();

    return Redirect::home()->with('success', 'User created!');
});
```
# License

MIT