You will continue to develop your application from the point you arrived at the end of week 2. The material that follows comes with the assumption that you have done all the exercises of the previous week. In case you have not done all of them, you can take the sample answer to the previous week from the model answer found in the exercise submission system. If you already got most of the previous week exercises done, it might be easier if you complement your own answer with the help of the material.

## Rails developer workflow

The optimal way to program on Rails is evidently different than eg. Java. As a rule, when you are programming on Rails you should _avoid_ writing lots of code, and testing it only later on when you go to the page which implements your code. A partial reason for this is the fact Rails is a dynamically typed and interpreted language, which makes it impossible for even the best IDEs to check the program syntax. On the other hand, the fact that it is an interpreted language together with its console tools (the console itself and the debugger) makes it possible to test the functionality of smaller code chunks before they are put into the edited code file.

Let us take for example what we implemented last week, the implementation of the avarage value of beer ratings, and let us follow a natural Rails workflow.

Each beer contains a collection of ratings:


```ruby
class Beer < ApplicationRecord
  belongs_to :brewery
  has_many :ratings
end
```


We have to create the method <code>average</code> to the beer

```ruby
class Beer < ApplicationRecord
  belongs_to :brewery
  has_many :ratings, dependent: :destroy

  def average
    # code here
  end
end
```

If we wanted to do it in "Java's way" we could find the sum by going through all the ratings item after item and we could divide it by the number of items.

All Ruby's things which have something to do with collections (for instance tables and <code>hash_many</code> field) contain the auxiliary methods provided by the Enumerable module (ks. ks. http://ruby-doc.org/core-2.5.1/Enumerable.html). Now we want to use the auxiliary methods to find out the avarage value.

When you write the code, you should _definitely_ use your console. In fact, the debugger would be an even better option than the console. The debugger will open the console straight in the context for which we are writing the code. Add the <code>binding.pry</code> command to the method call, which starts the debugger:

```ruby
class Beer < ApplicationRecord
  belongs_to :brewery
  has_many :ratings, dependent: :destroy

  def average
    binding.pry
  end
end
```

Open the console (command _rails c_ on the command line), read from the database an object containing the ratings, and call the method <code>average</code>:

```ruby
irb(main):026:0> b = Beer.first
   (0.1ms)  SELECT sqlite_version(*)
  Beer Load (5.0ms)  SELECT "beers".* FROM "beers" ORDER BY "beers"."id" ASC LIMIT ?  [["LIMIT", 1]]
=>
#<Beer:0x00007f044a848a00
...
irb(main):027:0> b.average
[7, 14] in /myapp/app/models/beer.rb
     7|   def to_s
     8|     "#{name} #{brewery.name}"
     9|   end
    10|
    11|   def average
=>  12|     binding.pry
    13|   end
    14| end
=>#0	Beer#average at /myapp/app/models/beer.rb:12
  #1	<main> at (irb):27
  # and 28 frames (use `bt' command for all frames)
(rdbg)
```

a debugger session will open inside the method. You will find all the information about that beer.

You have access to the object itself, by using the reference <code>self</code>


```ruby
(rdbg) self
#<Beer:0x00007f044a848a00
 id: 1,
 name: "Iso 3",
 style: "Lager",
 brewery_id: 1,
 created_at: Mon, 08 Aug 2022 17:13:09.108046000 UTC +00:00,
 updated_at: Mon, 08 Aug 2022 17:13:09.108046000 UTC +00:00>
```

and you have access to the object fields using either dot notation or simply the field name:

```ruby
(ruby) self.name
"Iso 3"
(rdbg) style
"Lager"
(rdbg)
```

Notice that if you want to modify the object field value inside a method, you have to use dot notation:

```ruby
def methid
  #following methods print to console the value of field 'name'
  puts self.name
  puts name

  # initiates a variable 'name'  inside the method ja gives it a value
  name = "StrongBeer"

  # edit the value of field 'name'
  self.name = "WeakBeer"
end
```

This means you can refer to beer ratings from inside a beer method using the field name <code>ratings</code>:


```ruby
(rdbg) ratings
  Rating Load (3.6ms)  SELECT "ratings".* FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
[#<Rating:0x00007f044927afe8
  id: 2,
  score: 22,
  beer_id: 1,
  created_at: Fri, 19 Aug 2022 14:09:06.293428000 UTC +00:00,
  updated_at: Fri, 19 Aug 2022 14:09:06.293428000 UTC +00:00>,
 #<Rating:0x00007f0449285f38
  id: 3,
  score: 17,
  beer_id: 1,
  created_at: Fri, 19 Aug 2022 14:09:11.750743000 UTC +00:00,
  updated_at: Fri, 19 Aug 2022 14:09:11.750743000 UTC +00:00>]
```

Take a look at the singular ratings:
```ruby
(ruby) ratings.first
#<Rating:0x00007f044927afe8
 id: 2,
 score: 21,
 beer_id: 1,
 created_at: Fri, 19 Aug 2022 14:09:06.293428000 UTC +00:00,
 updated_at: Fri, 19 Aug 2022 14:09:06.293428000 UTC +00:00>
```

if you want to sum up ratings, you have to take the value of the <code>score</code> field from each rating object:

```ruby
(ruby) ratings.first.score
21
```

The <code>map</code> method of the enumerable module provides us with a way to make a new collection out of an old one. You can retrieve the items of the new collection from the original collection, by executing a mapping function to each of the orginal items.

If you use the name <code>r</code> to refer to the items of the original collection, the mapping function will be simple:

```ruby
(ruby) r = ratings.first
#<Rating:0x00007f044927afe8>
(ruby) r.score
21
```

You can try what <code>map</code> will do:

```ruby
(ruby) ratings.map { |r| r.score }
[22, 17]
```

the mapping function is given as a code chunk between curly brackets which is given as parameter to the method <code>map</code>. The code chunk could also be defined using the <code>do end</code> pair. They both bring about the same result:

```ruby
(ruby) ratings.map do |r| r.score end
[22, 17]
```

The map method makes use of the rating collection to help you build the values of the table ratings. You'll have to sum these values next.

Rails has added the method [sum](http://apidock.com/rails/Enumerable/sum) to all Enumberables. Try that out on the table you got with map.

```ruby
(ruby) ratings.map { |r| r.score }.sum
39
```

In order to find out the avarage value, you will still have to devide the sum by the total number of items. Check how the <code>count</code> method works:

```ruby
(ruby) ratings.count
  Rating Count (2.2ms)  SELECT COUNT(*) FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
2
```

and then form a oneliner to find the avarage value:


```ruby
(ruby) ratings.map { |r| r.score }.sum / ratings.count
  Rating Count (2.2ms)  SELECT COUNT(*) FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
19
```

you will see that the result is rounded erroneously. The problem is evidently that both the devidend and divisor are integers. Change to float one of them. Before you do it, check how the method works to change integers to floats:

```ruby
> 1.to_f
=> 1.0
```

If you don't know how to do something with Ruby, Google knows.

Think of a proper search term and you will have an answer quite certainly. You'll have to be careful though, and check out a couple of Google answers at least. You will have to make sure at least that the answer is for a version of both Ruby and Rails which are modern enough.

In Ruby and Rails there is typically a ready-made method or gem for almost everything, so you should always google or check the documentation instead of reinventing the wheel.

You can make now the final version of your code to find the avarage value:

```ruby
(ruby) ratings.map { |r| r.score }.sum / ratings.count.to_f
  Rating Count (2.4ms)  SELECT COUNT(*) FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
