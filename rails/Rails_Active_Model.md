## Building Models with Active Model

`Active Model` was originally created to hold the behavior shared between `Active
Record` and `Active Resource`. As with `Abstract Controller`, the desired functionalities 
can be `cherry-picked` by including only the modules you need.
`Active Model` is also responsible for defining the application programming
interface (API) required by `Rails controllers and views`, so any other 
`object relational mapper` (ORM) can use `Active Model` to ensure Rails behaves
exactly as it would with `Active Record`.

Let’s explore both facets of `Active Model` in this chapter by writing a plug-in
called `Mail Form` that we’ll use in our controllers and views. `Mail Form’s` goal
is to receive a hash of parameters sent by a POST request, validate them, and
email them to a specified email address. This abstraction will allow us to
create fully functional contact forms in just a couple of minutes!

### Creating Our Model

`Mail Form` objects belong to the `models part` in the `model-view-controller`
architecture, as they receive the information sent through a form and deliver
it to a recipient specified by the business model. Let’s structure `Mail Form`
in the same way `Active Record` works: we’ll provide a class named `MailForm::Base`
that contains the most common features we expect in a `model`, such as the
`ability to specify attributes`, and seamless integration with `Rails forms`. As we
did in the previous chapter, let’s use `rails plugin` to create our new plug-in:

> $ rails plugin new mail_form

Our first feature is to implement a `class method` called `attributes()` that allows
a developer to `specify` which `attributes` the `Mail Form` object contains. Let’s
create a model inside `test/fixtures/sample_mail.rb` as a fixture to use in our tests:

```ruby
# mail_form/1_attributes/test/fixtures/sample_mail.rb
class SampleMail < MailForm::Base
  attributes :name, :email
end
```

Then we’ll add a test to ensure the defined `attributes` `name` and `email` are
available as accessors in the `Mail Form` object:

```ruby
# mail_form/1_attributes/test/mail_form_test.rb
require "test_helper"
require "fixtures/sample_mail"
class MailFormTest < ActiveSupport::TestCase
  test "sample mail has name and email as attributes" do
    sample = SampleMail.new
    sample.name = "User"
    assert_equal "User", sample.name
    sample.email = "user@example.com"
    assert_equal "user@example.com", sample.email
  end
end
```

When we run the test suite with `rake test` , it fails because `MailForm::Base` is not
defined yet. Let’s define it in `lib/mail_form/base.rb` and `implement` the `attributes()`
method:
```ruby
# mail_form/1_attributes/lib/mail_form/base.rb
module MailForm
  class Base
    def self.attributes(*names)
      attr_accessor(*names)
    end
  end
end
```

Our implementation delegates the creation of `attributes` to `attr_accessor()` . Before
we run our tests again, we need to ensure that `MailForm::Base` is loaded. One
option would be to explicitly `require "mail_form/base"` in `lib/mail_form.rb` . However,
let’s use Ruby’s `autoload()` instead:

```ruby
# mail_form/1_attributes/lib/mail_form.rb
module MailForm
  autoload :Base, "mail_form/base"
end
```

`autoload()` allows us to `lazily load` a `constant` when it is first referenced. So we
note that `MailForm` has a constant called `Base` defined in `mail_form/base.rb` . When
`MailForm::Base` is referenced for the first time, `Ruby loads` the `mail_form/base.rb` file.
This is frequently used in Ruby gems and in Rails itself for a fast booting
process, as it does not need to load everything up front.

With `autoload()` in place, our first test passes. We have a simple model with
attributes, but so far we haven’t used any of Active Model’s goodness. Let’s
do that now.

#### Adding Attribute Methods

`ActiveModel::AttributeMethods` is a `module` that `tracks all defined attributes`, allowing
us to add a `common behavior` to all of them `dynamically`. To show how it
works, let’s define two convenience methods, `clear_name()` and `clear_email()` , which
will clear out the value of the associated attribute when invoked. Let’s write
a test first:

