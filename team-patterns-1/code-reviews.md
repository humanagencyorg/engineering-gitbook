---
description: How they work.
---

# Code Reviews

## Why Code Reviews?

Code reviews provide a final check before code can be merged into `master`.

Code reviews are the final step in producing easy to read, easy to understand code:

1. &#x20;Planning - talk about how you want to implement the story.
2. Pair Programming - work together with your pair to implement.
3. **Code Review -** 3rd set of eyes to make sure your code is easy to read and understand.

{% hint style="info" %}
If a reviewer outside of the story can understand your code in Code Review, there is a very good chance that the engineer who maintains your code will as well!
{% endhint %}

## Code Review Etiquette

Whether you are providing the code review or receiving the code review, please embrace the following:

1. **Use soft words** - Written words come off much harsher than spoken words.  Be overtly more kind and avoid using negative words.
2. **First seek to understand** - Start your comment with a question. After asking your question, you can provide your thoughts.
3. **Talk in Person** - A quick 5 minute conversation is worth 5 comments.  Talk in person if any comment is unclear.

## Receiving Code Reviews

The goal of the Code Review is to make sure _others can easily understand your code._

### Make is Easy to Review

See the section on [Pull Requests](pull-requests.md) on how to structure your PR to make it easy to understand.  Be sure to tag your PR with `Code Review`.

### Face to Face Conversations

Followup on questions about your code face to face.  This will give you the opportunity to talk through any comments.

### Required Reviews

Your PR will require at least two reviews.  One of these reviews must be from the CTO.

### Graciously accept feedback

Code reviews require a lot of time and focus from the reviewers.  All comments should be viewed as an opportunity to improve.    **All comments should be marked as resolved before merging**.

## Providing Code Reviews

### Leave Timely Feedback

If a review is requested, please have feedback on the PR ASAP and ideally within 4-5 hours.  You want to leave time to followup up on any questions.

Any PR comments after a PR has passed QA should be acknowledged, but can be implemented in a followup PR.

### Leave Early Feedback

Comments in code reviews can be addressed more easily the earlier they are identified.  If you are a required reviewer, use [Github Notifications](https://docs.github.com/en/account-and-profile/managing-subscriptions-and-notifications-on-github/setting-up-notifications/configuring-notifications) and [Github file viewed](https://github.blog/2019-07-01-mark-files-as-viewed/) to view each commit and track changed files.

### Mind your Tone

When leaving feedback, first seek to understand what the engineer is thinking.  Ask a question to dig deeper into the decision.  After that provide your feedback.

{% hint style="info" %}
Remember words read more harshly than spoken language.  Something you might say jokingly in real life can come across as harsh or aggressive when written.
{% endhint %}

### Request Changes vs. Leave Comments

If you are not sure about a suggestion or if it is minor leave your feedback as a comment.

If your feedback prevents the PR from being deployed, leave your review as `Request Changes` .  This will require the comment to be resolved before merging.

Prefer to leave your comments as a "Review" instead of individual comments to prevent comment spam.

~~~~
