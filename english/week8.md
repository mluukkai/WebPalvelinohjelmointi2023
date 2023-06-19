## Hotwire

Rails 7 introduces Hotwire to help creating dynamic views in Rails easier, with minimal Javascript.

**What problem do Hotwire techniques intend to solve?**

The real benefit of Rails in the past has been that it allows creating **full applications** quickly. However, "Full application" definition has changed in 15 years. Users are accustomed to having dynamic, mobile friendly experiences: faster, partial, dynamic page loads and interactivity.

To deliver, developers have needed other software tools such as ReactJS library to create the services. This adds to the complexity of the apps and also diminshes the View part of RAils MVC into a REST/GraphQL API.

So with the introduction of Hotwire, Rails sets out to handle the complexities of rapidly changing Javascript world and provide a more uniform way of handling needs of Full-stack applications.

**What does Hotwire include:**

- Turbo - speed up pages by dividing page into components and make dynamic updates
- Stimulus - allow Javascript functionality with minimal Javascript
- Strada (not yet available as a framework and not part of this course) - building iOS and Anrdoid applications using Rails and Turbo
  - Strada functionality is currently being developed as [turbo-ios](https://github.com/hotwired/turbo-ios) and [turbo-android](https://github.com/hotwired/turbo-android)

## Turbo Frames here...



## Turbo Streams

Read [documentation for Turbo streams](https://turbo.hotwired.dev/handbook/streams).

**The point of Turbo streams** is to allow page updates in fragments. For example, if you have a page with a list of beers, you can append or remove just a single beer from the list in response to a change, instead of having to do a full page reload.

In modern web applications this kind of behavior is usually achieved by using a server-side REST API or GraphQL API that provides the needed information in JSON format. Then frontend side queries for the information, receives the JSON and uses that to render the needed HTML and updates the DOM accordingly. 

Turbo intends to do this more simply, by streaming ready-made HTML to the browser and handling the required actions under the hood.

Key concepts here are **Actions**, **Targets** and **Templates**, ie. What action should be done to which target elements using what template data.

## Actions

**Actions** define what should be done to a page element. Turbo Streams supports 7 different actions:

- `append`/`prepend` element to a collection (f.ex. add beer to list)
- Insert element `before`/`after` another element
- `remove` element
- `update`/`replace` element (almost same, but update will do the substitution more subtly)

## Identifying targets in the view

For actions to work, Turbo needs to know which part of the webpage the action is targeting so you need to identify the page elements in views accordingly. Targeting can be done either to single element or to multiple elements.

For a **single element**, you need a unique ID that can be used to identify the element. You can achieve this by explicitly creating the id value in the view, or using Turbo frame tag, but most convenient way is to use the Rails [dom_id](https://api.rubyonrails.org/classes/ActionView/RecordIdentifier.html) helper to autogenerate the tag.

For example:

```erb
<div id="<%= dom_id beer %>">
  <%= beer.name %>
</div>
```

This would result in a unique id like:

```
<div id="beer_55">Karhu</div>
```

You can also target **multiple elements** by using [css selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors). For example, if your beers also have class elements based on style, such as:

```
<div class="lager" id="beer_55">Karhu</div>
<div class="ipa" id="beer_62">S-marketin ipa</div>
<div class="lager" id="beer_71">Aura</div>
```

you could use the remove action to remove all lagers from the list by targeting the class ".lager".

## Design templates as partials

To make use of Turbo streams, you must design view templates as partials that can be rendered individually. If you want to stream changes to beers, such as append new beer rows to a list of beers, then beer row should be a partial.

Let's now make the Beers index page ready for streaming.

Extract your row rendering logic in `app/views/beers/index.html.erb` from:

```erb
<tbody>
  <% @beers.each do |beer| %>
     <tr>
  	  <td><%= link_to beer.name, beer %></td>
	  <td><%= link_to beer.style.name, beer.style %></td>
	  <td><%= link_to beer.brewery.name, beer.brewery %></td>
	  <td><%= round(beer.average_rating) %></td>
    </tr>
  <% end %>
</tbody>

```

To new partial file `app/views/beers/_beer_row.html.erb`:

```erb
<tr>
  <td><%= link_to beer.name, beer %></td>
  <td><%= link_to beer.style.name, beer.style %></td>
  <td><%= link_to beer.brewery.name, beer.brewery %></td>
  <td><%= round(beer.average_rating) %></td>
</tr>

```

And change the original code to use this partial :

```erb
<tbody id="beer_rows">
  <% @beers.each do |beer| %>
    <%= render partial: "beer_row", locals: {beer: beer} %>
  <% end %>
</tbody>
```

Pay attention to new ID that we give to the table. We need `beer_rows` to **target** the **action** of appending new beers as children of the table. 

Now let's allow adding new beers directly from index page. Define a Turbo Frame to include a part of our already existing code in the index page `app/views/beers/index.html.erb`

Substitute:

```
<%= link_to('New Beer', new_beer_path) if current_user %>
```

With:

```
<% if current_user %>
  <%= turbo_frame_tag "new_beer", src: new_beer_path %>
<% end %>
```

And in `app/views/beers/_new.html.erb` define which part of the view you want to show in the frame:

```
<h1>New beer</h1>

<%= turbo_frame_tag "new_beer" do %>
  <%= render "form", beer: @beer %>
<% end %>

...
```

![kuva](../images/ratebeer-w8-1.png)

This allows us to add beers directly from the index. But on closer look, adding a beer does a full update of the page. That is not want we want. To only append the created beer to the list and keep the rest of the page as it was, we need to change the response in controller `app/controllers/beers_controller.rb`:

```ruby
def create
	@beer = Beer.new(beer_params)

	respond_to do |format|
		if @beer.save
			format.turbo_stream { render turbo_stream: turbo_stream.append("beer_rows", partial: "beer_row", locals: { beer: @beer }) }
			format.html { redirect_to beers_path, notice: "Beer was successfully created." }
			format.json { render :show, status: :created, location: @beer }
		else
			format.html { render :new, status: :unprocessable_entity }
			format.json { render json: @beer.errors, status: :unprocessable_entity }
		end
	end
end
```

This is only adding one line of code. But a lot of things are needed for this to work:

(1) When browser creates the new request, Turbo framework adds `text/vnd.turbo-stream.html` to the request's `Accept` headers.

![kuva](../images/ratebeer-w8-2.png)

(2) Based on the Accept header, the controller knows to only return a turbo_stream template instead of a full page update

(3) `turbo_stream.append` renders a response HMTL fragment with action "append", targeting "beer_rows" and using partial `_beer_row.html.erb`, resulting in following fragment:

```
<turbo-stream action="append" target="beer_rows">
  <template>
    <tr id="beer_7">
      <td><a href="/beers/7">Fancy beer</a></td>
      <td><a href="/styles/1">Bulk</a></td>
      <td><a href="/breweries/1">Iso Panimo</a></td>
      <td>0.0</td>
    </tr>
  </template>
</turbo-stream>"
```

(4) Based on the ID we added to the table body earlier `<tbody id="beer_rows">`, Turbo knows to append the template as the last child of the table body. To test, try removing or changing the ID and see what happens.

## ActionCable provides dynamic updates

So far, we have been using Turbo Streams to update page fragments using the normal HTTP request-response logic. Browser sends a request that accepts turbo_stream as a response. This works fine and keeps the flow dynamic and ligthweight for a single user.

What if we want to stream the fragment to all browsers that are currently viewing the beers index without them needing to hit refresh? This has been quite simple since Rails version 5 which introduced ActionCable, a Rails way of using WebSockets (see [WebSockets in 100 seconds](https://www.youtube.com/watch?v=1BfCnjr_Vjg) for a very quick overview and [Rails ActionCable documentation](https://edgeguides.rubyonrails.org/action_cable_overview.html) for more details).

In order for our browser to start listening to changes, we need to define a stream/channel we want to listen to. In the top of our view `app/views/beers/index.html.erb` we want to "subscribe" to the updates by declaring:

```
<%= turbo_stream_from "beer_index" %>
```

This establishes a WebSocket connection between the browser and the server.

And in order to publish updates, we can do this in our Beer model `app/models/beer.rb`

```
after_create_commit -> { broadcast_append_to "beer_index", partial: "beers/beer_row", target: "beer_rows" }

```

This means that each time a new beer is created, an `append` action is triggered, broadcasted to channel `beer_index`, targeting element with ID `beer_rows` and using the `_beer_row.html.erb` partial to create the template. This is almost exactly the same thing that we did earlier by responding to a client request with the fragment. The difference is that the HTML fragment is broadcasted to all browsers subscibed to this channel. And the fact that it is using Websockets for communication. You can verify it (almost) works by opening two browser windows side by side and creating new a new beer.

![kuva](../images/ratebeer-w8-3.png)

As you notice, we have a problem. The user who is adding new beers, gets duplicate entries on the list. Pause here for a moment and think why this happens.

The reason is because the current user actually receives the fragments twice. Once as a HTTP response to the form submission, triggered by the create-action in controller. And another time as a WebSocket update, triggered by the after_create_commit in the model.

There are multiple ways to solve this.
1. The stream template in HTTP response could be commented out from the controller. The downside is that if there are problems with WebSockets, then the user does not see the effect of submitting a new beer.
2. Or we might check who the logged in user is and only conditionally trigger the after_create_commit hook from the model.
3. Or use the simplest way by giving each row a unique identifier. Let's go with that solution here:

In `app/views/beers/_beer_row.html`:

```erb
<tr id="<%= dom_id beer %>">
  <td>...
  
```

This way each row has a unique identifier which prevents the duplication. This unique identifier is actually required if we would like to, for example, trigger a remove action to a beer.

You can observer the WebSocket connection details using browser developer tools:
![kuva](../images/ratebeer-w8-4.png)

Note that we used a simple string "beer_index" to create a unique identifier for the channel. That was sufficient since there is only one beer_index. But in some cases we would like to use an object to identify the stream. For example, if we would implement ability to add new beers to a particular brewery from the Brewery page and stream the added data only to that page, we would want to use something like  `@brewery`instead of `"beer_index"`. This way, multiple users could be on different brewery pages and streaming would be able to target those pages. 


## Exercise 1

Change the beers list to include a button "X" (or text "remove") that will remove the beer from the database. 

a) Make it work first without using Turbo, allowing a full page reload after delete.
b) Then implement dynamically removing the deleted beer from the list by using Turbo stream from controller.
c) Then use WebSockets to stream the removal to all browsers.
d) And finish it off by introducing a confirmation popup. When user clicks the remove button, a confirmation with text "Are you sure you want to remove beer X?" should pop up, with options "Cancel" and "Remove".

## Exercise 2

In the front page, there are "Number of active breweries" and "Number of retired breweries". Make these numbers dynamic, so that if any user adds or retires a brewery, the changes are streamed to reflect these numbers in real time.


## ActionCable, Redis and Fly.io

In production environment, ActionCable that provides the base for working with Turbo streams requires redis. So you should have redis installed.

If you don't alread have a redis instance, or if there were issues installing redis (it might fail on running fly launch), you should create it from command line:
```
# Check if you already have an instance
fly redis list

# If not, create one:
fly redis create
```

And fetch the redis address from the console. Then configure the server:
```
fly redis status <your-redis-instance-name>
# Will give detailed information, including a line:
# Private URL = redis://default:somethingsomething...something.upstash.io

fly secrets set REDIS_URL=redis://default:somethingsomething...something.upstash.io
```

By default fly configures your gemfile with redis gem version higher than 5. If you are running Rails version lower than 7.0.4,, you need an older version of the gem. So in your Gemfile, replace the redis config with:

```
gem "redis", ">= 3", "< 5"
```
