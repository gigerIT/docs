---
title: Pest v3 Now Available
description: Today, we're thrilled to announce the release of Pest 3, a major update that features mutation testing, arch presets, team management, and more.
---

# Pest v3 Now Available

Today, we're thrilled to announce the release of **Pest 3**. As we announced at Laracon US, Pest 3 introduces mutation testing, arch presets, team management, new configuration API, multiple architectural testing improvements, and more. Check out Pest's creator, Nuno Maduro, live demonstrating what's new in Pest 3:

<div class="content-center" markdown="0">
    <iframe width="100%" height="315" src="https://www.youtube.com/embed/BNhbgcNJyAk" title="Introducing Pest 3.0 | Nuno Maduro at Laracon US 2024 in Dallas, TX" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
</div>

Below, we'll cover all the juicy details about this release. And as usual, you can find the [upgrade guide](/docs/upgrade-guide) in our website.

- **[Mutation Testing](#mutation-testing)**: An innovative technique used to evaluate the quality of your application's test suite.
- **[Arch Presets](#arch-presets)**: A set of predefined architectural rules that you can use to test your application's architecture.
- **[Team Management](#team-management)**: A new feature that allows you to manage tasks and todos with your team directly from the console.
- **[New Configuration API](#new-configuration-api)**: A new configuration API that is more intuitive and easier to use.
- **[More Architectural Testing Improvements](#more-architectural-testing-improvements)**: A bunch of new architectural expectations and improvements.
- **[And More Much...](#miscellaneous-improvements)**

<a name="mutation-testing"></a>
## Mutation Testing

Pest 3 introduces [Mutation Testing](/docs/mutation-testing), an innovative technique used to evaluate the quality of your application's test suite by detecting "untested" code. Unlike code coverage, mutation testing is not solely about "covered" code, but more about the **actual quality of the tests**.

The way mutation testing works is by introducing small changes (mutations) to the source code and verifying if the tests are failing against these changes. This process helps developers identify weak spots in their test suites and improve the overall quality of their tests.

To get started with mutation testing, head over to your test file, and be specific about which part of your code your test covers using the `covers()` function.

```php
covers(TodoController::class);

it('list todos', function () {
    $this->getJson('/todos')->assertStatus(200);
});
```

Then, run Pest PHP with the `--mutate` option to start mutation testing.

```bash
./vendor/bin/pest --mutate
# or in parallel...
./vendor/bin/pest --mutate --parallel
```

Pest will then re-run your tests against "mutated" code and see if the tests are still passing. If a test is still passing against a mutation, it means that the test is not covering that specific part of the code. As, as result, Pest will output the mutation and the diff of the code.

```diff
UNCOVERED  app/Http/TodoController.php  > Line 44: UpdateReturnValue - ID: 76d17ad63bb7c307

class TodoController {
    public function index(): array
    {
         // in your controller, pest will mutate this line returning an empty array instead of all todos...
-        return Todo::all()->toArray();
+        return [];
    }
}

  Mutations: 1 untested
  Score:     33.44%
```

Once you have identified the untested code, you can write additional tests to cover it.

```diff
covers(TodoController::class);

it('list todos', function () {
+    Todo::factory()->create(['name' => 'Buy milk']);

-    $this->getJson('/todos')->assertStatus(200);
+    $this->getJson('/todos')->assertStatus(200)->assertJson([['name' => 'Buy milk']]);
});
```

Once you have written the additional tests, you can re-run Pest with the `--mutate` option to see if the mutation is now "tested" and covered because you test the return value of the `index` method.

```bash
  Mutations: 1 tested
  Score:     100.00%
```

The higher the mutation score, the better your test suite is. A mutation score of 100% means that all mutations were "tested", which is the goal of mutation testing.

Now, if you see "untested" or "uncovered" mutations, or are a mutation score below 100%, typically means that you have **missing tests** or that **your tests are not covering all the edge cases**.

Our plugin is deeply integrated into Pest PHP. So, each time a mutation is introduced, Pest PHP will:

- **Only run the tests covering the mutated code** to speed up the process.
- **Cache as much as possible** to speed up the process on subsequent runs.
- If enabled, use **parallel execution to run multiple tests** in parallel to speed up the process.

There is so much more to explore with Mutation Testing, like `@pest-mutate-ignore` or `--mutate --everything`. You can learn more about it in our [Mutation Testing](/docs/mutation-testing) section.

<a name="arch-presets"></a>
## Arch Presets

As you may know, [Architecture testing](#arch-testing) enables you to specify expectations that test whether your application adheres to a set of architectural rules, helping you maintain a clean and sustainable codebase.

It's one of the most popular features of Pest, and with Pest 3, we're introducing **Arch Presets**. Arch Presets are a set of predefined architectural rules that you can use to test your application's architecture. These presets are designed to help you get started with architecture testing quickly and easily.

Here are the available Arch Presets in Pest 3:

<div class="collection-method-list" markdown="1">

- [`php`](#preset-php)
- [`security`](#preset-security)
- [`laravel`](#preset-laravel)
- [`strict`](#preset-strict)
- [`strict`](#preset-strict)

</div>

<a name="preset-php"></a>
### `php`

The `php` preset is a predefined set of expectations that can be used on any php project. It's not coupled with any framework or library.

It avoids the usage of `die`, `var_dump`, and similar functions, and ensures you are not using deprecated PHP functions. [source code](https://github.com/pestphp/pest/blob/3.x/src/ArchPresets/Php.php)

```php
arch()->preset()->php();
```

<a name="preset-security"></a>
### `security`

The `security` preset is a predefined set of expectations that can be used on any php project. It's not coupled with any framework or library.

It ensures you are not using code that could lead to security vulnerabilities, such as `eval`, `md5`, and similar functions. [source code](https://github.com/pestphp/pest/blob/3.x/src/ArchPresets/Security.php)

```php
arch()->preset()->security();
```

<a name="preset-laravel"></a>
### `laravel`

The `laravel` preset is a predefined set of expectations that can be used on [Laravel](https://laravel.com) projects.

It ensures you project's structure is following the well-known Laravel conventions, such as controllers only have `index`, `show`, `create`, `store`, `edit`, `update`, `destroy` as public methods and are always suffixed with `Controller` and so on. [source code](https://github.com/pestphp/pest/blob/3.x/src/ArchPresets/Laravel.php)

```php
arch()->preset()->laravel();
```

<a name="preset-strict"></a>
### `strict`

The `strict` preset is a predefined set of expectations that can be used on any php project. It's not coupled with any framework or library.

It ensures you are using strict types in all your files, that all your classes are final, and more. [source code](https://github.com/pestphp/pest/blob/3.x/src/ArchPresets/Strict.php)

```php
arch()->preset()->strict();
```

<a name="preset-relaxed"></a>
### `relaxed`

The `relaxed` preset is a predefined set of expectations that can be used on any php project. It's not coupled with any framework or library.

It is the opposite of the `strict` preset, ensuring you are not using strict types in all your files, that all your classes are not final, and more. [source code](https://github.com/pestphp/pest/blob/3.x/src/ArchPresets/Relaxed.php)

```php
arch()->preset()->relaxed();
```

Just like regular architecture tests, you may ignore specific expectation targets using the `ignoring()` method.

```php
arch()->preset()->security()->ignoring('md5');

arch()->preset()->laravel()->ignoring(User::class);
```

To get started with Arch Presets, please refer to our [Architecture Testing](/docs/arch-testing#arch-presets) section.

<a name="team-management"></a>
## Team Management

Pest 3 also introduces **Team Management**, a new feature that allows you to manage tasks and todos with your team directly from the console. With Team Management, you can create, assign, and track tasks, as well as view the status of each task.

To get started with team management in Pest, you need to specify the project's URL in your `Pest.php` configuration file. This URL will be used to link todos to the corresponding project management system.

```php
pest()->project()->github('my-organization/my-repository');
```

If you are using a different version control system, you can use the `gitlab`, `bitbucket`, `jira`, or `custom` methods instead.

Finally, you can create todos by using the `todo()` method. Also, you may use the `assignee`, `issue`, arguments to assign todos to specific team members or link them to issues in your project management system.

```php
it('has a contact page', function () {
    //
})->todo(assignee: 'taylor@laravel.com', issue: 123);
```

Also, it is often helpful to provide additional context for a todo. Pest allows you to write notes for a todo by providing a string to the `note` argument of the `todo()` method.

```php
it('has a contact page', function () {
    //
})->todo(note: <<<NOTE
    Given I am a user
    When I visit the contact page
    Then I should see a contact form
NOTE);
```

Once a todo is completed, you can mark it as work in progress by using the `wip()` method or mark it as done by using the `done()` method.

```php
it('has a contact page', function () {
    //
})->wip(assignee: 'taylor@laravel.com', issue: 123); // or ->done()
```

Finally, you can view todos separately from the rest of your test suite by including the `--todos` option when running Pest. You can also filter todos by assignee by providing their name to the `--assignee` option, or filter todos by issue by providing the issue number to the `--issue` option.

```bash
./vendor/bin/pest --todos --assignee=taylor # or --issue=123
```

There is so much more to explore with Team Management, you can learn more about it in our [Team Management](#team-management) section.

<a name="new-configuration-api"></a>
## New Configuration API

Pest 1 / Pest 2's configuration API was a little bit confusing, the `uses()` function that was originally made only for having the `$this` variable within closure bound to the test case instance, ended up being used for pretty much everything.

In Pest 3, we've introduced a new configuration API that is more intuitive and easier to use. The new configuration API is based on the `pest()` function, which allows you to configure Pest using a fluent and expressive API.

> Note: the `uses()` function is still available in Pest 3, and we don't have plans to remove it. However, we recommend using the new configuration API for new projects.

```diff
-uses(TestCase::class)->in(__DIR__);
+pest()->extends(TestCase::class);

-uses(TestCase::class, RefreshDatabase::class)->in('Features');
+pest()->extends(TestCase::class)->use(RefreshDatabase::class)->in('Features');

-uses()->compact();
+pest()->printer()->compact();
```

<a name="more-architectural-testing-improvements"></a>
## More Architectural Testing Improvements

### New Expectations

Again, Pest comes with a bunch of new architectural expectations and improvements. Some of them are already being used in the new Arch Presets, but you can use them individually as well.

- [`toHaveAllMethodsDocumented()`](/docs/arch-testing#expect-toHaveAllMethodsDocumented) - Asserts that a class has all its methods documented.
- [`toHaveAllPropertiesDocumented()`](/docs/arch-testing#expect-toHaveAllPropertiesDocumented) - Asserts that a class has all its properties documented.
- [`toHaveFileSystemPermissions()`](/docs/arch-testing#expect-toHaveFileSystemPermissions) - Asserts that a file has the expected file system permissions.
- [`toHaveLineCountLessThan`](/docs/arch-testing#expect-toHaveLineCountLessThan) - Asserts that a file has less than a given number of lines.
- [`toHaveMethods()`](/docs/arch-testing#expect-toHaveMethod) - Asserts that a class has the expected methods.
- [`toHavePrivateMethodsBesides()`](/docs/arch-testing#expect-toHavePrivateMethodsBesides) - Asserts that a class has private
- [`toHavePrivateMethods()`](/docs/arch-testing#expect-toHavePrivateMethods) - Asserts that a class has private
- [`toHaveProtectedMethodsBesides()`](/docs/arch-testing#expect-toHaveProtectedMethodsBesides) - Asserts that a class has protected
- [`toHaveProtectedMethods()`](/docs/arch-testing#expect-toHaveProtectedMethods) - Asserts that a class has protected
- [`toHavePublicMethodsBesides()`](/docs/arch-testing#expect-toHavePublicMethodsBesides) - Asserts that a class has public
- [`toHavePublicMethods()`](/docs/arch-testing#expect-toHavePublicMethods) - Asserts that a class has public
- [`toUseTrait()`](/docs/arch-testing#expect-toUseTrait) - Asserts that a class uses a trait
- [`toUseTraits()`](/docs/arch-testing#expect-toUseTraits) - Asserts that a class uses traits

### New Ignoring Ways

As you may know, you can ignore specific expectation targets using the `ignoring()` method. In Pest 3, we've introduced `@pest-arch-ignore-line` and `@pest-arch-ignore-next-line` annotations to ignore specific lines or the next line.

```php
final class User // @pest-arch-ignore-line
{
    // @pest-arch-ignore-next-line
    public function name()
    {
        //
    }
}
```

### Tear Down Improvements

In Pest 3, we've introduced a new `after()` method that allows you to run a specific "teardown" callback after a specific test or group of tests using describe.

```php
afterEach(function () {
    // This will run after each test...
});

it('may list todos', function () {
    //
})->after(function () {
    // This will run after this test only...
});
```

<a name="miscellaneous-improvements"></a>
## Miscellaneous Improvements

Because Pest 3 is based on PHPUnit 11, you can now use any PHPUnit 11 feature within Pest. Also, Pest 3 also comes with a bunch minor bug-fixes and improvements, below are some of the them:

- FEAT: Type Coverage now checks for missing types on constants.
- FEAT: Better error messages when static closures are used on tests + wrong arguments on datasets.
- FEAT: Adds basic support for static analysis tools within test closures.
- FEAT: Overall static analysis improvements on expectations and the entire API surface.
- FEAT: Possibility of deleting the `phpunit.xml` file and having Pest working out of the box.
- FIX: Exit code being computed incorrectly when using `--fail-on-xxx` CLI options.
- FIX: Describe blocks now support more than one method call when chaining methods.
- FIX: Runtime exceptions before the first test are now caught and displayed.
- FIX: Having coverage report failing with `--min=100` option when result less than 100 but bigger than 99.5.
- And much more...

---

There's never been a better time to dive in into testing and start using Pest. If you're ready to get started with Pest 3 right away, check out our [installation guide](/docs/installation) for step-by-step instructions. And if you're currently using Pest 2, we've got you covered with detailed upgrade instructions in our [upgrade guide](/docs/upgrade-guide).

Thank you for your continued support and feedback. We can't wait to see what you build with Pest 3!

---

Thank you for reading about Pest 3.0's new features! If you're considering a testing framework for your next project, here's why you should give Pest a try: [Why Pest →](/docs/why-pest)