19.5
```

The code is now ready and tested, so you can copy it into a method:

```ruby
class Beer < ApplicationRecord
  belongs_to :brewery
  has_many :ratings, dependent: :destroy

  def average
    ratings.map{ |r| r.score }.sum / ratings.count.to_f
  end

end
```

Test the method now: exit from the debugger, _loading_ the new code, retrieving the object and executing the method:


```ruby
irb(main):001:0> b = Beer.first
irb(main):002:0> b.average
  Rating Load (1.8ms)  SELECT "ratings".* FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
  Rating Count (2.3ms)  SELECT COUNT(*) FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
=> 19.5
```

The following test will reveal that there is something wrong, however:

```ruby
irb(main):003:0> b = Beer.second
irb(main):004:0> b.average
  Rating Load (2.3ms)  SELECT "ratings".* FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 2]]
  Rating Count (3.1ms)  SELECT COUNT(*) FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 2]]
=> NaN
```

Rating avarage value of the second beer in the database is <code>NaN</code>. Go back to your debugger. Write the command <code>binding.pry</code>in the method for the avarage value, reload the code and call the method for the problematic object:

```ruby
irb(main):008:0> b.average
[7, 15] in /myapp/app/models/beer.rb
     7|   def to_s
     8|     "#{name} #{brewery.name}"
     9|   end
    10|
    11|   def average
=>  12|     binding.pry
    13|     ratings.map{ |r| r.score }.sum / ratings.count.to_f
    14|   end
    15| end
```

Evaluate the expression parts in the debugger:

```ruby
(ruby) ratings.map{ |r| r.score }.sum
  Rating Load (2.5ms)  SELECT "ratings".* FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 2]]
0
(ruby) ratings.count.to_f
  Rating Count (2.1ms)  SELECT COUNT(*) FROM "ratings" WHERE "ratings"."beer_id" = ?  [["beer_id", 2]]
0.0
```


We are dividing integers by zero. See the result of the operation:

```ruby
(ruby) 0/0.0
NaN
```

In order to prevent a number being devided by zero, the method will have to handle the case separately:

```ruby
def average
  return 0 if ratings.empty?
  ratings.map{ |r| r.score }.sum / ratings.count.to_f
end
```

We are using an oneliner-if and the collection method <code>empty?</code> which evaluates whether the collection is empty and will be evaluated as true if it is so. This is Ruby's way to check emptiness, whereas a Java user would write:

```ruby
def average
  if ratings.count == 0
    return 0
  end
  ratings.map{ |r| r.score }.sum / ratings.count.to_f
end
```

In any case, you should comply to the peculiar style of the language you are using, expecially if you are dealing with a project where there are various different developers.

If using the debugger has not become a routine yet, do review last week's debugger material.

## Rubocop: it's all about style

In bigger software projects teams usually set common styling policies, eg. naming conventions, how brackets are placed, where to use space and where not to. Rail conventions already cover some of these, namely on class and  method naming level.

Let's implement [Rubocop](https://github.com/rubocop-hq/rubocop), which will help us define styling rules for our project and to enforce them. Rubocop is a similar _static analysis_ tool such as [ESLint](https://eslint.org/) from the JavaScript world and [checkstyle](http://checkstyle.sourceforge.net/) for Java.

Rubocop is installed from your command line
 
    gem install rubocop

The styling rules monitored by Rubocop are defined in _.rubocop.yml_ that is placed in the project root. Create the file in your project (note the dot at the start of the name) and copy content for it from [here](https://github.com/mluukkai/WebPalvelinohjelmointi2022/main/misc/.rubocop.yml).

The rules defined there are based on the [Relaxed Ruby](https://relaxed.ruby.style/) style but they are a bit stricter on some points. The file contents also define that some files are to be left out of any style checks.

A code style check is executed with the command _rubocop_ on the command line.

There are quite a few problems in the code, for example:

<pre>
app/models/beer.rb:8:5: C: Layout/EmptyLineAfterGuardClause: Add empty line after guard clause.
    return 0 if ratings.empty?
    ^^^^^^^^^^^^^^^^^^^^^^^^^^
</pre>

Line 8 of file _beer.rb_ breaks the rule [Layout/EmptyLineAfterGuardClause](http://docs.rubocop.org/en/latest/cops_layout/#layoutemptylineafterguardclause).

Documentation of the [rules](http://docs.rubocop.org/en/latest/cops/) tells us what this is all about: The problem is that there is no empty line following the first line, which is a so called _guard clause_, of our recently made method _average_. 

```ruby
def average
  return 0 if ratings.empty?
  ratings.map{ |r| r.score }.sum / ratings.count.to_f
end
```

The next error

<pre>
app/models/concerns/rating_average.rb:9:38: C: Layout/SpaceAroundOperators: Surrounding space missing for operator +.
    ratings.reduce(0.0){ |sum, r| sum+r.score } / ratings.count    
                                     ^
</pre>

breaks the rule that [mathematical operators must have empty spaces before and after](http://docs.rubocop.org/en/latest/cops_layout/#layoutspacearoundoperators).

Many of our problems have to do with missing or extra spaces and line-breaks:
<pre>
app/models/concerns/rating_average.rb:10:6: C: Layout/TrailingWhitespace: Trailing whitespace detected.
  end
     ^^
app/models/concerns/rating_average.rb:11:1: C: Layout/EmptyLinesAroundModuleBody: Extra empty line detected at module body end.
app/models/concerns/rating_average.rb:12:4: C: Layout/TrailingBlankLines: Final newline missing.
end

app/models/rating.rb:7:1: C: Layout/TrailingWhitespace: Trailing whitespace detected.
</pre>


> ## Exercise 1
>
> Fix all style errors in your code to fit our defined styling rules.
>
> NOTE: You can also run the check for just one  file/directory. Eg. command _rubocop app/models/beer.rb_ checks file _beer.rb_
>
> NOTE2: If you don't quite understand some error, check the [documentation](https://docs.rubocop.org/rubocop/.

> ## Exercise 2
>
>Add a rule that forbids methods over 15 lines long. Check that _rubocop_ announces if you you have a too long method in your code.
> 
> Instructions for defining style rules can be found in the Metrics section of [documentation](https://docs.rubocop.org/rubocop/cops_metrics.html).

Rubocop might mention in its reports that some errors could be fixed automatically:


```bash
31 files inspected, 19 offenses detected, 19 offenses autocorrectable
```

You can fix such errors automatically with the command `rubocop -A`. 

From now on, we recommend that you make sure that any code you create follows Rubocop's rules. You can edit the already configured rules to your liking if you wish.
## User and session

Next, you will expand your application, so that users will be able to register a user name for themselves in the system.
You will soon modify the functionality so that each rating will belong to a registered user.

![mvc picture](http://yuml.me/4abc9b51.png)

Start with creating a user object which has a user name, and later add a password too.

Create a model, a view, and a controller for the user, with the command <code>rails g scaffold user username:string</code>

New users are created according Rails conventions with the form at the address <code>users/new</code>. It would be more natural however, if the address were <code>signup</code>. Add an optional route in routes.rb.

```ruby
get 'signup', to: 'users#new'
```

So the HTTP GET request to the signup address will also be handled by the Users controller method <code>new</code>.

HTTP is a stateless protocol, which means that all the requests executed with an HTTP protocol are mutually independent. If we want to implement a state in our Web application, for instance user registration or a web store shopping basket, the information of a Web session state will have to be transmitted together with every browser HTTP request. The most common way to transmit state information is to use cookies, see http://en.wikipedia.org/wiki/HTTP_cookie

To tell a long story short, the idea behind cookies is the following: when the browser tries to access a Web site, the server can answer the browser and send a request to store a coockie. After that, the browser will add a cookie to all the HTTP requests for the Web site. A cookie is nothing else than a small amount of data, and the server can make use of the cookie data as it prefers to recognise browser owning the cookie.

Rails application developers will not have to work with cookies directly, because thanks to cookies, Rails has implemented __sessions__ which work at higher level and which are used by the application to "remember" browser information, such as the user identity, and the time of various HTTP requests, see
http://guides.rubyonrails.org/action_controller_overview.html#session.

Let's try to use sessions to remember an user's latest rating. In Rails applications, you have access to the session of a user (or of a browser) who did an HTTP request through the object <code>session</code> which works like a hash.

Store the rating session by adding the following chunk in the rating controller:

```ruby
def create
  # save the creted rating into a variable
  rating = Rating.create params.require(:rating).permit(:score, :beer_id)

  # save the rating to a session
  session[:last_rating] = "#{rating.beer.name} #{rating.score} points"

  redirect_to ratings_path
