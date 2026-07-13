# Conventions Notes

## JavaScript Linter (learn-ops-client)

### Configuration
Extends: react-app
Plugins active: import, flowtype, jsx-ally, react-hooks

### Rules I found (at least 5)
Write each rule name, its severity, and what it enforces as a plain sentence.
1. no-array-constructor : warn : disallows array constructors
2. no-delete-var : warn : disallows deleting variables. Can't use the delete operator on a variable
3. no-eval : warn : disallows the use of eval() preventing potentially dangerous, unnecessary, and slow code
4. no-labels : warn : disallow labeled statements 
5. no-loop-func : warn : disallow function declarations that contain unsafe references inside loop statements

### First warning fixed
File and line: LearnOps. js line 8
Rule: no-unused-vars
Fix: deleted unused variable/import 

## Python Linter (learn-ops-api)

### Configuration
Disabled rules and what each would flag:
broad-except: flags except Exception - catching all exceptions can hide bugs
arguments-differ: when a subclass overrides a method with a different parameter signature
missing-function-docstring: requires a docstring on every function

Good names allowed: i, j, ex, pk

What pylint-django likely adds:
- where the error is located and the category 

### First warning fixed
File and line: LearningAPI/metrics.py line 34 & 1
Rule and category: Final newline missing C/ Missing module docstring C
Fix: added a newline at bottom of module and added a docstring to the top 