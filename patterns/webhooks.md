---
description: Receiving updates from external services.
---

# Webhooks

## Overview

A webhook can be used to be alerted to a change our application that a change took place in an external service.  A couple of things:

1. **are not guaranteed** to be delivered in the order of the events which occurred.
2. **are not guaranteed** to be unique and could be duplicated.
3. **are not guaranteed** to be retried (varies by service)
4. **are not guaranteed** to succeed. Webhooks will time out for requests that block for too long (varies by service).

Given these constraints, we apply the following best practices when building webhooks:

**Webhook endpoints should delegate to workers** - In the event of high load, it is important that webhooks are captured and not lost if an application is not able to respond quickly enough to prevent the webhook from timing out.  Each webhook should enqueue the contents of the webhook so that it can be processed asynchronously.

**Never trust data the payload of the webhook** - Given that the order of webhooks is not guaranteed, race conditions can occur when data from the webhook payload is used to update the database.  Any data in the webhook data payload should be considered out of data upon receipt. &#x20;

{% hint style="info" %}
Query the API to get the most up to date information about the resource. &#x20;

Given we are listening to webhooks from **Product A**.  The webhook contains the `id` and `current_price` of **Product A.  **When a webhook is received, query `/api/v1/products/{id}` to get the most recent price of** Product A.**
{% endhint %}

**Mind the Open, Closed Principle** - The [Open Closed Principle](https://blog.cleancoder.com/uncle-bob/2014/05/12/TheOpenClosedPrinciple.html) states that a class should be open to extension, but closed to modification.  Consider moving most logic out of the webhook handling.

**Save webhooks in the database** - Consider persisting the contents of a webhook to the database.  It will be easier to reference the webhook in workers as well as debug webhooks.

**Webhooks should be authenticated** - Webhooks should have a mechanism that enables our application to verify the authenticity of the sender.  This varies by service.

{% hint style="info" %}
Stripe provides a [library](https://stripe.com/docs/webhooks/signatures) for verifying webhooks.  Twilio provides [a header](https://www.twilio.com/docs/usage/webhooks/webhooks-security) or rack based [middleware](https://www.twilio.com/docs/usage/tutorials/how-to-secure-your-sinatra-app-by-validating-incoming-twilio-requests#).
{% endhint %}

## Local Development

**Debugging Webhooks** - Webhooks can be tricky to debug because your webhook request will timeout if you place a breakpoint in your code.  It is better to debug in a worker that can be retried or use print statements.

**Webhooks require SSL** - On production webhooks will require SSL as any request.  Locally, this can vary by service.  In localy development, SSL may be required as well.

{% hint style="info" %}
Stripe will not require SSL if you use the [Stripe CLI to listen to hooks](https://stripe.com/docs/stripe-cli/webhooks).
{% endhint %}

{% hint style="info" %}
Use ngrok make your local environment accessible to the internet over an SSL connection.
{% endhint %}

## Example <a href="example" id="example"></a>

Here is an example of receiving webhooks from Stripe.

```ruby
module Api
  module V1
    class StripeWebhooksController < ::Api::V1::ApplicationController
      def create
        status = 200
        payload = request.body.read

        json = JSON.parse(payload, symbolize_names: true)
        HaLogger.info payload
        begin
          event = Stripe::Event.construct_from(json)
          StripeServices::HandleWebhookEvent.call(event)
        rescue JSON::ParserError, Stripe::SignatureVerificationError
          status = 400
        end

        render json: {}, status: status
      end
    end
  end
end

module StripeServices
  class HandleWebhookEvent < ApplicationService
    def initialize(event)
      @event = event
    end

    def call
      case @event.type
      when "capability.updated"
        stripe_account_id = @event.data.object.account

        check_capabilitities(stripe_account_id)
    end

    private

    def check_capabilitities(stripe_account_id)
      return unless Asker.find_by(stripe_user_id: stripe_account_id)

      CheckCapabilitiesJob.perform_later(stripe_account_id)
    end
  end
end
```

## Testing <a href="testing" id="testing"></a>

Webhooks should be tested using request specs.  The format of the webhook payload should be determined by first doing a [spike](spike.md).  A couple of considerations:

**Request Specs should not stub** - Request specs should verify that a record in the database changed.  They should not stub.  These specs should run Sidekiq inline so that jobs are automatically processed/

**Controller Specs for unit testing** - Controller specs can be used to stub out the call to your worker.

{% hint style="info" %}
The [stripe-ruby-mock gem](https://github.com/stripe-ruby-mock/stripe-ruby-mock) provides examples of many of the webhook payloads.
{% endhint %}

```ruby
require "rails_helper"

RSpec.describe "Stripe webhooks", stripe: true, type: :request do
  context "when stripe returns invalid JSON" do
    it "returns bad request" do
      event = StripeMock.mock_webhook_event("capability.updated.card_payments")

      allow(Stripe::Event).to receive(:construct_from).
        and_raise(JSON::ParserError)

      post api_v1_stripe_webhooks_path, params: event, as: :json

      expect(response.status).to eq 400
    end
  end

  context "when stripe returns verfication error" do
    it "returns bad request" do
      event = StripeMock.mock_webhook_event("capability.updated.card_payments")

      allow(Stripe::Event).to receive(:construct_from).
        and_raise(Stripe::SignatureVerificationError.new("bogus", "sign_body"))

      post api_v1_stripe_webhooks_path, params: event, as: :json

      expect(response.status).to eq 400
    end
  end

  context "when event is capability.updated" do
    it "calls CheckCapabilitiesJob for that stripe user" do
      event =
        StripeMock.mock_webhook_event("capability.updated.card_payments")
      stripe_user_id = event.data.object.account
      create(:asker,
             stripe_user_id: stripe_user_id,
             capability_card_payments: "inactive")
      check_capabilities_job = class_double("CheckCapabilitiesJob").
        as_stubbed_const
      allow(check_capabilities_job).to receive(:perform_later)

      post api_v1_stripe_webhooks_path, params: event, as: :json

      expect(check_capabilities_job).to have_received(:perform_later).
        with(stripe_user_id).once
    end

    it "logs the card payment capability" do
      event = StripeMock.
        mock_webhook_event("capability.updated.card_payments")
      allow(HaLogger).to receive(:info)

      post api_v1_stripe_webhooks_path, params: event, as: :json

      expect(HaLogger).to have_received(:info).
        with(event.to_json).once
    end
  end
end
```