end
```

add the following chunk of code to the application layout (tiedostoon app/views/layouts/application.html.erb) to make sure the rating will be seen in all pages:

```erb
<% if session[:last_rating].nil? %>
  <p>no ratings given</p>
<% else %>
  <p>previous rating: <%= session[:last_rating] %></p>
<% end %>
```

Try your application now. There is nothing stored in the session at the beginning, and the value of <code>session[:last_rating]</code> is <code>nil</code>, meaning that the page should say "no ratings given". Create a rating and see that it is saved in the session. Create a new rating again and see that it will overwrite the session data.

Open your application in an incognito window or another browser now. You will see that the session value is <code>nil</code> in the other browser. This means that the session is dependent on the browser.

## Signing in

The idea is implementing the registration functionality so that when users sign in, the <code>ID</code> of the corresponding <code>User</code> object is  saved in the session. When users sign out, the session is reset.

Attention: almost any kind of object can be saved in the session basically, and you could save in the session also the <code>User</code> object corresponding to the users who signed in. It is a best practice however (see http://guides.rubyonrails.org/security.html#session-guidelines) to store as little data as possible in the session (you can save up to 4kB of information in Rails sessions, by default). You should store just the the amount of data you need to identify users who signed in, whereas their other information can be retrieved from the database if needed.

Create now a controller to sign in and out of your application. Typically, you should follow Rails RESTful idea and conventional path names to implement the signing in functionality.

You can think of the session as something which is born when users sign up, and a session can almost be considered as the same kind of resource as a beer, fo instance. Accondingly, the controller for signing up will be called <code>SessionsController</code>.

A session resource is anyway different than beers, for instance, because users either are or are not signed in, in a particular moment. Differently than beers, a user can have not many but maximum one session. Differently than beers, it does not make sense to have a list with all sessions. Routes should be written in singular and this can be at least done when you create a session routes into routes.rb with the <code>resource</code> command:

    resource :session, only: [:new, :create, :destroy]

**Attention: make sure you write the routes.rb definition exactly in the form above, and not _resources_ as it is in the definitions of other paths.**

The sign up address is now **session/new**. The POST call to the address **session** executes the signing in, creating a session for the user. Users sign out when their session is destroyed, with the execution of a POST-delete call to the address **session**.

Create a controller for sessions (in the file app/controllers/sessions_controller.rb):

```ruby
class SessionsController < ApplicationController
  def new
    # render the signing up page
  end

  def create
    # retrieves from the database the user that matches the username
    user = User.find_by username: params[:username]
    # saves the user ID who signed up (if the user exists)
    session[:user_id] = user.id if not user.nil?
    #regirects the user to their own page
    redirect_to user
  end

  def destroy
    # resets the session
    session[:user_id] = nil
    # redericts the application to the main page
    redirect_to :root
  end
end
```

Notice that even though routes are written in the singular form now (**session** and *session/new**), the controller and the view directory spelling conventions should follow Rails normal plural form.

The code of the sign up page app/views/sessions/new.html.erb is below:

```erb
<h1>Sign in</h1>

<%= form_with url: session_path, method: :post do |form| %>
  <%= form.text_field :username %>
  <%= form.submit "Log in" %>
<% end %>
```

Differently than the form you made for the ratings (review [the information from last week](https://github.com/mluukkai/webdevelopment-rails/main/week2.md#form-and-post)), the form you are going to create is not based on an object and you will create it with method <code>form_with</code>-, see http://guides.rubyonrails.org/form_helpers.html#dealing-with-basic-forms

Sending the form will cause an HTTP POST request to the session_path (notice the singular) that is the address **session**.

The method to handle the call takes the user ID which was saved in the <code>param</code> object and retrieves the corresponding user object from the database to save the object ID to the session, if the object exists. At the end, users are redirected to their own page. Once more, the controller code looks like below:

```ruby
def create
  user = User.find_by username: params[:username]
  session[:user_id] = user.id if not user.nil?
  redirect_to user
end
```

Attention 1: the command <code>redirect_to user</code> is a short form for <code>redirect_to user_path(user)</code>, see [week 1](https://github.com/mluukkai/webdevelopment-rails/main/week1.md#review-naming-conventions-for-paths-and-controllers).

Attention 2: Instead of the <code>if not</code> combination we can use <code>unless</code> in Ruby, and the second line of the method could have been written like

```ruby
  session[:user_id] = user.id unless user.nil?
```

The best form for the command is however
```ruby
  session[:user_id] = user.id if user
```

You see, in Ruby all values apart from _nil_ and _false_ are evaluated as true. Therefore now the command is performed if _user_ is something else than _nil_ and this is exactly how we want it to work.

Add the following code to the application layout to add the name of the signed in user to all the pages (you can delete now the session code we added for training in the last section):

```erb
<% if not session[:user_id].nil? %>
  <p><%= User.find(session[:user_id]).username %> signed in</p>
<% end %>
```

Users can now sign in the application at the address [http://localhost:3000/session/new](/session/new) (assuming users have been created in the application at http://localhost:3000/signup). Signing out does not work yet.

**ATTENTION:** if you see the error message <code>uninitialized constant SessionController></code> **make sure that you properly defined all the routes in routes.rb, like**

```ruby
  resource :session, only: [:new, :create, :destroy]
