# Retrospectives

## Structure

Retrospectives provide the structure for the team to reflect on the previous sprint and note three things:

1. What was the team doing well and should continue doing?
2. What pain or issues did the team experience during their work?
3. What can you do to remedy the issues?

Retrospectives have varying formats.  Generally, we keep it pretty simple:

1. &#x20; Everyone takes a turn saying what went well.
2. &#x20;Everyone takes a turn saying things that were difficult.
3. &#x20;Everyone takes a turn suggesting ideas on how to improve.

{% hint style="info" %}
When talking about things that were difficult, try to describe the problem as much as possible; avoid thinking of the solution.  This will allow better conversation in when discussing solutions as a team.
{% endhint %}

## Facilitating

The facilitator is the "Emcee" for the retrospective.  Here is a list of their general responsibilities:

### Pace the meeting

Here's what you should do to ensure that the meeting progresses smoothing through topics

* **Record people's comments**.  Try to write these as close to what they say as possible.  When the participant is done speaking ask them if the notes match their comment.
* **Encourage and limit conversation**.  Generally give any given topic 5 minutes of discussion.  If it looks like a topic will go long, add an action item to discuss the topic after the retrospective.
* **Wrap it up**.  As the facilitator you are responsible for making sure the meeting ends at the specified time.  Retrospectives should only be extended if you can ask the participants if they would like to extend the meeting and they all agree.

### Record Followup Items

This is how a retrospective ideas get put into action.  Every comment from the "Should change" section should have an owner.  If no one wants to own it, then it is likely not important enough for us to change.

Your job as facilitator is to translate these ideas into action items with an owner.  This should take you 20 minutes after the completion of the retro.

* Create items in Notion or your task management system.
* Share all of the created items in Slack and tag the owner of the task.
* Ensure that the items are appropriately prioritized so that they do not get lost.

### Example

Here is an example of a team retrospective.

```
Start Time 10AM / 6PM
End Time 11AM / 7PM

## 🔥 Positives

- Hiring new front end developer Lesia! +7
- QA Reference Book 👍. Props to @Samantha Bursac @Iryna Lopina +3
    - Very descriptive with screenshots 👍
- Everyone working on the Billing story +6
    - We split this story to small pieces
    - Addressed some issues with existing flow
    - Cool to ship with small pieces +1
- Good things happening with schedule @Young Jung leading story +1
- Closed Beta for Test Analytics with BuildKite! 🙌 @Alex Beznos
- Dependabot is back alive +1
- New instance type for Buildkite will save some money 🙌 @Alex Beznos
- Team has been working hard and good patience when QA and PO kick back stories +1
- Live Preview - Team Effort 🙌 +2
- Spike with **Official** Notion API 🍻 +1
- Fun time working together in UA 🇺🇦
- New devices in the office 🖥
- New Help deck articles coming 📝
- Great to have team members visit and collaborate in person
- Shoutout to Vitalii for holding up the front end for the entire team! +2
- Congrats to Vova and Lena and Ivan 👶
- Swat and Andrey getting over the line as a pair
- QA @Iryna Lopina @Samantha Bursac
- Shoutout to @Vlad Solin
- 

## 🔧 Concerns, questions, opportunities, things to improve

- QA missed some obvious bugs +1
    - There were misspellings in the app.
    - We do not have regression plan
    - Started working on this but in progress
- Small stories are great, but the order of stories can be difficult to understand +1
    - some things can be tested, but other things cannot be tested. +1
        - Upgrade story is connected to the remove branding story
    - how can we better represent the order that stories ship
- ✅ Code Reviews give our team pain +4
    - Can we figure out rules to avoid conflicts in the review.
    - Story in PO and received comments on code
- ✅ Dwaynebot does not work and causes pain +5
- ✅ Heroku review application creation is really slow! 🐢 +2
- ✅ We have a list of URLs which should never be changed easily
    - URLs that we provide to Stripe
    - URLs for widget
    - these require surgical precision!~
    - Caused a bug with Stripe Express Connect Authorization (quiet failure!)
- Hide Branding branch had to be recreated from scratch +1
    - ✅ Perhaps working branches can use rebase flow +1
        - Would simplify merging branches
- ✅ Count of Percy snapshots is high!
    - We pay for all of the snapshots
    - It is difficult to know about all snapshots; tough to know about duplicated snapshots
- Couple of issues found in PO during the past few weeks

## What to do about it

- Everyone giving an update on their card at the end of the day
- Rewrite DwayneBot
    - Create a card and prioritize for rewriting Dwaynebot
    - Just get out of Alex's way and give him some time!
        - Alex already has spike, Slack integration, just needs time.
- Code Reviews
    - Create some engineering documentation about Code Reviews
        - Ideas:
            - If it is past Code Review column, then Code Review is closed or can only be suggestions
            - Fast feedback on Code Reviews.  If a review is requested, then it needs to be reviewed immediately so they can get feedback (4-5 hours)
            - Have a meeting in the calendar when everyone can request reviews.  All reviews should happen during meeting.
            - New team members unsure about approving can leave comments if they are not 100% about good PR
            - Tickets in PO should not have tag for Code Review.  Perhaps we should remove it after it is past.
            - No Code Review label means no comments.
            - How do we handle early feedback on Code Reviews?
                - Mike needs to review PRs early so that he can get through backlog
                - Mike needs to give feedback earlier so that it can be less work to change later on.
                - Early is okay, but late is really bad!  If it is in PO and you receive comments that is annoying!
            - Leaving a PR description helps the reviewers get into the PR more quickly and understand it.
            - Leaving comments on code for the Code Review
    - Heroku review applications being slow
        - Should we not do anything about it now? not so critical
        - Add ENV variable that allow NPM caches to be cached.  will save 88 seconds!
        - Create card.
        - Automatically create review apps with PR.
        - Iryna and Sam can grab the review app URL from Heroku and add to card.
- Percy snapshots
    - Task to cleanup and prioritize
        - Ben can work with an engineer
            - raw export of every view name
            - ax unnecessary snaphots
        - cut processing time for snapshots
        - cut costs for snapshots
    - Task for eliminating flaky Percy snapshots
- We have a list of URLs which should never be changed easily
    - Check on Buildkite
    - Custom rubocop rule that would let us know that we are changing a dependent URL
        - List of all URLs which should never change
        - How will we change over time?
            - Will require backward compatibility?
            - Will need logger or monitoring to see if old URLs usable.
        - Monitoring heath checkers for URLs
        - Create task
- Perhaps working branches can use rebase flow +1
    - Will Github have a button that allows us to `update and rebase`
```

The tasks were defined in a task manager using the retro notes and were shared out to the team via Slack.

![](<../.gitbook/assets/Screen Shot 2021-10-27 at 11.33.22 AM.png>)

![](<../.gitbook/assets/Screen Shot 2021-10-27 at 11.32.37 AM.png>)
