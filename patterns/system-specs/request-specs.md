# Request Specs

## Guidelines

Request specs provide 1 to 1 testing for every endpoint in an application.  Here are a few notes about request specs:

### Request specs are end to end

The Request spec should start at the API endpoint and go all the way to the database.  Request specs are preferred over Controller specs because:

1. &#x20;Request specs traverse the router, the middleware and both rack requests and responses.
2. &#x20;Request specs are fast as of Rails 5.

Controller specs can still be used for unit testing specific controller functionality.  Since Controller specs are considered unit tests, that would be the correct place to use mocks and stubs.

### Request specs do not stub

Request specs do not use stubbing and mocking of Ruby objects.  The goal of the Request spec is that it goes end to end.  It starts with the API request and traverses all of the code.

The exception here of course are Sidekiq jobs.  Sidekiq should be run with `Sidekiq::Testing.fake!`.  This [Sidekiq testing mode](https://github.com/mperham/sidekiq/wiki/Testing) adds all enqueued jobs to a `jobs` array.  Using the `rspec-sidekiq` [gem](https://github.com/philostler/rspec-sidekiq) you can check to see if the job was enqueued with Rspec matcher `have_enqueued_sidekiq_job `

Request specs do not typically run workers inline because those jobs take place outside of the request and would require a second request to retrieve the results.

### Request specs interact with the database

In contrast to System specs, Request specs do interact with the database.  Objects will need to be created directly through factories and expectations can be verified by checking values saved in the database.

Since Request specs do interact with the database, Request specs are the ideal place for writing end 2 end tests to verify that values are retrieved from or saved to the database correctly.

### **Request specs stub external API calls**

In contrast with System specs, Request specs can use libraries like `Webmock` to verify that a specific API was called.  Since request specs do not span multiple API calls, the stubbing and mocking of API requests should be manageable within the spec.

This does come down to preference and ease.  If there is a fake library available, generally that will be easier than stubbing the request using Webmock.  For example, `stripe-ruby-mock` allows the test to intercept all API calls to Stripe and store objects in memory.  If you want to check that a subscription was created, it is easier to use `stripe-ruby-mock` to see if a subscription was created as opposed to stubbing the request to create the subscription.&#x20;
