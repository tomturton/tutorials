Laravel 5.1 introduced us to a wonderful easy-to-use [application testing library](https://laravel.com/docs/testing), built on top of [PHPUnit](https://phpunit.de/).
It can help you write tests to visit a page, complete a form, browse the DOM (with [Symfony's DOM crawler](http://symfony.com/doc/current/components/dom_crawler.html) component), read the database and more.
You can mock events, and even Laravel facades.

If you haven't started using automated testing yet, but are interested, then this is a great starting point to learn from.

Testing any feature that updates the database needs to managed carefully, however. You usually don't want your development database to be filled up with thousands of records of dummy test data.
Luckily, Laravel offers some solutions.


## Database transactions

One of the best parts of the testing library is being able to switch on *database transactions*, which allows you to safely perform queries on your local database with the knowledge that after each test, the changes will be reverted. You simply add the `DatabaseTransactions` trait to your test class.

This works beautifully in the vast majority of cases, but I have recently come across an issue when needing to use multiple databases in a project.
Currently, Laravel only runs transactions on the default database. If you are testing a non-default database, transactions will have no effect, and your test queries will not revert!

This presents a real problem. In my case I was building several Laravel sites, each with its own database, plus access to a common database of users and roles etc.


## Database migrations

Another option is to use database migrations, which will run your migrations before and rollback after each test. To use, just add the `DatabaseMigrations` trait to your test class.

This will work as expected if you are testing muliple databases (as long as your migrations are setup to use the right connections).
However I find that it's better to use test versions of these databases when using the `DatabaseMigrations` trait.

To do this you will need to create duplicates for each database you wish to test. Importantly they need to use the same database driver (eg MySQL), but they can be called anything.

Next, in your `phpunit.xml` file, add an environment variable setting for each database, to override the default database names.

~~~ xml
...
<env name="DB_ACCOUNT_DATABASE" value="accounts_test"/>
<env name="DB_SITE_DATABASE" value="site_test"/>
...
~~~

Now you can run normal local databases that can change over time with more example data, and test databases that are only used by PHPUnit with [Faker entries](https://laravel.com/docs/testing#model-factories) (for example).


*[TDD]: Test-driven development: the practice of writing automated tests before adding functionality
*[application testing]: testing the software with normal use-cases, as if from a customer's perspective
