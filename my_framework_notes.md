

**Project**

- A collection of files used to develop, test, build, and distribute software

- Dependancy Manager 

  - Bundler which uses `Gemfile` and `Gemfile.lock`

- **Gemfile **

  - ```
    source 'https://rubygems.org'
    
    ruby '3.1.0'
    
    gem 'minitest', '~> 5.10'
    gem 'minitest-reporters', '~> 1.1'
    gem 'rake'
    ```

  - Run `bundle install` to create a Gemfile.lock file 

  - Add `require 'bundler/setup'` to each ruby file

- **Rakefile**

  - It uses a DSL supplied by Rake, but it is just basic Ruby code

  - `bundle exec rake -T`- displays all taks associated with Rakefile

  - `bundle exec rake hello`- runs whats in the `hello` task

  - ```
    require "rake/testtask"
    require "bundler/gem_tasks"
    
    desc 'Say hello'
    task :hello do
      puts "Hello there. This is the 'hello' task."
    end
    
    desc 'Run tests'
    task :default => :test
    
    Rake::TestTask.new(:test) do |t|
      t.libs << "test"
      t.libs << "lib"
      t.test_files = FileList['test/**/*_test.rb']
    end
    ```

- **Gemspec**

  - `project.gemspec` file

  - ```
    Gem::Specification.new do |s|
      s.name        = 'todolist_project'
      s.version     = '1.0.0'
      s.summary     = 'Todo List Manager!'
      s.description = 'This is a simple todo list manager!'
      s.authors     = ['Pete Williams']
      s.email       = 'pw@example.com'
      s.homepage    = 'http://example.com/todolist_project'
      s.files       = ['lib/todolist_project.rb', 'test/todolist_project_test.rb']
    end
    ```

    




**Servers**

![Zoomed in Server](https://da77jsbdz4r05.cloudfront.net/images/handling_requests_manually/server-zoom-web-app-data.png)

**HTTP request**

1) request line - (GET/POST + path + HTTP version)
2) headers - Metadata (ex/ content-type)
3) message body - Optional

**HTTP response**

1. Status Code (HTTP/1.1 200 OK)
2. Headers
3. Body

**SUMMARY**

- Although it is not something you'd normally do, it is possible to interact with HTTP manually because it is a *text-based protocol*.

- HTTP is built on top of *TCP*, which is a protocol of the transport layer and handles reliable communication between two computers.

- *URLs* are made up of many components: a *scheme*, a *host*, a *port*, a *path*, and *parameters*.

- *Query parameters* are parameters that are included in a URL. They are appended to the path using `?`. Parameters are specified in the URL using the form `key=value`.

- Parameters after the first are appended to the URL using `&`.

- HTTP is stateless, which means that each request is handled separately by the server. By carefully crafting URLs and parameters, stateful interactions can be built on top of HTTP.

  

**Web Server Examples**

- Webrick, Thin, PUMA, PHUSION

**Rack**

![Server Rack](https://da77jsbdz4r05.cloudfront.net/images/working_with_sinatra/server-zoom-rack.png)

- Rack is a generic interface to help application developers connect to web servers, so it works with many other servers besides WEBrick

- **Abstracts low-level tasks** like parsing information from `HTTP` requests and formatting the return value of your application back into an `HTTP`response that the server can understand. Applications can then be fully focused on business logic .

- **Generalizes application-to-server communication**. Rack-based apps — whether they’re written using Ruby, Sinatra, Rails, or another framework — can establish socket connections using any Rack-supported servers, whether that’s WEBrick, Puma, Passenger, etc..

- **Provides an architecture for using modular pieces of functionality**. Rack middlewares give the web developer a low-level opportunity to implement all kinds of functionalities by creating a middleware stack of chained Rack-based applications as part of the their application.

- *What does Rack give to the Web Application?*

  - Rack takes plain text HTTP requests and formats them into Ruby data structures that can be easily read by your application

- *What does Rack want from a Web Application?*

  - Takes the return value from your application and formats it into an HTTP-compliant text to be sent back to the client as an HTTP response
  - Application must define a `call(env`) method and it must return an Array object with 3 items:
    - A String or Integer object that represents the Status Code
    - A Hash object with key-value pairs of content headers and their values
    - An object that responds to the `each` method and holds the body of the response
  - What is `env`?
    - It is a Hash object with all the info about an incoming request that the app or framework can use
    - Comes from HTTP request, server, and Rack 

