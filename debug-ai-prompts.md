## Step 2: Claude's trace of the update method

Read LearningAPI/views/student_view.py and focus on the `update` method.

A user reports that updating their Slack handle appears to succeed, but the change is gone after a page refresh. Walk me through what the `update` method does step by step, and identify what would cause a change to `slack_handle` to not persist to the database.

## Step 4: Ask Claude to write the verification test

Look at LearningAPI/tests/test_student_update.py. A test class is already set up with a user, a token, and an NssUser with slack_handle='@old_handle'. The method `test_patch_slack_handle_persists_to_database` currently contains only `fail("Not Implemented yet")`.

Write the body of that method so it:
1. Makes a PUT request to `/students/{pk}/` with a new slack_handle value
2. Asserts the response status is 204
3. Re-fetches the NssUser from the database
4. Asserts the re-fetched record's slack_handle matches the value