```ruby
# mail_form/2_attributes_prefix/test/mail_form_test.rb
test "sample mail can clear attributes using clear_ prefix" do
  sample = SampleMail.new
  sample.name = "User"
  sample.email = "user@example.com"
  assert_equal "User", sample.name
  assert_equal "user@example.com", sample.email

  sample.clear_name
  sample.clear_email
  assert_nil sample.name
  assert_nil sample.email
end
```

Invoking `clear_name()` and `clear_email()` sets their respective `attribute` value back
to `nil` . With `ActiveModel::AttributeMethods` , we can define both `clear_name()` and
`clear_email()` dynamically in four simple steps, as outlined in our new `MailForm::Base`
implementation:

```ruby
# mail_form/2_attributes_prefix/lib/mail_form/base.rb
module MailForm
  class Base
    include ActiveModel::AttributeMethods # 1) attribute methods behavior
    attribute_method_prefix 'clear_' # 2) clear_ is attribute prefix
    
    def self.attributes(*names)
      attr_accessor(*names)
      
      # 3) Ask to define the prefix methods for the given attribute names
      define_attribute_methods(names)
    end

    protected
    
    # 4) Since we declared a "clear_" prefix, it expects to have a
    # "clear_attribute" method defined, which receives an attribute
    # name and implements the clearing logic.
    
    def clear_attribute(attribute)
      send("#{attribute}=", nil)
    end
  end
end
```

`ActiveModel::AttributeMethods` uses `method_missing()` to compile 
both the `clear_name()` and `clear_email()` methods
when they are first accessed. Their implementation invokes `clear_attribute()` ,
passing the `attribute name` as a parameter.


If we want to define `suffixes` instead of a `prefix` like `clear_ `, we need to use the
`attribute_method_suffix()` method and implement the method with the chosen suffix
logic. As an example, let’s implement `name?()` and `email?()` methods, which should
`return true` if the respective attribute value is `present`, as in the following test:

```ruby
# mail_form/3_attributes_suffix/test/mail_form_test.rb
test "sample mail can ask if an attribute is present or not" do
  sample = SampleMail.new
  assert !sample.name?
  sample.name = "User"
  assert sample.name?
  sample.email = ""
  assert !sample.email?
end
```

When we run the test suite, our new test fails. To make it pass, let’s define `?`
as a `suffix`, changing our `MailForm::Base` implementation to the following:

```ruby
# mail_form/3_attributes_suffix/lib/mail_form/base.rb
module MailForm
  class Base
    include ActiveModel::AttributeMethods
    attribute_method_prefix 'clear_'
    
    # 1) Add the attribute suffix
    attribute_method_suffix '?'
    
    def self.attributes(*names)
      attr_accessor(*names)
      define_attribute_methods(names)
    end
      
    protected
    def clear_attribute(attribute)
      send("#{attribute}=", nil)
    end
      
    # 2) Implement the logic required by the '?' suffix
    def attribute?(attribute)
      send(attribute).present?
    end
  end
end
```

Now we have both `prefix` and `suffix` methods defined and the tests are passing.
But what if we want to define both the `prefix` and the `suffix` at the `same time`?
We could use the `attribute_method_affix()` method, which accepts a hash specifying
both the `prefix` and the `suffix`.

`Active Record` uses `attribute methods` extensively. An example is the
`attribute_before_type_cast()` method, which `uses _before_type_cast` as a `suffix` to return
`raw data`, as received from `forms`. The `dirty functionality`, which is also part
of `Active Model`, is built on top of `ActiveModel::AttributeMethods` and defines a
handful of methods like `attribute_changed?()` , `reset_attribute!()` , and so on. We can
check the dirty implementation source code in the Rails repository.


### Aiming for an Active Model–Compliant API:

Even though we added attributes to our models to store form data, we need
to ensure that our model complies with the `Active Model API`; otherwise, we
won’t be able to use it in our controllers and views.

As usual, we’ll achieve this compliance through `test-driven development`,
except this time we won’t need to write the tests—Rails already provides all
of them in a module called `ActiveModel::Lint::Tests`.
When included, this module defines several tests asserting that 
each method required in an `Active Model–compliant API` exists. Each of these 
tests expects an instance variable named `@model` to return the object we 
want to assert against. In our case, `@model` should contain an instance 
of `SampleMail` , which will be compliant if `MailForm::Base` is 
compliant. Let’s create a new test file called `test/compliance_test.rb`
with the following:

