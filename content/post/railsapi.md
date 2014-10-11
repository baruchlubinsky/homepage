+++
Categories = ["Development", "Rails"]
Description = "Steps to create a Rails API, using rails-api and Postrgres as a database."
Tags = ["development", "rails", "postgres", "tutorial", "aws"]
Title = "Rails API backed by Postgres"
Section = "blog"
Date = 2014-08-20T12:27:12Z
+++

I've recently decided to move my Rails API backend from MongoDB to PostgreSQL. Mongo served me very well during the early stages of development, when I was rapidly iterating the schema, but now that the app is being released, PostgreSQL is a much better option. It is way easier to administer (using a managed instance as described below), and it has the flexibility of JSON-type fields that allows me to easily model my data.

Here are the step for setting up a [Rails-API](https://github.com/rails-api/rails-api) project with a [PostgreSQL](http://www.postgresql.org/).

Development 
==

Install PostgreSQL using [Homebrew](http://brew.sh/) and create a user for your application.

	$: brew install postgresql
	$: psql -d postgres
	posgres=#: create role 'username' with createdb login password 'password';
	posgres=#: ctrl+d

Follow the instructions from Homebrew to cause Postres to run at start up, otherwise run: 

    $: postgres -D /usr/local/var/postgres

Create the Rails-API project:

	$: gem isntall rails-api
	$: gem install pg
	$: rails-api new app

Edit `config/database.yml` with username and password. And set the adapter to be 'postgresql'. Then use Rails generators and ActiveModel as usual.

	$: cd app
	$: rails g model Organisation name:string
	$: rake db:migrate
	$: rake db:setup 

Production
==

[Amazon Web Services](http://aws.amazon.com/) makes it very easy to host Rails applications using [OpsWorks](http://aws.amazon.com/opsworks/) and [RDS](http://aws.amazon.com/rds/). 

Simply create a Postgres instance in RDS for use by your API. I've found it helpful (although I don't believe it should be strictly necessary) to set the database name at creation time. 

Then in OpsWorks create a [Rails App Server Layer](http://docs.aws.amazon.com/opsworks/latest/userguide/workinglayers-rails.html). Add an App with your API code and connect it to your RDS instance with appropriate authentication settings. 

At the time of writing there is a [bug](https://github.com/aws/opsworks-cookbooks/issues/136) in OpsWorks that requires you to set the database adapter explicitly at deploy time. In the OpsWorks console, when deploying the Rails app, add the following "Custom JSON":

    {"deploy":{"api":{"database":{"adapter":"postgresql","type":"postrgesql"}}}} 

Where `api` is the name you've given to your app in OpsWorks.