```

> ## Exercise 3
>
> Implement all the changes above and make sure that signing in works out smoothly with an existing user ID (so that a signed in user is shown on the page.) You can create an user at [http://localhost:3000/signup][/signup]). Even though signing out is not possible, you can sign in with a new ID and the old signing in will be overwritten.

## Controller and view auxiliary method

Making a database request in the view code is quite bad (as we did a moment ago with the code added to the application layout). Add the following method to the class <code>ApplicationController</code>:

```ruby
class ApplicationController < ActionController::Base
  # defines that the method current_user is accessible also from views
  helper_method :current_user

  def current_user
    return nil if session[:user_id].nil?
    User.find(session[:user_id])
  end
end
```

Because all the application controllers inherit the class <code>ApplicationController</code>, the method you are defining will be available to all the controllers. We also defined the method <code>current_user</code> as a helper method, which will be available not only to all the controllers but to all views, too. We can change the code added to the application layout in the following way:

```erb
<% if not current_user.nil? %>
  <p><%= current_user.username %> signed in</p>
<% end %>
```

We can also style it more nicely:

```erb
<% if current_user %>
  <p><%= current_user.username %> signed in</p>
<% end %>
```

Just <code>current_user</code> is enough as a condition as Ruby evaluates `nil` as false.


The signing in address __sessions/new__ is annoying. Create another more natural address and call it __signin__. Define also a route to sign out. Implement these two things by adding the following code to routes.rb:

```ruby
  get 'signin', to: 'sessions#new'
  delete 'signout', to: 'sessions#destroy'
```

The signing in form can be found now at the address [http://localhost:3000/signin][/signin] and signing out is possible through the HTTP DELETE request to the address _signout_.

The following would also have worked

```ruby
  get 'signout', to: 'sessions#destroy'
```

so that singing out would happen through HTTP GET. It is not a best practice, though, that HTTP GET requests modify the application status. Stick to the the REST philosophy conventions, which tell to destroy resources with HTTP DELETE requests. In this case, the resource is only a broader concept, users signing in.

> ## Exercise 4
>
> Modify the navigation bar in the application layout so that the bar will contain links to sign in and out. Notice that you should use HTTP DELETE for the signing out functionality. Rails version 7  links don't have direct support for using delete but you can [make links use delete method](https://github.com/heartcombo/devise/issues/5439#issuecomment-997927041)
>
> In addition to the previous two, add a link to the page with all users to the navigation bar, as welll as the signed up user name, which should link to the user's personal page. When the user is signed in, the bar should also show a link to create a new beer rating.
>
> Remember: you can see the defined routes and path methods using the command <code>rails routes</code> from the command line, or you can go to whatever unexisting application address, like [http://localhost:3000/wrong](http://localhost:3000/wrong)

At the end of the exercise, your application will look more or less like the following, if a user is signed in:

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-1.png)

and if users are not signed in, it will be like below (notice also the sign up link now):

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-2.png)

## User ratings

Now we'll make ratings belong to an user. After that, associations between objects should look like this:

![pic](http://yuml.me/4abc9b51.png)

The change will be nothing new at model level:

```ruby
class User < ApplicationRecord
  has_many :ratings   # user has many ratings
end

class Rating < ApplicationRecord
  belongs_to :beer
  belongs_to :user   # rating belongs to an user

  def to_s
    "#{beer.name} #{score}"
  end
end
```

Our solution does not work like this, however. Because of the connections, you need a reference to the user ID as foreign key in your _rating_ database table. All the changes in Rails databases are implemented in Ruby code with the help of _migrations_. Create a migration which adds a new column. Generate a migration file from the command line first, using the command:

    rails g migration AddUserIdToRatings

A file will appear in the directory _db/migrate_, with the following contents

```ruby
class AddUserIdToRatings < ActiveRecord::Migration[7.0]
  def change
  end
end
```

Notice that the directory already contains its own migration files for all the database tables created. Each migration will contain the information about the change in the database as well as how to cancel such change if needed. If the migration is simple enough, so that Rails can derive also  the cancel operation based on the addition you executed, it will be enough for the migration if it contains only the method <code>change</code>. If the migration is more complex, you will have to define the methods <code>up</code> and <code>down</code> which define separately how to execute the migration and how to cancel it.


This time, the migration we need is simple:

```ruby
class AddUserIdToRatings < ActiveRecord::Migration[7.0]
  def change
    add_column :ratings, :user_id, :integer
  end
end
```

In order to implement the migration change, execute the well-known commande <code>rails db:migrate</code> from the command line.

Migration are a vast field, and we will go back to them later on in the course. More information about migrations can be found at the address http://guides.rubyonrails.org/migrations.html

You will see from the console, that the connection between objects is implemented correctly:

```ruby
> u = User.first
> u.ratings
  Rating Load (0.3ms)  SELECT "ratings".* FROM "ratings" WHERE "ratings"."user_id" = ?  [["user_id", 1]]
=> []
```

The ratings you have given previously do not have an user at the moment:

```ruby
> r = Rating.first
> r.user
 => nil
>
```

We will define the first user created as the user of all the existing ratings:

```ruby
> u = User.first
> Rating.all.each{ |r| u.ratings << r }
>
```

**ATTENTION:** creating ratings from the user interface does not work properly at the moment, because the ratings created in this way will not belong to any user. You will fix this soon.

> ## Exercise 5
>
>  Add the followin things in the user page, which is the view app/views/users/show.html.erb:
> – the amount of that user's ratings and their avarage (attention: use the module defined last week, <code>RatingAvarage</code> to find the avarage!)
> – a list of the user ratings and the possibility to delete them
>
> Instead of the partial, make changes directly to app/views/users/show.html.erb, completely remove the reference to partial from that file

The user page will look more or less like below:

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-3.png)


Creating new ratings from the www-page does not work yet, because ratings are not connected to the signed in user. Change the rating controller so that a signed in user will be linked to the rating created.

```ruby
def create
  rating = Rating.new params.require(:rating).permit(:score, :beer_id)
  rating.user = current_user
  rating.save
  redirect_to current_user
end
```

Notice that <code>current_user</code> is the method we just added to the class <code>ApplicationController</code>, and it returns the signed up user by executing the code:

```ruby
  User.find(session[:user_id])
```

After we create a rating, the controller redirects the browser to the page of the signed in user.

> ## Exercise 6
>
> Change your application so that users can not delete ratings at the page with all ratings. Also, it should be possible to see the name of who created a rating next to the rating, and there should be a link to their page.

The page with all ratings should look like below, after doing the exercise:

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-4.png)

## Polishing the signing up

You application will give you pains at the moment, if users try to sign in with a user name which does not exist.

Change your application to dedirect users back to the sign in page, if signing in does not work out. Change the session controller like below:

```ruby
def create
  user = User.find_by username: params[:username]
  if user.nil?
    redirect_to signin_path
  else
    session[:user_id] = user.id
    redirect_to user
  end
end
```

change the code above further to give messages to user, explaining what happened:

```ruby
def create
  user = User.find_by username: params[:username]
  if user.nil?
    redirect_to signin_path, notice: "User #{params[:username]} does not exist!"
  else
    session[:user_id] = user.id
    redirect_to user, notice: "Welcome back!"
  end
