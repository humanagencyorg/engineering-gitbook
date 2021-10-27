# Testing

**Avoid lets - **Inspired by the [Thoughtbot blog post "Let's Not"](https://thoughtbot.com/blog/lets-not), our team belief is that we avoid using `let` in Rspec tests.  As the blog post describes, it introduces a "Mystery Guest".  Tests are more difficult to read and understand what is happening.  If a test is getting too long, it may be an indication that there is an opportunity to refactor the source code.&#x20;
