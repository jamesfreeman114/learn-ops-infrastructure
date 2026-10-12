# Debug Notes: The Invisible Save

## 1. Bug report summary

The user's Slack handle isn't updated after they go to change it. They fill out the form, click submit, and no errors appear, but the new Slack handle isn't showing up

## 2. What the debugger showed

| Question | Answer |
|----------|--------|
| Value of `student.slack_handle` before line 87 | None |
| Value of `student.slack_handle` after line 88 | 'slack-id-test-123' |
| Next line executed after line 90 | return Response(None, status=status.HTTP_204_NO_CONTENT)|

## 3. Root cause

student.slack_handle was being updated in memory, but was never being changed in the database because .save() wasn't called before the method was returned. 

## 4. The fix

student.save() - Added on line 89 after "slack_handle" was changed so save the new value to the database.

## 5. What the test verifies

Describe in plain language what `test_patch_slack_handle_persists_to_database` checks and why that is the right thing to test.

The test creates a new user and sends a put request to /students/{nss_user.id} a new slack handle. It first checks for the correct status code on the response by using assertEqual() to verify those values are exactly the same. Then it retrieves the updated version of the user and uses assertEqual() once again to verify the value for nss_user.slack_handle is exactly the same as what was sent in the put request.