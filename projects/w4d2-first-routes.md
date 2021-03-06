# First Routes & Controllers

In this project we'll start playing with Rails routing. To start,
generate a new, blank Rails project.

## First Routes

Go to `config/routes.rb` and generate your first routes with:

    resources :users

Remember that this one line actually generates **eight routes** for
us. Run `rake routes` to see what those routes are.

Woohoo! We've set up our first eight **API endpoints**. Each route you
have is an API endpoint, which encapsulates a single action your app
can take.

Lest you scream "magic!", do the following:

* Comment out the `resources :users` line.
* Write out the eight routes using the route 'matching' syntax.  For
  example: `get 'users/:id' => 'users#show'`.

Run `rake routes` again and ensure that the routes you've written
match exactly the routes generated by the `resources` helper. **NB**:
you'll probably be missing some names for some of the routes (they're
listed in the left-most column); You can name your routes by adding an
`as` option. `get 'users/new' => 'users#new', :as => 'new_user'`.

Remember that all a route does is match on the **HTTP method** and the
**url path** (it does this with a **regular expression**, if you know
what that is already). The controller then sends the request on to the
specified action on the specified controller.

We have our initial routes now and have the endpoints necessary to
manage a `User` resource. Notice though that our routes point to a
`UsersController`, which we don't actually have yet. Nor do we have a
`User` model, we'll add that later, too. Soon!

## First Controller

Each API endpoint creates/reads/updates/destroys a resource,

The router defines API endpoints (urls), and records which controller
and action to invoke. Each API enpoint has a conventional meaning:
create/read/update/destroy a resource. The controllers and their
actions are the ones actually doing the **CRUD** ing.

Generate your first controller with:

```
$ rails generate controller Users
```

Note that controllers are always plural; a controller manages requests
that pertain to a collection of **resources**. A resource is anything
in your application that you will be CRUDing.

Let's go take a look at the controller that was generated.

```
# app/controllers/application_controller.rb
class UsersController < ApplicationController
end
```

Controllers inherit from `ApplicationController` which is a controller
itself, but one that never actually handles any requests directly.
`ApplicationController` is where you'd put helper methods that you
want to share across all controllers. Take a look at it (it's right
there in the `app/controllers` folder):

```
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
end
```

`ActionController::Base` provides all the bells & whistles that Rails
controllers have; it's like `ActiveRecord::Base` in that respect.  All
your controllers will inherit the features it provides since it is in
the inheritance chain:

    UsersController < ApplicationController < ActionController::Base

`protect_from_forgery` helps protect against cross-site request
forgery (CSRF) by checking the authenticity of a certain token for
POST requests. We won't worry about that for now.

**Just for this assignment, comment out `protect_from_forgery`.** Then
you won't need to include the authenticity token in your POST params;
we'll learn about what that means later.

Alright, now that we have some routes and a matching controller, we
have everything we need to start handling requests.

## First Launch

We have our API endpoints setup and they map to a controller which
we've created. How do we actually start taking requests?

```
$ rails server
=> Booting WEBrick
=> Rails 3.2.13 application starting in development on http://0.0.0.0:3000
=> Call with -d to detach
=> Ctrl-C to shutdown server
```

Rails ships with a lightweight web server called WEBrick.

