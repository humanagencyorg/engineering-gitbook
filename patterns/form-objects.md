# Form Objects

## Overview

FormObjects are a common pattern in Rails.  This app adopted the [ThoughtBot form object](https://thoughtbot.com/blog/activemodel-form-objects).

FormObjects have a couple of purposes:

* **abstracting away the database schema from client side forms** - FormObjects gives us the freedom to change the database schema freely and shield the UI from those changes.\

* **triggering actions before / after save** - FormObjects can hold all of the before / after save actions that need to be performed on a form.  Attaching before / after save actions to a ActiveRecord object can be problematic because the creation of the record and the before / after actions are coupled -&#x20;

## Example

A couple of notes about implementing a FormObject:

**Errors** - To return errors to the clientside, override the `errors` method.  Additionally you can `include ActiveModel::Validations`, to add validations onto your model.

**attr\_accessors** - These are all of the attributes that the `ActiveMode::Model` will possess.  Alternatively, you could override `attributes`.

**save** - can be overridden to allow you to customize logic that runs before and after the object is saved. Alternatively, you could use relations like `belongs_to` and `has_many` and not override `save`.

**Single Transaction** - All records persisted should be inside a single transaction.  If necessary a transaction block can be used, but it is preferred to build objects using `json` and rely on `auto-save`

```ruby
class UgcTextResponse < ApplicationResponse
  include ActiveModel::Model

  attr_accessor :content, :step_resolver, :response

  validates :content, presence: true
  validates :content, length: { maximum: 1000 }

  delegate :last_step?, to: :step_resolver

  def save
    return unless valid?

    response.update!(attributes)
    step_resolver.reload!

    return true unless last_step?

    CompletedResponse.new(response).save!
  end

  private

  def attributes
    { ugc_text: content }
  end
end
```

## Testing

Form objects are unit tested.  A couple of notes:

**Check persistence logic** - It should be checked that objects are persisted to the database correctly.  If using a transaction block, it should be checked the transaction rolls back correctly.

**Mock data** -** **Data that the form object uses can be mocked.  Mocked data can be created through `FactoryBot` or via mocked methods on a Service.

**Stub side effects** - All external changes like API calls, workers, etc. should be stubbed and verified that the method was called with the correct arguments.

```ruby
require "rails_helper"

RSpec.describe WhitelabelDomainFormObjects::Create do

  describe "validations" do
    describe "asker" do
      it "invalid if asker not present" do
        params = {
          domain: "exp.hello.com",
        }

        instance = described_class.new(params)

        expect(instance.save).to be_falsy
        messages = instance.errors.messages
        expect(messages.keys).to eq(%i[asker])
        expect(messages[:asker]).to eq(["can't be blank"])
      end
    end

    describe "domain" do
      it "invalid if domain not present" do
        asker = create(:asker)
        params = {
          asker: asker,
        }

        instance = described_class.new(params)

        expect(instance.save).to be_falsy
        messages = instance.errors.messages
        expect(messages.keys).to eq(%i[domain])
        expect(messages[:domain]).to eq(["can't be blank"])
      end
    end
  end

  describe "#errors" do
    context "when form object have no errors" do
      it "returns domain_record errors" do
        asker = create(:asker)
        params = {
          asker: asker,
          domain: "exp",
        }

        instance = described_class.new(params)

        expect(instance.save).to be_falsy
        messages = instance.errors.messages
        expect(messages.keys).to eq(%i[domain])
        expect(messages[:domain]).to eq(["is invalid"])
      end
    end
  end

  describe "#save" do
    it "creates whitelabel_domain" do
      asker = create(:asker)
      params = {
        domain: "exp.hello.com",
        asker: asker,
      }
      proxy_domain = "expl.ink"

      instance = described_class.new(params)

      ClimateControl.modify WHITELABEL_PROXY_DOMAIN: proxy_domain do
        expect { instance.save }.to change(WhitelabelDomain, :count).
          from(0).
          to(1)
      end
      domain = WhitelabelDomain.last
      expect(domain.domain).to eq(params[:domain])
      expect(domain.asker).to eq(asker)
    end

    context "when domain invalid" do
      it "not creates domain" do
        asker = create(:asker)
        params = {
          domain: "exp.hello.com",
          asker: asker,
        }
        create(:whitelabel_domain, domain: params[:domain])
        proxy_domain = "expl.ink"

        instance = described_class.new(params)

        ClimateControl.modify WHITELABEL_PROXY_DOMAIN: proxy_domain do
          expect { instance.save }.not_to change(WhitelabelDomain, :count)
        end
      end
    end
  end
end
```