- **What makes a Rack App?**

  - `Config.ru` rackup file- tells us what to run and how to run it 

    - ```
      my_framework/
       ├── Gemfile
       ├── Gemfile.lock
       ├── config.ru
       ├── hello_world.rb
       ├── advice.rb
       └── views/
             └── index.erb
      ```

    - ```
      # hello_world.rb
      class HelloWorld
        def call(env)
          case env['REQUEST_PATH']
          when '/'
            template = File.read("views/index.erb")
            content = ERB.new(template)
            ['200', {"Content-Type" => "text/html"}, [content.result]]
          when '/advice'
            piece_of_advice = Advice.new.generate
            [
              '200',
              {"Content-Type" => 'text/html'},
              ["<html><body><b><em>#{piece_of_advice}</em></b></body></html>"]
            ]
          else
            [
              '404',
              {"Content-Type" => 'text/html', "Content-Length" => '48'},
              ["<html><body><h4>404 Not Found</h4></body></html>"]
            ]
          end
        end
      end
      ```

    - ```
      # config.ru
      require_relative 'hello_world'
      run HelloWorld.new
      ```

    - Start Rack Application `$ bundle exec rackup config.ru -p 9595`

  - `call(env)` method - This example doesn't use `config.ru` file 

```
require 'rack'
class MyApp
  def call(env)
    body = "<h2>Hello in Style!</h2>"
    ['200', { "content-type" => "text/html" }, [body]]
  end
end
Rack::Handler::WEBrick.run MyApp.new
```

`ruby myapp.rb`

`http://www.localhost:8080`



**Sinatra**

- Web development Framework.. another example is Rails

- Productivity tool meant to help developers speed up development by automating common tasks

- In Gemfile 

  - ```
    source "https://rubygems.org"
    
    ruby '3.2.2'
    
    group :development do  #If you were using PUMA and want to use webrick
      gem 'webrick'
    end
    
    gem "sinatra"
    gem "sinatra-contrib"
    gem "erubis"
    gem "rack-test"
    gem "minitest"
    gem "redcarpet"
    gem "bcrypt"
    gem "pg"
    ```

    

- ```
  require "sinatra"
  require "sinatra/reloader"
  require "tilt/erubis"
  
  configure do    #session is a hash - access through session[:key]
  	enable :sessions
  	set :session_secret, 'secret'
      set :erb, :escape_html => true
  end 
  
  configure do   #Escape user html
    set :erb, :escape_html => true
  end
  
  before do
    @contents = File.readlines("data/toc.txt")
  end
  
  helpers do (helpers are for view templates)
    def slugify(text)
      text.downcase.gsub(/\s+/, "-").gsub(/[^\w-]/, "")
    end
  end
  
  get "/" do
    erb :home
  end 
  
  get "/chapters/:number" do 
    #We can access this through #params[:number]
    #Or @number = params[:number] *Now we can access @number in ERB
  end 
  
  not_found do
    redirect "/a/good/path"
  end
  ```

- **Run Sinatra App**

`$ bundle exec ruby book_viewer.rb`

`[http://localhost:4567](http://localhost:4567/)`

