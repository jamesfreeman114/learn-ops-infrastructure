## Step 2: Claude's trace of the update method

The cause: the `update` method changes `slack_handle` on the Python object but never calls `student.save()`, so nothing is written to the database. It still returns 204, so the frontend thinks the update worked.

## What `update` does, step by step

See [student_view.py:77-100](learn-ops-api/LearningAPI/views/student_view.py#L77-L100).

**1. Look up the student (line 84)**
```python
student = NssUser.objects.get(pk=pk)
```
This runs a `SELECT` and gives you `student`, a Python object built from that database row. Think of it like fetching JSON into a local variable in JavaScript: changing the variable doesn't change the server.

**2. Check permissions (line 86)**
```python
if request.auth.user == student.user or request.auth.user.is_staff:
```
`request.auth.user` is the logged-in user, worked out from their token. The update is allowed if they're editing their own record or they're staff. Otherwise you get a 401 (line 94).

**3. Copy the new values onto the object (lines 87-90)**
```python
if "slack_handle" in request.data:
    student.slack_handle = request.data["slack_handle"]
```
`request.data` is the parsed JSON body of the PUT request. This line only changes the **in-memory** attribute. The database row hasn't changed yet.

**4. Return success (line 92)**
```python
return Response(None, status=status.HTTP_204_NO_CONTENT)
```
A 204 means "succeeded, nothing to send back." The frontend gets this and shows success.

**5. Error handling (lines 96-100)**
If the student doesn't exist you get a 404, and any other exception gives a 500.

## The bug

Between steps 3 and 4 there's no call to `student.save()`.

In Django, setting an attribute on a model instance does **not** touch the database. Only `.save()` sends the `UPDATE ... SET slack_handle = ...` SQL. Without it, the changed `student` object is simply thrown away when the function returns.

That's why the user sees success and then loses the change on refresh: the next GET reads the row, which never changed.

## A second bug next to it

Lines 89-90 check for and assign `gitub_handle` (missing the "h"). The model field is `github_handle` ([nssuser.py:16](learn-ops-api/LearningAPI/models/people/nssuser.py#L16)), so:
- A client sending `"github_handle"` won't match the `if`, so it's silently ignored.
- Even if `"gitub_handle"` were sent, `student.gitub_handle = ...` would just add a stray Python attribute that Django doesn't know about. It wouldn't be saved even after you add `.save()`.

## Your turn to fix it

Since you like to write the fix yourself, here are some hints:
- Add one line after the `if` blocks and before the `return`. It's a method on `student`.
- Optional: `save()` accepts `update_fields=[...]` to write only certain columns. Look it up in the Django docs.
- Fix the field name in both places on lines 89-90.

To check it, send a PUT with a new `slack_handle`, then do a GET (or look at the row in the database) and confirm the value changed.


## Step 3: Comparison to debugger findings

Claude confirmed the cause of the bug was that .save() was never called, and provided more detail on what Django does with SQL when .save() runs. Claude's suggestion on where to put .save() was different that what I did. Manually, I put save on the very next line within the "if" block (Line 89). Claude recommended putting it on line 91 before the request returns(Which would include both the slack and github handle changes). Claude pointed out the bug would still persist if a user went to change their github handle instead.

## Step 4: Ask Claude to write the verification test


### Replace lines 30-32 with this

```python
        # Step 2: the API should report success
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)

        # Step 3: re-fetch the record from the database
        refreshed = NssUser.objects.get(pk=nss_user.id)

        # Step 4: the new value should actually be stored
        self.assertEqual(refreshed.slack_handle, "@newhandle")
```

### Why each line is there

- **`self.assertEqual(response.status_code, ...)`**: `APITestCase` comes from Python's `unittest.TestCase`, so you get assertion methods like `assertEqual(actual, expected)`. It's similar to `expect(x).toBe(y)` in Jest. `status.HTTP_204_NO_CONTENT` is just the number 204 with a readable name, and it's already imported on line 2.
- **`NssUser.objects.get(pk=...)`**: this matters most. You **can't** check `nss_user.slack_handle`, because `nss_user` is the in-memory object you created before the request. It still says `@old_handle` whatever the API did. A fresh `.get()` runs a new `SELECT`, so you're checking what's actually stored. That's exactly the bug from before.
  - Another way you'll see: `nss_user.refresh_from_db()`, which reloads the existing object from the database in place.
- **The final `assertEqual`** is the check that would have caught the missing `.save()`.

### Two things to know

1. **Don't add a trailing slash to the URL.** Your request said `/students/{pk}/`, but [urls.py:25](learn-ops-api/LearningPlatform/urls.py#L25) sets up the router with `trailing_slash=False`. The existing `f"/students/{nss_user.id}"` is correct, and adding `/` would give you a 404.
2. **This test should fail at first.** If you haven't added `student.save()` to the view yet, the 204 check passes but the last assertion fails (`'@old_handle' != '@newhandle'`). That's what you want: it shows the test catches the bug. Add the `.save()`, run it again, and it should pass. Writing a failing test before the fix is called "red, then green."

Once `fail(...)` is gone, the `from pytest import fail` import on line 7 isn't used anymore. You can delete it.

