# What is Laravel?
Laravel is a popular open-source PHP framework, in version 5.2 at the time of writing.
It was originally launched by Taylor Otwell in 2011 as he was fed up with the limitations and slow development of CodeIgniter (one of the most popular PHP frameworks of the time).
However, Laravel is not that similar to CodeIgniter, except that it uses the MVC design pattern. Laravel is more similar to Ruby on Rails, in many ways.


# Why use Laravel, or other frameworks?
If you are developing a very simple PHP application, there is probably no need to use a large framework, such as Laravel.

Frameworks are designed to give you a platform to build your application upon, while taking care of the architecture and common functions.
This might be hard to appreciate if you've never used a framework before, but bear with me.
Frameworks help to organise your code (especially in larger apps), and usually try to make a lot of common tasks easier and more robust, such as querying databases, adding security, and form validation.

The learning curve for Laravel can be a little steep for beginners, but the docs are good, and there are some fantastic learning resources out there.
Despite the learning curve, you will quickly see the benefits, and find it can be really fun to use.
One unexpected benefit I found, was that Laravel has taught me to become a better PHP programmer.


# What features does Laravel offer?
Laravel makes all these things easy and robust (and much, much more)

- ACL (authorisation)
- Authentication
- Database relations
- Database version-control
- Encryption
- Emails
- Event listeners
- Form requests and validation
- HTML templating
- HTTP responses
- Integration testing
- Localisation
- Model factories
- Pagination
- RESTful routing
- Task scheduling
- Templating


