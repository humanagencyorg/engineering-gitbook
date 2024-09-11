# Serializers

## Overview

The app currently uses [Alba](https://github.com/okuramasafumi/alba) as our JSON serializer.  You can read more about Alba in their [documentation](https://okuramasafumi.github.io/alba/).

The benefit of Alba is that provides a flexible way to model JSON responses based on Ruby objects.

Alba serializers are stored in `app/resources`

## Example <a href="#example" id="example"></a>

```
# frozen_string_literal: true

class ApiKeyResource < ApplicationResource
  attributes :id, :token

  attribute :created_at do |resource|
    resource.created_at.strftime("%B %d, %Y")
  end
end
```

## Testing <a href="#testing" id="testing"></a>

Alba can be tested with simple unit specs.

```
# frozen_string_literal: true

require "rails_helper"

RSpec.describe ApiKeyResource do
  subject { described_class.new(record) }

  let(:record) { create(:api_key, :with_bearer) }
  let(:data) { subject.serializable_hash }
  let(:json) { JSON.parse(subject.to_json, symbolize_names: true) }

  it "has attributes" do
    expect(data).to have_key(:id)
    expect(data).to have_key(:created_at)
    expect(data).to have_key(:token)
  end

  it "matches json schema" do
    expect(json).to eq(
      {
        data: {
          id: record.id,
          token: record.token,
          created_at: record.created_at.strftime("%B %d, %Y")
        }
      }
    )
  end
end
```
