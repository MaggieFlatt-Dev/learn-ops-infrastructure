## Step 2: Claude's trace of the update method 

Prompt:
> look at LearningAPI/views/student_view.py and explain why the slack handle change is not being saved in update

## Step 4: Claude writes the verification test

## Step 5: Test failure output

```
LearningAPI/tests/test_student_update.py::StudentPartialUpdateTests::test_patch_slack_handle_persists_to_database FAILED [100%]

=================================== FAILURES ===================================
____ StudentPartialUpdateTests.test_patch_slack_handle_persists_to_database ____
LearningAPI/tests/test_student_update.py:32: in test_patch_slack_handle_persists_to_database
    self.assertEqual(nss_user.slack_handle, "@newhandle")
E   AssertionError: '@old_handle' != '@newhandle'
E   - @old_handle
E   + @newhandle
---------------------------- Captured stderr setup -----------------------------
Using existing test database for alias 'default'...
=========================== short test summary info ============================
FAILED LearningAPI/tests/test_student_update.py::StudentPartialUpdateTests::test_patch_slack_handle_persists_to_database
1 failed in 5.39s
```

Confirms the diagnosis from Step 2: the response is 204 (success), but `nss_user.slack_handle` is still `'@old_handle'` after `refresh_from_db()`, because `update()` never calls `.save()`.

Follow-up prompt:
> run the test using docker compose exec api pytest LearningAPI/tests/test_student_update.py -v

## Step 6: Ask Claude for the fix

Prompt:
> What single line do I need to add and where in the method does it belong to fix the bug?

## Step 7: Rerun test 

Prompt:
> rerun the test using docker compose to confirm it passes now
