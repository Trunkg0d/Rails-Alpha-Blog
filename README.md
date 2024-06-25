#### 1. Details
- Model name: article
- Class name: Article -> Capitalized A and singular, CamelCase
- File name: article.rb -> singular and all lowercase, snake_case
- Table name: articles -> plural of model name and all lowercase

Additional example:
- Model name: user
- Class name: User -> Capitalized U and singular, CamelCase
- File name: user.rb -> singular and all lowercase, snake_case
- Table name: users -> plural of model name

#### 2. Generate a migration to create a table (in this example articles):
```rails generate migration create_articles```

- To add attributes for the table in the migration file, add the following inside create_table block:
	`t.string :title`
- To run the migration file, run the following command from the terminal:
	`rails db:migrate`
- The first time you run the migration file, it will create the database, the articles table and a schema.rb file.
- To rollback or undo the changes made by the last migration file that was run, you may use the following command:
	`rails db:rollback`
- If you have run the rollback step, then you can update the previous migration file and add the following line to add a description attribute (column) to the articles table:
	```t.text :description```
- To run the newly edited migration file again, you can run `rails db:migrate`

Note: This above line will only work if you had rolled back the prior migration.

To generate a new migration file to add or make changes to your articles table you can generate a new file:
`rails generate migration name_of_migration_file`

Then within the def change method in the migration file you can add the following lines:
`add_column :articles, :created_at, :datetime`
`add_column :articles, :updated_at, :datetime`

You can run the newly created migrations file by running `rails db:migrate` from the command line and check out the `schema.rb` file to check that the changes were reflected properly.
#### 3. Models and rails console - text references
To create an article model, create an `article.rb` model file under `app/models` folder and fill it in:
```ruby
class Article < ApplicationRecord

end
```

Note: Make sure ApplicationRecord is CamelCase.

Now, provided you have the articles table already, you can use the Rails console and work with the articles table using this `article.rb` model file.

- To start the rails console, type in `rails console`  or `rails c` from the terminal.
	- Once in the console, you can exit it at any time by typing in `exit` followed by enter/return.
	- You can test out your connection to the articles table by typing the following command from within your rails console:
		`Article.all`
	- If you get an empty collection/array-like structure as a response, you're good to go.

- To create a new article, you can use any of the following methods:
```ruby
Article.create(title: "first article", description: "Description of first article") # make sure Article is capitalized if using this method
article = Article.new
article.title = "second article"
article.description = "description of second article"
article.save
article = Article.new(title: "third article", description: "description of third article")
article.save
```
- To check all the articles that exist in your articles table, you can use the following command:
	`Article.all`
#### 4. CRUD ops from rails console - text directions and code

- To find an article by id you can use the find method like below:
	`Article.find(1) # replace 1 with id of article you want to find`

- You can save this to a variable and use it like below

```ruby
	article = Article.find(1)
	article.title # to display (get) the title
	article.description # to display (get) the description
```

- You can use the methods below to view the first and last articles of the articles table:

	1. `Article.first` # display the first article in the articles table
	2. `Article.last` # display the last article in the articles table

- You can update an article by finding it first and then using setters for the attributes that the model provides like below:
```ruby
	article = Article.find(id of article you want to edit)
	article.title = "new title"
	article.description = "new description"
	article.save
```

- You can delete an article by using the destroy method. A sample sequence could be like below:
```ruby
	article = Article.find(id of article you want to delete)
	article.destroy
```

#### 5. Validations

```ruby
class Article < ApplicationRecord

  validates :title, presence: true, length: {minimum: 6, maximum: 100}

  validates :description, presence: true, length: {minimum: 10, maximum: 300}

end
```
#### 6. Forms
To create a new article from the browser (front-end), you'll need a form to get input from the user. Since we're dealing with articles which have title and description, you want to give the user an ability to fill-in the title and description of the article they are trying to create. The form is displayed via the new route/action and the form submission is handled by the create action.

First you'll need to expose these two routes in the `config/routes.rb` file, so the file looks like below:

You will need to add the new and create actions in the `articles_controller.rb` file like below:
```ruby
def new
end

def create 
end  
```
You will also need to create a view template for the new view. So, in the app/views/articles folder create a new file called `new.html.erb` and fill it in like below:
```ruby
<h1>Create a new article</h1>

<%= form_with scope: :article, url: articles_path, local: true do |f| %>
	<p>
		<%= f.label :title %><br/>
		<%= f.text_field :title %>
	</p>
	<p>
		<%= f.label :description %><br/>
		<%= f.text_area :description %>
	</p>
	<p>
		<%= f.submit %>
	</p>
<% end %>
```
#### 7. Messaging - validation and flash messages
In order to display the validation messages, we have to add an if else block to our create action. This is done to check for if the save happened, if not (else clause) then we display the new form again with the messages displayed. The create action would look like below:
```ruby
def create
	@article = Article.new(params.require(:article).permit(:title, :description))
	if @article.save
		redirect_to @article
	else
		render 'new'
	end
end
```
In order to display the messages, we add the following code block to the `new.html.erb` template (above the form code):
```html
<% if @article.errors.any? %>
	<h2>The following errors prevented the article from being saved</h2>
	<ul>
		<% @article.errors.full_messages.each do |msg| %>
			<li><%= msg %></li>
		<% end %>
	</ul>
<% end %>
```
To make the code work for the first time when the new form is displayed, we have to initiate an `@article` instance variable in the new action of the articles controller. Otherwise, the code `@article.errors.any?` will fail (as there is no `@article` instance variable available at the time).
```ruby
def new
	@article = Article.new
end
```
To display messages to the user using the flash messages helper, update the create action with the additional flash line like below:
```ruby
def create
	@article = Article.new(params.require(:article).permit(:title, :description))
	if @article.save
		flash[:notice] = "Article was created successfully."
		redirect_to @article
	else
		render 'new'
	end
end
```
Once the flash helper has the key of `'notice'` which has value of `'Article was created successfully'` via the create action, you can use this helper in your views (upon the redirect) to display the message to the user. Therefore we add the following code to the `app/views/layouts/application.html.erb` file within the body tag:
```html
<% flash.each do |name, msg| %>
	<%= msg %>
<% end %>
```
