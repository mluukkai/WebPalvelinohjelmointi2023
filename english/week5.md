
You will continue to develop your application from the point you arrived at the end of week 4. The material that follows comes with the assumption that you have done all the exercises of the previous week. In case you have not done all of them, you can take the sample answer to the previous week he model answer found in the exercise submission system.

## Utilizing open interfaces: bar search

A great part of modern Internet services makes use of open interfaces which provides useful data to enrich applications functionality.

The best interface among the ones available seems to be  Beermapping API <http://beermapping.com/api/>, which makes it possible to search for beer restaurants.

Applications which make use of beermaping API need a singular API key. You can retrieve a key at https://beermapping.com/api/, after logging in to the page (after logging in edit the url in your browser's address bar back to https://beermapping.com/api/). This is a common procedure in use for the larger part of modern free interfaces.

The API's services are listed at the page http://beermapping.com/api/reference/

For instance, you can find out the beer restaurants in a defined location by making an HTTP get request for the address <code>http://beermapping.com/webservice/loccity/[apikey]/[city]<location></code>

The location is passed as a part of the URL.

You can make requests with the browser or from the command line with the curl program. You'll find out beer restaurants in Espoo in the following way:

```ruby
mluukkai@melkki$ curl https://beermapping.com/webservice/loccity/731955affc547174161dbd6f97b46538/espoo
<?xml version='1.0' encoding='utf-8' ?><bmp_locations><location><id>12411</id><name>Gallows Bird</name><status>Brewery</status><reviewlink>https://beermapping.com/location/12411</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=12411&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=12411&amp;d=1&amp;type=norm</blogmap><street>Merituulentie 30</street><city>Espoo</city><state></state><zip>02200</zip><country>Finland</country><phone>+358 9 412 3253</phone><overall>91.66665</overall><imagecount>0</imagecount></location><location><id>21108</id><name>Captain Corvus</name><status>Beer Bar</status><reviewlink>https://beermapping.com/location/21108</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=21108&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=21108&amp;d=1&amp;type=norm</blogmap><street>Suomenlahdentie 1</street><city>Espoo</city><state>Etela-Suomen Laani</state><zip>02230</zip><country>Finland</country><phone>+358 50 4441272</phone><overall>0</overall><imagecount>0</imagecount></location><location><id>21496</id><name>Olarin panimo</name><status>Brewery</status><reviewlink>https://beermapping.com/location/21496</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=21496&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=21496&amp;d=1&amp;type=norm</blogmap><street>Pitkäniityntie 1</street><city>Espoo</city><state>Etela-Suomen Laani</state><zip>02810</zip><country>Finland</country><phone>045 6407920</phone><overall>0</overall><imagecount>0</imagecount></location><location><id>21516</id><name>Fat Lizard Brewing</name><status>Brewery</status><reviewlink>https://beermapping.com/location/21516</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=21516&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=21516&amp;d=1&amp;type=norm</blogmap><street>Lämpömiehenkuja 3</street><city>Espoo</city><state>Etela-Suomen Laani</state><zip>02150</zip><country>Finland</country><phone>09 23165432</phone><overall>0</overall><imagecount>0</imagecount></location><location><id>21545</id><name>Simapaja</name><status>Brewery</status><reviewlink>https://beermapping.com/location/21545</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=21545&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=21545&amp;d=1&amp;type=norm</blogmap><street>Kipparinkuja 2</street><city>Espoo</city><state>Etela-Suomen Laani</state><zip>02320</zip><country>Finland</country><phone></phone><overall>0</overall><imagecount>0</imagecount></location></bmp_locations>
```

As you'll notice, the response is in XML. This is a bit outdated because the currently most popular format to exchange information among Web services is Json.

If you use your browser, you'll see the returned XML in a form which can be read more easily for humans:

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-1.png)

**ATTENTION: do not use the API key which is shown here but register one for your own.**

**ATTENTION2:** In Fall 2022, the API doesn't find any bars in Espoo, try another city! The API's coverage for Finland is lacking.

Make now the search functionality for your application beer restaurants.

Create a page for this at the address places, so go to route.rb and define

    get 'places', to: 'places#index'

and create the controller:

```ruby
class PlacesController < ApplicationController
  def index
  end
end
```

plus the view app/views/places/index.html.erb, which will only look like a search box at first:

```erb
<h1>Beer places search</h1>

<%= form_with url: places_path, method: :post do |form| %>
  city <%= form.text_field :city %>
  <%= form.submit "Search" %>
<% end %>
```

The form sends HTTP POST requests to places_path. So define an appropriate route for it in routes.rb

    post 'places', to:'places#search'

In this way, you defined that the method name is <code>search</code>. Extend the controller like below:

```ruby
class PlacesController < ApplicationController
  def index
  end

  def search
    render :index
  end
end
```

The idea is that the <code>search</code> method retrieves a restaurant list from beermapping API. The restaurants will then be added to index.html and that's why the method <code>search</code> renders the view template <code>index</code> at the end.

The <code>search</code> method has to make an HTTP request from the controller to the beermapping API page. The best way to make HTTP requests in Ruby is using the HTTParty gem, see https://github.com/jnunemaker/httparty. Add the following to your Gemfile:

    gem 'httparty'

Set up the gem by running the usual command from the command line, <code>bundle install</code>

Try to look for Helsinki restaurants by hand from the console now (remember to restart your console):

```ruby
> api_key = "731955affc547174161dbd6f97b46538"
> url = "http://beermapping.com/webservice/loccity/#{api_key}/"
> response = HTTParty.get "#{url}helsinki"
```