# What should I know before learning Laravel?
Beyond a little experience with Object-Oriented PHP, it would certainly help to be familiar with the [Composer](https://getcomposer.org/) package manager, namespaces, database queries, and perhaps an understanding of [REST](/posts/rest-basics/).

Actually, while being easy for beginners, Laravel will introduce you to many modern PHP practices so even with a few days of experience, you will come out a better programmer.

I have written this post to whet your appetite. It is not a technical guide.
If you want to try out Laravel, then the [official docs](http://laravel.com/docs/) are all you need to get started.


# Routing and controllers

One of the very first things you might do when starting a new Laravel project is defining the routes to each of the pages and processes you need.

For instance, a simple 'items' page could be defined like this:

~~~~ php
Route::get('items', function () {
   return 'list of items'; 
});
~~~~

In the above example, if the `GET` HTTP method is used to request the page at `http://mysite.com/items`. In this case I have used a closure, as the response is very simple to compute and display.


~~~~ php
Route::post('items', function () {
    // ...
    // database logic
    // ...

   return redirect('items')
});
~~~~

The above route will be called when the browser/client requests the same page, but with a `POST` method. In this case they are redirected back to the `GET` route at the end of the closure.

## Controllers
Most of the time, using closures would only clutter up the routes file; so it is more sensible to employ controllers.

~~~~ php
Route::get('items/{id}', 'ItemController@show');
~~~~

The above route would be for a `GET` request to `http://mysite.com/items/5` where `5` is any valid Item ID. Instead of using a closure, this route calls the `show` method of the `ItemController` class. The ID of the item is passed as a parameter to this method:

~~~~ php
class ItemController extends Controller
{
    public function show($id)
    {
        return 'all about item with ID: ' . $id;
    }
}
~~~~

## RESTful resources
Laravel includes the full range of popular HTTP verbs, including `PUT`, `PATCH`, and `DELETE`, which makes it ideal for building REST APIs.
In fact, to create all the routes you might need for a typical RESTful resource, you could use the `resource` method:

~~~~ php
Route::resource('items', 'ItemController');
~~~~

This route definition expects the `ItemController` class to have several methods, by default:

Method | Parameters | HTTP request example | Description
--- | --- | --- | ---
`index()` | / | GET /items | List items
`create()` | / | GET /items/create | Form to create a new item
`store($request)` | Laravel request object | POST /items | Store a new item 
`show($id)` | Item ID | GET /items/id | Retrieve data on a specific item 
`edit($id)` | Item ID | GET /items/id/edit | Form to edit an existing item
`update($request, $id)` | Laravel request object, Item ID | PUT or PATCH /items/id | Save changes to an existing item
`delete($id)` | Item ID | DELETE /items/id | Delete a specific item


# Database migrations
Laravel lets you write database table structures in code, which you then run (migrate) to create the tables.

There are a few advantages to doing this:

- It's version control for database structure. Databases often evolve as the project expands, and creating new instructions to alter the structure of a database allows you to keep a record of how it has evolved.
- It also lets you define how to revert changes, if you decide to rollback to a previous version of the database structure.
- The table-writing code is largely driver-agnostic; meaning you could later switch to using a different database type.
- Laravel uses a nice syntax for creating fields, which really minimises what you need to write.

I had not considered either of these points before discovering Laravel database migrations. Now I don't ever want to go back to building the structure directly in MySQL, for anything larger than a simple two-table database.

~~~ php 
Schema::create('posts', function (Blueprint $table) {
    $table->increments('id'); // sets up an auto-incrementing unsigned integer as the primary key
    $table->integer('user_id')->unsigned()->nullable();
    $table->string('title')->unique();
    $table->text('body');

    $table->timestamps();  // adds `created_at` and `updated_at` timestamp fields
    $table->softDeletes(); // adds a `deleted_at` timestamp field
    
    // create a foreign key
    $table->foreign('user_id')
        ->references('id')
        ->on('users')
        ->onUpdate('cascade')
        ->onDelete('restrict');
}
~~~

In the console:

~~~
> php artisan migrate // migrate the database
> php artisan migrate:rollback // roll back structure to the last migration
~~~


# Models and Relationships
One of the biggest strengths of Laravel is its model interface.

You can think of a model object as a database record. The fields in the row become properties of the model object.
There is a model class for each table in your database, which you can configure with properties and built-in traits, that define how Laravel should treat the table.

'Eloquent' is Laravel's ORM query interface, that takes most of the hassle and frustration out of writing raw SQL.
Eloquent uses a chainable query syntax to talk to the database, which can ultimately be used to return a model or collection of models.

- Model classes allow you to define relations between other tables, so that Laravel can write intelligent queries.
- Eager loading is supported to tackle the 'N+1 problem', to avoid making additional queries during loops.
- Common queries can be defined in query scopes.
- Laravel will also cater for polymorphic relations.

Manually writing SQL for most of the above can be a real chore and requires good experience with SQL joins. Eloquent can be used to bypass nearly all of the pain of writing efficient SQL.

~~~ php
class User extends Model
{
    public function posts()
    {
        return $this->hasMany('App\Post');
    }

    public function country()
    {
        return $this->belongsTo('App\Country');
    }
}
~~~

~~~ php
$user = User::where('email', 'tom@tomturton.co.uk')->firstOrFail();

$capitalCity = $user->country->capital;
~~~

There are a few extra things I love about Laravel models:

- Timestamps can automatically be converted to [Carbon](http://carbon.nesbot.com/) instances, for super easy date manipulation and formatting.
- Relationships can be referred to as an object property to fetch the relationship model(s), or as a method to chain onto the query.
- 'Soft deletes' will pretend to delete a model, by simply marking it as deleted in the database. It will never show up again unless explicitly asked for. Not only does this allow recovery later, but it means none of that model's relationships will be broken.
- Laravel comes with a default User model and controller, which is preconfigured for authentication, authorisation and password resetting.


# Views and templating
Laravel's *Blade* templating system is another killer feature, as views are so simple and often fun to write. It's rare you will need to open `<?php ?>` tags in a Blade template (although there's nothing stopping you).

- A simple inheritance system allows you to write master templates that child views can extend.
- Data can be passed safely to avoid cross-site scripting attacks, or as-is when necessary.
- Blade also includes simple flow control such as `if`, `else` and `foreach` statements.
- There are lots of helper functions such as getting the URL to your routes, injecting a CSRF token into a form, accessing the request object, and many more.

~~~ php
@section('menu')
    @if (count($user->posts))
        <ul>
        @foreach ($user->posts as $post)
            <li>
                <a href="{{ route('posts.show', $post->id) }}">{{ $post->title }}</a>
                <time datetime="{{ $post->created_at->toRfc3339String() }}">
                    {{ $post->created_at->format('j M Y') }}
                </time>
            </li>
        @endforeach
        </ul>
    @else
    <p>You have no posts.</p>
    @endif
@endsection
~~~


# Form validation
Laravel includes a comprehesive form validation system, which allows you to easily check if a submitted form satisfies your requirements, and if not returns the user to the form.
Validation options include matching regular expresions, checking the database for existing records, and seeing if two fields match.
Custom validation rules can also be created, and all error messages can be also be customised.

~~~ php
public function rules()
{
    return [
        'first_name'            => 'min:2',
        'last_name'             => 'min:2',
        'email'                 => 'required|email|unique:users,email',
        'password'              => 'required|min:8|confirmed',
        'password_confirmation' => 'required|min:8'
    ];
}
~~~


# ACL authorisation
A relatively new, but powerful feature is Laravel's authorisation components.
It is really easy to create automatic rules to check user permissions, including in form requests and Blade templates.

I also like to use ACL as a feature toggle system, while I am developing an application. Instead of creating a feature branch in Git, I often prefer to hide a new feature behind an authorisation gate, and simply toggle it off if I need to quickly patch something for a hotfix.

~~~ php
@can('update-post', $post)
    <a href="/posts/123/update">Update post</a>
@endcan
~~~


# Automated integration testing
If you're one of countless developers who like the idea of automated testing, but haven't yet gotten around to implimenting it into your workflow, then hopefully Laravel's built-in integration testing package will encourage you to start.

Laravel uses a really quick and easy platform to write tests, with a straightforward syntax; so it is perfect for general application testing.

~~~ php
$this->visit('/password/update')
     ->type('1234', 'old_password')
     ->type('qwerty', 'password')
     ->type('qwerty', 'password_confirmation')
     ->press('Save new password')
     ->see('Your password has been updated');
~~~


# And more...
I've highlighted some of my favourite features of the Laravel framework, but there's mountains more. See the [Laravel website](https://laravel.com/) and documentation for a comprehensive overview.


*[ORM]: Object-relational mapping
*[ACL]: Access control list