![img](https://da77jsbdz4r05.cloudfront.net/images/working_with_sinatra/server-zoom-sinatra.png)

**ERB**

- In `views` directory `.erb` files

- Use `<% %>` for Ruby code and `<%= %>` to render Ruby code

```
<ul class="pure-menu-list">
  <% @contents.each do |chapter| %>
    <li class="pure-menu-item">
      <a href="#" class="pure-menu-link"><%= chapter %></a>
    </li>
  <% end %>
</ul>
```

```
<ul class="pure-menu-list">
  <% @contents.each_with_index do |chapter, index| %>
    <li class="pure-menu-item">
      <a href="/chapters/<%= index + 1 %>" class="pure-menu-link"><%= chapter %></a>
    </li>
  <% end %>
</ul>
```

- Form submitting 

  - ```
    <form action="/search" method="get">
      <input name="query" value="<%= params[:query] %>"/>
      <button type="submit">Search</button>
    </form>
    ```

- Header Links 

  - In .erb file 
  
  - ```
      <% content_for :header_links do %>
        <a class="list" href="/lists">All lists</a>
      <%end%>
    ```
  
  - In layout.erb
  
  - ```
            <%== yield_content :header_links %>
    ```
  
    - The <%== disables auto escaping 
  
- Linking a page 

  - ```
    <a href="/lists/<%=params[:list_id]%>">Cancel</a>
    ```
  
    
  

**ERB Layouts**

```
<html>
  <head>
    <title>Is it your birthday?</title>
  </head>
  <body>
    <%= yield %>
  </body>
</html>
```

In the app .rb file 

```
get '/' do
  erb :index, layout: :post #layout.erb is used by default
end
```



**Switch between rubies**

```
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"

rbenv local 3.2.2
```

**Server-Side vs Client-Side**

- Gemfile- Server-Side
- Ruby Files- Server-Side
- StyleSheets- Client-Side
- Javascript Files- Client-Side
- View Templates- Server-Side

**SUMMARY**

- *Sinatra* is a small web framework.
- HTTP requests are handled in Sinatra by creating `routes` for a path or set of paths.
- Routes are created using methods named after the HTTP method to be handled. So, a `GET` request is handled by a route defined using the `get` Sinatra method.
- *View templates* can be written in many *templating languages*, such as *ERB*. They provide a place to define the HTML display of a response outside of the route handling it. Templating languages are usually better suited to describing HTML than plain Ruby.
- A *layout* is a view template that surrounds a specific response's template. They are used to provide shared HTML that is used by all views, and often include links to stylesheets and JavaScript files.
- Routes can specify *parameters* by using colon followed by the parameter name: `/chapters/:number`. In this case, the value would be accessible within the route using `params[:number]`.
- Code placed in a `before` block is executed at the beginning of each new request.
- *View helpers* are Ruby methods that are used to minimize the amount of Ruby code included in a view template. These methods are defined within a `helpers` block in Sinatra.
- A user can be sent to a new location in response to a request with *redirection*. This is done in Sinatra using the `redirect` method. The redirection is accomplished by setting the `Location` header in the response. The client looks at the URL in the location header and sends out a new HTTP GET request for the associated resource, which in turn navigates the client to that new location.
- The files in a project can be identified as either *server-side* or *client-side* code based on where they will be evaluated.

**Heroku**

- Developers use Heroku to deploy, manage, and scale modern apps. Our platform is elegant, flexible, and easy to use, offering developers the simplest path to getting their apps to market.

1) Make project a git repository

2) ```ruby
   require "sinatra"
   require "sinatra/reloader" if development?
   require "tilt/erubis"
   require "sinatra/content_for"
   ```

​      This goes in main .rb file 

5. Specify Ruby version in Gemfile 

   1. `ruby "3.2.0"`
      1. Run `bundle install`

6. Add this to Gemfile to use PUMA as the web server

   1. ```
      group :production do
        gem "puma"
      end
      ```

   2. Run `bundle install`

7. Create a `config.ru` file 

   1. ```
      require './todo'
      run Sinatra::Application
      ```

8. Create a `Procfile`

   1. `web: bundle exec puma -t 5:5 -p ${PORT:-3000} -e ${RACK_ENV:-development}`

