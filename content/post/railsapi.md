+++
Categories = ["Development", "Rails"]
Description = "Steps to create a Rails API, usiong rails-api and Postrgres as a database."
Tags = ["development", "rails", "postgres", "tutorial"]
Title = "Rails API backed by Postgres"
Section = "blog"
Date = 2014-08-20T12:27:12Z
Draft = true
+++

	$: brew install postgresql
	$: gem install pg
	$: gem isntall rails-api
	$: rails-api new app
	$: psql -d postgres
	posgres=#: create role username with createdb login password 'password';
	ctrl+d
	$: cd app

Edit `config/database.yml` with username and password.

	$: rails g model Organisation name:string
	$: rake db:migrate
	$: rake db:setup 