end
```

If you want that your message will be seen in the sign up page, add the element below to the view ```app/views/sessions/new.html.erb```:

```erb
<p style="color: red"><%= notice %></p>
```

The element is ready in the user page template (unless you have deleted it by mistake), so the message will work there.

The __flashes__ are the messages connected to eg. redirections and that are remembered until the next HTTP request and that are shown on the page when needed. They are implemented in Rails thanks to the sessions, more about this at http://guides.rubyonrails.org/action_controller_overview.html#the-flash

## Validating object fields

Our application has a small problem now: it is possible to create many users with the same user name. In the <code>create</code> method of our user controller there should be the functionality to check that the <code>username</code> is not used.

A versitile mechanism to validate object fields comes built-in with Rails, see http://guides.rubyonrails.org/active_record_validations.html.

Validating the unicity of user names is simple, and you just need to add a short chunk of code to your User class:

```ruby
class User < ApplicationRecord
  include RatingAverage

  validates :username, uniqueness: true

  has_many :ratings
end
```

If you try to create a user which exists already, you will see that Rails will be able to generate an appropriate error message automatically.

Rails (or more properly, ActiveRecord) executes the object validations right before trying to store the object in the database, for instance with the operations <code>create</code> or <code>save</code>. If the validation fails, the object will not be stored.

Add other validations right away too. Add the requirement that user ID length should be at least three-character long, implementing the following line to the User class:

```ruby
  validates :username, length: { minimum: 3 }
```

If various validation rules concern the same attribute, they can all be connected under one <code>validates :attribute</code> call:

```ruby
class User < ApplicationRecord
  include RatingAverage

  validates :username, uniqueness: true,
                       length: { minimum: 3 }

  has_many :ratings
end
```

The controllers created with Rails scaffold generator are implemented in a way that if the validation works out and the object is stored in the database, the browser is redirected to the page of the object created. If the validation does not work out, it shows again the form to create objects and the error message is rendered in the page with the form.

How can the controller know, whether the validation worked out? The validation happens when it tries to save something in the database. If the controller saves the object with the method <code>save</code>, the controller can test the method return value whether the validation worked out or not:

```ruby
  @user = User.new(parameters)
  if @user.save
  	# the validation worked out, so the browser is redirected to the appropriate page
  else
    # the validation did not work out, so the view template :new is rendered
  end
```

The controller generated by scaffold is a bit more complex:

```ruby
def create
  @user = User.new(user_params)

  respond_to do |format|
    if @user.save
      format.html { redirect_to user_url(@user), notice: "User was successfully created." }
      format.json { render :show, status: :created, location: @user }
    else
      format.html { render :new, status: :unprocessable_entity }
      format.json { render json: @user.errors, status: :unprocessable_entity }
    end
  end
end
```

First of all, where does <code>user_params</code> come from which is used as parameter to create the object? You'll see that the following method is defined at the bottom of the file:

```ruby
    def user_params
      params.require(:user).permit(:username)
    end
```

so the first line of the method <code>create</code> is the same as

```ruby
   @user = User.new(params.require(:user).permit(:username))
```

So what does <code>respond_to</code> at the end of the method do? If the object is created with a normal form and and the browser expects to receive an HTML answer, the functionality will be this, by default:

```ruby
if @user.save
  redirect_to user_url(@user), notice: "User was successfully created."
else
  render :new, status: :unprocessable_entity 
end
```

the code chunk of the entry <code>format.html</code> (which technically is a method call) is executed in the code chunk of the command <code>respond_to</code> (which is also a method). However, if the HTTP POST call to create a user object was made so to expect an answer in json-form (which would happen for instance if the request was done in Javascript for a different service or Web page), it would execute the code of <code>format.json</code>. The syntax could look strange at first, but you'll soon get acquainted with it.

We'll continue with validations. Define that beer ratings have to be integers between 1-50:

```ruby
class Rating < ApplicationRecord
  belongs_to :beer
  belongs_to :user

  validates :score, numericality: { greater_than_or_equal_to: 1,
                                    less_than_or_equal_to: 50,
                                    only_integer: true }

  # ...
end
```

If users create inappropriate ratings, they won't be saved any more. You will notice however, that users won't receive any error message. The problem is that you created the form by hand, it does not contain error report functionality like the forms generated automatically with scaffold. Also the controller will never check whether the validation worked out.

Change first the rating controller method <code>create</code> so that it renders again the form to create ratings if the validation failed:


```ruby
def create
  @rating = Rating.new params.require(:rating).permit(:score, :beer_id)
  @rating.user = current_user

  if @rating.save
    redirect_to user_path current_user
  else
    @beers = Beer.all
    render :new, status: :unprocessable_entity
  end
end
```

The method creates first a Rating object with the command <code>new</code>, and this is not saved in the database yet. Then, it executes the database saving process with the method <code>save</code>. During this, it validates the object, and if this fails, the method returns false, and the object will not be saved in the database. In such case, the new view template will be rendered. Rendering the view template requires that the beer list is stored in the variable <code>@beers</code>.
[Rails 7 won't render the error messages](https://stackoverflow.com/questions/71751952/rails-7-signup-form-doesnt-show-error-messages) unless we return also a symbol :unprocessable_entity using HTTP status code 422. Read more about status codes from [wikipedia](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) or with pictures [here](https://http.cat/).

As you try to create an erroneous rating, the user remains in the view showing the form (which technically is rendered again after the POST call). However there is no error message, yet.

When the validation fails, Rails validator saves the error messages in the field <code>@rating.errors</code> (which belongs to the object <code>@ratings</code>.

Change the form to show the value of <code>@rating.errors</code>, if the field contains something:

```erb
<h2>Create new rating</h2>

<%= form_for(@rating) do |f| %>
  <% if @rating.errors.any? %>
    <%= @rating.errors.inspect %>
  <% end %>

  <%= f.select :beer_id, options_from_collection_for_select(@beers, :id, :to_s) %>
  score: <%= f.number_field :score %>
  <%= f.submit %>

<% end %>
```

If you create an erroneous rating now, you will be able to find out the reason from the object stored in the field <code>@rating.errors</code>.

Take the view template views/users/_form.html.erb as example and change your form (views/ratings/new.html.erb) like below:

```erb
<h2>Create new rating</h2>

<%= form_for(@rating) do |f| %>
  <% if @rating.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@rating.errors.count, "error") %> prohibited rating from being saved:</h2>

      <ul>
      <% @rating.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <%= f.select :beer_id, options_from_collection_for_select(@beers, :id, :to_s) %>
  score: <%= f.number_field :score %>
  <%= f.submit %>