9. Test your `Procfile` locally 

   In terminal: `bundle exec heroku local -p 5555`

   Visit at [localhost:5000](http://localhost:5555/]

   

   10. `$ heroku apps:create ls-rb175-book-viewer` in terminal 
   11. `$ git push heroku main` in terminal #or `master`



**Security**

- Is using POST as the HTTP method for a request more secure than GET? Why or why not?
  - **No.** Any request sent as plain text, regardless of the HTTP method used, is equally vulnerable to being seen while in transit on the network.
- How can a web application be secured so that any data being sent between the client and server can not be viewed by other parties?
  - Serving the application over **HTTPS** is the only way to protect a user's data.

**jQuery**

- JavaScript code that is executed in the user's browser in addition to Ruby code that runs on the server. We will be using the [jQuery](http://jquery.com/) library in the client-side code

- These go in the `public/javascripts/application.js` folder 

- ```javascript
  // public/javascripts/application.js
  $(function() {
  
    $("form.delete").submit(function(event) {
      event.preventDefault();
      event.stopPropagation();
  
      var ok = confirm("Are you sure? This cannot be undone!");
      if (ok) {
        var form = $(this);
  
        var request = $.ajax({
          url: form.attr("action"),
          method: form.attr("method")
        });
  
        request.done(function(data, textStatus, jqXHR) {
          if (jqXHR.status === 204) {
            form.parent("li").remove();
          } else if (jqXHR.status === 200) {
            document.location = data;
          }
        });
      }
    });
  
  });
  ```

- ```ruby
  # todo.rb
  
  # Delete a todo list
  post "/lists/:id/destroy" do
    id = params[:id].to_i
    session[:lists].delete_at(id)
    if env["HTTP_X_REQUESTED_WITH"] == "XMLHttpRequest"
      "/lists"
    else
      session[:success] = "The list has been deleted."
      redirect "/lists"
    end
  end
  
  ...
  
  # Delete a todo from a list
  post "/lists/:list_id/todos/:id/destroy" do
    @list_id = params[:list_id].to_i
    @list = load_list(@list_id)
  
    todo_id = params[:id].to_i
    @list[:todos].delete_at todo_id
    if env["HTTP_X_REQUESTED_WITH"] == "XMLHttpRequest"
      status 204
    else
      session[:success] = "The todo has been deleted."
      redirect "/lists/#{@list_id}"
    end
  end
  ```



**Testing Sinatra Applications**

1. Add `minitest` and `rack-test` to the project's `Gemfile` and run `bundle install` to install them.
2. Create a `test` directory within the project. Inside that directory, create a file called `cms_test.rb` and add to it any testing setup code.
3. Write a test that performs a `GET` request to `/` and validates the response has a successful response and contains the names of the three documents.
4. Write a test that performs a `GET` request to `/history.txt` (or another document of your choosing) and validates the response is successful and contains some of the content of that document.

- Since Sinatra builds on top of the [Rack](http://chneukirchen.org/blog/archive/2007/02/introducing-rack.html) library, testing Sinatra applications can take advantage of the [Rack::Test](https://github.com/brynary/rack-test)library for testing Rack applications.

- Write tests to prevent regression

- Minitest is Ruby's default testing library (DSL)

- **Test Suite:** this is the entire set of tests that accompanies your program or application. You can think of this as *all the tests* for a project.

- **Test:** this describes a situation or context in which tests are run. For example, this test is about making sure you get an error message after trying to log in with the wrong password. A test can contain multiple assertions.

- **Assertion:** this is the actual verification step to confirm that the data returned by your program or application is indeed what is expected. You make one or more assertions within a test.

  - `assert_equal` takes two parameters: the first is the expected value, and the second is the test or actual value.

- `test/app_test.rb`

  - Make a request to your application.

    Use `get`, `post`, or other HTTP-method named methods.

  - Access the response.

    The response to your request will be accessible using `last_response`. This method will return an instance of `Rack::MockResponse`. Instances of this class provide `status`, `body`, and `[]` methods for accessing their status code, body, and headers, respectively.

  - Make assertions against values in the response.

    Use the standard assertions supplied by `Minitest`.

  - Run `bundle exec ruby test/app_test.rb`

```
ENV["RACK_ENV"] = "test"

require "minitest/autorun"
require "rack/test"

require_relative "../app"

class AppTest < Minitest::Test
  include Rack::Test::Methods

  def app
    Sinatra::Application
  end

  def test_index
    get "/"
    assert_equal 200, last_response.status
    assert_equal "text/html;charset=utf-8", last_response["Content-Type"]
    assert_equal "Hello, world!", last_response.body
  end
end
```

| Assertion                          | Description                                 |
| :--------------------------------- | :------------------------------------------ |
| `assert(test)`                     | Fails unless `test` is truthy.              |
| `assert_equal(exp, act)`           | Fails unless `exp == act`.                  |
| `assert_nil(obj)`                  | Fails unless `obj` is `nil`.                |
| `assert_raises(*exp) { ... }`      | Fails unless block raises one of `*exp`.    |
| `assert_instance_of(cls, obj)`     | Fails unless `obj` is an instance of `cls`. |
| `assert_includes(collection, obj)` | Fails unless `collection` includes `obj`.   |

- When we use `assert_equal`, we are testing for *value equality*. Specifically, we're invoking the `==` method on the object. If we're looking for more strict *object equality*, then we need to use the `assert_same` assertion.

  - We can define an `==` method in main file to clarify `assert_equal`

    - ```
        def ==(other)                    
          other.is_a?(Car) && name == other.name
        end
      ```

- SEAT 

  - Set up the necessary objects.

    - ```
        def setup   #In test file to have @car accessible through test
          @car = Car.new
        end
      ```

  - Execute the code against the object we're testing.

  - Assert that the executed code did the right thing.

  - Tear down and clean up any lingering artifacts.



**Helpful Ruby methods**

- Get file names from directory 

  - ```
    Dir.children("files")  #files is the directory 
    ```

- Open and read files 

  - ```
      file = File.open("./files/#{params[:page]}")
      @file_contents = file.readlines.map(&:chomp)
    ```

    * This returns an array that you can iterate through in the erb file 

  - ```
    get "/:filename" do
      file_path = root + "/data/" + params[:filename]
    
      headers["Content-Type"] = "text/plain"
      File.read(file_path)
    end
    ```

​				* This doesn't use an erb file and sets the content-type to "text/plain"