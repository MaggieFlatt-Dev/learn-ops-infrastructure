# Trace Notes - Create a Student Note

## Request Path

| Layer | File | Class / Function | What it does |
|-------|------|-----------------|--------------|
| UI dialog |  StudentNoteDialog.js| createStudentNote| displays all the student notes & allows user to create new note |
| API helper | Fetch.js | fetchIt | Handles all fetch requests|
| URL router | urls.py | router.register(r'notes', views.StudentNoteViewSet, 'note') | When request comes in redirects to StudentNotesViewSet|
| View | student_note_view.py | StudentNoteViewSet | handles the CRUD operations of the student notes|
| Serializer | student_note_view.py | StudentNoteSerializer | coverts python object to json for the frontend |
| DB | | | note gets saved to database|
| UI refresh | PeopleProvider.js | getStudentNotes | fetches all student notes|

## Sequence Diagram

[Excalidraw link](https://excalidraw.com/#...)

![Diagram](./trace-diagram.png)