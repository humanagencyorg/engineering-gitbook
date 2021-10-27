# The 5 Second Rule

Test Driven Development is about more than just automated tests.  It is about communication.  Here are some best practices to make sure your tests are living up to their promise.

### The 5 Second Rule

> The 5 second rule states that another engineer should be able to verbally describe what a test is doing in under 5 seconds. &#x20;

If it takes longer than 5 seconds, you should consider refactoring your test.  Here are some practices you should consider to achieve the 5 second Rule.

#### Match the title to the Expectation

This is a big one.  If you do this, more than likely your test will pass the 5 second rule.

By matching your spec title to the expectations, it forces you to do a couple of things:

1. Created a clear purpose for that particular spec.
2. Makes your spec only have one expectation.

#### 4 Phase Test

The[ Thoughtbot Blog](https://thoughtbot.com/blog/four-phase-test) originally talked about the 4 phase test.  They specified 4 phases for tests:

1. &#x20;**setup** - create any objects under test
2. **exercise **- execute the object under test
3. **verify** - verify that the object achieved its behave
4. **teardown** - do any necessary cleanup

Our team follows the same structure by placing an empty line between each phase.  We want it easy for the human eye to be able to pick up on each test stage.

```ruby
RSpec.describe ExperienceEvents::FindActions, type: :service do
  context "if experience has no events" do
    it "returns an empty array" do
      type = "experience_completion"
      experience = create(:experience)

      result = described_class.call(experience, event_type: type)

      expect(result).to eq([])
    end
  end
end
```

#### Avoid lets

The Thoughtbot Blog also surfaced the idea of avoiding `let` in specs in their [blog post "Let's Not"](https://thoughtbot.com/blog/lets-not).

This practice is one of the more controversial ideas from Thoughtbot.  Here are their main points:

1. You should never have to debug a test.  That automatically fails the 5 second rule right?
2. Only create the data that you need for the test.  Every extra detail in a test must be considered by the next engineer.

This logic can also be applied to `subject` , `its`, and `before`.  Every time one of these constructs is engaged, you have to search a different part of the spec to understand how that spec works.  Let's keep easy by keeping everything you need to know right there in the spec.

In the spirit of Thoughtbot, let's not when it comes to using `let` and all of its friends.

#### Only relevant details only please

When the next developer works on the spec you wrote, they will (hopefully) be thoughtfully considering every detail that included.  It is implied that if something is included in a spec, it must be essential to the successfully getting the final result.

When you are using factories, you should override and manually set the attributes that are essential to the successful verification of the spec.

When you are creating test data, you should only create the minimum amount of test data required to make the spec pass.  If something is not used or not necessary to get the final answer, please delete it.



