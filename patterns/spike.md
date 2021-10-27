---
description: A repeatable way to verify the behavior of external systems.
---

# Spike

## Overview

Spikes help the team do two things:

1. Verify exactly how an external system behaves.
2. Allow others to easily and exactly recreate behavior.

Before you write any code that interacts with an external API, write a spike first so and your team are certain you understand how it behaves.

**Docs Can Lie** - Or at least tell partial truths.  The only way to truly understand an external service is to run code against the external service. &#x20;

**One file, maybe two** - Spikes are typically only one file.  If you have multiple files, please define a directory to store your spike.

**Inline Gem dependencies** - Spikes should run with a single `ruby` or `rackup`command.  Define your gems in your ruby file so no external dependencies need to be installed.

**Use Sinatra (or other Rack framework)** - To keep the spike small, use Sinatra to define all of your endpoints in a single file.

**Use dotenv** - Spikes are committed to version control.  Externalize any sensitive data (like API keys) to ENV variables. &#x20;

**Pass Arguments** - Use ENV variables to pass arguments into your spike.

**Add documentation** - Add links to pertinent documentation that explains the API you are using.

## Example <a href="example" id="example"></a>

### Single File Spike

Here is a spike to evaluate the Twilio Usage API.

```ruby
require "bundler/inline"
gemfile do
  gem "activesupport"
  gem "dotenv"
  gem "pry"                  # command line debugger
  gem "pry-byebug"           # provides next, continue, step for pry
  gem "pry-rails"            # automatically use pry on the console
  gem "twilio-ruby"
end

require "dotenv"
require "pry"
require "active_support"
require "active_support/core_ext"
require "twilio-ruby"
Dotenv.load(".env")

account_sid = ENV.fetch("TWILIO_ACCOUNT_SID")
auth_token = ENV.fetch("TWILIO_AUTH_TOKEN")

# Based on
# https://www.twilio.com/docs/phone-numbers/api/incomingphonenumber-resource#create-an-incomingphonenumber-resource

@client = Twilio::REST::Client.new(account_sid, auth_token)

messages = @client.messages.list(
  date_sent_after: 3.days.ago,
  limit: 1000,
)

puts messages.to
```

### Multi File Spike

Here is a spike to evaluate how the Plaid Stripe integration works.

```ruby
# plaid/config.ru
require_relative "./process_ach"

run PlaidExampleServer.new

# plaid/process_ach.rb
require "bundler/inline"
gemfile do
  gem "activesupport"
  gem "dotenv"
  gem "pry"                  # command line debugger
  gem "pry-byebug"           # provides next, continue, step for pry
  gem "pry-rails"            # automatically use pry on the console
  gem "stripe"
  gem "plaid"
  gem "sinatra"
end

require "dotenv"
require "pry"
require "active_support"
require "active_support/core_ext"
require "plaid"
require "sinatra/base"
Dotenv.load("../../../.env.development")

Stripe.api_key = ENV["STRIPE_SECRET_KEY"]
Stripe.api_version = "2019-09-09"
PLAID_CLIENT_ID = ENV.fetch("PLAID_CLIENT_ID").freeze
PLAID_SECRET = ENV.fetch("PLAID_SECRET").freeze
CLIENT_USER_ID = "some_user_id".freeze

unless defined? STRIPE_JS_HOST
  STRIPE_JS_HOST = "https://js.stripe.com".freeze
end

unless defined? STRIPE_CONNECT_HOST
  STRIPE_CONNECT_HOST = "https://connect.stripe.com".freeze
end

# Checklist
# Connecting to a Plaid Account
# Confirm the user wants to charge
# Make the charge
# Handling Plaid Errors
# Write a system test for Plaid

class PlaidExampleServer < Sinatra::Base
  get "/" do
    puts "We have a visitor!"
    "Hello.  This is the plaid test server."
  end

  get "/charge" do
    puts "Create a new client"
    @client = Plaid::Client.new(env: :sandbox,
                                client_id: PLAID_CLIENT_ID,
                                secret: PLAID_SECRET)

    puts "We are going to create a charge"
    puts "First we need to create token to generate a clientside link"

    public_token = @client.link_token.create(
      user: {
        client_user_id: CLIENT_USER_ID,
      },
      client_name: "My App",
      products: ["transactions"],
      country_codes: ["US"],
      language: "en",
    )

    erb :new_charge, locals: {
      name: "books",
      link_token: public_token.link_token,
    }
  end

  post "/charge" do # rubocop:disable Metrics/BlockLength
    @client = Plaid::Client.new(env: :sandbox,
                                client_id: PLAID_CLIENT_ID,
                                secret: PLAID_SECRET)
    puts "Now that we have the token, let's get the details of the account"
    puts "We need the account details so that the user can authorize"
    body = JSON.parse request.body.read
    public_token = body["public_token"]
    account_id = body["account_id"]
    exchange_token_response = @client.item.public_token.exchange(public_token)
    access_token = exchange_token_response["access_token"]
    stripe_response = @client.processor.stripe.bank_account_token.create(
      access_token, account_id
    )
    stripe_bank_account_token = stripe_response["stripe_bank_account_token"]

    # now we create a customer with their bank account
    customer = Stripe::Customer.create(
      {
        name: "Some name",
        email: "some@gmail.com",
        source: stripe_bank_account_token,
      },
      # this would be the stripe connect account
      # stripe_account: @response.ask.asker.stripe_user_id,
    )
    Stripe::Charge.create(
      amount: 100,
      currency: "usd",
      customer: customer,
    )
  end
end

# plaid/views/new_charge.rb
<html>

  <body>
    <p>Checkout our <%= name %></p>
    <button id="link-button">Link Account</button>

    <script src="https://cdn.plaid.com/link/v2/stable/link-initialize.js"></script>
    <script type="text/javascript">
      (async function() {

        const configs = {
          // 1. Pass the token generated in step 2.
          token: '<%= link_token %>',

          onSuccess: async function(public_token, metadata) {
            // 2a. Send the public_token to your app server.
            // The onSuccess function is called when the user has successfully
            // authenticated and selected an account to use.
            await fetch('/charge', {
              method: 'POST',
              body: JSON.stringify({ public_token: public_token, account_id: metadata.account_id }),
            });
          },
          onExit: async function(err, metadata) {
            // 2b. Gracefully handle the invalid link token error. A link token
            // can become invalidated if it expires, has already been used
            // for a link session, or is associated with too many invalid logins.
            if (err != null && err.error_code === 'INVALID_LINK_TOKEN') {
              linkHandler.destroy();
              linkHandler = Plaid.create({
                ...configs,
                token: await fetchLinkToken(),
              });
            }
            if (err != null) {
              // Handle any other types of errors.
            }
            // metadata contains information about the institution that the
            // user selected and the most recent API request IDs.
            // Storing this information can be helpful for support.
          },
        };

        var linkHandler = Plaid.create(configs);

        document.getElementById('link-button').onclick = function() {
          linkHandler.open();
        };
      })();
    </script>
  </body>
</html>
```

## Testing <a href="testing" id="testing"></a>

No testing is necessary for a spike.  You should simply be able to run the spike and quickly get feedback about the external system.
