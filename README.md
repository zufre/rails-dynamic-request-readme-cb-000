# Rails Dynamic Request

## Review

You already know how to create a static request, which is where you create a page that doesn't take any parameters and simply renders a view, an example would be: ```localhost:3000/about```. For Rails to process this request, the ```route.rb``` file contains a route such as:

```ruby
get 'about', to "static#about"
```

This is mapped to the ```static``` controller and ```about``` action, which renders the ```about.html.erb``` view template.

## Dynamic Requests

If you are comfortable with how static requests you will pick up on dynamic requests quickly. An example scenario for a dynamic request would be where a user navigates to a page in the application such as ```localhost:3000/posts/42``` and is displayed a post with the id of 42. Behind the scenes what is happening is below:

1. The ```route.rb``` file takes in the request and processes it like normal, except this time it also parses the ```42``` parameter and passes it to the posts' controller.

2. From that point the controller parses the ```42``` parameter and runs a query on the Post model, storing the result in an instance variable.

3. Lastly the controller passes the instance variable to the associated view which renders that specific post record details to the client.


## Code Implementation

In order to setup a dynamic request feature we will start by writing a test to verify that the page exists (you can ignore the FactoryGirl calls, I'm simply using it to create a new post so we have an object to work with):

```ruby
# spec/features/post_spec.rb

require 'rails_helper'

feature 'navigate to post pages' do
  subject { FactoryGirl.create(:post) }

  scenario 'on the show page' do
    visit "/posts/#{subject.id}"
    expect(page.status_code).to eq(200)
  end

end
```

Running ```bundle exec rspec``` gives us an expected error of: ```ActionController::RoutingError: No route matches [GET] "/posts/1"```. To correct this error let's draw a route that maps to a show action in the posts' controller:

```ruby
get 'posts/:id', to: 'posts#show'
```

Here you will notice something that's different from the static route, the ```/:id``` tells the routing system that this route can receive a parameter, and that parameter will be passed to the controller's show action. With this route in place let's run our tests again, you will see we get a different failure this time that says: ```AbstractController::ActionNotFound: The action 'show' could not be found for PostsController```. This means that we need to create a corresponding ```show``` action in the posts' controller, let's get this failure fixed with the code below:

```ruby
# app/controllers/posts_controller.rb

class PostsController < ApplicationController
  def show
  end
end
```

Running the tests now you will get a failure saying that that we have a missing view template, let's fix that by creating a view file that is mapped to our ```show``` action by running the following terminal command (we also need to create a posts directory): ```mkdir app/views/posts && touch app/views/posts/show.html.erb```

Running the tests now and we're all green, which means that the request will be properly routes through the controller and view and returns a HTTP status code of ```200```.

Now that we have the routing configured, let's build a test to see if the post content is rendered on the show page with the title being in an ```h1``` tag and the description in a ```p``` tag, below are the scenarios (if you reference the initial spec setup, ```subject``` holds the post that we created with FactoryGirl):

```ruby

# Post factory for reference:
# title "My Great Post"
# description "My amazing description"

scenario 'title is shown on the show page in a h1 tag' do
  visit "/posts/#{subject.id}"
  expect(page).to have_css("h1", text: "My Great Post")
end

scenario 'description is shown on the show page in a p tag' do
  skip
end
```

This gives us a failure that says: ```expected to find css "h1" with text "My Great Post" but there were no matches```. To fix this we first need to capture the post's id in the controller's show action, let's do that here:

```ruby
# app/controllers/posts_controller.rb

def show
  @post = Post.find(params[:id])
end
```

In this line, our show action is running a database query on the Post model that will return a post that has the id that matches the route parameters, and it will store this record in the ```@post``` instance variable and make it available to the ```show.html.erb``` file. Let's get our spec passing by placing the post's title on the show view template:

```ERB
<% # app/views/posts/show.html.erb %>
<h1><%= @post.title %></h1>
```

We're back to all green! Now let's implement the description spec:

```ruby
scenario 'description is shown on the show page in a p tag' do
  visit "/posts/#{subject.id}"
  expect(page).to have_css("p", text: "My amazing description")
end
```
This will give us a failure since there are no matches on the template yet, to implement this fix, update the view:

```ERB
<% # app/views/posts/show.html.erb %>
<h1><%= @post.title %></h1>
<p><%= @post.description %></p>
```

Now we're all passing, you now know how to create dynamic routes in Rails! However we would be remiss if we didn't follow the full "Red, Green, Refactor" TDD workflow. There are a few elements of the application that can be refactored. First let's add some syntactic sugar to our route file.

Remove:

```ruby
get 'posts/:id', to: 'posts#show'
```

Replace with:

```ruby
resources :posts, only: :show
```

Running the tests again everything is still passing. Now let's update the specs. In each instance of:

```ruby
visit "/posts/#{subject.id}"
```

Replace that with:

```ruby
visit post_path(subject)
```

The tests are still passing and it's considered a better practice to leverage build in route methods as opposed to hard coding paths. Lastly, let's refactor the controller, our ``Post.find``` method works great, but what happens when we want to make this available to other methods in the controller? It would be better if we had a before action that will store the ```@post``` instance variable for the methods that should have it. The ```posts_controller.rb``` file should be refactored to look something like this:

```ruby
class PostsController < ApplicationController
  before_action :set_post, only: :show
  
  def show
  end

  private

    def set_post
      @post = Post.find(params[:id])
    end
end
```

Now when we add a new method that needs the ```@post``` instance variable you can simply add it to the before action list instead of having duplicate code throughout the controller.

Running the tests for a final time you can see that they're all still passing, nice work!