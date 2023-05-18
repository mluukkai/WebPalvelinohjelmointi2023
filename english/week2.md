You will continue to develop your application from the point you arrived at the end of week 1. The material that follows comes with the assumption that you have done all the exercises of the previous week. In case you have not done all of them, you can take the sample answer of the previous week. If you already got most of the previous week exercises done, it might be easier if you complement your own answer with the help of the material.

## Sensible editor

Hopefully you are already using a sensible editor at this point, that is, something else than nano, gedit or notepad. Recommendable editors include RubyMine and Visual Studio Code. See [here](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/wadror.md#editoriide) for more.

Nowadays Visual Studio Code is very popular. If you use [VSC](https://code.visualstudio.com/), it is very much recommended to install the [Ruby plugin](https://code.visualstudio.com/docs/languages/overview).

In the end, when choosing an editor, the most important aspect is that is pleasant to use.
## Application layout

You want to put a navigation bar in your page like modern Web sites, placing a link to the lists with beers and breweries at the top of _all_ pages.

You can generate a navigation bar with the help of the method <code>link_to</code> and path helpers by adding the following links to each page:

```erb
<%= link_to 'breweries', breweries_path %>
<%= link_to 'beers', beers_path %>
```

If you had sharp eyes you might have noticed last week already that view templates do not contain all the HTML code of the pages. For instance, the view template for one beer, /app/views/beers/show.html.erb, looks like below:

```erb
<p style="color: green"><%= notice %></p>

<%= render @beer %>

<div>
  <%= link_to "Edit this beer", edit_beer_path(@beer) %> |
  <%= link_to "Back to beers", beers_path %>

  <%= button_to "Destroy this beer", @beer, method: :delete %>
</div>
```

If you look at the HTML code of the page of one beer using the _view source code_ option, you will see, that the page has much more HTML code than it is defined in the template (a part of the head contents has been removed):
```html
<!DOCTYPE html>
<html>
<head>
  <title>Ratebeer</title>
  <link data-turbolinks-track="true" href="/assets/application.css?body=1" media="all" rel="stylesheet" />
  <script data-turbolinks-track="true" src="/assets/jquery.js?body=1"></script>
  <meta content="authenticity_token" name="csrf-param" />
  <meta content="hZaC8o95xUbekA3PTsVZ+JmkVj9CCn5a4Kw8tF96WOU=" name="csrf-token" />
</head>
<body>

<p id="notice"></p>

<p>
  <strong>Name:</strong>
  Iso 3
</p>

<p>
  <strong>Style:</strong>
  Lager
</p>

<p>
  <strong>Brewery:</strong>
  1
</p>

<a href="/beers/1/edit">Edit</a> |
<a href="/beers">Back</a>


</body>
</html>
```

The page contains the type definition of the document, the head element which defines the style files and Javascript files to use, and the body element which defines the page contents (see more at http://www.w3.org/community/webed/wiki/HTML/Training).

The view template of the beer page contains only the HTML code which comes with the body element.

It's typical that all the pages of the application are the same except for the contents of the body element. In Rails, we can define the parts in common to all pages in the application _layout_, the file app/views/layouts/application.html.erb. The file contents will be the following by default:

```erb
<!DOCTYPE html>
<html>
  <head>
    <title>Ratebeer</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
  </head>

  <body>
    <%= yield %>
  </body>
</html>
```

The auxiliary methods inside the Head element define the style and javascript files which are used by the application. The auxiliary method <code>csrf_meta_tags</code> adds to the file the logic to eliminate CSRF attacs (see the [link](http://stackoverflow.com/questions/9996665/rails-how-does-csrf-meta-tag-work) for more information). As you may have guessed, the <code>yield</code> command inside the body element helps to render the contents defined by the view template of each page.

We can display a navigation bar in all pages by modifying the body element of our application layout in the following way:

```erb
<body>
  <div class="navibar">
    <%= link_to 'breweries', breweries_path %>
    <%= link_to 'beers', beers_path %>
  </div>

  <%= yield %>

</body>
```

The navigation bar has been set up in the div element which contains the _navibar_ class. We can modify its layout with the help of our CSS files.

Add the following to the file app/assets/stylesheets/application.css:

```css
.navibar {
  padding: 10px;
  background: #EFEFEF;
}
```

When you reload the page, you'll notice your application will almost look professional.

## routes.rb

The Routing component on Rails
(see http://api.rubyonrails.org/classes/ActionDispatch/Routing.html, http://guides.rubyonrails.org/routing.html) is in charge of handling the HTTP requests of the application and of routing them to the appropriate method controller.

The file <code>config/routes.rb</code> contains the information of how to route the requests to the various URLs. At this point, the file contents look like this:

```ruby
Rails.application.routes.draw do
  resources :beers
  resources :breweries
end
```

Later on, we will get to know the routes which are added by the method <code>resources</code>.

Let us get started by  setting the webpage containing a list of the breweries as the default page of the application. This will happen by adding the following line to the routes file

```ruby
root 'breweries#index'
```

The address http://localhost:3000/ will now lead to a page with all breweries.

What we wrote above is but the more classier way to say:

```ruby
get '/', to: 'breweries#index'
```

this means that when an HTTP GET request arrives to the path '/', it is routed and handled by the <code>index</code> method of the <code>BreweriesController</code>.

If we read the documentation, we should pay attention that a controller's methods are often called _actions_, in Rails. In any case, we have decided to use the name controller method in the course.

Similarly, you could also add the following line to routes.rb
  
```ruby
get 'kaikki_bisset', to: 'beers#index'
```

(kaikki_bisset is Finnish for all beers)

In such case, the GET requests to the URL http://localhost:3000/kaikki_bisset would lead to the page of all beers. Try that it works.

An interesting thing of the routes.rb file is that, even though it looks like a configuration file of pure text, all the contents are written in Ruby. The file lines are method calls. For instance the line

```ruby
get 'kaikki_bisset', to: 'beers#index'
```

calls the get method with parameters that are the string '/kaikki_bisset' and the hash <code>to: 'beers#index'</code>. This uses a newer syntax for hash expressions. If we use the old syntax, the part of the hash which defines the routing would be written <code>:to => 'beers#index'</code>, and the line of routes.rb would be:

```ruby
get 'kaikki_bisset', :to => 'beers#index'
```

we could also use brackets in the method call, and define the hash using curly brackets. The following might look clumsy but it is correct to define the route:

```ruby
get( 'kaikki_bisset', { :to => 'beers#index' } )
```

The elastic syntax (together with other characteristic features of the language) allows for a form which aims at the fluency of natural languages while configuring and programming applications. The style is well known in English with the name _Internal DSL_, see http://martinfowler.com/bliki/InternalDslStyle.html.

## Rating beers

Let us add the possibility to rate beers, that is to say, to give them a grade from 0 to 50. We will not use the generator that we have discovered last week (<code>rails generate scaffold...</code>), but we are going to create one ourselves, instead.

We want that all the rating are available at the address http://localhost:3000/ratings. Let us try to see what happens in our browser when we try to reach the URL.

The result in the error expression <code>No route matches [GET] "/ratings"</code> which tells us that the HTTP GET request which was done to the address was not matched by any defined route.

Let us add the route by creating the following line in the routes file:

```ruby
get 'ratings', to: 'ratings#index'
```

In Rails conventions, this defines that the index method of the RaitingsController class will be in charge of the ratings page.

Attention: if you wrote <code>match 'ratings' => 'ratings#index'</code>, you would get almost the same result. As it is typical in Rails, there are different ways to define the same thing in routes.rb, too.

Check out the page again in you browser. 

The error exception has changed to a new form, <code>uninitialized constant RatingsController</code>. This means that when the GET request arrives to the ratings address, the defined route tries to lead it so that it would be handled by the <code>index</code> method of the controller which is defined in the class <code>RatingsController</code>.

Let us define a controller in the file /app/controllers/ratings_controller.rb.

```ruby
class RatingsController < ApplicationController
  def index
  end
end
```

Pay attention to the name conventions and file location – Rails looks for the controller in the folder /app/controllers. If you place the controller somewhere else, Rails will not find it.

Try out the page with the browser once more.

You will receive the following error exception

```
RatingsController#index is missing a template for request formats: text/html
```

This happens beacause Rails tries to render the default view template which corresponds to the controller method and that should be located in /app/views/ratings/index.html.erb. Such file is not found, however.

Let us create the file /app/views/ratings/index.html.erb with the following contents (you will also need to create the directory _/app/views/ratings_):

```erb
<h2>List of ratings</h2>

<p>To be completed...</p>
```

and now the page works!

Note again Rails conventions and the file location, which is defined carefully. Because it is a view template which is called by the RatingsController, the view template is placed in the directory /views/ratings.

Another reminder from [last week](https://github.com/mluukkai/webdevelopment-rails/blob/main/week1.md#the-connection-between-controller-and-views): the <code>index</code> controll method renders the index view (which is located in the appropriate directory) at the end of the execution by default. The code

```ruby
class RatingsController < ApplicationController
  def index
  end
end
```

does the same thing as:

```ruby
class RatingsController < ApplicationController
  def index
    render :index    # renders the view template /app/views/ratings/index.html
  end
end
```

In any case, we do not explicitely call the render method if the default file is rendered – that is to say the template with the same name of the controller method.

## Creating the model by hand: done, almost...

One beer has many ratings, which means that the object model should be updated to look as follows:

![One beer has many ratings](http://yuml.me/5c8a236c.png)

We need a database table and the corresponding model object.

It is good to use migrations __always__ when you want to make changes on Rails, for instance when adding a table. The migrations are files which have to be placed in the directory db/migrate, and where we note the Ruby operations which modify the database. We will better familiarize ourselves with migrations later on. Now, we use Rails' ready-made _model generator_ to create our model. The generator not only creates a model object but it also generates automatically the migration we need.

Ratings have an integer <code>score</code> and a foreign key, which links to the rated beer. According to Rails conventions, the foreign key name has to be <code>beer_id</code>.

You can create the model and the migration which generates the database by giving the following command in the command line:

    rails g model Rating score:integer beer_id:integer

and the database table by executing the following migration in the command line

    rails db:migrate

Differently than the _scaffold_ generator which we used last week, the model generator does not create a controller or view templates.

**A reminder from last week:** the files generated by Rails generators (scaffold, model, ...) can be deleted with the command *destroy*:

    rails destroy model Rating

If you have already executed the migration and you notice that the code created by the generator has to be destroyed, it is **extremely important** that you first cancel the migration with the command

    rails db:rollback

In order to establish the connections at object level too (check [last week material](https://github.com/mluukkai/webdevelopment-rails/blob/main/week1.md#beers-and-the-one-to-many-connection), the classes have to be updates in the following way

```ruby
class Beer < ApplicationRecord
  belongs_to :brewery
  has_many :ratings
end

class Rating < ApplicationRecord
  belongs_to :beer
end
```
```

Each beer has many ratings and a rating belongs to one sole beer always.

Start the Rails console running the command <code>rails c</code> from the command line. Notice that if your console was open already, you can start to use the new code in the console by running <code>reload!</code>. Create a few ratings:

```ruby
> b = Beer.first
> b.ratings.create score: 10
> b.ratings.create score: 21
> b.ratings.create score: 17
```

The ratings are added to the first beer which is found in the database. Notice the way it was created; you could have run the same thing with the more complex

```ruby
b.ratings << Rating.create(score:15)
```


## Missing foreign key

Let's try to create a beer without a brewery: 

```ruby
irb(main)> b = Beer.create name:"anonymous", style: "watery"
=> #<Beer:0x00007f4444abc8b0 id: nil, name: "anonymous", style: "watery", brewery_id: nil, created_at: nil, updated_at: nil>
irb(main)>
```

_id_ and the time stamps do not get any values. It looks like the beer wasn't saved into the databse at all.

If we use beer method _errors_, we find the reason for the failed save.

```ruby
irb(main)> b.errors
=> #<ActiveModel::Errors [#<ActiveModel::Error attribute=brewery, type=blank, options={:message=>:required}>]>
```

So the beer won't be saved to the database without information about its brewery. We can fix this by defining the brewery and calling the method _save_ for the beer:

```ruby
> b.brewery = Brewery.find_by(name: 'Koff')
> b.save
   (0.1ms)  begin transaction
  Beer Create (1.9ms)  INSERT INTO "beers" ("name", "style", "brewery_id", "created_at", "updated_at") VALUES (?, ?, ?, ?, ?)  [["name", "anonymous"], ["style", "watery"], ["brewery_id", 1], ["created_at", "2022-09-11 18:21:40.830949"], ["updated_at", "2022-09-11 18:21:40.830949"]]
   (0.8ms)  commit transaction
```

The reason for failing to save the beer was that by default Rails demands that if an object refers to another object via a foreign key and _belongs_to_ is used in the code to form the association (like we do in the case of beers)

```ruby
class Beer < ApplicationRecord
  belongs_to :brewery

  # ...
end
```

the foreign key [cannot be uninitialized](https://blog.bigbinary.com/2016/02/15/rails-5-makes-belong-to-association-required-by-default.html).

>## Exercise 1
>
>The console use routine is extremely important for Rails developers. Do the following things by hand:
>
>create a new brewery "BrewDog", the founding date is 2007<br/>
>add two beers to the brewery
>* Punk IPA (style IPA)
>* Nanny State (style lowalcohol)
>add a couple of ratings to both beers
>
>Go through last week [material](https://github.com/mluukkai/webdevelopment-rails/blob/main/week1.md) in case you need and check the parts about console use.
>
>Return this exercise by adding the directory _exercises_ to your application. The directory has to contain the file exercise1, with the copy-pasted console session

Our database contains ratings now, and we want to make sure they appear on a page with all ratings.

> ## Exercise 2
>
> Add all the ratings on a ratings page. You can use the brewery controller <code>index</code> method and the corresponding template as model. When you make the rating list, use the following style, for instance
>
> ```erb
> <ul>
>  <% @ratings.each do |rating| %>
>    <li> <%= rating %> </li>
>  <% end %>
> </ul>
> ```
>
> Also add the information about the total amount of ratings to the page.

At this point, the page should look more or less like this:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/images/ratebeer-w2-1.png)

The rating is rendered in a quite unpleasant way. This depends on the fact that the li element contains only the object name, and because we haven't defined the method <code>to_s</code> which turns the ratings to strings, the method in use is the default <code>to_s</code> method which was inherited by all classes from the parent class Object.

Soon, we will define a _partial_ file for the ratings which breaks down the code and lets us create an easily readable display for our ratings. Before doing it, we will first see a couple of things about object method definitions.

## A couple of clarifications about Rails model objects

Spend a second to inspect the class <code>Brewery</code>:

```ruby
class Brewery < ApplicationRecord
  has_many :beers
end
```
```

The brewery has a <code>name</code> and a founding <code>year</code>. We can reach them by hand from the console:


```ruby
> b = Brewery.first
> b.name
=> "Koff"
> b.year
=> 1897
>
```

Technically, <code>b.year</code> is a method call. For each column which is defined by the schema of the database table, Rails creates a field into the model object. It creates an attribute and the method to read and change the attribute value. These automatically generated methods look more or less as follows:

```ruby
class Brewery < ApplicationRecord
  # ..

  def year
    read_attribute(:year)
  end

  def year=(value)
    write_attribute(:year, value)
  end
end
```

The methods helps us to read and change the value of the object attribute. The method which changes the value does not yet implement the change in the database, which happens only when we call the <code>save</code> method, they are an example of 'getters and setters' which are generated automatically.

Outsite the object, we retrieve object attributes by using 'dot notation':

    b.year

What about inside the object? Create a method to demonstrate how attributes are handled inside brewery:

```ruby
class Brewery < ApplicationRecord
  has_many :beers

  def print_report
    puts name
    puts "established at year #{year}"
    puts "number of beers #{beers.count}"
  end
end
```

As it is also in Java for instance, methods can be called with their name from within the object (<code>beers</code> is a method, too!)

You find an example on how to use the method:

```ruby
> b = Brewery.first
> b.print_report
Koff
established at year 1897
number of beers 2
```

You could have called the methods from inside the object by using Ruby's 'this,' that is, the object <code>self</code> reference:

```ruby
  def print_report
    puts self.name
    puts "established at year #{self.year}"
    puts "number of beers #{self.beers.count}"
  end
```

Next, create a method to 'restart' the brewery, in such case the establishment year changes to year 2022:

```ruby
def restart
  year = 2022
  puts "changed year to #{year}"
end
```

Try it out:

```ruby
> b = Brewery.first
> b.year
=> 1897
> b.restart
changed year to 2022
> b.year
=> 1897
>
```

as you notice, the year change does not work as we expected! As far as the method <code>restart</code> is concerned, there is no method call with <code>year = 2022</code>

    def year=(value)

There is no method call which could assign a new value to the attribute. Instead, a local variable called <code>year</code> is created in the method and it is given value 2022.

If you want to make the assignment work, you have to call the method through a <code>self</code> reference:

```ruby
def restart
  self.year = 2022
  puts "changed year to #{year}"
end
```
The functionality will now fulfill your expectations:

```ruby
> b = Brewery.first
> b.year
=> 1897
> b.restart
changed year to 2022
> b.year
=> 2022
>
```

**Attention:** In Ruby, instance variable are defined with an <code>@</code> character at the beginning. Instance variables _are not_ the same thing as the object attributes which are saved in the database thanks to ActiveRecord. The following code would not work as you could expect:

```ruby
  def restart
    @year = 2022
    puts "changed year to #{@year}"
  end
```

The <code>code</code> inside the brewery is an attribute which is saved in the database by ActiveRecord, whereas <code>@year</code> is an instance variable of the object. Instance variables are not much used in Rails. Instance variables are used on Rails mostly to transmit information from the controlers to the views.

> ## Exercise 3
>
> Change the ratings page so that each rating is displayed better as a string, eg. "karhu 35", which contains the name of the rated beer, followed by its rating value.
>
> The following link might be useful to form strings: https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/rubyn_perusteita.md#merkkijonot
>
>You can do everything directly in file views/partials/index.html.erb or optionally you create a partial template for rating. This handles diplaying a single rating.
>
>You can use eg. \_beer.html.erb and the responding index.html.erb files as a guide. Remember how partials files are named!

After you have done the exercise, the rating pages should look more or less like this:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/images/ratebeer-w2-2.png)

Attention: when you create new code in your application, it is a good practice to make trials by hand in your console. Below, we try to use the default method <code>to_s</code> to return the value of the rating:

```ruby
> r = Rating.last
> r.to_s
=> "#<Rating:0x007f8054b1cb10>"
>
```

Now we will define a <code>to_s</code> method for the rating:

```ruby
class Rating < ApplicationRecord
  belongs_to :beer

  def to_s
    "written rating"
  end
end
```

and we try to run it again from the console:

```ruby
> r.to_s
=> "#<Rating:0x007f8054b1cb10>"
```

It looks like the change has not been implemented anyway, what is wrong?

In order to make sure the change of the code is implemented, you have to reload the new code in the console using the command <code>reload!</code>, then you should retrieve the object from the database again and use this one. 


```ruby
> reload!
Reloading...
=> true
> r.to_s
=> "#<Rating:0x007f8054b1cb10>"
> r = Rating.last
> r.to_s
=> "written rating"
>
```

As you see above, reloading the code is not enough in itself, because the object stored in the variable <code>r</code> contains the old code still.

> ## Exercise 4
>
> Create the method <code>avarage_rating</code> to the class <code>Beer</code> to find the avarage value of the beer ratings. Add the avarage value to the beer page __if__ the beer has ratings
>
> The contents of the view template can be made conditional with something like this
>
> ```erb
> <% if beer.ratings.empty? %>
>  beer has not yet been rated!
> <% else %>
>  beer has some ratings
> <% end %>
> ```
>Remember to return the rating value as a floating point number. For this you can use the  `to_f` method.

The beer page should look more or less like the picture below, after you have done the exercise (notice that after last week, you could be showing the brewery ID instead of the brewery name on the page. If so, change your view so that it corresponds to the picture):

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/images/ratebeer-w2-3.png)

> ## Exercise 5
>
> The module enumerable (see https://ruby-doc.org/core-3.1.2/Enumerable.html) contains a large extent of auxiliary methods to parse object collections.
>
> Object collection classes can include the modul enumerable functionality by inheriting it.
>
> Get acquainted with the methods <code>map</code> and <code>reduce</code> (see for instance [reduce](https://ruby-doc.org/core-3.1.2/Enumerable.html#reduce), [map](https://ruby-doc.org/core-3.1.2/Enumerable.html#map) and google for further information) and change (in case you need) the method which calculates the rating avarage value so that it makes use of reduce or map and sum.
>
> Calculating the avarage value is easier in this case if you use ActiveRecord methods, see http://api.rubyonrails.org/classes/ActiveRecord/Calculations.html

Using the console, add a rating to one previously unrated beer. The beer page should now look like the one below:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/images/ratebeer-w2-4.png)

The page has one small, but annoying grammar mistake:

    beer has 1 ratings

> ## Exercise 6
>
> Get acquainted with the auxiliary method <code>pluralize</code> which is ready made in Rails (http://apidock.com/rails/ActionView/Helpers/TextHelper/pluralize) and use it to make sure the page beer page is grammatically correct (in case there is one rating, the method should print 'beer has 1 rating.')

## Form and post

Make it possible that ratings are created by hand in your application from the www-page.

According to Rails convensions, the form to create a Rating object has to be found at the address ratings/new, and the form can be accessed thanks to the <code>new</code> method of the ratings controller.

Create the appropriate route in routes.rb

```ruby
get 'ratings/new', to:'ratings#new'
```

We add the <code>new</code> method to the ratings controller (whose actual name is RatingsController). The method makes sure the form is rendered. It looks simple:

```ruby
  def new
    @rating = Rating.new
  end
```

The method only creates a new Rating object and passes it as <code>@rating</code> to the new.html.erb view template which is rendered by default through the method. The object is created with a <code>new</code> command, so it is not saved into the database.

Create now the next view, the file /app/views/ratings/new.html.erb:

```erb
<h2>Create new rating</h2>

<%= form_for(@rating) do |f| %>
  beer id: <%= f.number_field :beer_id %>
  score: <%= f.number_field :score %>
  <%= f.submit %>
<% end %>
```

Go now to the page which contains the form, at the address http://localhost:3000/ratings/new

The HTML code which is rendered with the help of the view looks more or less like the one below (you find the code by going to the page and choosing _view page source_ from the browser):

```erb
<form action="/ratings" method="post">
  beer id: <input name="rating[beer_id]" type="number" />
  score: <input name="rating[score]" type="number" />
  <input name="commit" type="submit" value="Create Rating" />
</form>
```


As you can see, a normal HTML form is generated (you find more details at http://www.w3.org/community/webed/wiki/HTML/Training#Forms).

The address where the form is sent to is /ratings and the HTTP method used is not GET but POST. There are two fields which are numbers, and their values are sent to users with a POST call as values of the variables <code>rating[beer_id]</code> and <code>rating[score]</code> arvoina.

Rails method <code>form_for</code> creates a form which works correctly and sends the data to the right address automatically. The form has input fields for all the attributes on the object in parameter.

You can read more about how to create forms with the <code>form_for</code> method at the address
 http://guides.rubyonrails.org/form_helpers.html#dealing-with-model-objects

If we try to create ratings,nothing seems to happen. The browser's developer console however shows us that the browser has done a POST request to http://localhost:3000/ratings but the server has replied with 404.

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/images/w2-post.png)


This means we have to create a route in the file config/routes.rb for sending the form:

```ruby
post 'ratings', to: 'ratings#create'
```

According to Rails conventions, the method which is in charge of creating a new object is called <code>create</code>. Make its foundation:

```ruby
  def create
    raise
  end
```

At this point, the method does not make anything else than throwing an exception (the method call <code>raise</code>).

Try to send information with the form, now. The exception thrown in the controller method causes an error message. Rails adds ample diagnostics in the error page, for instance the hash which is contained by the HTTP request parameters. The diagnostics looks like the one below:

```ruby
{"authenticity_token"=>"[FILTERED]",
 "rating"=>{"beer_id"=>"1", "score"=>"2"},
 "commit"=>"Create Rating"}
```

The information sent with the form is contained within the hash.

The hash containing the parameters is saved in the controller variable <code>params</code>.

The new pieces of information about the ratings are the values of the hash key <code>:rating</code>, and we can retrieve them with the command <code>params[:rating]</code>. This is an hash itself and its vaule is <code>{"beer_id"=>"1", "score"=>"2"}</code>. In other words, you can retrieve the rating with the command <code>params[:rating][:score]</code>.

## debugger

Inspect the thing with the controller by hand, making use of Rails debugger.

Rails has already configured a [debugger](https://github.com/ruby/debug) for your use. However, Rails' default debugger doesn't perform well in some situations so let's install an alternative, [pry-byebug](https://github.com/deivid-rodriguez/pry-byebug), by adding the following to Gemfile

```ruby
group :development, :test do
  gem 'pry-byebug'
end
```

and execute the command <code>bundle install</code> from the command line and restart your Rails application.

Add the command <code>binding.pry</code> to the beginning of the controller, at the point where you want to inspect the code.

```ruby
def create
  binding.pry
end
```

When you create a new rating using the form, the application stops at the <code>binding.pry</code>> command. An interactive console view will open in the terminal where Rails is running:

```ruby
Started POST "/ratings" for ::1 at 2022-07-20 14:02:51 +0300
Processing by RatingsController#create as TURBO_STREAM
  Parameters: {"authenticity_token"=>"[FILTERED]", "rating"=>{"beer_id"=>"12", "score"=>"12"}, "commit"=>"Create Rating"}
[7, 15] in ~/ratebeer/app/controllers/ratings_controller.rb
     7|     def new
     8|       @rating = Rating.new
     9|     end
    10|
    11|     def create
=>  12|       binding.pry
    13|     end
    14|
    15| end
=>#0    RatingsController#create at ~/ratebeer/app/controllers/ratings_controller.rb:12
  #1    ActionController::BasicImplicitRender#send_action(method="create", args=[]) at ~/.rbenv/versions/3.1.2/lib/ruby/gems/3.1.0/gems/actionpack-7.0.3/lib/action_controller/metal/basic_implicit_render.rb:6
  # and 73 frames (use `bt' command for all frames)
(ruby)
```

The arrow tells you where the execution was paused. Analyse the contents of the <code>params</code> variable, now:

```ruby
(rdbg) params
#<ActionController::Parameters {"authenticity_token"=>"2pGKvP6I-RYAoEbZr6eJltrNZt_T0YlQvO4K7EOyMFrF1W_OzJoPTKd39LBQoMyG5u_ScQrLjztIcB8TyWpDTw", "rating"=>#<ActionController::Parameters {"beer_id"=>"12", "score"=>"12"} permitted: false>, "commit"=>"Create Rating", "controller"=>"ratings", "action"=>"create"} permitted: false>
(ruby) params[:rating][:beer_id]
"12"
(ruby) params[:rating][:score]
"12"
```

You can execute whatever code from the debugger console as if it was a Rails console, in case you need.

The most important commands of the debugger might be _step, next, continue_, and _help_. Step executes the next step in the code, along with the possible method calls. Next executes the whole next line. Continue lets the program execution continue in the normal way.

More info about the debugger https://guides.rubyonrails.org/debugging_rails_applications.html#debugging-with-the-debug-gem


## Saving a rating

Inside the controller, <code>params[:rating]</code> contains all the information which is needed to create a new rating. Also, because it is a hash like <code>{"beer_id"=>"1", "score"=>"30"}</code>, it can be given straight as parameter to the method <code>create</code>. It should be possible to create a rating by using the command:

```ruby
Rating.create params[:rating]  # which means the same as Rating.create beer_id:"1", score:"30"
```

So change your controller code to look like the following:

```ruby
  def create
    Rating.create params[:rating]
  end
```

Try to create a new rating now. Against all our best hopes, the action fails and we are thrown an error message

```
ActiveModel::ForbiddenAttributesError
```

What is it about?

If the command to create the rating had been

```ruby
Rating.create beer_id: params[:rating][:beer_id], score: params[:rating][:score]
```

which meens exactly the same as the form above because <code>params[:rating]</code> is __exactly the same__ hash as <code>beer_id:params[:rating][:beer_id], score:params[:rating][:score]</code>), we wouldn't have met any error message. Because of [information security issues](http://en.wikipedia.org/wiki/Mass_assignment_vulnerability) Rails does not allow "high-handed" mass assignments (that is to say, giving all parameters as hash) of a  <code>params</code> variable when the object is created.

Starting from Rails 4, we have to specify what contents of the hash <code>params</code> we can mass-assign when we create the objects. For this, the controller uses the methods <code>require</code> and <code>permit</code> of <code>params</code>.

The idea is that first we use require to retrieve the hash which contains the information of the object which has to be created from params:

```ruby
params.require(:rating)
```

after this, we use permit to specify the fields where value mass assignment can be allowed:
```ruby
params.require(:rating).permit(:score, :beer_id)
```
Our controller looks like the following:


```ruby
  def create
    Rating.create params.require(:rating).permit(:score, :beer_id)
  end
```

More information on how to handle form parameters in https://edgeguides.rubyonrails.org/action_controller_overview.html#strong-parameters.

Try now to create the rating. ATTENTION: when you use a form to create a rating, make sure that the beer ID input to the form corresponds to a beer ID which is available in the database!

Creating the rating works now, check it with the console or from the page of all ratings. At least on Chrome creating a rating causes a situation where the browser seems to stay on the same page but the page "freezes". The reason for this is revealed in the log message printed into application consol:

```
↳ app/controllers/ratings_controller.rb:12:in `create'
No template found for RatingsController#create, rendering head :no_content
Completed 204 No Content in 55ms (ActiveRecord: 16.4ms | Allocations: 12093)
```
So, because no view template has been configured for the create operation, the browser sends an empty response, that is, an answer that contains no HTML code. Chrome however seems to keep the previous page displayed when it receives an empty response.

## Rederecting

We could create the template now, but we decide that the user browser is __redirected__ to the page containing all ratings after a new rating is created. Change the controller code to look like the one below:

```ruby
def create
  Rating.create params.require(:rating).permit(:score, :beer_id)
  redirect_to ratings_path
end
```

<code>ratings_path</code> is a path auxiliary method which is provided by Rails and it means the same as "/ratings"

If you have created ratings where <code>beer_id</code> does not reflect an existing beer ID, you will now most likely be thrown an error message. You can distroy these ratings by hand from the Rails console in the following way:

```ruby
    Rating.last        # shows the rating which has been created last, check if its beer_id is incorrect
    Rating.last.delete # removes the rating which was created last
```

You can distroy the beerless ratings also with the following one-liner:

```ruby
Rating.all.select{ |r| r.beer.nil? }.each{ |r| r.delete }
```

Select creates a table which will contain the collection items we have gone through if the option in the code chunk is true. <code>r.beer.nil?</code> returns <code>true</code> if the object <code>r.beer</code> is <code>nil</code>.

The command above can also be written in a shorter form like

```ruby
Rating.all.select{ |r| r.beer.nil? }.each(&:delete)
```

What does the command <code>redirect_to ratings_path</code> exactly do when we use it in the controller? Usually the controller renders the appropriate view template, and the code retrieved is then returned to the browser, which renders the page on the screen.

When the browser is redirected, the server sends a response which is equipped with a statuscode 302, which does not contain any HTML at all. The response contains only an address to where browser will automatically make the HTTP GET request. The redirection remains unnoticed for the browser user.

Try what happens if you put something like <code>redirect_to "http://www.cs.helsinki.fi"</code> as final redirection when a new rating is created!

## redirect_to vs render

Technically, it would have been possible to avoid using redirect and render the page with all ratings straight from the controller that creates the new rating:

```ruby
def create
  Rating.create params.require(:rating).permit(:score, :beer_id)
  @ratings = Rating.all
  render :index
end
```

Even though the result will look exactly the same for the website visitors, the solution is not the best for a couple of reasons. First of all, you have to copy to the <code>create</code> method all the code contained in the <code>index</code> method, which is needed to form the view. (Now there is not much to copy, but the case is not always as simple) 

Another reason concerns the browser's behaviour. If our controller rendered the page, and the browser user refreshed the page after creating one beer, some of the older browsers could resend the form contents. This is because the previous action of the browser, that the refreshing then re-executes, is actually the HTTP POST request that took care of sending the form.
 
This problem does not exist with redirections: the page shown to visitors after the POST command is the page which is retrieved with the HTTP GET triggered by the redirection.

The rule of thumb is that you should *always* use redirection with controllers that handle HTTP POST methods for forms. This is true not only on Rails, but more generally in Web-programming, see http://en.wikipedia.org/wiki/Post/Redirect/Get, with the only exeption if the controller operation does not work for reasons like the information sent with the form is uncorrect.

Let us underline this important difference once again:

* when the controller method end with the command <code>render :something</code> (which often happens implicitely), your Rails application generates an HTML page, and the server sends it to the browser to be rendered
* when the controller finds the command  <code>redirect_to address</code>, the server sends a redirect request together with a statuscode 302 to the browser, requesting the browser to make the HTTP GET request to the address defined by the controller method. This happens automatically and the browser user will not notice the redirection

**Every** Web developer has to understand the part above!


## Path helper methods

Rails creates path methods (or path helpers) automatically to all the routes which are defined in routes.rb. Thanks to them, you will not need to hard-code the addresses of the various different pages.

For instance, the redirection address after creating a new rating could have been hard-coded like the example below, instead of using the auxiliary function <code>ratings_path</code>:


```ruby
def create
  Rating.create params.require(:rating).permit(:score, :beer_id)
  redirect_to 'ratings'
end
```

As usual, hard-coding does not make sense with addresses either.

The available paths which are generated automatically can be viewed from the command line, using the command <code>rails routes</code>

```ruby
mluukkai@melkki.~/ratebeer$ rails routes
      Prefix Verb   URI Pattern                   Controller#Action
       beers GET    /beers(.:format)              beers#index
             POST   /beers(.:format)              beers#create
    new_beer GET    /beers/new(.:format)          beers#new
   edit_beer GET    /beers/:id/edit(.:format)     beers#edit
        beer GET    /beers/:id(.:format)          beers#show
             PATCH  /beers/:id(.:format)          beers#update
             PUT    /beers/:id(.:format)          beers#update
             DELETE /beers/:id(.:format)          beers#destroy
   breweries GET    /breweries(.:format)          breweries#index
             POST   /breweries(.:format)          breweries#create
 new_brewery GET    /breweries/new(.:format)      breweries#new
edit_brewery GET    /breweries/:id/edit(.:format) breweries#edit
     brewery GET    /breweries/:id(.:format)      breweries#show
             PATCH  /breweries/:id(.:format)      breweries#update
             PUT    /breweries/:id(.:format)      breweries#update
             DELETE /breweries/:id(.:format)      breweries#destroy
        root GET    /                             breweries#index
     ratings GET    /ratings(.:format)            ratings#index
 ratings_new GET    /ratings/new(.:format)        ratings#new
             POST   /ratings(.:format)            ratings#create
```

For instance, the last 3 routes tell us the following things:
* the method call <code>ratings_path</code> generates a link to the address "ratings" which is directed to the <code>index</code> method of the ratings controller.
* the method call <code>ratings_new_path</code> generates a link to the address "ratings/new" which is directed to the <code>new</code> method of the ratings controller. This will render the rating form.
    * Attention: as you see if you compare the routes above, <code>ratings_new_path</code> is not the same as the creation path for new beers, for instance, we will correct the problem later
* The POST call to the address "ratings" is directed to the <code>create</code> method of the ratings controller

As we have already noticed,the information of the command <code>rails routes</code> arrives to the web page to render in error situations. The page even provides you with an interactive tool, which you can use to see how the application routes the input example path:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/images/ratebeer-w2-6.png)


> ## Exercise 7
>
> In the page with all ratings, add a link to create a new rating. Add a link to the list with all ratings in the navigation bar of your application.

## Beer selection from a list

Creating a new rating is not the nicest thing to do now, because the user has to know the beer ID. Let's change the rating so that the user can choose the beer he wants to evaluate out of a list.

If we want that the form to create a list, the controller in charge of displaying the form has to retrieve the list from the database and save it into a variable. Extend the controller in the following way:

```ruby
class RatingsController < ApplicationController
  def new
    @rating = Rating.new
    @beers = Beer.all
  end

  # ...
end
```

If you consult page http://guides.rubyonrails.org/form_helpers.html#making-select-boxes-with-ease and make a couple of trials, you will find out that the form to create a rating has to be modified in the following way:

```erb
<%= form_for(@rating) do |f| %>
  <%= f.select :beer_id, options_from_collection_for_select(@beers, :id, :name) %>
  score: <%= f.number_field :score %>

  <%= f.submit %>
<% end %>
```

This means that the value of <code>beer_id</code> of the form is generated with the _select_ element of the HTML form. You can select the options of this element using the view method <code>options_from_collection_for_select</code> from the list of beers contained by the <code>@beers</code> variable. This is done by setting the beer ID (the second parameter :id) as value and the beer name (third parameter :name) is shown to the form users.

The third parameter defines what individual options are shown on the form. In this case, the result of the methd _name_ for each beer. In Ruby references to method names are defined as symbols, that is, as strings starting with a colon.

**Attention:** you can test the view methods from the console too. The methods can be called through the <code>helper</code> object:

```ruby
> b = Beer.all
> helper.options_from_collection_for_select(b, :id, :name)
=> "<option value=\"1\">Iso 3</option>\n<option value=\"2\">Karhu</option>\n<option value=\"3\">Tuplahumala</option>\n<option value=\"4\">Huvila Pale Ale</option>\n<option value=\"5\">X Porter</option>\n<option value=\"6\">Hefeweizen</option>\n<option value=\"7\">Helles</option>\n<option value=\"8\">Lite</option>\n<option value=\"9\">IVB</option>\n<option value=\"10\">Extra Light Triple Brewed</option>\n<option value=\"13\">Punk IPA</option>\n<option value=\"14\">Nanny State</option>"
>
```

> ## Exercise 8
>
> Create the method <code>to_s</code> for your beers so that the text will show both the beer and the brewery name
>
> Modify the form to create ratings. Users should not see the value of the name field of the beers they will choose from; instead, they should find the string of text which is returned by the method <code>to_s</code> of the object.

> ## Exercise 9
>
> Do the same change to the form to create beers (in the file views/beers/_form.html.erb) and to the controller which takes care of showing the form (beers#new). The user should not have to define by hand the brewery of the beer they want to create, but they should be able to choose the brewery out of a list.
>
> Change the controller to create new beers (beers#create), so that the browser is redirected to the list with all beers after a new beer is created. The list address should be generated with a path helper. By default, the browser is redirected to the page of the new beer with the command <code>redirect_to @beer</code>: change this.
>
> The form created automatically by scaffolding contains code for reporting errors, we will get acquainted with this better later.

> ## Exercise 10
>
> By this time, the style of the beers created are given as strings. We will change our application later on so that the beer styles are also stored in the database.
>
> Before we do it, adopt a temporary solution. Change your application so that the style of the beer created can be chosen from a list, which is built based on the table supplied by the controller.  The code of the <code>new</code> method of the beer controller will be modified as follows:
>
>Controller
>
> ```ruby
> def new
>  @beer = Beer.new
>  @breweries = Brewery.all
>  @styles = ["Weizen", "Lager", "Pale ale", "IPA", "Porter", "Lowalcohol"]
> end
> ```
>
> The view will have to generate the selection options of the form based on the <code>@styles</code> table. Instead of using the method <code>options_from_collection_for_select</code> to generate the selection options, in this case you better use the method <code>options_for_select</code>, see
> http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-options_for_select

Strangely enough, after all these changes you will not be able to edit the beer information. The reason is that the view template which generates the form is the same view template which is used to create a new beer and to edit the beer information (app/views/beers/_form.html.erb). After the changes, the view requires that the variable <code>@breweries</code> contains the brewery list and the variable <code>@styles</code> contains the brewery styles. You access the page to modify beer information after executing the controller method <code>edit</code>, and we will have to modify the controller in the following way to fix the problem:

```ruby
  def edit
    @breweries = Brewery.all
    @styles = ["Weizen", "Lager", "Pale ale", "IPA", "Porter", "Lowalcohol"]
  end
```

It is very typical, that the controller methods <code>new</code> and <code>edit</code> contain much of the same code. It might make sense to extract the common code and make a new method out of it.

##  REST and routing

REST (representational state transfer) is an HTTP-protocol-based architecture model which is expecially used to implement web-based applications. The idea behind it is simple: the resources for editing and retrieval are defined by addresses, the request methods describe the operation for the resources, and the request body contains the data for the resources, in case they are needed.

Read now http://guides.rubyonrails.org/routing.html till point 2.5. Rails makes it easy to observe a structure such as REST. If you are interested in it, you can read more about REST from 
[here](https://en.wikipedia.org/wiki/Representational_state_transfer), for instance.

Change the rating paths to the file routes.rb so that we use the  ready-made <code>resources</code> definition:

```ruby
  # comment or remove the old definitions
  #get 'ratings', to: 'ratings#index'
  #get 'ratings/new', to: 'ratings#new'
  #post 'ratings', to: 'ratings#create'

  resources :ratings, only: [:index, :new, :create]
```

Because you don't need routes like **delete**, **edit** ja **update**, use the <code>:only</code> qualifier to choose only the routes you need. Take a look at the paths defined for the application by running the command <code>rails routes</code> from the command line (or from the web page with an erroneous URL):

```ruby
     ratings GET    /ratings(.:format)            ratings#index
             POST   /ratings(.:format)            ratings#create
  new_rating GET    /ratings/new(.:format)        ratings#new
```

The result is the same as the one before, but the name of auxiliary method <code>rating_new_path</code> follows now Rails conventions and it's called <code>new_rating_path</code>.

Change the old path method call which is used in the template app/views/ratings/index.erb.html with the new one.

## Removing a rating

Add the possibility to remove ratings. First, change routes.rb and add the appropriate route:

```ruby
resources :ratings, only: [:index, :new, :create, :destroy]
```

Then add a link in the ratings list to delete a rating:

```erb
<ul>
  <% @ratings.each do |rating| %>
    <li> <%= render rating %> <%= button_to 'delete', rating_path(rating.id), method: :delete %> </li>
  <% end %>
</ul>
```

Rails conventions imply that you delete objects using the HTTP DELETE method. If want to delete a rating where the ID is 5, clicking the delete button triggers a HTTP DELETE request to the address ratings/5.

As we have mentioned before, the parameter of <code>link_to</code> can be the object which is called instead of the call <code>rating_path(rating.id)</code>. The code above can also be shortened to:

```erb
<ul>
  <% @ratings.each do |rating| %>
    <li> <%= render rating %> <%= button_to 'delete', rating, method: :delete %> </li>
  <% end %>
</ul>
```

In order to make deletion work, you have to define the method <code>destroy</code> for the controller, which executes the deletion.

The URL to the method is ratings/[the ID of the object to delete]. According to Rails conventions, the method finds the ID of the object to delete thanks to the <code>params</code> object. The deletion happens when the object is retrieved from the database and its <code>delete</code> method is called:

```ruby
def destroy
  rating = Rating.find(params[:id])
  rating.delete
  redirect_to ratings_path
end
```

A redirection will be executed at the end, which leads back to the page with all ratings. Because of the redirection, the browser sends a new GET request to the application for the /ratings address, and the method ratings#index is executed again.

> ## Exercise 11
>
> Deleting ratings has an issue: if users are not careful they might eventually destroy ratings without meaning to do it.
>
> Make it so that users are required to confirm whether they really mean to destroy a rating. See [here](https://stackoverflow.com/a/70994323) for help

## Orphan objects

If you remove beers with ratings from your application, the ratings which belong to the deleted beer remain in the database. Most likely, this causes that the rating page renders erroneously.

> ## Exercise 12
>
> Remove some beers with ratings and go to the page with all ratings. You will see the error message <code>undefined method `name' for nil:NilClass</code>
>
> The error is cause by the fact that it tries to call <code>beer.name</code> from the method <code>to_s</code> of the rating object.
>
> Delete the orphan ratings by hand from the console. Try to think first of a command/some commands, which can help you to make a list of the orphan ratings. If you can't think of it yourself, you can find a ready-made answer for the exercise above in this page.

The ratings which belong to a beer can be deleted easily automatically. Along side the beer model code <code>has_many :ratings</code>, you should mark that ratings are dependent on beers, and that they should be destroyed if beers are distroyed:

```ruby
class Beer < ApplicationRecord
  belongs_to :brewery
  has_many :ratings, dependent: :destroy

  # ...
end
```

The orphan issue is solved now.


> ## Exercise 13
>
> Implement the same change for the breweries. When a brewery is deleted, its beer should also be deleted.
>
> Create a brewery with at least one beer with ratings. Delete the brewery and make sure, the beers of the brewery and their ratings are deleted.
>
>If you can't yet access individual breweries from the all breweries page, fix it now!

## Inderect object connection

Your application is created in a way so that ratings belong to beers and that beers belong to breweries. This means that a set of ratings belong to each brewery, indirectly. Rails provides you with a simple way to go from the breweries to the ratings directly:

```ruby
class Brewery < ApplicationRecord
  has_many :beers
  has_many :ratings, through: :beers
end
```

the connection is defined as a "database connection," but it is specified that the connection happens through other beers. Now the brewery has the method <code>ratings</code> which returns the ratings.

Implement this connection in your code and test the following thing from your console (not before doing <code>reload!</code>):

```ruby
> k = Brewery.find_by name:"Koff"
> k.ratings.count
 => 5
```

> ## Exercise 14
>
> Change the page which shows the information of the singular breweries so that it tells the amount of ratings of their beers as well as the avarage value. Add the method <code>average_rating</code> to the brewery to do this.
>
> Make sure the count for the number of ratings is grammatically correct, as it was in exercise 6. If there are no ratings, do not show the avarage.

The brewery page should be look more or less like the picture below after you have implemented the changes. 

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/images/ratebeer-w2-8.png)

You will see, that beer and brewery both a method called <code>avarage_rating</code> which also works in the same way. You can not leave the code like this.

## Moving common code to a module

We notice that beer and brewery both have an identically named method <code>average_rating</code> that also work identically. It is not acceptable to leave our code this way.
> ## Exercise 15
>
> Ruby provides you with a way to share methods between two classes with the help of modules, see https://github.com/mluukkai/WebPalvelinohjelmointi2022/blob/main/web/rubyn_perusteita.md#moduuli
>
> Modules have different uses – forming namespaces, for instance. However, now we are interested in the _mixin_ inheritance which can be implemented with modules.
>
> Get acquainted well enough with modules and refactor your code so that the method <codee>average_rating</code> is moved to a module which is contained by the classes <code>Beer</code> and <code>Brewery</code>.
>As we the now created module is only used by models, it is sensible to define it as a [concern](https://api.rubyonrails.org/classes/ActiveSupport/Concern.html) and place the file defining it in the directory _app/models/concerns_
>
> ```ruby
> module RatingAverage
>  extend ActiveSupport::Concern
>
>  # ...
> end
> ```
>
> - Attention: if the name of your module is <code>RatingAvarage</code>, exactly like in the example, because of Ruby naming conventions it has to be placed in the file <code>app/models/concerns/rating_average.rb</code>. In fact, even though classes names are CamelCase and start with capital letters, their files names follow the snake_case.rb style.

After you have done the exercise, the class Brewery should look more or less like below (assuming your module is called RatingAvarage):

```ruby
class Brewery < ApplicationRecord
  include RatingAverage

  has_many :beers
  has_many :ratings, through: :beers
end
```

and the method <code>avarage_rating</code> should still work like before:

```ruby
> b = Beer.first
> b.average_rating
=> #<BigDecimal:7fa4bbde7aa8,'0.17E2',9(45)>
> b = Brewery.first
> b.average_rating
=> #<BigDecimal:7fa4bfbf7410,'0.16E2',9(45)>
>
```

## A simple protection

At the end of the week, you now want to make your application in a way so that only the administrator is allowed to delete breweries. In week 3, you will use a more far-reaching way for autentification; implement a shortcut now, using [http basic authentication](http://en.wikipedia.org/wiki/Basic_access_authentication). See http://api.rubyonrails.org/classes/ActionController/HttpAuthentication/Basic.html

At the same time, you can get to know Rails controllers _filter methods_, see hhttp://guides.rubyonrails.org/action_controller_overview.html#filters. You can use them to implement easily a particular functionality, so that some particular methods are executed before (before_action) a defined controller method.

We'll define a filter method called <code>authenticate</code> for the brewery controller (which is defined as <code>private</code>). The filter has to be executed before each method of the brewery controller:

```ruby
class BreweriesController < ApplicationController
  before_action :set_brewery, only: %i[ show edit update destroy ]
  before_action :authenticate

  # ...

  private

  # ...

  def authenticate
    raise "perform authentication"
  end
end
```

The filter method throws an exeption. Therefore, you will be sent an exeption every time you go to whatever page of the breweries. Check this out in the browser.

Define the filter method execution so that it affects only when breweries are deleted:

```ruby
class BreweriesController < ApplicationController
  before_action :set_brewery, only: %i[ show edit update destroy ]
  before_action :authenticate, only: [:destroy]

  # ...

  private

  # ...

  def authenticate
    raise "perform authentication"
  end
end
```

Check again with your browser that other pages work, but deleting a brewery causes an error.

Implement http-basicauth authentication then (you can read more at http://blog.dcxn.com/2011/09/30/the-simplest-possible-authentication-in-rails-http-auth-basic/)

Hard code an "admin" user name with the password "secret":

```ruby
class BreweriesController < ApplicationController
  before_action :set_brewery, only: [:show, :edit, :update, :destroy]
  before_action :authenticate, only: [:destroy]

  # ...

  private

  # ...

  def authenticate
    authenticate_or_request_with_http_basic do |username, password|
      if username == "admin" and password == "secret"
        return true
      else
        raise "Wrong username or password" # username/password was wrong
      end
    end
  end
end
```

And your application will work as desired!

ATTENTION: After you have given the right user name and password once, the browser will not ask for the ID when you go to the page again. Open a new incognito window, if you want to test the sign-in process again!

The idea behind the method <code>authenticate_or_request_with_http_basic</code> is that the application requests the browser to send a username and password. These are later forwarded to the code block between <code>do</code> and <code>end</code> through the parameters <code>username</code> and <code>password</code>. If the code block is true, the page will be shown to the user.

Because the code block has the same value as the id condition, it can be simplified like this:

```ruby
def authenticate
  authenticate_or_request_with_http_basic do |username, password|
    raise "Wrong username or password" unless username == "admin" and password == "secret"
    
    return true
  end
end
```


HTTP Basic authentication is useful for the pages simple protection needs. In more complex situations however, you will have to look for other solutions to provide better data protection.

You'd better note, that the HTTP Basic authentication should not be used for other purposes than the HTTPS protocol, because user name and password are sent [Base64](http://en.wikipedia.org/wiki/Base64) encoded. This means that anyone could get to the headers and find out the password. A solution which is slightly better is the [Digest access authentication](http://en.wikipedia.org/wiki/Digest_access_authentication). In this way, users sign in with an identifier which is calculated with a one-way function and not with user name and password. Using Digest authentication is simple on Rails, see 
http://api.rubyonrails.org/classes/ActionController/HttpAuthentication/Digest.html

> ## Exercise 16
>
> Expand the solution so that the program would accept other user name-password pairs which are hard coded. The available IDs are hard coded in hashes which are defined in a method. The method has to workwith hashes of arbitrary length.
>
> ```ruby
>   def authenticate
>    admin_accounts = { "pekka" => "beer", "arto" => "foobar", "matti" => "ittam", "vilma" => "kangas" }
>
>    authenticate_or_request_with_http_basic do |username, password|
>      # do something here
>    end
>  end
> ```
> If you test the functionality, remember that you have to use an incognito browser if you want to sign in again after giving the right combination of user name and password already once.
>
>HINT: Coming up with the correct code might be easiest with the help of a debugger. Pause the application execution:
>
>```ruby
> authenticate_or_request_with_http_basic do |username, password|
> binding.pry
> end
>```
>and try what values variables _admin_accounts_, _username_ and _password_ contain ja form the right command.
>
> HINT 2: The code block should be evaluated either as true or untrue depending on whether the password is correct. The value doesn't however nececessarily have to be either _true_ or _false_ because Ruby interprets also other values as either true (truthy) or untrue (falsy). For example _nil_ is is interpreted as untrue/falsy. See more eg. at https://learn.co/lessons/truthiness-in-ruby-readme.

## Application to Internet

To end your week, it is time to deploy again your application to either Heroku or Fly.io. Deployment to Fly.io might go without problems as Fly.io automatically executes any database migrations defined in the application. Not so with Heroku.

## Problems with Heroku

If you try to naviage to the page with all ratings, you will find the old evil error message:

![picture](https://github.com/mluukkai/WebPalvelinohjelmointi2015/raw/master/images/ratebeer-w2-12.png)

Tracing down errors in the application running in the production mode is always a bit more difficult than when it is running in the developer mode, where Rails provides the application developer with various ways to find out the problems.

In the production mode, the problems' reason has to be found from the application logs. As we mentioned last week already, you can find heroku application logs with the command <code>heroku logs</code>.

Once more, you'll find the reason of your problems:

```ruby
> heroku logs
2020-08-20T13:34:55.379420+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5] Processing by RatingsController#index as HTML
2020-08-20T13:34:55.381470+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5]   Rendering ratings/index.html.erb within layouts/application
2020-08-20T13:34:55.384735+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5]   Rating Load (1.2ms)  SELECT "ratings".* FROM "ratings"
2020-08-20T13:34:55.385523+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5]   Rendered ratings/index.html.erb within layouts/application (3.9ms)
2020-08-20T13:34:55.385780+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5] Completed 500 Internal Server Error in 6ms (ActiveRecord: 1.2ms)
2020-08-20T13:34:55.386820+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5]
2020-08-20T13:34:55.386846+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5] ActionView::Template::Error (PG::UndefinedTable: ERROR:  relation "ratings" does not exist
2020-08-20T13:34:55.386848+00:00 app[web.1]: LINE 1: SELECT "ratings".* FROM "ratings"
2020-08-20T13:34:55.386849+00:00 app[web.1]: ^
2020-08-20T13:34:55.386850+00:00 app[web.1]: : SELECT "ratings".* FROM "ratings"):
2020-08-20T13:34:55.386958+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5]     1: <h2>List of ratings</h2>
2020-08-20T13:34:55.386960+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5]     2:
2020-08-20T13:34:55.386966+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5]     3: <ul>
2020-08-20T13:34:55.386968+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5]     4:  <% @ratings.each do |rating| %>
2020-08-20T13:34:55.386970+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5]     5:    <li> <%= rating %> <%= link_to 'delete', rating_path(rating.id), method: :delete, data: { confirm: 'Are you sure?' } %> </li>
2020-08-20T13:34:55.386972+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5]     6:  <% end %>
2020-08-20T13:34:55.386973+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5]     7: </ul>
2020-08-20T13:34:55.386977+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5]
2020-08-20T13:34:55.387016+00:00 app[web.1]: [fc20f584-1aef-4d93-8bff-c1a55e5cb6f5] app/views/ratings/index.html.erb:4:in `_app_views_ratings_index_html_erb___3457620989041177195_70202650345860'
```


