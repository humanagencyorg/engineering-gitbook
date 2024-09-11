# Stories

A story represents a complete use case that provides value to the user.

A good story should:

* enumerate exactly who we intend on helping
* show how our change will work
* explain why our change is needed

We use the Getting Things Done story format to communicate user needs and how we are going to help them.  Getting Things Done story format looks like the following:

```
When a person I want to help is in a particular state
They can do something new and amazing
So that their problem is solved
```

In addition to Getting Things Done, a story should also define acceptance criteria that provide more details about how the story will work.  The acceptance criteria should **UI agnostic**.  The criteria should describe the various states that may be encountered and how they should be handled.

Here is an example of effective acceptance criteria:

```
when a creator is doing something
    they can press a button
        # describe what the button does
        # describe an alternative scenarios that might exist when the button is pressed
when a different user interacts with the app
    they can view the result
        # describe how the result looks
        # describe alternative scenarios that might influence how the result
```

Finally, a story can contain product notes.  These are additional notes that provide context to the story from the point of view of the user.  It may also reference behavior elsewhere in the app.

Here's an example of a story for email verification.

```
## Story

> When a creator has integrated Hubspot
They sync a Formli response or payment to a Hubspot Object
So that they update an existing object and run a workflow
> 

## Acceptance Criteria:

- [ ]  when a creator is creating an action
    - [ ]  they can choose a “Sync to Hubspot Object” action
        - [ ]  the creator can select the object
            - [ ]  “Contact”
            - [ ]  “Company”
            - [ ]  “Deal”
            - [ ]  “Ticket”
            - [ ]  or any custom object
        - [ ]  the creator can search for the object
            - [ ]  select a Hubspot attribute
            - [ ]  reference a value from Formli
        - [ ]  the attributes for that custom object are pulled into Formli
        - [ ]  referenced values from Formli can be mapped to the Hubspot object attributes
- [ ]  when the action runs
    - [ ]  the referenced value are updated for the searched object
        - [ ]  only mapped fields are updated (all unmapped fields are not updated)
    - [ ]  when a search returns more than one object
        - [ ]  only the first object is updated

## Product Notes

Hubspot Form submissions can only create new objects.   This is an issue anytime you want to update an existing record.
```
