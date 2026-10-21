# Conventions Notes

## JavaScript Linter (learn-ops-client)

### Configuration
Extends: react-app
Plugins active: import, flowtype, jsx-ally, react-hooks

### Rules I found (at least 5)
array-callback-return  - warning
no-iterator - warning
no-function-assign - warning
no-undef - error
react-hooks / rules-of-hooks - error


### First warning fixed
File and line: /learn-ops-client/src/components/LearnOps.js Line 8

Rule: no-unused-vars warning  'CohortProvider' is defined but never used

Fix: Removed the import


## Python Linter (learn-ops-api)

### Configuration
Disabled rules and what each would flag:

1. broad-except: flags except Exception: because catching all exceptions can hide bugs.
2. arguments-differ: fires when a subclass overrides a method with a different parameter signature. Django ViewSets override methods this way by design, so the rule is disabled.
3. missing-function-docstring: missing-function-docstring: requires a docstring on every function. Disabled here because descriptive test method names already communicate intent.

Good names allowed:

i,j, ex, pk


What pylint-django likely adds:

Takes django naming conventions and syntax into account when linting python code. Avoids false errors and includes Django-specific checks

### First warning fixed
File and line: LearningAPI/admin.py:34:0

Rule and category: C0301

Category: convention 
Rule name: Line too long

Fix: Started a new line before the for loop in this part of the code.
