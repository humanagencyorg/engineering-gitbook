# Combatting Flaky Specs

Flaky specs can be annoying at best and completely destructive to team productivity at worst.

There are two reasons why flaky tests can destroy the productivity of a team:

1. A green build is required to merge to production.  Even a 75% pass rate will cause a substantial bottleneck to production. &#x20;
2. Builds are expensive.  This is especially true when you use services like Percy that charges per snapshot.

We can break flaky tests into several categories.

### Flaky by negligence

This is a class of flaky specs that should never occur and should be caught by your pairing partner or in code review.

#### Expecting dynamic numbers and data in visual diff snapshots

There are a couple of sources for this type of flaky spec.  It is especially problematic in snapshot specs where we are doing a visual diff against the last build.

1. **Using the model id in a snapshot** - Guaranteed flaky spec.  The id of the model will always be different every test if your test suite is using database truncation (because the id index is never reset).
2. **Using relative times in a snapshot** - When you use a relative time like `2.hours.from.now` that will always be different every time you run the test.  Normally, that is okay, but in a snapshot the visual diff will be also always be different.  Consider using `Timecop` to freeze time on all snapshot specs.
