# README

## Start Blog App

```
rails new speed_blog
cd spee*
```

In Gemfile add:

```ruby
gem 'bootstrap-sass'
gem 'jquery-rails'
gem 'devise'
gem 'carrierwave'
```

`bundle install`

## Set Up Bootstrap

Go to app/assets/stylesheets/application.css, 

**change extension to .scss**

Add to bottom of document:

```css
@import "bootstrap-sprockets";
@import "bootstrap";
```

Go to app/assets/javascripts/application.js, replace `//= require rails-ujs` with the following:

```js
//= require jquery
//= require jquery_ujs
//= require bootstrap-sprockets
```

Go to app/views/layouts/application.html.erb and add a container around yield, so body section looks like this:

```html
  <body>
    <div class="container">
      <%= yield %>
    </div>
  </body>
```

## Set Up Devise, Users, and Scaffold for Post and Comment

### Devise basic

in Terminal

```bash
rails g devise:install
rails g devise User
rails db:migrate
```
##### Set root
Devise won't work without a root directory in routes! Go to config/routes.rb and add:

```ruby
root 'posts#index'
```

Devise is ready, time to set up the rest of our models in Terminal

```
rails g migration AddPropertiesToUser username:string avatar:string
rails g scaffold Post title:string entry:text user_id:integer
rails g scaffold Comment message:text post_id:integer user_id:integer
rails db:migrate
```



## Models & Associations

In app/models

###user.rb should add associations

```ruby
  has_many :posts
  has_many :comments
```

 and look like this:

```ruby
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
  has_many :posts
  has_many :comments
end
```

### post.rb should add associations

```ruby
	has_many :comments
	belongs_to :user
```

and look like this:

```ruby
class Post < ApplicationRecord
	has_many :comments
	belongs_to :user
end
```

### comment.rb should add associations

```ruby
	belongs_to :post
	belongs_to :user
```

and look like this:

```ruby
class Comment < ApplicationRecord
	belongs_to :post
	belongs_to :user
end
```

##Set up Comment form on Post Show Page

#### Optional: Remove redundant comment views/actions, redirect

Delete unnecessary files in app/views/comments. The ONLY necessary view is _form.html.erb.

Delete unnecessary actions in controller. The ONLY necessary actions are create & comment_params

####In Comment Controller, fix the create action so it redirects to the Post page when it works

Before:

```ruby
 format.html { redirect_to @comment, notice: 'Comment was successfully created.' }
```

After:

```ruby
 format.html { redirect_to post_path(id: @comment.post_id), notice: 'Comment was successfully created.' }
```

Comments Controller page:

```ruby
class CommentsController < ApplicationController

  # POST /comments
  # POST /comments.json
  def create
    @comment = Comment.new(comment_params)

    respond_to do |format|
      if @comment.save
        format.html { redirect_to post_path(id: @comment.post_id), notice: 'Comment was successfully created.' }
        format.json { render :show, status: :created, location: @comment }
      else
        format.html { render :new }
        format.json { render json: @comment.errors, status: :unprocessable_entity }
      end
    end
  end


  private


    # Never trust parameters from the scary internet, only allow the white list through.
    def comment_params
      params.require(:comment).permit(:message, :post_id, :user_id)
    end
end
```

#### In Posts controller, explain what a comment is

We want to have comments & the comment form on the Show page. It has to know what a new comment is in the show action.

```ruby
  def show
    @comment = Comment.new
  end
```

Entire Posts controller:

```ruby
class PostsController < ApplicationController
  before_action :set_post, only: [:show, :edit, :update, :destroy]

  # GET /posts
  # GET /posts.json
  def index
    @posts = Post.all
  end

  # GET /posts/1
  # GET /posts/1.json
  def show
    @comment = Comment.new
  end

  # GET /posts/new
  def new
    @post = Post.new
  end

  # GET /posts/1/edit
  def edit
  end

  # POST /posts
  # POST /posts.json
  def create
    @post = Post.new(post_params)

    respond_to do |format|
      if @post.save
        format.html { redirect_to @post, notice: 'Post was successfully created.' }
        format.json { render :show, status: :created, location: @post }
      else
        format.html { render :new }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /posts/1
  # PATCH/PUT /posts/1.json
  def update
    respond_to do |format|
      if @post.update(post_params)
        format.html { redirect_to @post, notice: 'Post was successfully updated.' }
        format.json { render :show, status: :ok, location: @post }
      else
        format.html { render :edit }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /posts/1
  # DELETE /posts/1.json
  def destroy
    @post.destroy
    respond_to do |format|
      format.html { redirect_to posts_url, notice: 'Post was successfully destroyed.' }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_post
      @post = Post.find(params[:id])
    end

    # Never trust parameters from the scary internet, only allow the white list through.
    def post_params
      params.require(:post).permit(:title, :entry, :user_id)
    end
end
```