The call will return an object of the class <code>HTTParty::Response</code>.  [The documentation](https://www.rubydoc.info/github/jnunemaker/httparty/HTTParty/Response) shows that the object can be enquired about the headers concerning the answer:

```ruby
> response = HTTParty.get "#{url}helsinki"
> response.headers
=> {"date"=>["Mon, 17 Sep 2018 20:43:11 GMT"],
 "server"=>["Apache"],
 "upgrade"=>["h2,h2c"],
 "connection"=>["Upgrade, close"],
 "set-cookie"=>["easylogin_session=eff28ad09a8f62046917a8c424e4b0b3; path=/"],
 "expires"=>["Mon, 26 Jul 1997 05:00:00 GMT"],
 "cache-control"=>
  ["no-store, no-cache, must-revalidate", "post-check=0, pre-check=0"],
 "pragma"=>["no-cache"],
 "last-modified"=>["Mon, 17 Sep 2018 20:43:11 GMT"],
 "vary"=>["Accept-Encoding"],
 "content-length"=>["972"],
 "content-type"=>["text/xml;charset=UTF-8"]}
>
```

The headers include metadata related to the answer of the HTTP request, e.g. _content-type_ tells the type of the answer.

```
"content-type"=>["text/xml;charset=UTF-8"]
```

The HTTP call status code can be found with:

```ruby
> response.code
 => 200
```

The status code (see https://www.rfc-editor.org/rfc/rfc9110.html#name-successful-2xx), is 200 now, which is fine, meaning that the request has succeeded.

The answer object method <code>parsed_response</code> will return the data as Ruby's hash:

```ruby
> response.parsed_response
=> {"bmp_locations"=>
  {"location"=>
    [{"id"=>"6742",
      "name"=>"Pullman Bar",
      "status"=>"Beer Bar",
      "reviewlink"=>"https://beermapping.com/location/6742",
      "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=6742&d=5",
      "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=6742&d=1&type=norm",
      "street"=>"Kaivokatu 1",
      "city"=>"Helsinki",
      "state"=>nil,
      "zip"=>"00100",
      "country"=>"Finland",
      "phone"=>"+358 9 0307 22",
      "overall"=>"72.500025",
      "imagecount"=>"0"},
     {"id"=>"6743",
      "name"=>"Belge",
      "status"=>"Beer Bar",
      "reviewlink"=>"https://beermapping.com/location/6743",
      "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=6743&d=5",
      "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=6743&d=1&type=norm",
      "street"=>"Kluuvikatu 5",
      "city"=>"Helsinki",
      "state"=>nil,
      "zip"=>"00100",
      "country"=>"Finland",
      "phone"=>"+358 10 766 35",
      "overall"=>"67.499925",
      "imagecount"=>"1"},
...
```

Even though the server returns the answer in XML format, the gem HTTParty parses it, and makes it possible that it is handled straight in the best form as Ruby hash.

You can get the restaurant table returned by the request in the following way:

```ruby
> places = response.parsed_response['bmp_locations']['location']
> places.size => 12
```

So, it knows 12 places in Helsinki. Inspect the first one:

```ruby
> places.first
=> {"id"=>"6742",
 "name"=>"Pullman Bar",
 "status"=>"Beer Bar",
 "reviewlink"=>"https://beermapping.com/location/6742",
 "proxylink"=>"http://beermapping.com/maps/proxymaps.php?locid=6742&d=5",
 "blogmap"=>"http://beermapping.com/maps/blogproxy.php?locid=6742&d=1&type=norm",
 "street"=>"Kaivokatu 1",
 "city"=>"Helsinki",
 "state"=>nil,
 "zip"=>"00100",
 "country"=>"Finland",
 "phone"=>"+358 9 0307 22",
 "overall"=>"72.500025",
 "imagecount"=>"0"}
```


Create an object only to present the restaurants, and call it by the name <code>Place</code>. Put the class in the models folder.

```ruby
class Place < OpenStruct
end
```

As we create the class from a hash representing a beer restaurant, we will make the class inherit Ruby's ready-made [OpenStruct](https://ruby-doc.org/stdlib-3.1.2/libdoc/ostruct/rdoc/OpenStruct.html) class' functionality.

With OpenStruct it is easy to "wrap" a hash into an object which makes referencing fields of the hash possible with the dot-notation.

E.g. if we have a normal hash:

```ruby
bar_hash = {
 "name"=>"Pullman Bar",
 "status"=>"Beer Bar",
  "city"=>"Helsinki"
}
```

we have to use brackets to reference to its fields:
```ruby
> bar_hash['name']
=> "Pullman Bar"
> bar_hash['city']
=> "Helsinki"
```

If we "wrap" a hash into an OpenStruct object:
```ruby
> bar = OpenStruct.new baari_hash
```

we can access all fields with dot-notation:
```ruby
bar.name
=> "Pullman Bar"
bar.city
=> "Helsinki"
```

in this way we got an object which is used similarly to normal Rails models such as Beer, Brewery, etc.

However, we do not want to use OpenStructs directly in our code. That is way we create the class Places for beer restaurants and this class inherits OpenStruct:

```ruby
class Place < OpenStruct
end
```

Defining a separate class makes the code clearer and makes it possible to add methods to class objects as necessary.

Our class in used by passing it the hash defining a beer restaurant as a constructor parameter:

```ruby
irb(main):011:0> baari = Place.new places.first
=> #<Place id="6742", name="Pullman Bar", status="Beer Bar", reviewlink="https://beermapping.com/location/6742", proxylink="http://beerma...
irb(main):012:0> baari.name
=> "Pullman Bar"
irb(main):013:0> baari.zip
=> "00100"
irb(main):014:0>
```

So write the code to initialize the controller. Hard-code the search criteria so that it starts from Helsinki and create only one Place object for the first place which is retrieved:

```ruby
class PlacesController < ApplicationController
  def index
  end

  def search
    api_key = "731955affc547174161dbd6f97b46538"
    url = "http://beermapping.com/webservice/loccity/#{api_key}/"

    response = HTTParty.get "#{url}helsinki"
    places_from_api = response.parsed_response["bmp_locations"]["location"]
    @places = [ Place.new(places_from_api.first) ]

    render :index, status: 418
  end
end
```

We also add the status code 418 to render. This is to let [Turbo](https://github.com/hotwired/turbo-rails) (which is used by Rails) know that it should rerender the same page after the POST request. With this, the tests will also work. If we were to set e.g. status code 303 [the tests would break](https://stackoverflow.com/a/30555199). This hack is an example of bad code, but as we navigate between Turbo and the tests, it is necessary.

Modify app/views/places/index.html.erb so that it shows the restaurants which have been found:

```erb
<h1>Beer places search</h1>

<%= form_with url: places_path, method: :post do |form| %>
  city <%= form.text_field :city %>
  <%= form.submit "Search" %>
<% end %>

<% if @places %>
  <ul>
    <% @places.each do |place| %>
      <li><%=place.name %></li>
    <% end %>
  </ul>
<% end %>
```

The code looks like working (notice that you will have to restart Rails server so that the gem HTTParty will be set up).

Expand the code to show all restaurants and to make use of a form parameter to search against the locality:

```ruby
  def search
    api_key = "731955affc547174161dbd6f97b46538"
    url = "http://beermapping.com/webservice/loccity/#{api_key}/"
    response = HTTParty.get "#{url}#{params[:city]}"

    @places = response.parsed_response["bmp_locations"]["location"].map do | place |
      Place.new(place)
    end

    render :index, status: 418
  end
```

The application works, but it will display an error if the locality does not have restaurants.

If you use the debugger, you will see what the locality list returned by the API looks like in such cases:

```ruby
{"id"=>nil, "name"=>nil, "status"=>nil, "reviewlink"=>nil, "proxylink"=>nil, "blogmap"=>nil, "street"=>nil, "city"=>nil, "state"=>nil, "zip"=>nil, "country"=>nil, "phone"=>nil, "overall"=>nil, "imagecount"=>nil}
```

So the return value is a hash. But if the search finds restaurants, the return value is a table which contains hashes. Fix the code taking this into consideration. The code will also take into consideration the case where the API returns a hash which does not correspond to an nonexistent place anyway. This is the case when a town has only one restaurant.

```ruby
class PlacesController < ApplicationController
  def index
  end

  def search
    api_key = "731955affc547174161dbd6f97b46538"
    url = "http://beermapping.com/webservice/loccity/#{api_key}/"
    response = HTTParty.get "#{url}#{params[:city]}"
    places_from_api = response.parsed_response["bmp_locations"]["location"]

    if places_from_api.is_a?(Hash) && places_from_api['id'].nil?
      redirect_to places_path, notice: "No places in #{params[:city]}"
    else
      places_from_api = [places_from_api] if places_from_api.is_a?(Hash)
      @places = places_from_api.map do | location |
        Place.new(location)
      end
      render :index, status: 418
    end
  end

end
```

The code looks quite bad so far, but we'll come back to this issue in a moment. Make sure the page shows more information about the bars. Define the keys which should be shown as static methods of the Place class:

```ruby
class Place < OpenStruct
  def self.rendered_fields
    [:id, :name, :status, :street, :city, :zip, :country, :overall ]
  end
end
```

below the improved code for index.html.erb:

```erb
<h1>Beer places search</h1>

<p id="notice"><%= notice %></p>

<%= form_with url: places_path, method: :post do |form| %>
  city <%= form.text_field :city %>
  <%= form.submit "Search" %>
<% end %>

<% if @places %>
  <table>
    <thead>
      <% Place.rendered_fields.each do |field| %>
        <th><%= field %></th>
      <% end %>
    </thead>
    <% @places.each do |place| %>
      <tr>
        <% Place.rendered_fields.each do |field| %>
          <td><%= place.send(field) %></td>
        <% end %>
      </tr>
    <% end %>
  </table>
<% end %>
```

The restaurants are now shows as an [HTML table](https://developer.mozilla.org/en-US/docs/Learn/HTML/Tables/Basics).

## Calling object methods with the _send_ method

The code generating the rows of the table is

```erb
<tr>
  <% Place.rendered_fields.each do |field| %>
    <td><%= place.send(field) %></td>
  <% end %>
</tr>
```

What is really happening here?

Before changes the view was rendered as follows:
```erb
<% @places.each do |place| %>
  <li><%= place.name %></li>
<% end %>
```

so for each restaurant its name was displayed, <code>place.name</code>

Our current code does the same as the one below which is in a more understandable format:
```erb
<tr>
  <td><%= place.id %></td>
  <td><%= place.name %></td>
  <td><%= place.status %></td>
  <td><%= place.street %></td>
  <td><%= place.city %></td>
  <td><%= place.zip %></td>
  <td><%= place.country %></td>
  <td><%= place.overall %></td>
</tr>
```

In Ruby, an object method can be also "indirectly" called by using the method <code>send</code>. So instead of writing  <code>place.name</code>, we can create a method call with the <code>place.send(:name)</code> syntax. A beer restaurant's row generation can be changed into 

```erb
<tr>
  <td><%= place.send(:id) %></td>
  <td><%= place.send(:name) %></td>
  <td><%= place.send(:status) %></td>
  <td><%= place.send(:street) %></td>
  <td><%= place.send(:city) %></td>
  <td><%= place.send(:zip) %></td>
  <td><%= place.send(:country) %></td>
  <td><%= place.send(:overall) %></td>
</tr>
```

As we also defined the method <code>Place.rendered_fields</code> to return the list <code>[ :id, :name, :status, :street, :city, :zip, :country, :overall ]</code>, we can generate the td-tags  with an <code>each</code> loop.

```erb
<tr>
  <% Place.rendered_fields.each do |field| %>
    <td><%= place.send(field) %></td>
  <% end %>
</tr>
```

Should you do this? It partly up to personal preferences. By defining the list of fields to render, we could also generate the table header row by looping:

```erb
<thead>
  <% Place.rendered_fields.each do |field| %>
    <td><%= field %></td>
  <% end %>
</thead>
```

If we now decided to add or remove some visible fields, it is enough to just edit the list defined in the class <code>Places</code>. No need to touch the template.

```ruby
class Place < OpenStruct
  def self.rendered_fields
    [ :id, :name, :status, :street, :city, :zip, :country, :overall ]
  end
end
```

## Names containing special characters

Your application hides a small issue still. If you try to look for New York beer restaurants you will run into troubles. The space has to be replaced with the code %20 in the URL. The change should not be made 'by hand', because the space is not the only character which has to be coded into the URL. As you might have thought, Rails provides a ready-made solution for this, the method <code>ERB::Util.url_encode</code>. Try the method out from the console:

```ruby
> ERB::Util.url_encode("St John's")
 => "St%20John%27s"
>
```

Implement the code change by replacing the HTTP GET request line with the following:

```ruby
response = HTTParty.get "#{url}#{ERB::Util.url_encode(params[:city])}"
```

> ## Exercise 1
>
> Implement the code above in your program. Also add a link to the beer restaurants search page in the navigation bar.

## Refactoring your Places controller

Rails controllers should not include application logic. It is a best practice to put external APIs in their own class. A good place for such class is in the _lib_ folder. Place the following code into the file _lib/beermapping_api.rb_:

```ruby
class BeermappingApi
  def self.places_in(city)
    url = "http://beermapping.com/webservice/loccity/#{key}/"

    response = HTTParty.get "#{url}#{ERB::Util.url_encode(city)}"
    places = response.parsed_response["bmp_locations"]["location"]

    return [] if places.is_a?(Hash) and places['id'].nil?

    places = [places] if places.is_a?(Hash)
    places.map do | place |
      Place.new(place)
    end
  end

  def self.key
    "731955affc547174161dbd6f97b46538"
  end
end
```

So the class defines a static method which returns a table of the beer restaurants which have been found in the towns defined by the parameter. If no restaurant is found, the table will be empty. The API class is not in its best format yet, because you cannot know completely what other methods you need.

To ensure that code in the _lib_ folder will work (not only on your computer but also in Heroku and Fly.io), you must add these two lines into the _config/application.rb_ file:

```ruby
config.autoload_paths << Rails.root.join("lib")
config.eager_load_paths << Rails.root.join("lib")
```

The addition should be placed inside _Application_ class definition

```ruby
module Ratebeer
  class Application < Rails::Application
    # ...

    # add here
  end
end
```

Restart the application for the changes to take effect.

The controller will be looking neat, by now:

```ruby
class PlacesController < ApplicationController
  def index
  end

  def search
    @places = BeermappingApi.places_in(params[:city])
    if @places.empty?
      redirect_to places_path, notice: "No locations in #{params[:city]}"
    else
      render :index, status: 418
    end
  end
end
```

## Testing beer restaurant search

You will want to create Rspec tests for the functionality you've implemented. The new functionality makes use of external services. Tests should be written in any case without making use of the external services. Luckily, it is easy to replace an external interface with Rails stub component.

You will want to divide the tests in two parts. The class <code>BeermappingApi</code> encapsulates the external interface, so replace its functionality by hard coding a new one with the help of stubs. The test will check whether the places page works properly, assuming that the <code>BeermappingApi</code> components works properly.

The functionality of the <code>BeermappingApi</code> component will then be tested separately with unit tests written with Rspec.

So get started with your tests on the functionality of the web page places. Create a file for the test, /spec/features/places_spec.rb

```ruby
require 'rails_helper'

describe "Places" do
  it "if one is returned by the API, it is shown at the page" do
    allow(BeermappingApi).to receive(:places_in).with("kumpula").and_return(
      [ Place.new( name: "Oljenkorsi", id: 1 ) ]
    )

    visit places_path
    fill_in('city', with: 'kumpula')
    click_button "Search"

    expect(page).to have_content "Oljenkorsi"
  end
end
```

The test gets started with an interesting command:

```ruby
allow(BeermappingApi).to receive(:places_in).with("kumpula").and_return(
  [ Place.new( name: "Oljenkorsi", id: 1 ) ]
)
```

The command "hard codes" a table containing one Place object in answer to the method <code>places_in</code> of the class <code>BeermappingApi</code>, as if the method was called with the parameter "kumpula".

When the tests make an HTTP request to the places controller, and as the controller calls the API method <code>places_in</code>, instead of executing the real code the places controller is returned a hard-coded answer.

> ## Exercise 2
>
> Extend your tests to cover the following exceptions:
> * if the API returns various beer restaurants, all of them are shown on the page
> * if the API does not find any beer restaurant in town (so if the return value is an empty table), the page should show the message "No locations in _place name_"
>
> Section [Polishing the signing up](https://github.com/mluukkai/WebPalvelinohjelmointi2023/blob/main/english/week3.md#polishing-the-signing-up) from week 3 may be helpful for the second task.

Let's move to testing the class <code>BeermappingApi</code>. The class makes an HTTP GET request for the Beermapping service with the help of the HTTParty library. You could stub the HTTParty get method like in the previous exercise. This would not be nice, though, because the method returns an <code>HTTPartyResponse</code> object and creating that by hand with stub is not the most fun thing to do.

A better option is using _webmock_ https://github.com/bblimke/webmock/, which makes it possible to stub at the level of the library used by HTTParty.

Get started with the gem by adding the line <code>gem 'webmock'</code> to the Gemfile **test-scope**:


```ruby
group :test do
  # ...
  gem 'webmock'
end
```

**ATTENTION: webmock has to be defined _only_ into test-scope, otherwise it will prevent all the HTTP requests made by your application!**

Run <code>bundle install</code>.

Add also the following line to the file ```spec/rails_helper.rb```:

```ruby
require 'webmock/rspec'
```

Using the webmock library is easy, altogether. For instance, the command below stubs the GET request to _every_ URL (which is defined with regexp <code>/.*</code>) to return information  about 'Lapin kulta' beer in XML form :

```ruby
stub_request(:get, /.*/).to_return(body: "<beer><name>Lapin kulta</name><brewery>Hartwall</brewery></beer>", headers:{ 'Content-Type' => "text/xml" })
```

So if you called <code>HTTParty.get("http://www.google.com")</code> after the command, you'd see the following

```xml
<beer>
  <name>Lapin kulta</name>
  <brewery>Hartwall</brewery>
</beer>
```

So you need the right "hard-coded" data for tests, the data which represents the XML returned by the Beermapping service HTTP GET request.

A way to generate a test input is to ask the interface itself for it, so make an HTTP GET request with the <code>curl</code> command from the command line:

```ruby
mluukkai@melkki$ curl http://beermapping.com/webservice/loccity/731955affc547174161dbd6f97b46538/turku
<?xml version='1.0' encoding='utf-8' ?><bmp_locations><location><id>18856</id><name>Panimoravintola Koulu</name><status>Brewpub</status><reviewlink>https://beermapping.com/location/18856</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=18856&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=18856&amp;d=1&amp;type=norm</blogmap><street>Eerikinkatu 18</street><city>Turku</city><state></state><zip>20100</zip><country>Finland</country><phone>(02) 274 5757</phone><overall>0</overall><imagecount>0</imagecount></location></bmp_locations>
```

Now you could copy-paste the information returned in XML form by the HTTP reqest to your test. If you want to be sure you place the XML right in the string, you should use a quite particular syntax
see http://blog.jayfields.com/2006/12/ruby-multiline-strings-here-doc-or.html where the string is placed between <code><<-END_OF_STRING</code> and <code>END_OF_STRING</code>.

You find below the test code which should be placed into spec/lib/beermapping_api_spec.rb (deciding to place the code in the lib subfolder because the test destination is an auxiliary class in the lib folder):

```ruby
require 'rails_helper'

describe "BeermappingApi" do
  it "When HTTP GET returns one entry, it is parsed and returned" do

    canned_answer = <<-END_OF_STRING
<?xml version='1.0' encoding='utf-8' ?><bmp_locations><location><id>18856</id><name>Panimoravintola Koulu</name><status>Brewpub</status><reviewlink>https://beermapping.com/location/18856</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=18856&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=18856&amp;d=1&amp;type=norm</blogmap><street>Eerikinkatu 18</street><city>Turku</city><state></state><zip>20100</zip><country>Finland</country><phone>(02) 274 5757</phone><overall>0</overall><imagecount>0</imagecount></location></bmp_locations>
    END_OF_STRING

    stub_request(:get, /.*turku/).to_return(body: canned_answer, headers: { 'Content-Type' => "text/xml" })

    places = BeermappingApi.places_in("turku")

    expect(places.size).to eq(1)
    place = places.first
    expect(place.name).to eq("Panimoravintola Koulu")
    expect(place.street).to eq("Eerikinkatu 18")
  end

end
```

So the test first defines that the HTTP GET request for the URL ending in the string "turku" (which was defined through the regexp <code>/.\*turku/</code>) should return a hardcoded XML; it defines in the header that the information returned is in XML form. Without this definition the HTTParty library will not know how to parse correctly the data of the HTTP request.


The test itself is straightforward: it checks the table returned by the BeermappingApi method <code>places_in</code>.

*Attention:* in the test you only stubbed the HTTP GET calls for the URLs that ended with "turku" (<code>/.*turku/</code>). If the test execution causes any other kind of HTTP call, the test will point this out:

```ruby
) BeermappingApi When HTTP GET returns no entries, an empty array is returned
     Failure/Error: places = BeermappingApi.places_in("kumpula")
     WebMock::NetConnectNotAllowedError:
       Real HTTP connections are disabled. Unregistered request: GET http://beermapping.com/webservice/loccity/731955affc547174161dbd6f97b46538/kumpula

       You can stub this request with the following snippet:

       stub_request(:get, "http://beermapping.com/webservice/loccity/731955affc547174161dbd6f97b46538/kumpula").
         to_return(:status => 200, :body => "", :headers => {})
```

As you'll understand from the error message, you can also stub the HTTP call for the singular URL string with the help of the command <code>stub_request</code>. The same test can also contain various <code>stub_request</code> calls, all defining answers for different URL requests.

> ## Exercise 3
>
> Extend the tests to include the cases below
> * if HTTP GET does not return any place, the method <code>places_in</code> should return an empty table
> * if HTTP GET returns various places, the method <code>places_in</code> should return a table of Place objects containing all the restaurants returned by the HTTP call as XML.
>
> The stubbed answers should be formed again with the help of the curl command with the API requests
>
> Remember to use debugger as help while testing

Stubbing and mocking methods and whole objects is a vast area of studies. You can read more on the topic in connection to Rspec at the following link http://rubydoc.info/gems/rspec-mocks/

Terms like stub or mock objects or "stubbing and mocking" are used rather carelessly. Luckily, Rails community uses the terms properly. To tell a long story short, stubs are objects where method answers have been hard-coded beforehand. Mock also provide hard-coded answers as stubs, but in addition to doing it, mocks help you to define your expectations on how should their methods be called. If the objects which have to be tested do not call mock methods as expected, this will produce an error.

More about Mocks and Stubs at: http://martinfowler.com/articles/mocksArentStubs.html

## Performance optimization

Your application will currently be making requests to the beermapping service every time that the restaurants of a city are retrieved. You could improve the application  by memorizing recent searches.

Rails provides an easy-to-use, key-value combination based cache.

The cache is not on by default. You can take it into use by executing <code>rails dev:cache</code> from the command line.

In _config/environments/development.rb_ also replace these rows

```ruby
config.cache_store = :memory_store
config.cache_store = :null_store
```

with this

```ruby
config.cache_store = :file_store, 'tmp/cache_store'
```

Also restart your console and application.

You can access your cache through the object saved in the variable <code>Rails.cache</code>. Try on your console:

```ruby
> Rails.cache.write "avain", "arvo"
 => true
> Rails.cache.read "avain"
 => "arvo"
> Rails.cache.read "kumpula"
 => nil
> Rails.cache.write "kumpula", Place.new(name: "Oljenkorsi")
 => true
> Rails.cache.read "kumpula"
 => #<Place:0x00000104628608 @name="Oljenkorsi">
```

You can store almost anything in cache. And the interface is really simple, see http://api.rubyonrails.org/classes/ActiveSupport/Cache/Store.html

The first method call will search from the database and save the object in the cache memory. The following call will receive the key object straight form the cache memory.

Rails cache saves key-value pairs in the file system by default. How the cache stores objects can be configured however, see http://guides.rubyonrails.org/caching_with_rails.html#cache-stores

Storing cache data in the file system during production is not optimal in terms of performance. A better option is [Memcached](http://memcached.org/) for instance; read more at https://devcenter.heroku.com/articles/building-a-rails-3-application-with-memcache

**Attention:** because our tests will soon start to test the code which makes use of Rails.cache, you'd better configure your cache to use the _central memory_ instead of file system to store information during tests. You can do it by adding the following line to the file _config/environments/test.rb_:

```ruby
config.cache_store = :memory_store
```

Modify the class <code>BeermappingApi</code> so that it will save the request results in the cache memory. If a request concerns a city which is available in cache, the result is returned from cache.

```ruby
class BeermappingApi
  def self.places_in(city)
    city = city.downcase

    places = Rails.cache.read(city)
    return places if places

    places = get_places_in(city)
    Rails.cache.write(city, places)
    places
  end

  def self.get_places_in(city)
    url = "http://beermapping.com/webservice/loccity/#{key}/"

    response = HTTParty.get "#{url}#{ERB::Util.url_encode(city)}"
    places = response.parsed_response["bmp_locations"]["location"]

    return [] if places.is_a?(Hash) and places['id'].nil?

    places = [places] if places.is_a?(Hash)
    places.map do | place |
      Place.new(place)
    end
  end

  def self.key
    "731955affc547174161dbd6f97b46538"
  end
end
```

The lowercase city name is used as key. The logic is quite simple: if restaurants corresponding to a key can be found in the cache (i.e. value is not nil) they are returned. If the cache doesn't contain restaurants for a certain city, they will be fetched with the method <code>get_places_in(city)</code>, saved into cache and returned to the method caller.

If you do a search for New York beer restaurants twice in a row, you'll see that the answer will be returned much faster the second time.

You have access to the data in the application cache memory also from console:

```ruby
> Rails.cache.read("helsinki").map(&:name)
 => ["Pullman Bar", "Belge", "Suomenlinnan Panimo", "St. Urho's Pub", "Kaisla", "Pikkulintu", "Bryggeri Helsinki", "Stadin Panimo", "Panimoravintola Bruuveri"]
>
```

It is also possible to delete the value stored in a defined key by hand from the console, in case you need:

```ruby
> Rails.cache.delete("helsinki")
 => true
> Rails.cache.read("helsinki")
 => nil
>
```

You could also simplify the code a bit by using the Rails.cache method [fetch](https://api.rubyonrails.org/classes/ActiveSupport/Cache/Store.html#method-i-fetch)

```ruby
class BeermappingApi
  def self.places_in(city)
    city = city.downcase
    Rails.cache.fetch(city) { get_places_in(city) }
  end

  def get_places_in(city)
    # ...
  end
end
```

If the cache contains data for the key it got as a parameter, the method returns the data in cache. If the cache doesn't contain any data corresponding to the key, the code block passed with the method will be executed and the return value of that block will be saved into cache. The command _fetch_ in itself will also return the code block's value.

## Outdated data

The problem with cache memory is when it comes to outdated data. So sometimes one may add restaurants to the beermapping page, and your cache memory will maintain the old data. Then you should make sure that the cache memory will not contain too old data.

One option is clearing the cache memory data from time to time with the command:

    Rails.cache.clear

A better solution is defining an expiring lifetime for the cached data.

> ## Exercise 4
>
> ### This is not the most important exercise of the week, so do not get stuck with it if you get problems
>
>  Define the expiring lifetime for the restaurant data that are saved in the cache memory, 1 week for instance. When you test the exercise, you should use a shorter lifespan however, like one minute.
>
> Passing the exercise does not require major changes in your code, you only need to fix _one_ line in fact. You find useful hints at http://guides.rubyonrails.org/caching_with_rails.html#activesupport-cache-store. Other information to manage the time settings at http://guides.rubyonrails.org/active_support_core_extensions.html#time
>
> **Attention:** as usual, you should test the expiring lifetime settings by hand from console!
>
> **Attention2:** if you mess up  the cache memory, remember <code>Rails.cache.clear</code> and <code>Rails.cache.delete key</code>

## Tests and cache

In exercise 3, you made tests for the class <code>BeermappingApi</code> with the help of Webmock. It is god to point out that cache memory affects tests and separately you may want to test the situations where data are not found from the cache memory (cache miss) and where they are already in cache (cache hit):

```ruby
require 'rails_helper'

describe "BeermappingApi" do
  describe "in case of cache miss" do

    before :each do
      Rails.cache.clear
    end

    it "When HTTP GET returns one entry, it is parsed and returned" do
      canned_answer = <<-END_OF_STRING
  <?xml version='1.0' encoding='utf-8' ?><bmp_locations><location><id>18856</id><name>Panimoravintola Koulu</name><status>Brewpub</status><reviewlink>https://beermapping.com/location/18856</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=18856&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=18856&amp;d=1&amp;type=norm</blogmap><street>Eerikinkatu 18</street><city>Turku</city><state></state><zip>20100</zip><country>Finland</country><phone>(02) 274 5757</phone><overall>0</overall><imagecount>0</imagecount></location></bmp_locations>
  END_OF_STRING

      stub_request(:get, /.*turku/).to_return(body: canned_answer, headers: { 'Content-Type' => "text/xml" })

      places = BeermappingApi.places_in("turku")

      expect(places.size).to eq(1)
      place = places.first
      expect(place.name).to eq("Panimoravintola Koulu")
      expect(place.street).to eq("Eerikinkatu 18")
    end

  end

  describe "in case of cache hit" do
    before :each do
      Rails.cache.clear
    end

    it "When one entry in cache, it is returned" do
      canned_answer = <<-END_OF_STRING
  <?xml version='1.0' encoding='utf-8' ?><bmp_locations><location><id>18856</id><name>Panimoravintola Koulu</name><status>Brewpub</status><reviewlink>https://beermapping.com/location/18856</reviewlink><proxylink>http://beermapping.com/maps/proxymaps.php?locid=18856&amp;d=5</proxylink><blogmap>http://beermapping.com/maps/blogproxy.php?locid=18856&amp;d=1&amp;type=norm</blogmap><street>Eerikinkatu 18</street><city>Turku</city><state></state><zip>20100</zip><country>Finland</country><phone>(02) 274 5757</phone><overall>0</overall><imagecount>0</imagecount></location></bmp_locations>
  END_OF_STRING

      stub_request(:get, /.*turku/).to_return(body: canned_answer, headers: { 'Content-Type' => "text/xml" })

      BeermappingApi.places_in("turku")
      places = BeermappingApi.places_in("turku")

      expect(places.size).to eq(1)
      place = places.first
      expect(place.name).to eq("Panimoravintola Koulu")
      expect(place.street).to eq("Eerikinkatu 18")
    end
  end
end
```

In the first <code>describe</code> block the <code>before :each</code> block clears the cache before the tests are run. So when the test uses the method call <code>BeermappingApi.places_in</code>, the restaurant information is fetched with a HTTP request. In the second describe block the method <code>BeermappingApi.places_in</code> is called twice. The first call makes sure that the information for the searched location is saved into cache. The second call is answered from cache and the result is tested.

The test is very redundant, and it should be refactored, but we won't stop for now.

**One more thing to keep in mind:** because you are testing a code which makes use of Rails.cache, you'd better configure the cache so that it makes use of the __central memory__ instead of the file system when it saves during tests. You can implement this adding the following line in the file _config/environments/test.rb_

```ruby
config.cache_store = :memory_store
```

## Saving application-specific data

The API key is written in your application code so far. This is of course not smart. There are many options to save the configuration information in Rails, see for instance https://guides.rubyonrails.org/configuring.html

The best option to save application-specific and not too complex data are environment variables. See the example below:

Set up <code>BEERMAPPING_APIKEY</code> as environment variable from the command line first.

```ruby
mluukkai@melkki$ export BEERMAPPING_APIKEY="731955affc547174161dbd6f97b46538"
```

Rails applications have access to environment variables through the hash variable <code<ENV</code>:

```ruby
> ENV['BEERMAPPING_APIKEY']
 => "731955affc547174161dbd6f97b46538"
>
```

Delete the hard-coded apikey and read it from the environment variable:

```ruby
class BeermappingApi
  # ...

  def self.key
    return nil if Rails.env.test? # while testing api is not needed, return nil
    raise 'BEERMAPPING_APIKEY env variable not defined' if ENV['BEERMAPPING_APIKEY'].nil?
    ENV.fetch('BEERMAPPING_APIKEY')
  end
end
```

The code contains an exception which is executed in case the apikey is not found.

The value of the environment variable will have to be defined if you search for beer restaurants. You can define the environment variable by starting your application as follows:

```ruby
mluukkai@melkki$ export BEERMAPPING_APIKEY="731955affc547174161dbd6f97b46538"
mluukkai@melkki$ rails s
```

or defining the environment variable together with the start command:

```ruby
mluukkai@melkki$ BEERMAPPING_APIKEY="731955affc547174161dbd6f97b46538" rails s
```

You can define the value of the environment variable (with the export command) in the file which is executed when the shell is started (the file format will be .zshrc, .bascrc or .profile according to the shell).

It's simple to set up the environment variable value in [Heroku](https://devcenter.heroku.com/articles/config-vars) and [Fly.io](https://fly.io/docs/reference/secrets/#setting-secrets ) too.

**ATTENTION** If you want to keep GitHub Actions in working order you need to define the environment variable in the workflow configuration, see https://docs.github.com/en/actions/learn-github-actions/environment-variables.

## More about the controller

Let's look more closely at the idea behind the <code>show</code> controller methods. A review will be beneficial also for the following exercises.

Take a look at a brewery controller. The controller method to show a singular brewery does not contain any code:

```ruby
  def show
  end
```
the view template app/views/breweries/show.html.erb renders by default anyway, and it points to the <code>@brewery</code> variable:

```ruby
<h2><%= @brewery.name %></h2>

<p>
  <em>Established year:</em>
  <%= @brewery.year %>
</p>
```

how will the variable get a value? The value is set in the controller as a _before filter_ defined in the method <code>set_brewery</code>.

```ruby
class BreweriesController < ApplicationController
  before_action :set_brewery, only: [:show, :edit, :update, :destroy]
  #...

  def set_brewery
    @brewery = Brewery.find(params[:id])
  end
end
```

so the controller defines that the code below should be executed always before the method <code>show</code>

```ruby
@brewery = Brewery.find(params[:id])
```

In turn, this loads the brewery object from memory and saves it into a variable for the view.

As you'll realize from the code, the controller retrieves the brewery ID through the hash <code>params</code>. How does this happen?

If you look at the application routes either with the command <code>rails routes</code> or through the browser (going to whatever invalid address like localhost:3000/foobar), you will see that the route information concerning singular breweries looks like this

```ruby
brewery_path	 GET	 /breweries/:id(.:format)	 breweries#show
```

so the form of the URL of a singular brewery is _breweries/42_, and the ending number stands for the brewery ID. As the route definition implies, the brewery ID is set as the value of the <code>:id</code> key in the <code>params</code> hash.

You could also define a 'parameter path' by hand. If you added the following chunk of code to routes.rb
(panimo means brewery in Finnish)

```ruby
   get 'panimo/:id', to: 'breweries#show'
```

you would have access to the brewery page from the address http://localhost:3000/panimo/42. Once again, the address would make use of the method <code>show</code> which would retrieve the ID in the same way from the <code>params</code> hash.

If you wanted to use another controller method and if you defined the route in the following way

```ruby
   get 'panimo/:panimo_id', to: 'breweries#nayta'
```

the controller method could look like the one below:

```ruby
def nayta
  @brewery = Brewery.find(params[:panimo_id])
  render :index
end
```

so this time you define the route so that the brewery ID was referenced through the <code>:panimo_id</code> key of the <code>params</code> hash.

## Restaurant page

> ## Exercises 5 and 6 (this equals to two exercises)
>
> Improve your application so that you can open a page with restaurant information by clicking on restaurant names. 
> - You'd better follow Rails conventions when you pick the restaurant URL, that is places/:id. Routes.rb could look something like this:
>
> ```ruby
> resources :places, only: [:index, :show]
> # which generates the same two paths as the following
> # get 'places', to: 'places#index'
> # get 'places/:id', to: 'places#show'
>
> post 'places', to: ' places#search'
> ```
>
>* Attention: the restaurant information are retrieved from cache a bit indirectly when users go to a restaurant page. In order to have access to this information you will have to "remember" the city where the restaurant was found, in addition to its ID – or the result of the last search operation. One way to do this is through sessions, see https://github.com/mluukkai/WebPalvelinohjelmointi2023/blob/main/english/week3.md#user-and-session
>
> Another way to implement this functionality is the so-called "Locquery Service," as described at the page http://beermapping.com/api/reference/ 
>
> **ATTENTION1** Because _Place_ is not an ActiveRecord class, the following will not work
>
> `link_to place.name, place`
>
> the link's target address needs to be defined in the longer form
>
> `link_to place.name, place_path(place.id)`
>
> **ATTENTION2** If you have trouble making a restaurant's name a clickable link, you can edit the table from the version using the _send_ method into a more simple version:
>
> ```erb
> <table>
>  <thead>
>    <th>id</th>
>    <th>name</th>
>    <th>status</th>
>    <th>street</th>
>    <th>city</th>
>    <th>zip</th>
>    <th>country</th>
>    <th>overall</th>
>  </thead>
>  <% @places.each do |place| %>
>    <tr>
>      <td><%= place.id %></td>
>      <td><%= place.name %></td>
>      <td><%= place.status %></td>
>      <td><%= place.street %></td>
>      <td><%= place.city %></td>
>      <td><%= place.zip %></td>
>      <td><%= place.country %></td>
>      <td><%= place.overall %></td>
>    </tr>
>  <% end %>
> </table>
> ```
>
> Check whether adding the restaurant page breaks any tests. If so, you can try to fix the test, even though it is not essential at this point.

After the exercise, your application can look something like this:

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-2.png)


## Rating beers straight from a beer page

Users have to rate beers from a separate page so far, and the beer is chosen out of a separate rating menu. It would be more natural if rating could happen straight from a beer page.

There are many optional ways to implement this. See below one of the easiest. You'll make use of the <code>form_for</code> helper, creating a form with the help of the object behind it. The **BeersController** show method will need a small change:

```ruby
def show
  @rating = Rating.new
  @rating.beer = @beer
end
```

So in case a beer is rated, a rating object already linked to it is created for the view template. The rating object is created with _new_, so it is not yet stored in the database. Notice that before executing the method <code>show</code>, the before filter executes a command which retrieves the beer to inspect from the database: <code>@beer = Beer.find(params[:id])</code>


The view template /views/beers/show.html.erb is modified as follows:

```erb
<p style="color: green"><%= notice %></p>

<%= render @beer %>

<% if current_user %>
  <h4>give a rating:</h4>

  <%= form_with(model: @rating) do |form| %>
    <%= form.hidden_field :beer_id %>
    score: <%= form.number_field :score %>
    <%= form.submit "Create rating" %>
  <% end %>

  <div>
    <%= link_to "Edit this beer", edit_beer_path(@beer) %>
    <%= button_to "Destroy this beer", @beer, method: :delete %>
  </div>
<% end %>
```

If you want that the form sends the beer ID, the field <code>beer_id</code> will have to be added into the form. You don't want that users are able to manipulate the field however, so it should be defined as <code>hidden_field</code> in the form.

Because the form is created with the helper <code>form_for</code>, it will be sent to <code>ratings_path</code> automatically with an HTTP POST request, which means that it is the rating controller <code>create</code> method which takes care of sending the form. The controller will work with no changes needed!

There is a small issue in this solution. If users try to give an invalid rating:

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-4.png)

the controller (that is, the rating controller <code>create</code> method) will render a new rating form instead of the beer view:

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-3.png)

A possible solution would be checking what address the create method has been accessed from and then rendering the right page based on the address found. We won't implement this change now, however.

First fix a bigger issue. You will see from the picture above, that if the rating validation fails (when trying to rate the  _Weihenstephaner Hefeweizen_ beer) the previously chosen beer is not selected anymore (instead it is now _Iso 3_).

The reason behind this is that the method <code>options_from_collection_for_select</code> which generates the options of the drop-down menu is not told which option it should choose by default, and so it picks the first object of the collection. You can specify the default option giving the method its fourth parameter:

```erb
options_from_collection_for_select(@beers, :id, :to_s, selected: @rating.beer_id)
```

So modify the view template app/views/ratings/new.html.erb to look like below:

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
  <%= f.select :beer_id, options_from_collection_for_select(@beers, :id, :to_s, selected: @rating.beer_id) %>
  score: <%= f.number_field :score %>
  <%= f.submit %>
<% end %>
```

In fact, other application forms have the same problem. Check what happens for instance when you try to create a beer without a name. Fix the form if you want.

> ## Exercise 7
>
> Make it possible to join beer clubs straight from beer club pages.
>
> You should stick to the same implementation principle as for the ratings on beer pages, so add a form in the page for beer club which should help you to create a new <code>Membership</code> object which belongs to the beer club and to the signed-in user. You can set values into hidden_field fields with the <code>value</code> parameter:
>
> ```erb
> <%= form_with(model: @membership) do |form| %>
>   <%= form.hidden_field :beer_club_id, value: @beer_club.id %>
>   <%= form.hidden_field :user_id, value: current_user.id %>
>   <%= form.submit "Join the beer club" %>
> <% end %>
> ``

And now some small changes to joining beer clubs

> ## Exercise 8
>
> The joining button should not be displayed if no user is signed in the system or if the user is already a member of the club.
>
> Change your code (the appropriate method of the membership controller) so that after joining a beer club the browser redirects to the beer club page and the page shows a message of the user having joined the club.

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-5.png)

> ## Exercise 9
>
> Extend your application functionality so that members can leave a beer club.
>
> Add a button for the beer club page which allows to leave the club. The button should be visible only if the sign-in user goes to a beer club page where he is a member already. When they press the button, user's memberships is destroyed, and they are redirected back to their own page. The page should show a message reporting the successful action, as the pictures below show.
>
> Hint: this functionality can be implemented in the same way as for joining a club, that is with a form on the beer club page. The HTTP method used in the form should be defined as "delete":
>
> ```erb
> <%= form_with(..., method: :delete) do |form| %>
>   ...
>   <%= form.hidden_field :beer_club_id, value: @beer_club.id %>
>   <%= form.hidden_field :user_id, value: current_user.id %>
>   <%= form.submit "End the membership" %>
> <% end %>
> ```
> There are many ways to complete this exercise. One way is to figure out the ID of the user's membership object which can then be set into a path.
>
> In order to use the form, the value of the <code>@membership</code> variable in the controller should be the object connecting the user to the club. If you do this exercise by using the ID of the <code>membership</code> object, it needs to be also let through the <code>membership_params</code> method in the controller.

If the user is the club member, the page should show a button to leave the club:

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-5a.png)

After leaving a club, the user should be redirected to their page and a message should appear:

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-5b.png)


## Migrations

You have been using Rails migrations already from the first week. Now it's time to dig deeper into the topic.

> ## Exercise 10
>
> Carefully read  http://guides.rubyonrails.org/migrations.html

## Beer style

> ## Exercises 11 – 13 (this equals three exercises)
>
> Expand your application so that the beer style won't be a string anymore; instead, the styles should be saved in the database. Each beer style should also have a textual description. The style description type should be defined as <code>text</code>; in fact, the default size of a field defined with the type <code>string</code> is only 255 characters.
>
> After the change, the beer-style relationship should be like this:

![picture](http://yuml.me/c5a711bb.png)

> Notice, that the <code>style</code> attribute which currently belongs to beer should be deleted so that there will be no association conflict between the accessor to be generated and the old field.
>
> It might be a bit challenging to make the change so that beers are linked automatically to the right style database tables.
> This will also work if you implement the change in multiple steps, for instance:
> * create an database table for the styles
> * in the table, create a row for each style name which is found in the _beers_table (this can be done from the console)
> * rename the _beers_ table _style_ column, call it something like _old_style_ (do it with migrations)
> * create a foreign key to _beers_ table for the styles (this also with migrations, this and the previous step can be done in the same migration)
> * work by hand on your console and connect beers and the _style_ objects with the help of the old_style column
>   * even better would be to do this with migrations aswell
> * remove _old_style_ from the beer table with the help of migrations
>
> **Notice that you should update Heroku/Fly.io instance simultaneously!**
>
> Suggestion: you can train migrating data. Copy the database, that is the file _db/development.sqlite3_, and if you mess up with the migration, you can always recover the old data from the copy. Debugger (binding.break) might also turn up useful when you make the migration.
>
> You can also move to the new styles in the database more directly by deleting the _style_ column from the beers and setting up the beer styles from the console, for instance.
>
> After the change has been implemented, when beers are created, their style will be chosen from a ready-made list as it is for breweries. Also add a link to the navigation bar to the styles page.
>
> An individual style's page should show list with all the beers of that style.
>
> **ATTENTION1**  If you define <code>belongs_to :style</code> for the class _Beer_ you will no longer be able to access the string type attribute \_style* with the dot-notation _beer.style_, rather you need to use _beer['style']_
>
> **ATTENTION2** make sure that it is still possible to create beers after the extention! You will have to change a few things, and maybe the most difficult to notice is the beer controller help method <code>beer_params</code>.

The beer style page will look something like this, after you have completed the exercise:

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-6.png)

A good beer style list and their descriptions is found at the address http://beeradvocate.com/beer/style/

> ## Exercise 14
>
> Saving the styles in the database will break most of the tests. Update your tests. Notice that you will have to fix also the FactoryBot factories.
>
> Even though the broken tests are many, keep calm. Solve the problems one test after the other, the same problems are accumulated in various different places and updating the tests will not be too difficult at the end.
>
> _NOTE_ You can delete Rails' auto-generated tests, e.g. the test _spec/views/styles/index.html.erb_spec.rb_

## Beer weather

> ## Exercise 15
>
> On the page showing a location's beer places add the current weather forecast for that location. There are dozens of services offering weather forecasts. Personally I used [https://weatherstack.com/](https://weatherstack.com/). Remember to handle the the API keys sensibly in your code!

After the exercise, the places page could look like

![picture](https://raw.githubusercontent.com/mluukkai/WebPalvelinohjelmointi2023/main/images/ratebeer-w5-7.png)

> ## Exercise 16
>
> Exercise 15 broke some tests. Fix them. You can mark this exercise as completed only if you also did the previous exercise

## Submitting the exercises

Commit all your changes and push the code to Github. Deploy to the newest version of Heroku or Fly.io, too. Remember to check with Rubocop that your code still adheres to style rules. 

Mark the exercises you have done at https://studies.cs.helsinki.fi/stats/courses/rails2023.

[Week 6](https://github.com/mluukkai/WebPalvelinohjelmointi2023/blob/main/english/week6.md)
