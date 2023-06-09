
### Rails CLI

```
# Create a new rails app
$ rails new project_name 

# Start the Rails server
$ rails s

# Rails console
$ rails c

# Install dependencies
$ bundle install

# View all routes
$ rails routes

# Toggle rails caching
$ rails dev:cache
```

### Rails generators CLI
```
# CRUD Scaffold (model, migrations, controller, views, test)
rails g scaffold Product name:string price:decimal

# CRUD Scaffold with one to many relationship field
rails g scaffold Invoice customer:references

# Delete scaffold created files
rails destroy scaffold Product

# Controller (name, action1, action2, ...)
$ rails g controller Products index show

# Model, migration and table columns
$ rails g model Product name:string active:boolean
```

### Migration
```
# Create new table migration
rails g migration Invoices

# Update existing table migration
rails g migration add_comment_to_invoices comment:text

# Run migration
$ rails db:migrate

# Rollback last migration
$ rails db:rollback

# Run database seed code
$ rails db:seed

# Delete and re-create db and run migrations
$ rails db:reset

# Create table migration exemple
create_table :products do |t|
  t.string :name
  t.decimal :price, precision: 8, scale: 2
  t.timestamps
end

# Create table with foreign key
create_table :invoices do |t|
  # null: false = dont allow null value in this db field
  t.references :customer, null: false, foreign_key: true
  t.timestamps
end

# Change table migration exemple
add_column :invoices, :comment, :text
```

### Routes
```
# Route maps to controller#action
get 'welcome', to: 'pages#home'

# Root page (root_path name helper)
root 'pages#home' 

# Named route 
get 'exit', to: 'sessions#destroy', as: :logout

# Create all the routes for a RESTful resource
resources :items

# HTTP    Verb Path    Controller#Action  Named Helper
# GET     /items           items#index    items_path
# GET     /items/new       items#new      new_item_path
# POST    /items           items#create   items_path
# GET     /items/:id       items#show     item_path(:id)
# GET     /items/:id/edit  items#edit     edit_item_path(:id)
# PUT     /items/:id       items#update   item_path(:id)
# DELETE  /items/:id       items#destroy  item_path(:id)

# Only for certain actions
resources :items, only: :index

# Resource with exceptions 
resources :items, except: [:new, :create]

# Nested resources
resources :items do
  resources :reviews
end
# create 7 RESTful for items and reviews 
# :reviews will have /items/item:id prefixing each routes
# GET /items/:item_id/reviews  reviews#index  item_reviews_path 

# Dynamic segment: params['id']
get 'products/:id', to: 'products#show'
# Query String: url /products/1?user_id=2
# params will be {'id' 'user_id'}

# Namespace Admin::ArticleController
# and prefix '/admin'
namespace :admin do
  resources :articles
end

# only prefix '/admin'
scope '/admin' do
  resources :articles, :comments
end

# Redirect
get '/stories', to: redirect('/articles')
```

### Model
```
# Model validation
validates :title, :description, :image_url, presence: true
validates :email, presence: true, format: { with: /\A[^@\s]+@[^@\s]+\z/, message: 'Must be a valid email address'}
validates :price, numericality: { greater_than_equal_to: 0.01 }
validates :title, uniqueness:  true
validates :title, length: { minimum: 3, maximum: 100 }
validates :type, inclusion: types.keys

# Model relationship
belongs_to :customer

# Relation with cascade delete
has_many :invoices, dependent: :destroy

#One to one
has_one :profile

# Hook methods 
before_destroy :ensure_not_reference_by_any_invoices 
before_save :downcase_email 

# create virtual password and password_confirmation 
# and bcrypt password_digest
# add method to model: user.authenticate(params[:password])
has_secure_password
```