<% end %>
```

When it finds valitation errors, the view template renders all the error message contained in <code>@rating.errors.full_messages</code>.

**Attention:** when the validation fails the redirection is not executed (why doesn't it work here?), but the view template is rendered instead, as it usually happens when you execute the <code>new</code> method.

You find help for the next exercises at
http://guides.rubyonrails.org/active_record_validations.html ja https://apidock.com/rails/v4.2.7/ActiveModel/Validations/ClassMethods/validates

> ## Exercise 7
>
> Add the following validations to your program
> * beer and brewery names are not empty
> * the brewery founding year is an integer between 1040-2022
> * the length of the username attribute of the User class is 3 – 30 characters

If you try to create a beer with an empty name, you get an error message
![kuva](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-9.png)

Why? If beer creation fails due to an error in validation the <code>create</code> method of beer controller executes the else branch. That is, re-renders the beer creation form. The form for beer creation uses the <code>@styles</code> variable containing the list of beer styles in generating the form. The reason for the error message is that this variable is not initialized in this case (as it is when coming from the controller method  <code>new</code>). The form also assumes that the variable <code>@breweries</code> contains the list of all breweries.
Therefore we can fix our issue by initializing these variables in the else branch:

```ruby
def create
  @beer = Beer.new(beer_params)

  respond_to do |format|
    if @beer.save
      format.html { redirect_to beers_path, notice: 'Beer was successfully created.' }
      format.json { render :show, status: :created, location: @beer }
    else
      @breweries = Brewery.all
      @styles = ["Weizen", "Lager", "Pale ale", "IPA", "Porter", "Lowalcohol"]
      format.html { render :new }
      format.json { render json: @beer.errors, status: :unprocessable_entity }
    end
  end
end
```


> ## Exercise 8
>
> ### doing this exercise is not essential to continue with the rest of the week's material, so you should not get stuck with it. You can also do this exercise after you have done the rest of the week.
>
> Improve exercise 7 validations so that the brewery founding year is an integer which is at least 1040 and maximum the current year. You cannot hard-code the year.
>
> Notice that this will not work as you want:
>
>   validates :year, numericality: { less_than_or_equal_to: Time.now.year }
>
> <code>Time.now.year</code> is evaluated when the program loads the class code. If the program starts to run at the end of 2021, in 2022 users will not to be able to register a 2022 brewery, because when the program started it evaluated 2021 as the year upper limit and the validation will fail.>
> A possible way is defining your own validation method http://guides.rubyonrails.org/active_record_validations.html#custom-methods
>
> You could find even an even shorter solution in terms of code, a hint could be lambda/Proc/whatever...


## Connections many to many

One beer has many ratings, and a rating has always one user, which means a beer has many users who made a rating. Similarly, a user has many ratings and a rating has one beer. This means that a user has many rated beers. The connection between beers and users is **many to many** where the rating table acts as a union table.

We can create this many to many connection at code level easily using the way we got acquainted with [last week](https://github.com/mluukkai/webdevelopment-rails/blob/main/week2.md#inderect-object-connection), the **has_many through** connection:

```ruby
class Beer < ApplicationRecord
  include RatingAverage

  belongs_to :brewery
  has_many :ratings, dependent: :destroy
  has_many :users, through: :ratings

  # ...
end

class User < ApplicationRecord
  include RatingAverage

  has_many :ratings
  has_many :beers, through: :ratings

  # ...
