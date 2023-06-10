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

- ```
  require "sinatra"
  require "sinatra/reloader"
  require "tilt/erubis"
  
  
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