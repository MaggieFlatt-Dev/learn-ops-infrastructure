# Debug Notes: The Invisible Save

## 1. Bug report summary

What did the user report? What behavior did you observe when reproducing it?
- User reported when they tried to add their slack Id to their profile they were receiving a 204 that is was created but the slack Id was not being saved. When I tried to reproduce the bug I noticed that the slackId was being held in state but was not being saved to the database once user clicked update. 

## 2. What the debugger showed

| Question | Answer |
|----------|--------|
| Value of `student.slack_handle` before line 87 | None |
| Value of `student.slack_handle` after line 88 |  my SlackId |
| Next line executed after line 90 | 92: return Response(None, status=status.HTTP_204_NO_CONTENT) |

## 3. Root cause

What was the underlying reason the change was not saved? What Django concept does this come back to?
- There was not a .save() anywhere in the update so the new slackId never got saved. In-Memory State vs. Persistent State

## 4. The fix

What line did you add? Where exactly in the method?
- I added student.save() to line 91 after the try and if block but before the exceptions and return 

## 5. What the test verifies

Describe in plain language what `test_patch_slack_handle_persists_to_database` checks and why that is the right thing to test.
- this test makes a PUT request to students/{pk} with the new slack_handle value. It asserts that the response status is 204, then refetches the NssUser from the database. The refetch asserts that the refetched slack_handle matches the value you sent. It's the right thing to test because that is where the bug has been detected. 