The *ratings* database table does not exist. The problem is solved if you execute the migrations:

```bash
heroku run rails db:migrate
```

Next you will generate a situation where your database will be put in a slightly inconsistent condition.

Start heroku console with the command <code>heroku run console</code> and create a beer which does not belong to any brewery

```ruby
> b = Beer.new name:"crap beer", style:"lager"
> b.save(validate: false)
```

and create a beer which belongs to an inexistent brewery (meaning that the reference key brewery ID is erroneous):

```ruby
> b = Beer.new name:"shitty beer", style:"lager", brewery_id: 123
> b.save(validate: false)
```

If you go now to the page with all beers, you will find the same unfortunate message "We're sorry, but something went wrong.". Once again, you will find the error by looking into the logs:

```ruby
2022-08-20T10:56:01.307817+00:00 app[web.1]: F, [2022-08-20T10:56:01.307761 #4] FATAL -- : [22db4647-3122-419e-8e83-e2e99bfe3606]
2022-08-20T10:56:01.307818+00:00 app[web.1]: [22db4647-3122-419e-8e83-e2e99bfe3606] ActionView::Template::Error (undefined method name' for nil:NilClass):
2022-08-20T10:56:01.307818+00:00 app[web.1]: [22db4647-3122-419e-8e83-e2e99bfe3606]     10:
2022-08-20T10:56:01.307819+00:00 app[web.1]: [22db4647-3122-419e-8e83-e2e99bfe3606]     11:   <p>
2022-08-20T10:56:01.307819+00:00 app[web.1]: [22db4647-3122-419e-8e83-e2e99bfe3606]     12:     <strong>Brewery:</strong>
2022-08-20T10:56:01.307820+00:00 app[web.1]: [22db4647-3122-419e-8e83-e2e99bfe3606]     13:     <%= link_to beer.brewery.name, beer.brewery %>
2022-08-20T10:56:01.307820+00:00 app[web.1]: [22db4647-3122-419e-8e83-e2e99bfe3606]     14:   </p>
2022-08-20T10:56:01.307821+00:00 app[web.1]: [22db4647-3122-419e-8e83-e2e99bfe3606]     15:
2022-08-20T10:56:01.307821+00:00 app[web.1]: [22db4647-3122-419e-8e83-e2e99bfe3606]     16:   <% if beer.ratings.empty? %>
2022-08-20T10:56:01.307821+00:00 app[web.1]: [22db4647-3122-419e-8e83-e2e99bfe3606]
2022-08-20T10:56:01.307822+00:00 app[web.1]: [22db4647-3122-419e-8e83-e2e99bfe3606] app/views/beers/_beer.html.erb:13
2022-08-20T10:56:01.307822+00:00 app[web.1]: [22db4647-3122-419e-8e83-e2e99bfe3606] app/views/beers/index.html.erb:7
2022-08-20T10:56:01.307822+00:00 app[web.1]: [22db4647-3122-419e-8e83-e2e99bfe3606] app/views/beers/index.html.erb:6
```

