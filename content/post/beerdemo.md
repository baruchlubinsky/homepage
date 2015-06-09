+++
Categories = ["Development", "Ember"]
Description = "Develop an application demonstarting how easy it is to build data driven applications using Ember.js."
Tags = ["development", "ember", "tutorial", "emberdata"]
Title = "Getting started with Ember and Ember Data using Ember CLI"
Section = "blog"
Date = 2014-08-13T18:38:22Z
+++

This article describes a minimal [Ember](http://emberjs.com/) application. Ember, like [Rails](http://rubyonrails.org/), is designed with a philosophy of convention over configuration. This demo is designed to demonstrate how little code is required to create a functional data driven website by harnessing [Ember Data](http://emberjs.com/guides/models/) and the [REST Adapter](http://emberjs.com/guides/models/the-rest-adapter/). 

# Getting started

I use [Ember CLI](http://www.ember-cli.com/) to build my site. Ember CLI uses ES6 Modules to organise code. This leads to well structured, logically layed out code (unlike traditional Javascript). For a backend I wrote a [Google App Engine](http://appengine.google.com/) program, it does not contain any business logic. All the data processing is handled by the Ember App.

The application keeps track of a list of beers, and comments about each beer. All the CRUD functions are implemented, and the data contains a "has-many" relationship. The rest of this article explains the files I added to the scaffolding from Ember CLI. [Clone the repo.](https://github.com/baruchlubinsky/beerdemo/tree/e6a7984c9692a62198ee06ba1961a819ed990c97)

# Models

The data model is very simple, two model and one relationship. See the [docs](http://emberjs.com/guides/models/the-rest-adapter/#toc_relationships) explaining how `async` affects the JSON. 

#### `app/models/beer.js`

{{< highlight javascript >}}
import DS from 'ember-data';

export default DS.Model.extend({
  name: DS.attr('string'),
  type: DS.attr('string'),
  comments: DS.hasMany('comment', {async: true}),
});
{{< /highlight >}}

#### `app/models/comment.js`

{{< highlight javascript >}}
import DS from 'ember-data';

export default DS.Model.extend({
  user: DS.attr('string'),
  message: DS.attr('string'),
  beer: DS.belongsTo('beer'),
});
{{< /highlight >}}

# Router

Comments are added at `beers.show` and records are deleted from `beers.edit`.

#### `app/router.js`

{{< highlight javascript >}}
import Ember from 'ember';

var Router = Ember.Router.extend({
  location: DemoENV.locationType
});

Router.map(function() {
	this.resource('beers', function() {
    	this.route('new');
    	this.route('show', {path: ':id'});
    	this.route('edit', {path: ':id/edit'});
  	});
});

export default Router;
{{< /highlight >}}

# Routes

`/beers` shows a list of all the beer records.

#### `app/routes/beers/index.js`

{{< highlight javascript >}}
import Ember from 'ember';

export default Ember.Route.extend({
	model: function() {return this.store.find('beer');},
});
{{< /highlight >}}

`/beers/new` provides allows a new record to be created.

#### `app/routes/beers/new.js`

{{< highlight javascript >}}
import Ember from 'ember';

export default Ember.Route.extend({
	model: function() {return this.store.createRecord('beer');},
	actions: {
		create: function() {
			var self = this;
			this.controller.get('model').save().then(
				function() {
					self.transitionTo('beers.index');
				});
		}
	}
});
{{< /highlight >}}

`/beers/:id/edit` updates a record. The `:id` from the URL is available to the model hook as `params.id`.

#### `app/routes/beers/edit.js`

{{< highlight javascript >}}
import Ember from 'ember';

export default Ember.Route.extend({
	model: function(params) {return this.store.find('beer', params.id);},
	actions: {
		save: function() {
			var self = this;
			self.controller.get('model').save().then(
				function() {
					self.transitionTo('beers.index');
				});
		},
		delete: function() {
			var self = this;
			var model = self.controller.get('model'); 
			model.destroyRecord().then(
				function() {
					self.transitionTo('beers.index');
				}, function (error) {
					Ember.Logger.debug(error);
				});
		}
	}
});
{{< /highlight >}}

`/beers/:id` displays an individual beer record and adds comments to it.

#### `app/routes/beers/show.js`

{{< highlight javascript >}}
import Ember from 'ember';

export default Ember.Route.extend({
	model: function(params) {
		return this.store.find('beer', params.id);
	},
	setupController: function(controller, model) {
    	controller.set('model', model);
    	controller.set('newComment', this.store.createRecord('comment'));
  	},
  	actions: {
		comment: function() {
			var self = this;
			var comment = self.controller.get('newComment');
			var beer = self.controller.get('model');
			beer.get('comments').addObject(comment);
			comment.save().then( // comment first to get an ID
				function() {
					beer.save().then(
						function() {
							self.controller.set('newComment', self.store.createRecord('comment'));
						}
					);
				}
			);
		},
		willTransition: function() {
			this.controller.get('newComment').destroyRecord();
			return true;
		}
	}
});
{{< /highlight >}}

The `willTransition` handler is necessary to clean up when the route is changed by the back button.

# Controllers and Views

This application works with the default controllers and views.

# Adapter

Connect Ember Data to the webservice. You're welcome to try out my instance, but if you want to use it more heavily please [create your own on the App Engine](https://github.com/baruchlubinsky/everyrest). 

#### `app/adapters/application.js`

{{< highlight javascript >}}
import DS from 'ember-data';

export default DS.RESTAdapter.extend({
	host: "http://everyrest.appspot.com",
	ajax: function(url, method, hash) {
 		hash = hash || {}; // hash may be undefined
 		hash.crossDomain = true;
 		hash.xhrFields = {withCredentials: false};
 		return this._super(url, method, hash);		
 	}
});
{{< /highlight >}}

# Serializer

See the [docs](// http://emberjs.com/api/data/classes/DS.RESTSerializer.html#method_serializeHasMany) to see how this causes a model to write an array of its "has-many" ID's to the server. This is not the defualt since most servers take responsibility for tracking relationships in the data.

#### `app/serializers/application.js`
{{< highlight javascript >}}
import DS from 'ember-data';

export default DS.RESTSerializer.extend({
	serialize: function(record, options) {
	    var json = this._super(record, options);
	    record.eachRelationship(function(name, relationship) {
	      if (relationship.kind === 'hasMany') {
	        json[name] = record.get(name).mapBy('id');
	      }
	    });
	    return json;
	  }
});) 
{{< /highlight >}}

# Templates

There is nothing special about the [templates](http://emberjs.com/guides/templates/handlebars-basics/) I have used in this app. Here is the template for `show` as an example.

#### `app/templates/beers/show.hbs`

{{< highlight html >}}
<h2>{{name}}</h2>
<h3>{{type}}</h3>
<h4>Comments</h4>
<ul>
{{#each model.comments}}
<li>
	<p>{{message}}</p>
	<em>{{user}}</em>
</li>
{{/each}}
</ul>
<h4>Post</h4>
<p>Name:</br>
{{input value=newComment.user}}</p>
<p>Comment:</br>
{{textarea value=newComment.message cols="40" rows="4"}}</p>
<button {{action 'comment'}}>Comment</button>
{{< /highlight >}}