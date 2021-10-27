# Serializers

## Overview

This is where you describe how the particular pattern works

**High level point** - Provide a explanation for the high level point

**High level point** - Provide a explanation for the high level point

**High level point** - Provide a explanation for the high level point

**High level point** - Provide a explanation for the high level point

## Example <a href="example" id="example"></a>

Post a code sample of the pattern.  This could be copied from the code base or contrived.

```ruby
# frozen_string_literal: true

module Api
  module V1
    module References
      class BlockSerializer
        include FastJsonapi::ObjectSerializer

        set_key_transform :camel_lower

        attribute :value, &:id

        attribute :title do |object|
          object.blockable.decorate.display_title
        end

        attribute :sub_title do |object|
          object.blockable.decorate.display_response_type
        end
      end
    end
  end
end
```

‌

## Testing <a href="testing" id="testing"></a>

Describe how you can test the pattern that you are documenting

```
Provide a code example here
```
