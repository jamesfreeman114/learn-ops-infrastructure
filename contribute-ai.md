## Step 3: /conventions learn-ops-client.

The project has one linting tool, ESLint, set up through the `react-app` preset.

## ESLint

**Command:**
```bash
cd ~/workspace/lms/learn-ops-client && npx eslint src
```
Add `--fix` to auto-fix what it can.

**Config:** `~/workspace/lms/learn-ops-client/package.json`, key `"eslintConfig": { "extends": "react-app" }`

It uses ESLint 8.41.0 and `eslint-config-react-app` 7.0.1. The project adds no custom rules on top of the preset.

### Plugins
- `eslint-plugin-react`: React and JSX checks
- `eslint-plugin-react-hooks`: rules for using hooks correctly
- `eslint-plugin-jsx-a11y`: accessibility
- `eslint-plugin-import`: import and export statements
- `eslint-plugin-flowtype`: Flow type annotations (not used in this project)

### Rule categories
| Category | Example rules | What they catch |
|---|---|---|
| Possible bugs | `no-undef` (error), `no-dupe-keys`, `array-callback-return`, `no-unreachable` | Undefined variables, repeated object keys, a `.map()` callback with no `return` |
| Code quality | `no-unused-vars`, `eqeqeq`, `default-case`, `no-useless-escape` | Unused variables, `==` instead of `===`, a `switch` with no `default` |
| React Hooks | `react-hooks/rules-of-hooks` (error), `react-hooks/exhaustive-deps` | Hooks called conditionally, missing `useEffect` dependencies |
| React/JSX | `react/jsx-no-undef` (error), `react/jsx-no-target-blank`, `react/no-direct-mutation-state` | Components used without importing them, unsafe `target="_blank"` links |
| Accessibility | `jsx-a11y/alt-text`, `jsx-a11y/anchor-is-valid` | `<img>` with no `alt`, `<a>` with no real `href` |
| Imports | `import/first` (error), `import/no-anonymous-default-export` | Imports placed after other code, unnamed default exports |
| Browser globals | `no-restricted-globals` (error) | Bare `event`, `name` or `location` used without `window.` |

### Disabled rules
**In the config:** none for this project. The preset turns off 9 rules, including `no-undef` and `no-unused-vars`, but only for `.ts`/`.tsx` files, and the project has none.

**In the code:**
| Location | Comment | What it would have flagged |
|---|---|---|
| `src/components/dashboard/student/StudentDashboard.js:20` | `// eslint-disable-next-line` | `no-unused-vars` on `firstName`. With no rule named, every rule is off for that line. |
| `src/components/utils/HumanDate.js:21` | `//eslint-disable-next-line` | `no-useless-escape` on the `'\/'` in `.replace()` |
| `src/components/cohorts/StudentCapstoneList.js:55` | `/* eslint-disable no-undef */` | `structuredClone`, a real browser API this ESLint version doesn't recognize. The comment turns `no-undef` off from line 55 to the end of the file. |
| `src/components/cohorts/StudentCardList.js:152` | `/* eslint-disable no-undef */` | Same as above: `structuredClone`, with `no-undef` off from line 152 to the end of the file. |

## Pylint

**Command:**
```bash
cd ~/workspace/lms/learn-ops-infrastructure && docker compose exec api pylint LearningAPI/
```

**Config:** `~/workspace/lms/learn-ops-api/.pylintrc`
```ini
[FORMAT]
good-names=i,j,ex,pk

[MESSAGES CONTROL]
disable=broad-except,arguments-differ,missing-function-docstring
```

### Plugins
- **`pylint-django`** (in `Pipfile`): teaches Pylint about Django, such as model `.objects` and related fields. `.pylintrc` doesn't load it, so Pylint reports Django code it doesn't understand as `no-member` errors.

### Rule categories
| Category | Example rules | What they catch |
|---|---|---|
| **E**: Error | `E1101 no-member`, `E0602 undefined-variable`, `E0307 invalid-str-returned` | Attributes that don't exist, undefined names, a `__str__` that might not return a string |
| **W**: Warning | `W0611 unused-import`, `W0613 unused-argument`, `W0102 dangerous-default-value` | Unused imports and arguments, `def f(x=[])` |
| **C**: Convention | `C0103 invalid-name`, `C0114 missing-module-docstring`, `C0301 line-too-long` | Names that break PEP 8, missing docstrings, lines over 100 characters |
| **R**: Refactor | `R0903 too-few-public-methods`, `R1705 no-else-return`, `R1725 super-with-arguments` | Code that is too complex or written in a roundabout way |
| **Django** (`pylint-django`) | `W5101 model-missing-unicode`, `E5110 django-not-configured` | Django-specific mistakes in models and setup |

`good-names=i,j,ex,pk` lets those short names pass `invalid-name`.

### Disabled rules
**In `.pylintrc`:**
| Rule | What it would have flagged |
|---|---|
| `broad-except` (W0718) | `except Exception:` blocks (see below) |
| `arguments-differ` (W0221) | A method that overrides a parent method with a different signature, e.g. DRF's `def list(self, request)` |
| `missing-function-docstring` (C0116) | Functions and methods without a `"""docstring"""` |

**Inline** (`# pylint: disable=...`):
| Location | Code | What it hides |
|---|---|---|
| `LearningAPI/views/team_maker_view.py:118` | `E1101` no-member | `team.students.all()` |
| `LearningAPI/models/coursework/capstone_timeline.py:12` | `E1101` | `self.capstone.student.user.first_name` |
| `LearningAPI/models/people/assessment.py:24, 31, 32` | `E1101` | `self.book.id`, `self.book.course.name` |
| `LearningAPI/models/coursework/course.py:10` | `E0307` invalid-str-returned | `__str__` return type |
| `LearningAPI/models/coursework/foundation_exercise.py:18` | `E0307` | Same as above |
| `LearningAPI/models/people/assessment.py:35` | `E0307` | Same as above |

The `E1101` lines are Django false alarms that `pylint-django` would handle.

### What `broad-except` would flag
It flags `except Exception:`, which catches **every** error instead of the specific one you expect. Example from `LearningAPI/views/project_view.py:30-35`:

```python
try:
    project.save()
    serializer = ProjectSerializer(project)
    return Response(serializer.data, status=status.HTTP_201_CREATED)
except Exception as ex:          # ← flagged
    return Response({"reason": ex.args[0]}, status=status.HTTP_400_BAD_REQUEST)
```

Any error inside the `try` becomes a 400 "bad request". That includes a typo in the serializer or a database outage, so real bugs get hidden. Catching a specific error such as `IntegrityError` limits the 400 response to the failure you actually expect. If the rule were enabled, Pylint would report this pattern **75 times** in `LearningAPI`.
