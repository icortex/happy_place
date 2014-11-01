happy_place
============
| Project                 |  Gem Release      |
|------------------------ | ----------------- |
| Gem name                |  happy_place      |
| License                 |  [MIT](LICENSE.txt)   |
| Version                 |  [![Gem Version](https://badge.fury.io/rb/happy_place.png)](http://badge.fury.io/rb/happy_place) |
| Continuous Integration  |  [![Build Status](https://travis-ci.org/mfpiccolo/happy_place.png?branch=master)](https://travis-ci.org/mfpiccolo/happy_place)
| Test Coverage           |  [![Coverage Status](https://coveralls.io/repos/mfpiccolo/happy_place/badge.png?branch=master)](https://coveralls.io/r/mfpiccolo/happy_place?branch=coveralls)
| Grade                   |  [![Code Climate](https://codeclimate.com/github/mfpiccolo/happy_place.png)](https://codeclimate.com/github/mfpiccolo/happy_place)
| Dependencies            |  [![Dependency Status](https://gemnasium.com/mfpiccolo/happy_place.png)](https://gemnasium.com/mfpiccolo/happy_place)
| Homepage                |  [http://mfpiccolo.github.io/happy_place][homepage] |
| Documentation           |  [http://rdoc.info/github/mfpiccolo/happy_place/frames][documentation] |
| Issues                  |  [https://github.com/mfpiccolo/happy_place/issues][issues] |

## Description

A happy place for javascript in Rails

## Installation

Add this line to your application's Gemfile:

```ruby
gem "happy_place"
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install happy_place

## Features
happy_place adds a js method to your controller.  This method accepts the following keyword arguments:

`js_class:` String name of the js class that you want to user

`function:` Sting of the name of the function you would like to call

`partial:` String of the partial name and path that you would like to render

`args:` Hash of arguments that you would like to be available in your js function

Both partial and keys passed from args will be available in your js function by accessing the object passed in.

```ruby
class ExampleContorller
  def index
    @examples = Example.all

    respond_to do |format|
      format.js {
        js(js_class: "ExampleClass",
           function: "doSomething",
           partial:  "some_partial",
           args:     {first_id: @examples.first.id}
        )
      }
      format.html
    end
  end
end
```

```coffeescript
class this.ExampleClass
  constructor: ->

  @doSomething: (args) ->
    html_to_append = args.partial
    first_id = args.first_id
    alert(html_to_append)
    alert(first_id)
```

happy_place will infer the js_class and function name if you name it the same as your controller and action.

```ruby
class ExampleContorller
  def index
    @examples = Example.all

    respond_to do |format|
      format.js {
        js(
           partial:  "some_partial",
           args:     {first_id: @examples.first.id}
        )
      }
      format.html
    end
  end
end
```

```coffeescript
class this.ExampleController
  constructor: ->

  @index: (args) ->
    html_to_append = args.partial
    first_id = args.first_id
    alert(html_to_append)
    alert(first_id)
```

If your controller action is only ever going to be hit by one format type or you want the same js to run for both html or js formats then you don't need the responds to and you can use the js method directly in the action definition.

Also if you do not need to pass along args or a partial then you can simply call js

```ruby
class ExampleContorller
  def index
    @examples = Example.all

    js
  end
end
```
```coffeescript
class this.ExampleController
  constructor: ->

  @index: ->
    alert "Huzza!)
```

## Naming and Directory Structure
Technically you can put your code anywhere you want but to make it to your happy place you should follow the naming and directory structure used by rails.

If you are adding code that is conroller and action specific, then add a directory called controllers in your `app/assets/javascripts` directory.  If your controllers are namespaced then namespace them just like you do in your rails controllers.  Here is an example of a namespaced coffee class:

```coffeescript
# app/assets/javascripts/controllers/admin/special/orders_controller
this.Admin or= {};
Admin.Special or= {};

class this.Admin.Special.OrdersController
  constructor: ->

  @index: (args) ->
    alert("Do some js stuff here...")
```

Make note of the or=.  This is to make sure that you don't overwrite the js object if it already exists.

Use this same naming and directory structure for all your js.  If you are creating service objects then put them in `app/assets/javascripts/services`

Remember to add your paths to the manifest so sprockets can load them:

```
//= require_tree ./controllers
//= require_tree ./services
```

Or require them explicitly:

`//= require controllers/admin/special/orders_controller`


## Example Usecases

### Stop using js views after remote actions!

Lets say you have a blog where you can see a list of posts. (Imagine that!)  You use the Posts#index to display this and it is loaded noramally with :html.

```html
<!-- posts/index.html.erb -->
<div class="posts-table">
    <%= render partial: "posts_table" %>
</div>
```

```html
<!-- posts/_posts_table.html.erb -->
<table>
  <thead>
    <tr>
      <th>ID</th>
      <th>Name</th>
      <th>Description</th>
      <th>Content</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <% @posts.each do |p| %>
      <tr>
        <td><%= p.id %></td>
        <td><%= p.name %></td>
        <td><%= p.description %></td>
        <td><%= p.state.humanize.titlecase %></td>
        <td class="button-column">
          <%= link_to post_path(p) %>
        </td>
      </tr>
    <% end %>
  </tbody>
</table>
```

However you want to be able to updat the posts table with a filter that uses ajax.  Normally, you would create view of `views/posts/index.coffee` with the following code:

```coffee
# views/posts/index.coffee
$(".posts-table).html("""<%=j render(partial: "/posts_table", posts: @posts ) %>""");
```

I really do not like this patern.  This leaves your you javascript spread out all over your application and it is hard to keep track your apps workflow.

Here is my happy place for this

Step 1.  Add the gem to gemfile and bundle

Step 2. In the controller action you are interested in handling with js:

```ruby
class PostsController
  def index
    @posts = Posts.all

    respond_to do |format|
      format.js { js partial: "posts" }
      format.html
    end
  end
end
```

Step 3.  in `app/assets/javascripts/controllers/posts_controller.coffee`

```ruby
class this.PostsController
  constructor: ->

  @index: (args) ->
    $(".posts_table").html(args.partial);
```

Step 4.  Add `assets/controllers` to your manifest application.js

```
//= require_tree ./controllers
```

Step 5.  Rejoice!

## Donating
Support this project and [others by mfpiccolo][gittip-mfpiccolo] via [gittip][gittip-mfpiccolo].

[gittip-mfpiccolo]: https://www.gittip.com/mfpiccolo/

## Copyright

Copyright (c) 2014 Mike Piccolo

See [LICENSE.txt](LICENSE.txt) for details.

## Contributing

1. Fork it ( http://github.com/mfpiccolo/happy_place/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

[![githalytics.com alpha](https://cruel-carlota.pagodabox.com/e1a155a07163d56ca0c4f246c7aa8766 "githalytics.com")](http://githalytics.com/mfpiccolo/happy_place)

[license]: https://github.com/mfpiccolo/happy_place/MIT-LICENSE
[homepage]: http://mfpiccolo.github.io/happy_place
[documentation]: http://rdoc.info/github/mfpiccolo/happy_place/frames
[issues]: https://github.com/mfpiccolo/happy_place/issues