end
```

And the many to many connection will work for users:

```ruby
User.first.beers
=> [#<Beer:0x00007fbe23b8a770
  id: 1,
  name: "Iso 3",
  style: "Lager",
  brewery_id: 1,
  created_at: Sun, 21 Aug 2022 15:25:05 UTC +00:00,
  updated_at: Sun, 21 Aug 2022 15:25:05 UTC +00:00>,
 #<Beer:0x00007fbe23b8a608
  id: 1,
  # ...
```

and for beers:

```ruby
irb(main):007:0> Beer.first.users
   (0.2ms)  SELECT sqlite_version(*)
  Beer Load (2.3ms)  SELECT "beers".* FROM "beers" ORDER BY "beers"."id" ASC LIMIT ?  [["LIMIT", 1]]
  User Load (2.0ms)  SELECT "users".* FROM "users" INNER JOIN "ratings" ON "users"."id" = "ratings"."user_id" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
=>
[#<User:0x00007faf15b47aa0
  id: 1,
  username: "mluukkai",
  created_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00,
  updated_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00>,
 #<User:0x00007faf15b4dc20
  id: 1,
  username: "mluukkai",
  created_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00,
  updated_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00>
]
```

It seems to work, but it feels odd to refer to users who rated a beer with the name <code>users</code>. A more natural way to refer to users who rated a beer could be perhaps <code>raters</code>. This works if you change the connection definition in the following way:

```ruby
has_many :raters, through: :ratings, source: :user
```

By default, <code>has_many</code> will look for a table whose name is the same as its first parameter. Because <code>raters</code> is not the name of the connection destination, this has to be defined apart using the _source_ option.

The new name of our connection will work now:

```ruby
irb(main):009:0> Beer.first.raters
   (0.2ms)  SELECT sqlite_version(*)
  Beer Load (2.2ms)  SELECT "beers".* FROM "beers" ORDER BY "beers"."id" ASC LIMIT ?  [["LIMIT", 1]]
  User Load (2.0ms)  SELECT "users".* FROM "users" INNER JOIN "ratings" ON "users"."id" = "ratings"."user_id" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
=>
[#<User:0x00007faf160f7748
  id: 1,
  username: "mluukkai",
  created_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00,
  updated_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00>,
 #<User:0x00007faf160bad48
  id: 1,
  username: "mluukkai",
  created_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00,
  updated_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00>]
```

Because the same user can create various ratings of the same beer, the user will be seen various times among the beer raters. If you want that one rater is seen only once, you can do like this:

```ruby
irb(main):010:0> Beer.first.raters.uniq
  Beer Load (1.7ms)  SELECT "beers".* FROM "beers" ORDER BY "beers"."id" ASC LIMIT ?  [["LIMIT", 1]]
  User Load (2.2ms)  SELECT "users".* FROM "users" INNER JOIN "ratings" ON "users"."id" = "ratings"."user_id" WHERE "ratings"."beer_id" = ?  [["beer_id", 1]]
=>
[#<User:0x00007faf15cfd020
  id: 1,
  username: "mluukkai",
  created_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00,
  updated_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00>]
irb(main):011:0>
```

It would also be possible to define that by default beer <code>raters</code> should return individual users only once. You could implement this by setting *[scope](https://guides.rubyonrails.org/association_basics.html#scopes-for-has-many) \_distinct* to the <code>has_many</code> attribute, limiting the sets of associated objects so that each object is shown only once:


```ruby
class Beer < ApplicationRecord
  #...

  has_many :raters, -> { distinct }, through: :ratings, source: :user

  #...
end
```

More about defining connections in normal and more complicated circumstances at http://guides.rubyonrails.org/association_basics.html

Attention: there is also another way to create many-to-many connections on Rails, <code>has_and_belongs_to_many</code>, see http://guides.rubyonrails.org/association_basics.html#the-has-and-belongs-to-many-association which might be useful if the only purpose of your connection table is to establish a connection.

However the trend is to use the has_many through combination and explicitely defined connection tables, instead of the method has_and_belongs_to_many (because of its various issues). Among the others, Chad Fowler suggests that users should avoid using has_and_belongs_to_many in his book [Rails recepies](http://pragprog.com/book/rr2/rails-recipes), Obie Fernandez gives the same suggestion in his autoritative work [Rails 5 Way](https://leanpub.com/tr5w)

> ## Exercises 9-10: Beer clubs
>
> ### This and the following exercise are not essential to continue with the week material. You can also do them after the other exercises of the week.
>
> Extend the system so that users can be members of _beer clubs_.
>
> Use scaffold to create <code>BeerClub</code> with the attributes <code>name</code> (a string) <code>founded</code> (an integer) and <code>city</code> (a string)
>
> Create a many-to-many connection between <code>BeerClub</code> and <code>User</code>. Create a connection table for this, the <code>Membership</code> model, with the foreign keys to the objects <code>User</code> and <code>BeerClub</Code> as attributes (they are <code>beer_club_id</code> and <code>user_id</code>). You can use scaffold for this model too.
>
> At this point, you can implement the functionality to add members to beer clubs similarly as the current beer rating functionality – adding the link "join a club" to the navigation bar, through which registered users can be added to one of the beer clubs in the list.
>
> List all the members in the beer club page, and similarly, list all the beer clubs that a person belongs to on their page. Add a link to the list with all beer clubs in the navigation bar.
>
> You don't need to implement the functionality to remove a user from a beer club, yet.
>
> In this exercise you need to be careful with Rails naming conventions. The class defining a BeerClub is written BeerClub, the matching foreign key is beer_club_id and in other objects eg. Memberships, the beer clubs are referenced to as beer_club.

> # Exercise 11
>
> Refine the previous exercise so that users cannot join the same beer club multiple times.
>
> There are many ways to accomplish this but using validations might not be the most sensible way. It doesn't really make sense to even offer beer clubs that the user is already a member of on the joining form.

The following two pictures will help you understand what your application should look like after exercises 9–11.

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-5.png)

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-6.png)

## Password

Modify your application again so that users will have a password. Because of information security issues, you should never save passwords to the database. In the database, we only store the password digest, which was calculated with a one-way function. Let's implement a migration for this:

    rails g migration AddPasswordDigestToUser

the code of your migration (see the folder db/migrate) should be like below:

```ruby
class AddPasswordDigestToUser < ActiveRecord::Migration[7.0]
  def change
    add_column :users, :password_digest, :string
  end
end
```

notice that the name of the added column must be <code>password_digest</code>.

Add the code below to <code>User</code> class:

```ruby
class User < ApplicationRecord
  include RatingAverage

  has_secure_password

  # ...
end
```

<code>has_secure_password</code> (see http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html) provides the class with the functionality to save the password _digest_ into the database and the user can be authenticated when needed.

Rails uses the <code>bcrypt-ruby</code> gem to store the digest. Get started with it by adding the following line to the Gemfile

    gem 'bcrypt', '~> 3.1.7'

After this, run <code>bundle install</code> from the command line to set up the gem.

Try out the new functionality from the console now (you will have to restart the console to set up the new gem). It is recommend to also restart the application at this point.

__Also, remember to execute the migration!__

The password functionality <code>has_secure_password</code> adds the attributes <code>password</code> and <code>password_confirmation</code> to the object. The idea is that the password and its confirmation are placed in these attributes. When an object is stored in the database – for instance with the <code>save</code> method call – the digest which is stored in the database as value in the column <code>password_digest</code> will be calculated. The proper password, the attribute of <code>password</code> is not stored in the database, and only its representation is recorded by the object.


Storing a password for our user

```ruby
> u = User.first
> u.password = "secret"
> u.password_confirmation = "secret"
> u.save
  TRANSACTION (0.1ms)  begin transaction
  User Exists? (2.9ms)  SELECT 1 AS one FROM "users" WHERE "users"."username" = ? AND "users"."id" != ? LIMIT ?  [["username", "mluukkai"], ["id", 1], ["LIMIT", 1]]
  User Update (15.3ms)  UPDATE "users" SET "updated_at" = ?, "password_digest" = ? WHERE "users"."id" = ?  [["updated_at", "2022-08-25 11:41:12.367244"], ["password_digest", "[FILTERED]"], ["id", 1]]
  TRANSACTION (10.8ms)  commit transaction
=> true
```

The authentication happens thanks to the method <code>authenticate</code> which was added to the <code>User</code> object:

```ruby
>  u.authenticate "salainen"
=>
#<User:0x00007f320cdbba38
 id: 1,
 username: "mluukkai",
 created_at: Sun, 21 Aug 2022 15:35:05.281921000 UTC +00:00,
 updated_at: Thu, 25 Aug 2022 11:41:12.367244000 UTC +00:00,
 password_digest: "[FILTERED]">
irb(main):006:0>
```

the method <code>authenticate</code> returns <code>false</code> if the password given as parameter is wrong. If the password is right. the method returns the object itself.

Implement the functionality to check the password when users sign in. Change first the sing-in page (app/views/sessions/new.html.erb) so that in addition to asking the user name, it asks for the password as well (notice that the type of the form field is *password_field*, which only shows stars instead of the written password):

```erb
<h1>Sign in</h1>

<p id="notice"><%= notice %></p>

<%= form_with url: session_path, method: :post do |form| %>
  username <%= form.text_field :username %>
  password <%= form.password_field :password %>
  <%= form.submit "Log in" %>
<% end %>
```

and change the sessions controller so that it uses the method <code>authenticate</code> to verify whether the form password is right.

```ruby
def create
  user = User.find_by username: params[:username]
  # check that user exists and the password is correct
  if user && user.authenticate(params[:password])
    session[:user_id] = user.id
    redirect_to user_path(user), notice: "Welcome back!"
  else
    redirect_to signin_path, notice: "Username and/or password mismatch"
  end
end
```

Try out whether signing in works (**attention: you will have to restart rails server to set up bcrypt-gem**). So far, signing in only works for the users whose passwords were added from the console by hand.

Add a password input field to the user creation (that is to say, to the view view/users/_form.html.erb):

```erb
<div>
  <%= form.label :password, style: "display: block"%>
  <%= form.password_field :password %>
</div>

<div>
  <%= form.label :password_confirmation, style: "display: block"%>
  <%= form.password_field :password_confirmation %>
</div>
```


The controller auxiliary method <code>user_params</code> which is in charge of creating users has to be modified so that it can retrieve the password and its confirmation sent from the form:


```erb
 def user_params
   params.require(:user).permit(:username, :password, :password_confirmation)
 end
```

Try to see what happens if you give an erroneous value to the password confirmation.

Attention: If you get the error message  <code>BCrypt::Errors::InvalidHash</code> upon trying to sign in, it is most likely caused by the user not having a password set. Add the password from console and try again.

> ## Exercise 12
>
> Implement a user validation to the class, to make sure the password is at least four characters longs and it contains at least one capital letter and one figure (You don't need to worry about the scandic letters (ä, ö, ... )).

**Attention**: you can test Ruby's regular expressions with the Rubular application: http://rubular.com/. You can of course solve this exercise with other techniques as well.


## Deleting only one's own ratings

At this point, an user is able to delete the ratings of anyone else. Modify your application so that users can remove only their own ratings. It will be simple if it's verified in the rating controller:

```ruby
def destroy
  rating = Rating.find params[:id]
  rating.delete if current_user == rating.user
  redirect_to user_path(current_user)
end
```

so we execute the remove operation only if the ```current_user``` is the same as the rating user.

There is no reason actually why you should show the rating remove link in other pages than in the personal page of the signed in user. So change the user show page like below:

```erb
<ul>
  <% user.ratings.each do |rating| %>
    <li><%= "#{rating.to_s}" %>
      <% if @user == current_user %>
        <%= link_to "Delete", rating, data: {turbo_method: :delete} %>
      <% end %>
    </li>
  <% end %>
</ul>
```

Notice that simply removing the **delete** link does not prevent deleting other users ratings, because it is extremely easy to make an HTTP DELETE operation to the urls of a of ratings. Therefore, it is essential to check the identity of the signed-in user in the control method which executes the deletion.

> ## Exercise 13
>
> Each user's page [http://localhost:3000/user/1](http://localhost:3000/user/1) contains the button **destroy this user** , which can be used to destroy users and a link **edit** to edit their information.
>
> Show the editing and destroying links/buttons only on the signed in user's personal user page. 
> Change also the <code>update</code> and <code>destroy</code> methods of the User controller so that updating or deleting the object information can only be done by the signed-in user.

> ## Exercise 14
>
> Create a new user name, sign in as that user, and then distroy the user. Deleting the user name will cause an annoying error. **You will get over it by deleting the browser cookies.** Try to think what caused the error and fix the bug in the application too, so that deleting a user would not bring about an error situation.

> ## Exercise 15
>
> Extend your application so that when an user is deleted, their ratings are also automatically deleted. See https://github.com/mluukkai/webdevelopment-rails/blob/main/week2.md#orphan-objects
>
> If you completed exercises 9-11, that is, implemented the beer clubs, make sure that destroying an user destroys also their beer club memberships.


## More adjustments

Among the user editing actions you can also change the <code>username</code>. This does not make much sense, so remove the option.

Creating a new user and editing it make use of the same form, which is defined in the file views/users/_form.html.erb. In Rails, forms generated by scaffolding are also partials and attached to other templates with the <code>render</code> call.

The view template for editing users is below:

```erb
<h1>Editing user</h1>

<%= render 'form' %>

<%= link_to "Show this user", @user %> |
<%= link_to "Back to users", users_path %>
```

first it renders the elements in the _form template, and then a couple of links. The form code is below:

```erb
<%= form_with(model: user) do |form| %>
  <% if user.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(user.errors.count, "error") %> prohibited this user from being saved:</h2>

      <ul>
        <% user.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div>
    <%= form.label :username, style: "display: block" %>
    <%= form.text_field :username %>
  </div>

  <div>
    <%= form.label :password, style: "display: block"%>
    <%= form.password_field :password %>
  </div>

  <div>
    <%= form.label :password_confirmation, style: "display: block"%>
    <%= form.password_field :password_confirmation %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>
```

This means we want to delete the following chunk of code from the form

```erb
<div>
  <%= form.label :username, style: "display: block" %>
  <%= form.text_field :username %>
</div>
```

_if_ we are editing the user information – that is, if the user object has been already created previously.


With the method <code>new_record?</code>, you can request from the object <code>@user</code> whether it has been already stored in the database. In this way, you will show the <code>username</code> field in the form only when it's being used for creating a new user.

```erb
<% if @user.new_record? %>
  <div>
    <%= form.label :username, style: "display: block" %>
    <%= form.text_field :username %>
  </div>
<% end %>
```

Your form will do now, but it is still possible to change the user name by sending an HTTP POST request with a new username straight to the server.

Implement another verification in the <code>update</code> method of the User controller, to prevent changing the user name:

```ruby
def update
  respond_to do |format|
    if user_params[:username].nil? and @user == current_user and @user.update(user_params)
      format.html { redirect_to @user, notice: 'User was successfully updated.' }
      format.json { head :no_content }
    else
      format.html { render action: 'edit' }
      format.json { render json: @user.errors, status: :unprocessable_entity }
    end
  end
end
```

The form to change the user information will look like below, after all the changes you've implemented:

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-7.png)

> ## Exercise 16
>
> The only information of users are their password now. Change the form used in modifying the user information so that it will look like the picture below. Notice, that the new user signup has to look like before.

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-8.png)

## Application to Internet

To end your week, it is time to deploy again your application to either Heroku or Fly.io. Deployment to Fly.io might go without problems as Fly.io automatically executes any database migrations defined in the application. Not so with Heroku.

## Problems with heroku

If the program updated version is deployed in heroku, you will run into problems again. The page with all ratings, the one with all users, and the signup link cause a well-known error:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w2-12.png)

As we saw [last week](https://github.com/mluukkai/webdevelopment-rails/blob/main/week2.md#problems-with-heroku) you will have to find the reason from heroku logs.

The page with all users causes the following error:

    ActionView::Template::Error (PG::UndefinedTable: ERROR:  relation "users" does not exist

so the *users* database table does not exist because the application recent migrations haven't been executed in heroku. The issue will be solved by executing the migrations:

    heroku run rails db:migrate

The signup page will also work after executing the migrations.

The issue with the rating page will not be solved with the help of migrations, and you will have to look into the logs to find a solution:

```ruby
2022-08-24T16:28:33.610096+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4] ActionView::Template::Error (undefined method `name' for nil:NilClass):
2022-08-24T16:28:33.610221+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]     2:
2022-08-24T16:28:33.610225+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]     3: <ul>
2022-08-24T16:28:33.610227+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]     4:  <% @ratings.each do |rating| %>
2022-08-24T16:28:33.610229+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]     5:    <li> <%= rating %> <%= link_to rating.user.username, rating.user %></li>
2022-08-24T16:28:33.610231+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]     6:  <% end %>
2022-08-24T16:28:33.610232+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]     7: </ul>
2022-08-24T16:28:33.610234+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]     8:
2022-08-24T16:28:33.610239+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4]
2022-08-24T16:28:33.610241+00:00 app[web.1]: [2fb11437-8b3c-4ec2-a65c-5f725a7e65b4] app/models/rating.rb:10:in `to_s'
```

The reason is the old one – the view code tries to call the <code>username</code> method of a nil object. It must be beacuse of the parameter in the <code>link_to</code> method

```ruby
    rating.user.username
```

the system contains ratings which don't belong to any user object.

Even though the database migration has been executed, a part of the data are still conform to the old database scheme. When it came to the database migration, it would have been wise to write a code to check that also the system data are brought to the form expected by the code, after the migration, meaning that each existing rating should belong to a user or otherwise the rating should have been removed.

Create a user in the database and use heroku console make so the first user created becomes the user of all existing ratings.

```ruby
> u = User.first
> Rating.all.each{ |r| u.ratings << r }
```

Now your application will work.

Let me repeat the conclusion of "Problems in heroku" from last week, to end this week too.

<quote>
Most commonly, the problems we have in production depend on the inconsistent state that some objects have got because of our changes in the database scheme. For instance, they may be belonging to objects which do not exist or the references might be missing. **It is a good practice to deploy the application in the production mode as often as possible**, in this way, you will know that the potential problems are caused by the changes you have just done and fixing them will be easier.
</quote>

## Rubocop

Remember to use Rubocop to check that your code still follows the configured style rules.

If you are using Visual Studio Code you can install the [ruby-rubocop](https://marketplace.visualstudio.com/items?itemName=misogi.ruby-rubocop) plugin. The editor will then notify you immediately of any styling errors:

![pic](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2022/main/images/ratebeer-w3-10.png)

## Submitting the exercises


Commit all your changes and push the code to Github. Deploy to the newest version to Heroku or Fly.io, too.

Mark the exercises you have done at https://studies.cs.helsinki.fi/stats/courses/rails2022.