Here is the cause:

    undefined method `name' for nil:NilClass

The line which causes our problem is

```erb
<%= link_to beer.brewery.name, beer.brewery %>
```

So there is a beer whose <code>brewery</code> field is <code>nil</code>. This can be because the value of the beer <code>brewery_id</code> is either <code>nil</code> or erroneous (in case of a brewery which has been deleted).

After you have found the reason, you'll have to find the culprit. Open your heroku console with the command <code>heroku run console</code> and retrieve the beers without brewery:

```ruby
> Beer.all.select{ |b| b.brewery.nil? }
=> [#<Beer id: 8, name: "crap beer", style: "lager", brewery_id: nil, created_at: "2020-08-20 13:37:21", updated_at: "2020-08-20 13:37:21">, #<Beer id: 9, name: "shitty beer", style: "lager", brewery_id: 123, created_at: "2020-08-20 13:38:51", updated_at: "2020-08-20 13:38:51">]
>
```

The next thing to do is fixing the objects which cause the problem. Because you created them yourself a moment ago for testing reasons, delete the objects (first store into a variable the objects returned by the previous operation which are in the variable <code>_</code>):

```ruby
> bad_beer = _
=> [#<Beer id: 8, name: "crap beer", style: "lager", brewery_id: nil, created_at: "2020-08-20 13:37:21", updated_at: "2020-08-20 13:37:21">, #<Beer id: 9, name: "shitty beer", style: "lager", brewery_id: 123, created_at: "2020-08-20 13:38:51", updated_at: "2020-08-20 13:38:51">]
> bad_beer.each{ |bad| bad.delete }
> Beer.all.select{ |b| b.brewery.nil? }
=> []
>
```

Most commonly, the problems we have in production depend on the inconsistent state that some objects have got because of our changes in the database scheme. For instance, they may be belonging to objects which do not exist or the references might be missing. **It is a good practice to deploy the application in the production mode as often as possible**, in this way, you will know that the potential problems are caused by the changes you have just done and fixing them will be easier.

Because it is a program in production, resetting the database (<code>rails db:drop</code>) in never aa acceptable way to "fix" a database inconsistency because the data in production cannot be destroyed. You should start from the beginning to learn reading logs and finding out problems as you are meant to do.

## Submitting the exercises

Commit all your changes and push the code to Github. Deploy to the newest version to Heroku or Fly.io, as well.

Mark the exercises you have done at https://studies.cs.helsinki.fi/stats/courses/rails2022.