```ruby
# mail_form/4_am_compliance/test/compliance_test.rb
require 'test_helper'
require 'fixtures/sample_mail'
class ComplianceTest < ActiveSupport::TestCase
  include ActiveModel::Lint::Tests
  def setup
    @model = SampleMail.new
  end
end
```

When we run rake test , we get several failures, all with this reason:

> The object should respond to `to_model`.

When `Rails controllers` and `view helpers` receive a `model`, they first call
`to_model()` and `manipulate` the `returned result` instead of the `model directly`.
This allows `ORM implementations` that don’t want to add `Active Model`
methods to their API to return a `proxy object` where these methods are defined.
In our case, we want to add `Active Model` methods directly to `MailForm::Base` .
Consequently, our `to_model()` implementation should return `self` , as shown here:

```ruby
def to_model
  self
end
```

Although we could add this method to `MailForm::Base` , we are not going to
implement it ourselves. Instead, let’s include `ActiveModel::Conversion` , which
implements `to_model()` exactly as we discussed, and three other methods
required by `Active Model`: `to_key()` , `to_param()` , and `to_partial_path()`.


The `to_key()` method should return an `array of keys` that uniquely identifies
the `model`, if any exists, and it is used by `dom_id()` in views. The `dom_id()` method
was added to Rails along with `dom_class()` and a bunch of other helpers to better
organize our views. For example, `div_for(@post)` , where `@post` is an `Active Record`
instance of the `Post class` with an `id of 42`, relies on both these methods to
create a div where the `id` attribute is equal to `post_42` and the `class` attribute is
`post` . For Active Record, `to_key()` returns an array containing the `record ID` from
the database.

On the other hand, `to_param()` is used in `routing` and can be overwritten in any
model to generate a unique URL for that model. When we invoke `post_path(@post)` ,
Rails calls `to_param()` in the `@post` object and uses its result to generate the final
URL. For Active Record, the default is to return the `ID` as a string.


Finally, we have `to_partial_path()` . This method is invoked every time we pass a
`record` or a `collection of records` to `render()` in our views. Rails will go through
each of these records and retrieve the `path to their partial`. For example, the
path to an `instance of the Post` class is `posts/post` .

It is important to understand not only what those methods do, but also what
they allow us to achieve. For example, by customizing `to_param()` , we can easily
change the `URLs` of our `objects`. Imagine a `Post` class with `id` and `title` attributes;
changing the URLs of those posts to include the `title` is as easy as this:

```ruby
def to_param
  "#{id}-#{title.parameterize}"
end
```

Similarly, imagine that each `Post` has a `different format`. It can be a video, a
link, or a bunch of a text, and each of those formats should be rendered differently. 
If we store the `format` of the blog post in the `format` attribute, we could
`render` each post as follows:

```ruby
@posts.each do |post|
  render partial: "posts/post_#{post.format}",
    locals: { post: @post }
end
```

However, by overriding `to_partial_path()` like this

```ruby
def to_partial_path
  "posts/post_#{format}"
end
```

our view would simply call

>render @posts

This not only makes our code cleaner, but also improves our application
performance. In the first example, we end up going through `Rails’s rendering
stack` many times, looking up `templates` and `duplicating efforts`. However, by
customizing `to_partial_path()` , we call `render()` just once, allowing Rails to efficiently
look up all `partials` in one take.

The default `to_partial_path()` implementation available in `ActiveModel::Conversion`
allows us to provide `partials` for `MailForm::Base` objects as in any Active Record
object. However, since our objects are `never persisted`, they aren’t `uniquely
identified`, meaning that both `to_key()` and `to_param()` should return `nil` . This is
exactly the behavior provided by `ActiveModel::Conversion` . Let’s include it in our
`MailForm::Base` class:

```ruby
# mail_form/4_am_compliance/lib/mail_form/base.rb
module MailForm
  class Base
    include ActiveModel::Conversion
```