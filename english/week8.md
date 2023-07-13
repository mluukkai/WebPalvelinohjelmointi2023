# Hotwire

## Table of Contents

* [Introduction to Hotwire](#introduction-to-hotwire)
  * [Why Hotwire?](#why-hotwire)
* [Introduction to Hotwire Components](#introduction-to-hotwire-components)
* [Hotwire Components in Detail](#hotwire-components-in-detail)
  * [Turbo Frames](#turbo-frames)
  * [Turbo Streams](#turbo-streams)
    * [Turbo Streams Actions](#turbo-streams-actions)
    * [Turbo Streams Targets](#turbo-streams-targets)
    * [Turbo Streams Templates](#turbo-streams-templates)
    * [Dynamic Updates with ActionCable](#dynamic-updates-with-actioncable)
  * [Stimulus](#stimulus)
    * [Stimulus Controllers](#stimulus-controllers)
    * [Stimulus Actions](#stimulus-actions)
    * [Stimulus Targets](#stimulus-targets)
    * [Stimulus Values](#stimulus-values)
    * [Lifecycle Methods](#lifecycle-methods)
    * [Application Example](#application-example)
* [Exercises](#exercises)
  * [Turbo Streams Exercises](#turbo-streams-exercises)
    * [Implementing Beer Removal with Confirmation Pop-up](#implementing-beer-removal-with-confirmation-pop-up)
    * [Dynamic Updating of Active and Retired Breweries](#dynamic-updating-of-active-and-retired-breweries)
  * [Stimus Exercises](#stimulus-exercises)
* [ActionCable, Redis, and Fly.io Integration](#actioncable-redis-and-flyio-integration)

## Introduction to Hotwire

Ruby on Rails version 7.x introduces a new functionality called [Hotwire](https://hotwired.dev/), aimed at simplifying the creation of dynamic views with minimal reliance on Javascript. Hotwire empowers Rails developers to incorporate partial reloading of user interface elements in a similar fashion to popular Javascript libraries like [React](https://react.dev/), all while leveraging the familiar syntax of the Ruby language.

### Why Hotwire?

Throughout its history, the Rails framework has been renowned for its ability to enable the rapid development of __fully-featured applications__. However, over the past 15 years, the concept of a _"fully-featured application"_ has evolved significantly. Today's users have come to expect dynamic, mobile-friendly experiences that encompass faster, partial, and interactive page loads.

To meet these expectations, developers have had to rely on additional software tools, such as the React library, to build the necessary functionality. Unfortunately, this approach adds complexity to the applications and often diminishes the role of the View component within the Rails [MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) architecture, reducing it to merely serving as a [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) or [GraphQL](https://en.wikipedia.org/wiki/GraphQL) API.

With the introduction of Hotwire, Rails aims to tackle the challenges posed by the rapidly evolving Javascript landscape. This is in contrast to the more steadily-paced Ruby ecosystem, which tends to favor incremental and conservative evolution. Hotwire offers a more streamlined and cohesive approach to fulfilling the requirements of full-featured, full-stack applications. It achieves this by eliminating the reliance on disparate tools and aligning with the ethos of Rails as a comprehensive platform for web application development. Hotwire provides the tools to construct dynamic and interactive user experiences while maintaining consistency with the familiar Rails paradigms.

## Introduction to Hotwire Components

Hotwire encompasses three core components, each serving a specific purpose: Turbo, Stimulus, and Strada (not covered in this course).

1. __Turbo__

__Turbo Frames__ and __Turbo Streams__ enhance page loading speed by dividing the page into components and facilitating dynamic updates.

* __Turbo Streams over HTTP__ - Delivering updates directly through HTTP responses, reducing page reloads.
* __Turbo Streams over Action Cable__ - Real-time updates using Action Cable's WebSocket framework.
* __WebSockets with Turbo__ - Bidirectional communication for real-time updates and interactivity.

2. __Stimulus__

Stimulus is a lightweight Javascript framework that enhances interactivity and user interactions in server-rendered HTML views. By attaching JavaScript behavior to HTML elements, it improves the user experience without complex frameworks or extensive coding.

3. __Strada__

Strada is an extension of Hotwire that allows developers to build iOS and Android applications using Rails and Turbo. Currently, Strada is being developed as separate repositories: [turbo-ios](https://github.com/hotwired/turbo-ios) for iOS and [turbo-android](https://github.com/hotwired/turbo-android) for Android, respectively.

## Hotwire Components in Detail

### Turbo Frames

TBA

### Turbo Streams

The purpose of [Turbo Streams](https://turbo.hotwired.dev/handbook/streams) is to enable page updates in fragments. For example, when a page displays a list of beers, instead of performing a complete page reload, a single beer can be appended or removed from the list in response to a change.

In modern web applications, achieving this behavior often involves having a separate server-side REST API or GraphQL API, commonly referred to as the back-end, to provide the necessary information in JSON format. The front-end queries this back-end, receives the JSON data, and renders the required HTML while updating the DOM accordingly.

Turbo simplifies this process by streaming pre-rendered HTML, compiled on the back-end, directly to the browser and handling the necessary actions internally.

Key concepts in Turbo Streams include __actions__, __targets__, and __templates__. These concepts determine which action should be applied to specific target elements using specific template data.

#### Turbo Streams Actions

In Turbo Streams, __actions__ are a fundamental concept used to specify the changes or updates that should be performed on the client-side HTML DOM in response to a server-side event. An action represents a specific operation that can be applied to one or more target elements within a Turbo Stream response.

__Actions__ are defined using HTML-like syntax and consist of a combination of elements and attributes. Each action includes a target element, which represents the HTML element on the client-side that needs to be updated, and one or more operations that define how the target element should be modified.

The operations that can be applied to a target element include:

| Action    | Description |
|-----------|-------------------------------------------------------------------|
| `append`  | Appends new content after the target element.                     |
| `prepend` | Prepends new content before the target element.                   |
| `replace` | Replaces the content of the target element with new content.      |
| `update`  | Updates specific attributes or properties of the target element.  |
| `remove`  | Removes the target element from the DOM.                          |

#### Turbo Streams Targets

In order for __actions__ to function properly, Turbo requires the identification of target elements within the DOM. This can be achieved by assigning unique HTML `id` parameters to individual elements or by utilizing `class` parameters to target multiple elements.

For identifying a single element, one can explicitly create an ID value in the view or leverage the convenient Rails [dom_id](https://api.rubyonrails.org/classes/ActionView/RecordIdentifier.html) helper, which automatically generates the ID tag. For example:

```erb
<div id="<%= dom_id(brewery) %>">
  <%= brewery.name %>
</div>
```

This would result in a unique ID like:

```erb
<div id="brewery_55">Sinebrychoff</div>
```

Alternatively, when targeting __multiple elements__ based on specific [CSS class selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors), such as:

```erb
<div class="active" id="brewery_55">Sinebrychoff</div>
<div class="active" id="brewery_62">Laitilan Wirvoitusjuomatehdas</div>
<div class="retired" id="brewery_71">Pyynikin käsityöläispanimo</div>
```

you could use the remove action to remove all retired breweries from the list by targeting the class ".retired".

#### Turbo Streams Templates

To leverage the capabilities of Turbo Streams, view templates should be designed as [partials](https://guides.rubyonrails.org/layouts_and_rendering.html#using-partials) that can be rendered individually. This enables targeted streaming of changes to specific components. For example, when streaming updates for breweries and appending new breweries to a list, the brewery row should be implemented as a partial.

1. __Extracting Row Rendering Logic__

To prepare the Breweries index page for streaming, extract the row rendering logic (created in Exercise 1) from `app/views/breweries/_breweries_list.html.erb`:

**app/views/breweries/_breweries_list.html.erb**
```erb
<tbody>
  <% breweries.each do |brewery| %>
    <tr %>">
      <td><%= link_to brewery.name, brewery, data: { turbo_frame: "_top"} %></td>
      <td><%= brewery.year %></td>
      <td><%= brewery.beers.count %></td>
      <td><%= round(brewery.average_rating) %></td>
    </tr>
  <% end %>
</tbody>
```

To new partial file `app/views/breweries/_brewery_row.html.erb`:

**app/views/breweries/_brewery_row.html.erb**
```erb
<tr %>">
  <td><%= link_to brewery.name, brewery, data: { turbo_frame: "_top"} %></td>
  <td><%= brewery.year %></td>
  <td><%= brewery.beers.count %></td>
  <td><%= round(brewery.average_rating) %></td>
</tr>
```

And change the original code to use this partial :

```erb
<tbody id="<%= status %>_brewery_rows">
  <% breweries.each do |brewery| %>
    <%= render "brewery_row", brewery: brewery %>
  <% end %>
</tbody>
```

Pay attention to new ID that we give to the tbody element. We need `active_brewery_rows` or `retired_brewery_rows` ID to **target** the **action** of appending new breweries as children of the correct table. If in Exercise 1 you did not define local `status` or something similar containing `active`/`retired` information for the different brewery listings, you should do that now as it will help us later.

2. __Adding New Breweries from Index Page__

Next, let's enable the addition of new breweries directly from the index page. Replace the following code in `app/views/breweries/index.html.erb`:

**app/views/breweries/index.html.erb**
```erb
<%= link_to "New brewery", new_brewery_path if current_user %>
```

With the following Turbo Frame tag:

**app/views/breweries/index.html.erb**
```erb
<%= turbo_frame_tag "new_brewery", src: new_brewery_path if current_user %>
```

This Turbo Frame will include a part of our existing code from the `new_brewery` path.

3. __Defining Partial View for Adding New Beer__

In `app/views/breweries/_new.html.erb` specify which part of the view you want to show in the Turbo Frame:

**app/views/beers/_new.html.erb**
```erb
<h1>New brewery</h1>

<%= turbo_frame_tag "new_brewery" do %>
  <%= render "form", brewery: @brewery %>
<% end %>
# ...
```

![image](../images/ratebeer-w8-4.png)


4. __Modifying the Controller Response__

In order to append the created beer to the list without doing a full page update, we need to modify the response in the create action of the `app/controllers/breweries_controller.rb` file.

By adding the `format.turbo_stream` block, we specify that the response should be rendered as a Turbo Stream template with the action of appending the new brewery row to the target element with the ID `active_brewery_rows` or `retired_brewery_rows`. These steps enable the addition of breweries directly from the index page while only appending the created brewery to the list without refreshing the entire page.

**app/controllers/breweries_controller.rb**
```ruby
def create
  @brewery = Brewery.new(brewery_params)

  respond_to do |format|
    if @brewery.save
      format.turbo_stream {
        status = @brewery.active? ? "active" : "retired"
        render turbo_stream: turbo_stream.append("#{status}_brewery_rows", partial: "brewery_row", locals: { brewery: @brewery })
      }
      format.html { redirect_to brewery_url(@brewery), notice: "Brewery was successfully created." }
      format.json { render :show, status: :created, location: @brewery }
    else
      format.html { render :new, status: :unprocessable_entity }
      format.json { render json: @brewery.errors, status: :unprocessable_entity }
    end
  end
end
```

This process involves a simple addition of code, but several components are necessary to ensure its proper functioning:

1. When the browser initiates a new request, the Turbo framework includes the `text/vnd.turbo-stream.html` in the request's `Accept` headers. This informs the server that it expects a Turbo Stream template instead of a full page update.

![image](../images/ratebeer-w8-2.png)

2. The controller, based on the `Accept` header, recognizes the request's Turbo Stream format and responds by rendering a Turbo Stream template instead of a complete page. This ensures that only the necessary HTML fragments are sent back to the browser.

3. Using the `turbo_stream.append` method, a response HTML fragment is generated with the action set to `append`. This fragment targets the element with the identifier `brewery_rows` and utilizes the `_brewery_row.html.erb` partial to generate the content. Here is an example of the resulting fragment:

```html
<turbo-stream action="append" target="active_brewery_rows"><template><tr id="brewery_65">
  <td><a data-turbo-frame="_top" href="/breweries/65">Laitilan Wirvoitusjuomatehdas</a></td>
  <td>1995</td>
  <td>0</td>
  <td>0.0</td>
</tr></template></turbo-stream>
```

4. With the table body previously assigned an ID, such as `<tbody id="active_brewery_rows">`, Turbo knows to append the generated template as the last child of the table body element. It intelligently places the new content in the appropriate location. You can test this behavior by removing or altering the ID and observing the resulting outcome.

#### Dynamic Updates with ActionCable

[ActionCable](https://edgeguides.rubyonrails.org/action_cable_overview.html) enables dynamic updates by utilizing [WebSockets](https://en.wikipedia.org/wiki/WebSocket), providing real-time streaming of content to multiple browsers. Unlike traditional request-response logic, which requires the browser to send a request and await a response, ActionCable allows seamless updates without the need to refresh the page. This functionality has been available since Rails version 5 and is based on WebSockets technology. For a very brief tutorial on WebSockets, see [WebSockets in 100 seconds](https://www.youtube.com/watch?v=1BfCnjr_Vjg).

To establish a connection between the browser and the server for listening to changes, we define a stream or channel. In our view file, `app/views/breweries/index.html.erb`, we "subscribe" to updates using the following line of code:

**app/views/breweries/index.html.erb**
```erb
<%= turbo_stream_from "breweries_index" %>
```

This code establishes a WebSocket connection between the browser and the server, enabling the browser to receive real-time updates.

To publish updates, we utilize the Brewery model `app/models/brewery.rb`. Whenever a new brewery is created, the following code is triggered:

**app/models/brewery.rb**
```ruby

after_create_commit -> { broadcast_append_to "breweries_index", partial: "breweries/brewery_row", target: "active_brewery_rows" }, if: :active?
after_create_commit -> { broadcast_append_to "breweries_index", partial: "breweries/brewery_row", target: "retired_brewery_rows" }, if: :retired?
```

This code broadcasts an `append` action to the `breweries_index` channel, targeting the element with the ID `active_brewery_rows` or `retired_brewery_rows`. It uses the `_brewery_row.html.erb` partial to create the template for the new beer. Essentially, it replicates the same functionality we implemented earlier by responding to client requests with fragments. However, the difference lies in the fact that the HTML fragment is now broadcasted to all browsers subscribed to the `breweries_index` channel, thanks to the power of WebSockets.

You can test the functionality by opening two browser windows side by side and creating a new brewery. You'll observe that the updates are instantly reflected in both windows, demonstrating the real-time nature of ActionCable and WebSockets.

![image](../images/ratebeer-w8-3.png)

During the process, you may have noticed a problem: when a user adds new breweries, duplicate entries appear in the list. Let's take a moment to understand why this happens.

The reason is that the current user actually receives the fragments twice. Firstly, as an HTTP response to the form submission triggered by the create action in the controller. Secondly, as a WebSocket update triggered by the `after_create_commit` hook in the model.

To address this issue, there are several possible solutions:

1. Comment out the stream template in the HTTP response from the controller. However, this approach has a downside: if there are any issues with WebSockets, the user won't see the effect of submitting a new brewer.

2. Conditionally trigger the `after_create_commit` hook in the model based on the logged-in user. This approach ensures that the user only receives the WebSocket update once.

3. Opt for a simpler solution by giving each row a unique identifier. Let's proceed with this approach here.

In `app/views/breweries/_brewery_row.html`, add the following line of code:

**app/views/breweries/_brewery_row.html**
```erb
<tr id="<%= dom_id(brewery) %>">
  <td>...
```

By assigning a unique identifier to each row, we prevent duplication. This unique identifier becomes essential if we want to trigger actions, such as removing a specific brewery.

To observe the WebSocket connection details, you can use the browser's developer tools.

![image](../images/ratebeer-w8-4.png)

It's worth noting that in our example, we used a simple string, `breweries_index`, as the identifier for the channel since there is only one `breweries_index`. However, in certain scenarios, you may want to use an object to identify the stream. For instance, if you implement the ability to add new beers to a specific brewery from the Brewery page and stream the added data only to that page, you would want to use something like `@brewery` instead of `"breweries_index"`. This way, different brewery pages can be targeted with the streaming updates.

### Stimulus

[Stimulus](https://stimulus.hotwired.dev/) is a JavaScript framework designed to enhance interactivity in HTML, eliminating the need for extensive custom JavaScript development. By utilizing a set of JavaScript modules that can be seamlessly attached to HTML elements using specific attributes, Stimulus enables developers to create simpler and more maintainable code.

One of the notable advantages of Stimulus is its seamless integration with Rails, making it an ideal choice for web applications built on the Rails framework. Leveraging the latest browser technologies, Stimulus delivers a fast and seamless user experience, ensuring optimal performance.

Stimulus utilizes key concepts such as __controllers__, __actions__, __targets__, and __values__ to determine the appropriate actions to apply to specific target elements based on specific data.

__Benefits__

* Reduces the amount of custom JavaScript required to implement common functionality, streamlining development efforts.
* Provides a straightforward and intuitive API for dispatching and listening to events on webpages, simplifying event handling.
* Enhances performance by avoiding the overhead associated with full-fledged JavaScript frameworks.

__Considerations__

* Stimulus may not be the ideal choice for complex applications that demand extensive client-side functionality.
* Careful organization and structuring of Stimulus controllers is necessary to prevent bloated and unmanageable code.
* Developers who are unfamiliar with the Stimulus framework may experience a learning curve and added complexity during development.

#### Stimulus Controllers

When working with Stimulus, it is essential to follow a specific naming convention for __controller__ files. Each controller file should be named in the format `[identifier]_controller.js`, where the identifier corresponds to the data-controller attribute associated with the respective controller in your HTML markup.
By adhering to this naming convention, Stimulus can seamlessly link the controllers in your HTML with their corresponding JavaScript files.

Here is an example of a controller named `hello_controller.js` located at the file path `/app/javascript/controllers/hello_controller.js`:

**/app/javascript/controllers/hello_controller.js**
```javascript
import { Controller } from "@hotwired/stimulus";

export default class extends Controller {}
```

In the example below, a `<div>` element is associated with the `data-controller` attribute, which has the value `hello`. This binds it to the Stimulus controller named `hello_controller.js` (as shown above). It's important to note that the scope of a Stimulus controller includes the element it is connected to and its children, but not the surrounding elements. For instance, in the example below, the `<div>`, `<input>`, and `<button>` are part of the controller's scope, while the surrounding `<h1>` element is not.

```html
<h1>Greetings</h1>
<div data-controller="hello">
  <input type="text" />
  <button>Greet</button>
</div>
```

#### Stimulus Actions

__Actions__ in Stimulus are methods defined within a controller that respond to user events or changes in the application state. These actions are identified using the data-action attribute and can be triggered by various events, such as clicks, form submissions, or custom events.

For example, to add an action to a `<button>` element, you can use the data-action attribute with the format `event->controller#method`. In the following example, a click event listener is attached to the `<button>` element with the controller identifier `hello`, and it calls the `greet` method when clicked.

```html
<h1>Greetings</h1>
<div data-controller="hello">
  <input type="text" />
  <button data-action="click->hello#greet">Greet</button>
</div>
```

Stimulus provides default events for certain elements, which means that the click event can be omitted from the data-action attribute for buttons. For example, `<button data-action="hello#greet">Greet</button>` would achieve the same result.

Here is a complete list of elements and their default events:

| Element             | Default Event   |
|---------------------|-----------------|
| `a`                 | `click`         |
| `button`            | `click`         |
| `details`           | `toggle`        |
| `form`              | `submit`        |
| `input`             | `input`         |
| `input type=submit` | `click`         |
| `select`            | `change`        |
| `textarea`          | `input`         |

#### Stimulus Targets

__Targets__ in Stimulus are special attributes that allow a controller to reference and manipulate specific elements within its scope. To define targets, you need to add the `data-[controller name]-target` attribute to the HTML elements. Stimulus scans your controller class and identifies target names in the static `targets` array. It automatically adds three properties for each target name: `sourceTarget`, which evaluates to the first matching target element, `sourceTargets`, which evaluates to an array of all matching target elements, and `hasSourceTarget`, which returns boolean value `true` or `false` depending on the presence of a matching target.

```html
<h1>Greetings</h1>
<div data-controller="hello">
  <input data-hello-target="name" type="text" />
  <button data-action="hello#greet">Greet</button>
</div>
```

To add the target's name to the controller's list of target definitions, you need to update the `hello_controller.js` file accordingly. This will automatically create a property named `nameTarget` that returns the first matching target element. You can then use this property to read the value of the element and build the greeting string.

**/app/javascript/controllers/hello_controller.js**
```javascript
import { Controller } from "@hotwired/stimulus";

export default class extends Controller {
  static targets = ["name"];

  greet() {
    const name = this.nameTarget.value;
    console.log(`Hello, ${name}!`);
  }
}
```

#### Stimulus Values

__Values__ in Stimulus are a way to store and access data within a controller using the `value` method. These values can be declared in various ways, such as static values defined in the controller, attributes on HTML elements, or dynamic values. 

In the example below, the attribute `data-hello-greeting-value` is used to add a value to an HTML element. In this case, the value is set to `Welcome`.

```html
<h1>Greetings</h1>
<div data-controller="hello" data-hello-greeting-value="Welcome">
  <input data-hello-target="name" type="text" />
  <button data-action="hello#greet">Greet</button>
</div>
```

In the controller file `hello_controller.js`, a static values array is created, including the attribute name `greeting` with a type of `String`. By adding the attribute `data-hello-greeting-value="Welcome"` to the `<div>` element, the value `Welcome` is assigned to the `greetingValue` variable in the controller. This value can then be accessed and used within the controller's code.

**/app/javascript/controllers/hello_controller.js**
```javascript
import { Controller } from "@hotwired/stimulus";

export default class extends Controller {
  static targets = ["name"];
  static values = { greeting: String };

  greet() {
    const name = this.nameTarget.value;
    console.log(`${this.greetingValue}, ${name}!`);
  }
}
```

### Lifecycle Methods

Lifecycle methods in Stimulus provide a capability for executing code at specific stages in the lifecycle of a controller. These methods, defined within the controller class, offer hooks for initialization, connection to the DOM, and disconnection from the DOM.

Here is an example showcasing the available lifecycle methods in a Stimulus controller:

```javascript
import { Controller } from 'stimulus';

export default class extends Controller {
  initialize() {
    // Executed once when the controller is first instantiated
  }

  [name]TargetConnected(target) {
    // Executed anytime a target is connected to the DOM
  }

  [name]TargetDisconnected(target) {
    // Executed anytime a target is disconnected from the DOM
  }

  connect() {
    // Executed when the controller is connected to the DOM
  }

  disconnect() {
    // Executed when the controller is disconnected from the DOM
  }
}
```

These lifecycle methods provide developers with the flexibility to perform specific tasks at appropriate points in the controller's lifecycle. The `initialize` method can be used for initial setup or one-time actions. The `[name]TargetConnected` and `[name]TargetDisconnected` methods enable dynamic handling of targets as they are connected or disconnected from the DOM. The `connect` and `disconnect` methods offer a convenient way to set up or clean up resources associated with the controller when it is connected or disconnected from the DOM.

By leveraging these lifecycle methods, developers can ensure proper initialization, respond to changes in DOM state, and maintain organized and maintainable code in their Stimulus controllers.

### Application Example

Let's enhance the functionality of deleting user ratings by utilizing Stimulus to enable the simultaneous deletion of multiple ratings, offering a more efficient process compared to deleting them one by one.

1. Create a new partial file named `_ratings.html.erb` within the `/app/views/users` folder. Next, extract the ratings code section (shown below) from the `/app/views/users/show.html.erb` file and place it into the ratings partial file (`/app/views/users/_ratings.html.erb`).

**/app/views/users/_ratings.html.erb**
```erb
<ul>
  <% @user.ratings.each do |rating| %>
    <li>
      <%= "#{rating.score} #{rating.beer.name}" %>
      <% if @user == current_user %>
        <%= button_to 'delete', rating, method: :delete, form: { style:'display:inline-block;', data: { 'turbo-confirm': 'Are you sure?' } } %>
      <% end %>
    </li>
  <% end %>
</ul>
```

2. Delete the ratings code from the `/app/views/users/show.html.erb` file and insert the ratings partial using the `<%= render partial: 'ratings' %>` statement under the ratings header.

**/app/views/users/show.html.erb**
```erb
<h4>Ratings</h4>
<%= render partial: 'ratings' %>
```

3. Modify the partial by removing the list elements and delete button from the `/app/views/users/_ratings.html.erb` file.

**/app/views/users/_ratings.html.erb**
```erb
<div data-controller="rating" class="ratings mb-4">
  <% @user.ratings.each do |rating| %>
    <div class="rating">
      <% if @user == current_user %>
        <input type="checkbox" name="ratings[]" value="<%= rating.id %>" />
      <% end %>
      <span><%= "#{rating.score} #{rating.beer.name}" %></span>
    </div>
  <% end %>
  <% if @user == current_user %>
    <button data-action="rating#destroy">Delete selected</button>
  <% end %>
</div>
```

4. With the modified templates ready, let's update the `routes.rb` file (`/app/config/routes.rb`) to handle the ratings destroy action. Remove the `destroy` action from the ratings resources and add a separate delete method to handle the removal of rating IDs.

**/app/config/routes.rb**
```ruby
resources :ratings, only: [:index, :new, :create]
delete 'ratings', to: 'ratings#destroy'
```

5. Modify the `destroy` method within the `ratings_controller.rb` file (`app/controllers/ratings_controller.rb`) to handle the deletion of multiple rating IDs. Retrieve the rating IDs from the request body, loop through each ID, find the respective rating, and destroy it if the current user is the author of the rating. Implement a `rescue` block to handle any potential errors during the loop. After the destruction, ensure the user data is refreshed to reflect the changes.

**app/controllers/ratings_controller.rb**
```ruby
def destroy
  destroy_ids = request.body.string.split(',')
  destroy_ids.each do |id|
    rating = Rating.find_by(id: id)
    rating.destroy if rating && current_user == rating.user
  rescue StandardError => e
    puts "Rating record has an error: #{e.message}"
  end
  @user = current_user
  respond_to do |format|
    format.html { render partial: '/users/ratings', status: :ok, user: @user }
  end
end
```

6. Create the Stimulus ratings controller file `ratings_controller.js` within the `/app/javascript/controllers/` folder. Import the `Controller` class from the `@hotwired/stimulus` module and define a class that extends the `Controller` class. Within this class, implement the destroy method. Begin by displaying a confirmation dialog to the user. If the user confirms the deletion, retrieve the selected rating IDs from the checkboxes, include the CSRF token in the request headers, and initiate a `fetch` request to the `/ratings` endpoint with the selected rating IDs as the request body. Handle the response accordingly, updating the HTML of the ratings element if the response is successful, and logging any errors that occur during the process.

**/app/javascript/controllers/ratings_controller.js**
```javascript
import { Controller } from "@hotwired/stimulus";

export default class extends Controller {
  destroy() {
    const confirmDelete = confirm("Are you sure you want to delete these selected ratings?");
    if (!confirmDelete) {
      return;
    }

    const selectedRatingsIDs = Array.from(
      document.querySelectorAll('input[name="ratings[]"]:checked'),
      (checkbox) => checkbox.value
    );

    const csrfToken = document.querySelector('meta[name="csrf-token"]').content;
    const headers = { "X-CSRF-Token": csrfToken };

    fetch("/ratings", {
      method: "DELETE",
      headers: headers,
      body: selectedRatingsIDs.join(","),
    })
      .then((response) => {
        if (response.ok) {
          response.text().then((html) => {
            document.querySelector("div.ratings").innerHTML = html;
          });
        } else {
          throw new Error("Something went wrong");
        }
      })
      .catch((error) => {
        console.log(error);
      });
  }
}
```

## Exercises

### Turbo Streams Exercises

#### Implementing Brewery Removal with Confirmation Pop-up

Enhance the breweries list functionality by adding a button or text "X" for removing a brewery from the database (see [Rails views documentation](https://guides.rubyonrails.org/layouts_and_rendering.html#rendering-by-default-convention-over-configuration-in-action)). The implementation should follow these steps:

1. __Initial Removal (No Turbo, Full Page Reload)__
Initially, make the removal work without using Turbo, requiring a full page reload after the delete action.

2. __Dynamic Removal with Turbo Streams__
Improve the functionality by dynamically removing the deleted brewery from the list using Turbo Streams. Ensure the removal is reflected in the UI without requiring a full page reload.

3. __WebSocket Integration for Real-Time Updates__
Leverage WebSockets to stream the removal action to all connected browsers in real time.

4. __Confirmation Pop-up__
Enhance user experience by introducing a confirmation pop-up. When a user clicks the remove button, a confirmation dialog should appear with the text "Are you sure you want to remove brewery X and all beers associated with it?". The pop-up should provide options for "Cancel" and "Remove" actions.

#### Dynamic Updating of Active and Retired Breweries

Notice that _Number of Active Breweries_ and _Number of Retired Breweries_ require a full page reload to reflect the actual numbers. Make these numbers dynamic so that any addition or retirement of a brewery by any user triggers real-time updates. The changes should be streamed to reflect the updated counts instantly.

Hint: you can render multiple turbo stream messages from a controller response by placing them in an array.

### Stimulus Exercises

TBA

## ActionCable, Redis, and Fly.io Integration

To ensure seamless operation of __ActionCable__, the foundational component for Turbo streams, in a production environment, it is necessary to have [Redis](https://redis.io/) installed. Please follow the steps outlined below to ensure a proper configuration.

1. __Verify Redis Installation__

Check if you already have a Redis instance by running the following command in your terminal:

`$ fly redis list`

If no Redis instance is listed or if you encountered issues during the installation process (e.g., failure during `fly launch`), proceed to create a Redis instance using the command:

`$ fly redis create`

2. __Fetch Redis Address and Configure the Server__

Retrieve the Redis address by executing the following command in your terminal:

`$ fly redis status <your-redis-instance-name>`

This command will provide detailed information, including a line displaying the private URL in the format: `Private URL = redis://default:somethingsomething...something.upstash.io`. Configure the server by setting the Redis URL as a secret. Execute the command:

`$ fly secrets set REDIS_URL=redis://default:somethingsomething...something.upstash.io`

3. __Gemfile Configuration__

By default, Fly.io configures your `Gemfile` with a Redis gem version higher than 5. However, if you are running a Rails version lower than 7.0.4, you will need an older version of the gem. Update your `Gemfile` with the following Redis configuration:

```ruby
gem "redis", ">= 3", "< 5"
```
