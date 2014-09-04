+++
Categories = ["Development", "Ember", "Cloud"]
Description = "Create a custom file input component that allows files to be uploaded to a private bucket on Amazon Web Services (AWS) Simple Storage Solution (S3)."
Tags = ["development", "ember", "aws", "rails"]
Title = "Ember component for secure file upload to S3"
Section = "blog"
Date = 2014-09-04T17:42:00Z
+++

Last time I worked with seriously with Ember was before the addition of components (everything was a view). I'm enjoying learning it again in its current, much more mature, state. This article is about a component I created based on the standard HTML file input control. The component retrieves [temporary credentials for S3](http://docs.aws.amazon.com/AmazonS3/latest/dev/PresignedUrlUploadObject.html) from an API and produces a [FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData) object ready to post the selected file.

## Multi-tenant File Storage

This project is born out of a need to host sensitive user data on [AWS](http://aws.amazon.com/) using [S3](http://aws.amazon.com/s3/). Each user must be certain that their data is protected not only from public access but also from other users of the system. S3 provides functionality to restrict access to the files it hosts. The approach I have taken does not grant anyone access to the bucket. All requests must be temporarily granted permission. For uploading this is achieved using a "Presigned Post".

The bucket does not grant access to any user. Rather I create a [IAM role](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) that has rights to upload to the bucket. Then let the webserver running the backend API assume that role. The bucket must also allow [CORS](http://docs.aws.amazon.com/AmazonS3/latest/dev/cors.html) requests.

## Backend API

The backend I am using here is a [Rails-API](https://github.com/rails-api/rails-api) application. Of course any sort of backend will do, Rails is nice here because there is an official [AWS SDK for Ruby](http://docs.aws.amazon.com/AWSRubySDK/latest/frames.html).

The backend must provide the signed data for the POST. In reality it should also authenticate users before doing so. The following code can be used to create the fields that are required:

{{% highlight ruby %}}
require 'date'

class FilestoreController < ApplicationController
	def upload
		s3 = AWS::S3.new
		bucket = s3.buckets[Rails.configuration.filestore[:upload]]
		if !bucket.exists? 
			render json: {error: "Upload bucket #{bucket.name} does not exist."}, status: 404
			return
		end
		access_key_id = AWS.config.credentials[:access_key_id]
		policy = bucket.presigned_post({expires: Time.now.utc + 60*60})
		form = policy.fields
		render json: {form: {url: policy.url.to_s, fields: {
				"signature" => form['signature'],
				"policy" => form['policy'],
				"access_key_id" => access_key_id,
			}}}
	end
end
{{% /highlight %}}

This code uses the SDK to calculate the policy and signature fields as required. (The documentation about this is very confusing but it seems that this is enough.) For added security, the policy can made more specific. 

I like to host my Ember application separately from the backend. In that case you must have your Rails application set up for CORS. A nice simple way to do this is in the application controller.

{{% highlight ruby %}}
class ApplicationController < ActionController::API
	before_filter :cors

	def cors
    	response.headers.merge! 'Access-Control-Allow-Origin' => '*', 'Access-Control-Allow-Methods' => 'POST, PUT, GET, DELETE', 'Access-Control-Allow-Headers' => 'Origin, Accept, Content-Type'
  	end
end
{{% /highlight %}}

## File chooser

I wanted to create a component that could be used a "drop in replacement" for `<input type="file">`. So that it could be used withing existing javascript solutions for file uploads. This solution doesn't realise that perfectly, but I like the way that it uses an existing HTML element.

The component is defined (in an [Ember-CLI](http://www.ember-cli.com/) project) in `app/components/s3-upload.js`:

{{% highlight js %}}
import Ember from 'ember';

export default Ember.Component.extend({
	tagName: 'input',
	type: 'file',
	attributeBindings: ['disabled', 'alt', 'type'],
	folder: '',
	didInsertElement: function() {
		var self = this;
		self.set('disabled', true);
		self.set('alt', 'Authenticating...');
		Ember.$.ajax('http://url.for.backend/upload_form').then(
			function(response) {
				self.set('disabled', false);
				self.set('alt', 'Select a file to upload.');
				self.set('policy', response.form);
			},
			function(response) {
				this.set('alt', 'Unable to connect with S3.');
			}
		);
	},
	change: function() {
		var policy = this.get('policy');
		var form = new FormData();
		var input = this.get('element');
		var folder = this.get('folder');
		var key = folder + input.value.split(/(\\|\/)/g).pop();
		
		form.append("key", key);
		form.append("signature", policy.fields.signature);
		form.append("AWSAccessKeyId", policy.fields.access_key_id);
		form.append("policy", policy.fields.policy);
		form.append("file", input.files[0]);
		
		this.set('ajaxOptions', {
			url: policy.url,
			data: form,
			contentType: false,
			type: 'POST',
			processData: false,
			}
		);
	}
});
{{% /highlight %}}

No template is required. The component reveals an `ajaxOptions` property that contains all the data required to upload the file. Similar to the original element, the upload must be executed externally.

The `didInsertElement` event sends a request to the backend to retrieve the presigned fields that are saved in the components 'policy' property. Then when a file is selected a `FormData` object is created. S3 expects the uploaded file to be in the `file` field. The form is added to a hash of AJAx options ready to be posted. `processData` must be false to prevent jQuery from attempting to parse the file.

## Usage

The file selector is designed to be usable in various ways. The most simple one I came up with wraps it in another component including an upload button. The template for that component is:

{{% highlight html+django %}}
{{s3-upload ajaxOptions=formToPost disabled=disabled folder=folder}}
<button {{action 'doUpload'}} {{bind-attr disabled=disabled}}>Upload</button>
{{% /highlight %}}

This binds the disabled properties to eachother so that the upload button is disabled while the file input is disabled. The ajax options containing the file data are bound to `formToPost`. The code for this wrapper is:

{{% highlight js %}}
import Ember from 'ember';

export default Ember.Component.extend({
	actions: {
		doUpload: function() {
			var opts = this.get('formToPost');
			Ember.$.ajax(opts);
		}
	}
});
{{% /highlight %}}

The hash prepared in the `s3-upload` is ready to be used by `$.ajax`. This is a minimal implementation of that component. It is designed to be usable within a more complicated user interface - there are already many good javascript solutions out there. Of course the results of the ajax call should be handled.