As you can see, it loads your application in development mode (later
we'll discuss the other two modes: production and testing), and is
listening for requests at `http://0.0.0.0:3000`. The last part,
`:3000` specifies the port it is listening on. Rails defaults to port
3000 in development. The domain `http://0.0.0.0` can be accessed from
your browser as simply `http://localhost`.

In your browser, navigate to `http://localhost:3000`. Voila! A
running Rails app with what will become a very familiar index page.

## First Request

Create a script in your `bin/` folder; name it `my_script.rb`. We'll
be using it to make requests to our Rails app. We'll be able to run
this script with the `ruby` command.

Add `gem 'addressable'` to your `Gemfile` and `bundle install`.

Require `addressable/uri` and `rest-client` so that we can construct
our requests.

```ruby
# bin/my_script.rb
url = Addressable::URI.new(
  scheme: 'http',
  host: 'localhost',
  port: 3000,
  path: '/users.html'
).to_s

puts RestClient.get(url)
```

Make sure your rails server is still running (keep it in a tab in
Terminal). In another tab, run the script:

```
~/FirstRoutes$ ruby bin/my_script.rb
/Users/ruggeri/.rvm/gems/ruby-1.9.3-p194/gems/rest-client-1.6.7/lib/restclient/abstract_response.rb:48:in `return!': 404 Resource Not Found (RestClient::ResourceNotFound)
        from /Users/ruggeri/.rvm/gems/ruby-1.9.3-p194/gems/rest-client-1.6.7/lib/restclient/request.rb:230:in `process_result'
        from /Users/ruggeri/.rvm/gems/ruby-1.9.3-p194/gems/rest-client-1.6.7/lib/restclient/request.rb:178:in `block in transmit'
        from /Users/ruggeri/.rvm/rubies/ruby-1.9.3-p194/lib/ruby/1.9.1/net/http.rb:745:in `start'
        from /Users/ruggeri/.rvm/gems/ruby-1.9.3-p194/gems/rest-client-1.6.7/lib/restclient/request.rb:172:in `transmit'
        from /Users/ruggeri/.rvm/gems/ruby-1.9.3-p194/gems/rest-client-1.6.7/lib/restclient/request.rb:64:in `execute'
        from /Users/ruggeri/.rvm/gems/ruby-1.9.3-p194/gems/rest-client-1.6.7/lib/restclient/request.rb:33:in `execute'
        from /Users/ruggeri/.rvm/gems/ruby-1.9.3-p194/gems/rest-client-1.6.7/lib/restclient.rb:68:in `get'
        from bin/my_script.rb:11:in `<main>'
```

Okay, that didn't work; we got a 404 error. Why?

The server log will be where you'll go to see what's going on in your
application. All your `puts` and `p` statements in your application
will also go to the server log. **Always be looking at the server
log**; this is an essential debugging technique. We'll see what some
of the most important information is in just a second. Here's what we
see in the log:

```
Started GET "/users.html" for 127.0.0.1 at 2013-08-12 10:48:39 -0700

AbstractController::ActionNotFound (The action 'index' could not be found for UsersController):
...
```

Looks like a request came in; what's the error? It seems like it's
complaining that we don't actually have an `index` action setup in our
`UsersController`. Note that your application looked for an index
action because the router specified that a GET request to `/users`
maps to `UsersController#index`.

Let's fix it. Add an empty `index` action to your `UsersController`:

```ruby
class UsersController < ApplicationController
  def index
  end
end
```

Run the script again. It fails again, so look at the log:

```
Started GET "/users.html" for 127.0.0.1 at 2013-08-12 10:52:35 -0700
Processing by UsersController#index as HTML
Completed 500 Internal Server Error in 1ms

ActionView::MissingTemplate (Missing template users/index, application/index with {:locale=>[:en], :formats=>[:html], :handlers=>[:erb, :builder, :coffee]}. Searched in:
  * "/Users/ruggeri/FirstRoutes/app/views"
):
...
```

This time, it's complaining that there's a missing template. Wait a
minute; we never called `render`. Why is it trying to look for a
template at all? Because in the absence of an explicit `render`
statement, your controller will by default try to render a template
with the same name as the controller action - in this case, it was
looking for a template called `index.html.erb` in `app/views/users`.

We're not going to deal with views and templates just yet. To get rid
of this error, let's just add a simple render:

```ruby
class UsersController < ApplicationController
  def index
    render text: "I'm in the index action!"
  end
end
```

Try again. It should work! Now take a look at the server log.

```
Started GET "/users.html" for 127.0.0.1 at 2013-08-12 10:53:41 -0700
Processing by UsersController#index as HTML
  Rendered text template (0.0ms)
Completed 200 OK in 102ms (Views: 101.3ms | ActiveRecord: 0.0ms)
```

For every request, the server will tell you which controller and
action is processing it. In this case, it was the `UsersController`'s
`index` action.

Woohoo! Your RestClient call should have returned the string "I'm in
the index action!" Victory is yours. Congratulations on successfully
setting up, making, and processing your first Rails request.

## Playing with Parameters

Now we're going to focus on how data comes into our controllers from
the outside world.

The key method here is `#params`. `#params` is a method provided by
`ActionController::Base` that returns a hash of all the parameters
available. The parameters are sourced from three places:

* Route parameters (e.g. the `:id` from `/users/:id`)
* Query string (the part of the URL after the `?`: `?key=value`)
* POST/PATCH request data (the body of the HTTP request).

Go ahead and make some GET requests to `/users` playing around with
the query values. Check out the server log and notice that it logs
how the parameters are coming in:

```
Started GET "/users.html?var1=val1" for 127.0.0.1 at 2013-08-12 10:55:44 -0700
Processing by UsersController#index as HTML
  Parameters: {"var1"=>"val1"}
  Rendered text template (0.0ms)
Completed 200 OK in 0ms (Views: 0.3ms | ActiveRecord: 0.0ms)
```

Now make some POST requests to `/users` playing around with POST data
and seeing how the parameters come in. Think about what controller
action you need to POST to `/users` (check your routes again).

Now make a couple requests to `/users/:id` and see how the `:id`
parameter comes in.

### Nesting Parameters

Notice how all of our parameters come in at the top level of the
parameters hash. Let's say we wanted to structure it a bit differently
so that certain parameters came in nested under others (hash within
a hash) like so:

```
{
  'id' => 5,
  'some_category' => {
    'a_key' => 'another value',
    'a_second_key' => 'yet another value',
    'inner_inner_hash' => {
      'key' => 'value'
    }
  },
  'something_else' => 'aaahhhhh'
}
```

Here's how we would accomplish that:

```
url = Addressable::URI.new(
  scheme: 'http',
  host: 'localhost',
  port: 3000,
  path: '/users/5.json',
  query_values: {
    'some_category[a_key]' => 'another value',
    'some_category[a_second_key]' => 'yet another value',
    'some_category[inner_inner_hash][key]' => 'value',
    'something_else' => 'aaahhhhh'
  }
)
```

If we follow this bracket notation, Rails will nest parameters for
us. The rule is that the keys with brackets gets nested deeper in the
params.

Try it out a few times with both GETs and POSTs.

## Rendering JSON

For a few requests, play around with rendering JSON.

```
render json: {'a_key' => 'a value'}
```

Also play around with the `to_json` method that Rails adds to the
`Object` class. See how various Ruby data structures are converted to
JSON.

## Using Models

So, now we know how to set up routes, how to set up matching
controller actions, how to send and process incoming data through
parameters, and how to render something back to the requestor. Let's
mix in some models.

Build a `User` model with name and email. Write a migration to add
columns for `name` and `email`. Migrate your database and add a couple
users through the console. Add validations for presence of name and
email.

In your `UsersController#index`, grab all the users and render them as
JSON. Remember that when you hand `render json:` anything, it
automatically calls `to_json` on it for you.

Make the request in your script and make sure you're getting the right
JSON back. Check your server log and note that the SQL that ran is
logged there for you. All SQL queries your app makes will show up in
the server log - yet another useful piece of information that the log
contains.

```
Started GET "/users.json" for 127.0.0.1 at 2013-12-16 10:12:40 -0800
Processing by UsersController#index as JSON
  User Load (0.2ms)  SELECT "users".* FROM "users"
Completed 200 OK in 72ms (Views: 2.7ms | ActiveRecord: 3.6ms)
```

Congrats! Applications, and especially web APIs, are all about
connecting data in your database with the outside world. You've just
done that.

### Creating a User through the API

Let's begin to provide a way to create a new user through the
API. Here's a start for a `create` action:

```ruby
# app/controller/users_controller.rb
def create
  user = User.new(params[:user].permit(:user_attributes_here))
  user.save!
  render json: user
end
```

Likewise, let's write a method in `my_script.rb` to send the
attributes:

```ruby
# my_script.rb
def create_user
  url = Addressable::URI.new(
    scheme: 'http',
    host: 'localhost',
    port: 3000,
    path: '/users.json'
  ).to_s

  puts RestClient.post(
    url,
    { user: { name: "Gizmo", email: "gizmo@gizmo.gizmo" } }
  )
end
```

Notice how we've namespaced all the parameters of the user to create
under `:user`. This leverages mass-assignment to set all the uploaded
attributes. This is an extremely common Rails pattern: pretty much
every time we upload parameters we will nest them under in an inner
hash to use for mass assignment.

### Handling Submission Errors

What if the user doesn't upload valid parameters for a new cat?

```
puts RestClient.post(
  url,
  { user: { name: "Gizmo" } }
)
```

This doesn't upload the required email attribute. The controller will
create a `user` object, but when it calls `save!` the validation will
fail and an error will be raised.

To inform the user of what went wrong, it is typical to send back
error messages. Let's modify our controller code to send back the
errors as JSON in the event of failure:

```ruby
def create
  user = User.new(params[:user].permit(:name, :email))
  if user.save
    render json: user
  else
    render(
      json: user.errors.full_messages, status: :unprocessable_entity
    )
  end
end
```

Note that if the save of the user fails, we send back the errors to
the client. We also set the status code. By default the status code
will be 200 (OK); if something has gone wrong, use a non-200 code to
indicate this. In this case, we will return a status code
of 422. Rails gives us names for these various codes so that the code
is a bit more semantic. Here is a list of the
[Rails status code names][rails-codes].

[rails-codes]: http://guides.rubyonrails.org/layouts_and_rendering.html#the-status-option

Now build some other controller actions:

* show
* update (you'll want to use `ActiveRecord::Base#update`)
* destroy

While you're at it, try refactoring the `params[...].permit(...)`
stuff into its own method. If you need an example, check out the
[controllers reading][strong-params-example].

Play around with them. Think about what each action's purpose is, what
data is coming in (params), what your controller needs to do with
models, and what it ultimately should render.

[strong-params-example]: https://github.com/appacademy/rails-curriculum/blob/master/w4d2/basic-controllers.md#drying-out-strong-parameters
