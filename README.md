# ZencoderJS

A JavaScript library for interacting with the [Zencoder transcoding API](http://zencoder.com).

See [http://zencoder.com/docs/api](http://zencoder.com/docs/api) for more details on the API.

Requires [jQuery](http://www.jquery.com).

## Getting Started

The first thing you'll need to interact with the Zencoder API is a Zencoder API key. 
You'll want to use a "Read-Only API Key" which blocks any users that find the key in the javascript from creating jobs without your permission. 
You can find or generate your read-only API key in your Zencoder account under the [API tab](https://app.zencoder.com/api).

To get started, pass your API key to the Zencoder function:

```javascript
Zencoder('abcd1234abcd1234');
```

This will store your API key and automatically pass it with any requests to the API.

## Request Functions

All functions that send a request (get, post) to the Zencoder API have three possible arguments: options, callback, and error.

```javascript
Zencoder.Job(options, callback, error);
```

- options: A javascript object {} with options specific to the function. Sometimes a resource ID or other value can be used instead of an object as a shortcut.
- callback: A function that is called when the request is successful. Two arguments are passed to the callback:
  - data: A javascript object with the requested information or the response data from a post.
  - response: A Zencoder.Response object that contains details about the response including the raw content string returned from the API.
  
  ```javascript
  callback = function(data, response) {};
  ```
  
- error: A function that is called when there is an error performing the request. A Zencoder.HTTPError object is returned as the first argument and contains information about the error.

### Job Progress

The most likely use of the zencoder-js library is to poll the progress of a transcoding job. 
You could use this to show your users a progress percent or bar for a job or specific output.

In this case you have probably already created the transcoding job using another Zencoder API integration, and have stored the job ID that was returned by the Zencoder API. Make sure that ID is somehow accessible in javascript.

```javascript
var jobID = "1234"; // Written out by your application.
```

Create an instance (object) of the job in javascript using the Zencoder.Job.find() function.
The first argument passed to the callback function will be a job object, where you can access any job details passed back by Zencoder,
as well as output and input details.

```javascript
Zencoder.Job.find(jobID,
  function(myJob, response){

    // You can now access job details
    var jobState = myJob.state,

    // Including job outputs
    firstOutput = myJob.outputs[0];

  }, function(error){

    alert(error.message);

  }
);
```
Now choose the output you want to listen for progress on. 
Even if you only have one output in your job, the best place to listen for progress is on an output.
For this example we'll assume you're interested in the first output.
> focusing in on the callback function in this example

```javascript
function(myJob, response){

  var jobState = myJob.state,
      firstOutput = myJob.outputs[0];

  firstOutput.onProgress(function(event, output){
    
    // Learn more about output progress here: https://app.zencoder.com/docs/api/outputs/progress
    
    // State: Current state of the job (example: 'processing')
    var state = output.state,

    // Overall output progress as a percent (example: 50.01)
    progress = output.progress,

    // Current Proccessing Event. Only has a value if the current state is 'processing'.
    // possible values: downloading, transcoding, uploading
    current_event = output.current_event,

    // The progress through the current event as a percent (example: 50.01)
    current_event_progress = output.current_event_progress;
  });

}
```

### Parameters

When sending API request parameters you can specify them as a non-string object, which we'll then serialize to JSON:

```ruby
Zencoder::Job.create({:input => 's3://bucket/key.mp4'})
```

Or you can specify them as a string, which we'll just pass along as the request body:

```ruby
Zencoder::Job.create('{"input": "s3://bucket/key.mp4"}')
```

## Jobs

There's more you can do on jobs than anything else in the API. The following methods are available: `list`, `create`, `details`, `resubmit`, `cancel`, `delete`.

### create

The hash you pass to the `create` method should be encodable to the [JSON you would pass to the Job creation API call on Zencoder](http://zencoder.com/docs/api/#encoding-job). We'll auto-populate your API key if you've set it already.

```ruby
Zencoder::Job.create({:input => 's3://bucket/key.mp4'})
Zencoder::Job.create({:input => 's3://bucket/key.mp4',
                      :outputs => [{:label => 'vp8 for the web',
                                    :url => 's3://bucket/key_output.webm'}]})
Zencoder::Job.create({:input => 's3://bucket/key.mp4', :api_key => 'abcd1234'})
```

This returns a Zencoder::Response object. The body includes a Job ID, and one or more Output IDs (one for every output file created).

```ruby
response = Zencoder::Job.create({:input => 's3://bucket/key.mp4'})
response.code            # => 201
response.body['id']      # => 12345
```

### list

By default the jobs listing is paginated with 50 jobs per page and sorted by ID in descending order. You can pass two parameters to control the paging: `page` and `per_page`.

```ruby
Zencoder::Job.list
Zencoder::Job.list(:per_page => 10)
Zencoder::Job.list(:per_page => 10, :page => 2)
Zencoder::Job.list(:per_page => 10, :page => 2, :api_key => 'abcd1234')
```

### details

The number passed to `details` is the ID of a Zencoder job.

```ruby
Zencoder::Job.details(1)
Zencoder::Job.details(1, :api_key => 'abcd1234')
```

### progress

The number passed to `progress` is the ID of a Zencoder job.

```ruby
Zencoder::Job.progress(1)
Zencoder::Job.progress(1, :api_key => 'abcd1234')
```

### resubmit

The number passed to `resubmit` is the ID of a Zencoder job.

```ruby
Zencoder::Job.resubmit(1)
Zencoder::Job.resubmit(1, :api_key => 'abcd1234')
```

### cancel

The number passed to `cancel` is the ID of a Zencoder job.

```ruby
Zencoder::Job.cancel(1)
Zencoder::Job.cancel(1, :api_key => 'abcd1234')
```

### delete

The number passed to `delete` is the ID of a Zencoder job.

```ruby
Zencoder::Job.delete(1)
Zencoder::Job.delete(1, :api_key => 'abcd1234')
```

## Inputs

### details

The number passed to `details` is the ID of a Zencoder input.

```ruby
Zencoder::Input.details(1)
Zencoder::Input.details(1, :api_key => 'abcd1234')
```

### progress

The number passed to `progress` is the ID of a Zencoder input.

```ruby
Zencoder::Input.progress(1)
Zencoder::Input.progress(1, :api_key => 'abcd1234')
```

## Outputs

### details

The number passed to `details` is the ID of a Zencoder output.

```ruby
Zencoder::Output.details(1)
Zencoder::Output.details(1, :api_key => 'abcd1234')
```

### progress

*Important:* the number passed to `progress` is the output file ID, not the Job ID.

```ruby
Zencoder::Output.progress(1)
Zencoder::Output.progress(1, :api_key => 'abcd1234')
```

## Notifications

### list

By default the jobs listing is paginated with 50 jobs per page and sorted by ID in descending order. You can pass three parameters to control the paging: `page`, `per_page`, and `since_id`. Passing `since_id` will return notifications for jobs created after the job with the given ID.

```ruby
Zencoder::Notification.list
Zencoder::Notification.list(:per_page => 10)
Zencoder::Notification.list(:per_page => 10, :page => 2)
Zencoder::Notification.list(:per_page => 10, :page => 2, :since_id => 20)
Zencoder::Notification.list(:api_key => 'abcd1234')
```

## Accounts

### create

The hash you pass to the `create` method should be encodable to the [JSON you would pass to the Account creation API call on Zencoder](http://zencoder.com/docs/api/#accounts). No API key is required for this call, of course.

```ruby
Zencoder::Account.create({:terms_of_service => 1,
                          :email => 'bob@example.com'})
Zencoder::Account.create({:terms_of_service => 1,
                          :email => 'bob@example.com',
                          :password => 'abcd1234',
                          :affiliate_code => 'abcd1234'})
```

### details

```ruby
Zencoder::Account.details
Zencoder::Account.details(:api_key => 'abcd1234')
```

### integration

This will put your account into integration mode (site-wide).

```ruby
Zencoder::Account.integration
Zencoder::Account.integration(:api_key => 'abcd1234')
```

### live

This will put your account into live mode (site-wide).

```ruby
Zencoder::Account.live
Zencoder::Account.live(:api_key => 'abcd1234')
```

## Reports

### minutes

This will list the minutes used for your account within a certain, configurable range.

```ruby
Zencoder::Report.minutes(:from => "2011-10-30", :to => "2011-11-24")
```

## Advanced HTTP

### Alternate HTTP Libraries

By default this library will use Net::HTTP to make all API calls. You can change the backend or add your own:

```ruby
require 'typhoeus'
Zencoder::HTTP.http_backend = Zencoder::HTTP::Typhoeus

require 'my_favorite_http_library'
Zencoder::HTTP.http_backend = MyFavoriteHTTPBackend
```

See the HTTP backends in this library to get started on your own.

### Advanced Options

A secondary options hash can be passed to any method call which will then be passed on to the HTTP backend. You can pass `timeout` (in milliseconds), `headers`, and `params` (will be added to the query string) to any of the backends. If you are using Typhoeus, see their documentation for further options. In the following example the timeout is set to one second:

```ruby
Zencoder::Job.create({:input => 's3://bucket/key.mp4'}, {:timeout => 1000})
```


### SSL Verification

We will use our bundled SSL CA chain for SSL peer verification which should almost always work without a hitch. However, if you'd like to skip SSL verification you can pass an option in the secondary options hash.

**NOTE: WE HIGHLY DISCOURAGE THIS! THIS WILL LEAVE YOU VULNERABLE TO MAN-IN-THE-MIDDLE ATTACKS!**

```ruby
Zencoder::Job.create({:input => 's3://bucket/key.mp4'}, {:skip_ssl_verify => true})
```

Alternatively you can add it to the default options.

```ruby
Zencoder::HTTP.default_options.merge!(:skip_ssl_verify => true)
```

### Default Options

Default options are passed to the HTTP backend. These can be retrieved and modified.

```ruby
Zencoder::HTTP.default_options = {:timeout => 3000,
                                  :headers => {'Accept' => 'application/json',
                                               'Content-Type' => 'application/json'}}
```

## Advanced JSON

### Alternate JSON Libraries

This library uses the `multi_json` gem to encode and decode JSON. This fantastic gem lets you swap out the JSON backend at will and includes a working JSON encoder/decoder. You can check the [MultiJson](https://github.com/intridea/multi_json) project for more information on how to accomplish this.