#### In app/views/comments/_form.html.erb aka Comments Form

Set up the user id and post id to be automatic.

Before:

```html
  <div class="field">
    <%= form.label :post_id %>
    <%= form.number_field :post_id %>
  </div>

  <div class="field">
    <%= form.label :user_id %>
    <%= form.number_field :user_id %>
  </div>
```

After:

```html
  <div class="field">
    <%= form.hidden_field :post_id, value: @post.id %>
  </div>

  <div class="field">
    <%= form.hidden_field :user_id, value: current_user.id %>
  </div>
```

> Note: We just broke the app. It doesn't know who the "current_user" is because we haven't set up the controllers yet so you have to log in. Make sure this is fixed before you try to go on.

#### In app/views/posts/show.html.erb aka Posts Show page

Add a section where the comment form shows up and has a list of the comments.

```html
<div class="row">
	<div class="col-md-6">
		
		<%= render 'comments/form', comment: @comment %>

		<% @post.comments.each do |comment| %>
			<div class="well">
				<h5><%= comment.user.username %> says...</h5>
				<p><%= comment.message %></p>
			</div>
		<% end %>

	</div>
</div>
```

## Finish Setting Up Devise Users, load Carrierwave

#### Controller: app/controllers/application_controller.rb

Add the following code

```ruby
	before_action :authenticate_user!

	before_action :configure_permitted_parameters, if: :devise_controller?

	protected

	def configure_permitted_parameters
		devise_parameter_sanitizer.permit(:sign_up, keys: [:username, :avatar])

		devise_parameter_sanitizer.permit(:account_update, keys: [:username, :avatar])
	end
```

Controller should look like this when you're done

```ruby
class ApplicationController < ActionController::Base
	#some versions of Rails have a protect_from_forgery line here
    before_action :authenticate_user!

	before_action :configure_permitted_parameters, if: :devise_controller?

	protected

	def configure_permitted_parameters
		devise_parameter_sanitizer.permit(:sign_up, keys: [:username, :avatar])

		devise_parameter_sanitizer.permit(:account_update, keys: [:username, :avatar])
	end
end

```

#### Form: Devise Views

In Terminal, run

```
rails g devise:views
```

In app/views/devise/registrations, there are two files.

Both new.html.erb and edit.html.erb files will need these fields added sometime after the email and before the submit:

```html
  <div class="field">
    <%= f.label :username %>
    <%= f.text_field :username %>
  </div>

  <div class="field">
    <strong>Upload a profile pic:</strong>
    <%= f.file_field :avatar %>
  </div>
```

#### Carrierwave

in Terminal

```
rails g uploader Avatar
```

in User Model, add

```ruby
mount_uploader :avatar, AvatarUploader
```

so now the file is

```ruby
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
  has_many :posts
  has_many :comments
  mount_uploader :avatar, AvatarUploader
end
```

##Set up Posts

####Form view

In app/views/posts/_form.html.erb, fix the user_id field (probably starting on line 24) so that it automatically makes it the current user's id:

```html
  <div class="field">
    <%= form.hidden_field :user_id, value: current_user.id %>
  </div>
```

#### Index View

Replace post.user_id with post.user.username, and replace the User `<th>` section with an Author section 

```html
<p id="notice"><%= notice %></p>

<h1>Posts</h1>

<table>
  <thead>
    <tr>
      <th>Title</th>
      <th>Entry</th>
      <th>Author</th>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @posts.each do |post| %>
      <tr>
        <td><%= post.title %></td>
        <td><%= post.entry %></td>
        <td><%= post.user.username %></td>
        <td><%= link_to 'Show', post %></td>
        <td><%= link_to 'Edit', edit_post_path(post) %></td>
        <td><%= link_to 'Destroy', post, method: :delete, data: { confirm: 'Are you sure?' } %></td>
      </tr>
    <% end %>
  </tbody>
</table>

<br>

<%= link_to 'New Post', new_post_path %>

```

#### Show View

Finesse & rearrange

```html
<p id="notice"><%= notice %></p>

<p>
  <strong>Title:</strong>
  <%= @post.title %>
</p>

<p>
  <strong>Author:</strong>
  <%= @post.user.username %>
</p>

<% if @post.user.avatar.url %>
	<%= image_tag  @post.user.avatar.url %>
<% end %>
<p>
  <strong>Entry:</strong>
  <%= @post.entry %>
</p>


<div class="row">
	<div class="col-md-6">
		
		<%= render 'comments/form', comment: @comment %>

		<% @post.comments.each do |comment| %>
			<div class="well">
				<h5><%= comment.user.username %> says...</h5>
				<p><%= comment.message %></p>
			</div>
		<% end %>

	</div>
</div>

<%= link_to 'Edit', edit_post_path(@post) %> |
<%= link_to 'Back', posts_path %>
```