### Controllers
```
# Hook before running any code
before_action :set_post, only: [:show, :edit, :update, :destroy]

# If you use Devise (authentification)
before_action :authenticate_user!

# 7 Restfull action short exemple 
def index
  # Search input name :q
  if session[:q].present?
    params[:page] = 1
    # Create instance variable @
    @posts = Post.where "title like ?", "%" + session[:q] + "%"
  else
    @posts = Post.all
  end
  @posts = @posts.order("created_at DESC")
  # Pagination with will_paginate gem
  @posts = @posts.paginate(page: params[:page], per_page: 3)
  session[:q] = nil
end

# By convention all action (even empty one)
# run hook before_action and render view template
def show
end

def new
  @post = Post.new
end

def edit
end

def create
  @post = Post.new(post_params)
    if @post.save
      redirect_to @post, notice: 'Created successfully!'
    else
      render :new 
    end
end

def update
  if @post.update(post_params)
      redirect_to @post, notice: 'Updated successfully!'
    else
      render :edit 
    end
end

  def destroy
    @post.destroy
    redirect_to posts_url, notice: 'Delete successfully!' 
  end

private
  # Use callbacks to share common methods between actions.
  def set_post
    @post = Post.find(params[:id])
  end

  # Only allow a list of trusted parameters through.
  def post_params
    params.require(:post).permit(:title, :body, :image_url)
  end
end

# render text
render plain: 'Hello User'
```

### Active Record
```
# Active record common methods
Article.all
# Throw error if not found
Article.find(params[:id])
# Do not throw error if not found
Article.find_by(product_id: product_id)
@category = Category.find_by!(slug: params['slug']) # Return Not Found Error (404 page in production)
Article.group(:product_id).sum(:quantity)
Article.where('quantity > 1')
Article.where(cat_id: cat_id, model: model)
Article.where(model: model).or(Article.where(cat_id: cat_id))
Article.join(:categories).where(categories: { id: 2 } )
Article.where("title LIKE ?", "%" + params[:q] + "%")
Article.count
Article.first
Article.last
Article.column_names # ['id', 'name', 'price']
Category.delete_all # delete all rows in Category table
product.category = Category.all.sample # random for Faker data
@products = Product.offset(5).limit(10).all # skip 5, take 10
```

### Template
```
<%# Comment tag %>

<%# Output return expression tag %>
<%= @user.name %>

<%# No output return expression tag %>
<% if @user.name == 'Mike' %>

<%# Layout file : app/view/layouts/application.html.erb %>

<%# In layout file replace expression with page content %>
<%= yield %>

<%# View for the route name helper path %>
<%= link_to 'About', about_path, class: 'nav-link' %>

<%# View for the route name helper path with params %>
<%= link_to 'Item record', item_path(@item) %>

<%# auto object to route map (/products/:id) %>
<%= link_to 'Show', @product %>

<%# Delete link %>
<%= link_to 'Destroy', product, method: :delete, data: { confirm: 'Are you sure?' } %>

<%# Post link %>
<%= button_to 'Add to Cart', line_items_path(product_id: product) %>

# link_to with slot
<%= link_to category_path do %>
  <%= @product.category.name %>
<% end %>

<%# Button form %>
<%= button_to 'Logout', logout_path, method: :delete %>

<%# Image asset (app/assets/images) %>
<%= image_tag "rails.png" %>

<%# Format currency %>
<%= number_to_currency(product.price) %>

<%# Safe Html render %>
<%= sanitize(product.description) %>

<%# Enable caching %>
<%= cache @products do %>

<%# Check url %>
<%= request.path.include?('post') ? 'active' : '' %>">

<%# Check current page %>
<%= current_page?('about') ? 'active' : '' %>">

<%# Render shared partial _navbar.html.erb %>
<%= render partial: 'shared/navbar' %>

<%# Render partial _form.html.erb %>
<%= render 'form', product: @product %>

<%# data tables iterations %>
<% @users.each do |user| %> <% end %>

<%# Render a for each _user.html.erb %>
<%= render @users %> 

<%# Conditional %>
<% if current_user.signed_in? %> 
  ...
<% else %> 
  ...
<% end %>

<%# Display errors %>
<% if @post.errors.any? %> 
<% if form.object.errors.any? %>

<% if @post.errors.empty? %>
<% @post.errors.full_messages.each do |message| %>
  <p class='error'><%= message %></p>
<%= user.errors.count %>
<%= post.errors[:description] %>

<%# Flash messages %>
<% flash.each do |msg_type, msg| %>
  <div class="alert alert-<%= msg_type %>">
    <%= msg %>
  </div>
<% end %>
```

