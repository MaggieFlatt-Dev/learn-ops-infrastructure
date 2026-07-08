## Step 2: Claude's trace of the update method

Result:

Found the bug in `update()` at `LearningAPI/views/student_view.py:77-100` — it never calls `.save()` on the model.

```python
def update(self, request, pk=None):
    student = NssUser.objects.get(pk=pk)
    if request.auth.user == student.user or request.auth.user.is_staff:
        if "slack_handle" in request.data:
            student.slack_handle = request.data["slack_handle"]
        if "gitub_handle" in request.data:
            student.gitub_handle = request.data["gitub_handle"]

        return Response(None, status=status.HTTP_204_NO_CONTENT)
```

Setting `student.slack_handle = ...` only changes the attribute on the Python object in memory. Django doesn't write anything to the database until `student.save()` is explicitly called. Since that call is missing, the view returns a "success" 204 response, but the change is discarded when the request finishes — nothing ever touches the database.

Two things noted:
1. **Missing save** — need to add `student.save()` after the `if` blocks, before the `return Response(...)`.
2. **Possible typo** — the second field is checked as `"gitub_handle"` (missing an "h") both in `request.data` and as the attribute name (`student.gitub_handle`). Need to confirm against the `NssUser` model whether the real field is `github_handle` — if so, this key never matches incoming data and also sets a nonexistent attribute.

## Step 3: Comparison to debugger findings

Claude found exactly what I found but he was far more in-depth than I was. I didn't write what line the bug was on and I didn't catch the misspelled github name either. As far as what the issue was, Claude was spot on with what I wrote

## Step 4: Claude writes the verification test

Result:

```python
self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)

nss_user.refresh_from_db()
self.assertEqual(nss_user.slack_handle, "@newhandle")
```

`refresh_from_db()` re-fetches the row from the database into the existing `nss_user` object, overwriting any in-memory-only changes with what's actually persisted. Checking `response.status_code` alone isn't enough proof of persistence, since the buggy view returns 204 regardless of whether `.save()` ran. The second assertion is what actually catches the missing save.

## Step 5: Test failure output

Ran with `pipenv run python -m pytest LearningAPI/tests/test_student_update.py`:

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

The test fails exactly as expected: the response comes back 204 (success), but the value in the database is still `'@old_handle'` because `update()` never calls `.save()`. This confirms the diagnosis from Step 2.

Note: running the test required `pipenv install --dev` first, since `pytest` and `pytest-django` (listed as dev-packages in the Pipfile) weren't installed in the virtualenv yet. That command regenerated `Pipfile.lock`.

Also re-ran it inside the actual containerized app, from `learn-ops-infrastructure` (where `docker-compose.yml` lives, mounting `../learn-ops-api` as `/app`):

```
docker compose exec api pytest LearningAPI/tests/test_student_update.py -v
```

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
Using existing test database for alias 'default' ('file:memorydb_default?mode=memory&cache=shared')...
=========================== short test summary info ============================
FAILED LearningAPI/tests/test_student_update.py::StudentPartialUpdateTests::test_patch_slack_handle_persists_to_database - AssertionError: '@old_handle' != '@newhandle'
1 failed in 0.85s
```

Same failure as the local pipenv run — confirms the bug reproduces inside the actual containerized app, not just the local dev environment.

## Step 6: Ask Claude for the fix

Result:

Add `student.save()`, placed after the two `if` blocks that set the attributes but before `return Response(...)` — while still inside the `if request.auth.user == student.user or request.auth.user.is_staff:` block:

```python
if "slack_handle" in request.data:
    student.slack_handle = request.data["slack_handle"]
if "gitub_handle" in request.data:
    student.gitub_handle = request.data["gitub_handle"]

student.save()          # <- add this line

return Response(None, status=status.HTTP_204_NO_CONTENT)
```

Reasoning: the attribute assignments only mutate the in-memory Python object; `.save()` is what issues the actual `UPDATE` to the database. It must run before the response is returned, and it must stay inside the authorization check so unauthorized requests can't persist changes.

## Step 7: Test passing output
After manually applying the fix, re-ran the test in the container to confirm:

```
docker compose exec api pytest LearningAPI/tests/test_student_update.py -v
```

```
LearningAPI/tests/test_student_update.py::StudentPartialUpdateTests::test_patch_slack_handle_persists_to_database PASSED [100%]

1 passed in 1.11s
```

Confirms `student.save()` persists the slack handle change to the database — bug fixed.