### Form
```
<%# Model form %>
<%= form_with(model: product), local: true do |form| %>
  <%= form.label :title %>
  <%= form.text_field :title, class: 'form-control' %>

  <%= form.submit %>
<% end %>

<%# Generic form %>
<%= form_with url: "/search", method: :get, local: true do |form| %>
  <%= form.label :query, "Search for:" %>
  <%= form.text_field :query, placeholder: 'search' %>
  <%= form.submit "Search", class: 'btn btn-primary' %>
<% end %>

<%# Multi-lines memo %>
<%= form.text_area :description, rows: 10, cols: 60 %>

<%# Collection Select (field, collection, key, value, label) %>
<%= form.collection_select :category_id, Category.all, :id, :name, {include_blank: '- Select a Category -'}, {class: 'form-select'}

<%# Select %>
<%= form.select :type, Customer.types.keys, prompt: 'Select a type' %>
<%= form.select :rating, (1..5) %>

<%# if form in new or edit mode change submit text %> 
<%= form.submit @product.new_record? ? 'Create' : 'Update', class: "mt-4 btn btn-primary" %>
```

### Flash, Session and Cookie
```
# Create flash (reset every new request)
flash[:success] = 'User created with success!'

# Create flash.now (reset every new view render)
flash.now[:error] = 'Please select s user!'

# Create session (reset every browser close)
session[:user_id] = user.id

# Check if session exist
session[:user_id].nil?

# Remove
session.delete(:user_id)

# Remove all
reset_session      

# Create cookie (reset at expiration date)
cookies.permanent[:remember_token] = remember_token

# Encrypted cookie
cookies.permanent.encrypted[:user_id] = user.id

# Delete cookie
cookies.delete(:user_id)
Database seed with faker
# bundle install 
$ gem 'faker' 

# db/seeds.rb
Customer.delete_all
10.times do |n|
    customer = Customer.new
    customer.name = Faker::Name.name
    customer.phone = Faker::PhoneNumber.phone_number
    customer.save
end

# Run seed script
$ rails db:seed

Devise Gem (authentification)
# Install
gem devise
bundle install
rails g devise:install # follow on-screen instruction
rails generate devise User
rails db:migrate

# Customize devise login, register, etc. views
rails generate devise:views

# Add in existing controllers
before_action :authenticate_user!

# User shared globals
user_signed_in?
current_user

# Logout link_to exemple
<%= link_to "Logout", destroy_user_session_path, method: :delete %>

Trix Editor
gem ‘image_processing'
rails action_text:install
bundle install
rails db:migrate

# Add to model class 
has_rich_text :body 

# Template use
<%= form.rich.text_area %>

# actiontext.scss (if using boostrap only)
trix-editor { 
  &.form-control { 
    height: auto; 
  } 
} 
```

### MISC
```
# Edit credentials encrypted list (API keys, secrets, etc.)
$ EDITOR="code --wait" bin/rails credentials:edit --environment=development

Clone a Rails project from Github
$ git clone https://github/myprojet…
$ cd projet_name
$ bundle install
$ yarn install --check-files
$ rails dev:cache # not always needed
$ rails db:migrate
$ rails db:seed
$ rails s
Heroku deployment

# Change Gemfile production database to Postgress
group :development do
 gem 'sqlite3'
end

group :production do
  gem 'pg'
end

# Create git and push to Heroku
$ heroku create
$ git remote
$ bundle install --without production
$ git status
$ git add -A
$ git commit -m 'heroku deployment'
$ git push origin master
$ git push heroku master
$ heroku run rails db:migrate
$ heroku run rails db:seed

# launch the upload site
$ heroku open 
```
