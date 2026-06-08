# learn-ops-api: Service Exploration

## 1. Top-level folders in `learn-ops-api`
 
| Folder          | Why does this folder need to exist?                      |
|-----------------|----------------------------------------------------------|
|.github          | sets up connection with github/actions                   |
|.vscode          | configuration files/project specific settings            |
|config           | deployment configuration files                           |
|LearningAPI      | the meat of the project. Wires everything together       |
|LearningPlatform | holds settings, URL configuration, and Python deployment |
|logs             | holds API requests and logs them                         |
|LogViewer        | html for displaying logs for readability                 |
|static           | custom css for nav-sidebar                               |
|staticfiles      | assets i.e. (media images, CSS, fonts, JS files)         |
|templates        | layout of webpage with logic on how to display data      |


## 2. Folders inside `LearningAPI`

| Folder      | What responsibility does it own and why? |
|-------------|---------------------------------------------------------------|
|fixtures     | data used to prepopulate the environments & run test          |
|migrations   | allows you to modify tables/fields, etc without writing SQL   |
|models       | represents data and the rules around it. Talks to database    |
|serializers  | coverts Python instance to JSON so it can be passed to client |
|tests        | unit tests to make sure application runs as intended/debug    | 
|views        | handles requests, applies logic, decides what to return       |


## 3. What is the Pipfile?
Pipenv's dependency file - the Python equivalent to package.json. Lists packages the project depends on. 

## 4. Key packages in the Pipfile

| Package             | What functionality does it provide and why?              |
|---------------------|----------------------------------------------------------|
| django              | the whole project, the config, the settings, entry-point |
| djangorestframework | ext that turns Django into a framework to build JSON APIs|
| django-allauth      | pluggable auth package. handles the flow of login/register so you don't have to|

## 5. What does `decorators.py` do?
a function that wraps another function. It runs additional logic before (and sometimes after) the wrapped function executes, without changing the wrapped function's own code. i.e is the user logged in?, does the user have permission?, what HTTP methods are allowed?

## 6. What is a serializer, and what serializers are defined here?
Coverts Python instance to JSON so it can be passed to client. You can't send a Python object over HTTP directly. And when data comes in from a request, you need to validate it before trusting it enough to save.
- captsone
- cohort
- nssuser_cohort
- nssuser
- proposal_status
- user


## 7. Models and what they represent

| Model      | Real-world thing it represents                   |
|------------|--------------------------------------------------|
|coursework  | capstones, books, exercises, completed on, attempts, learner  |
|people      | assessments, teams, group projects, scholarship students, cohort, student notes|
|skill       | core skill record, learning record|

## 8. Views vs. viewsets

| Type    | Example class | File path |
|---------|---------------|-----------|
| View    | BookViewSet   | LearningAPI -> views -> book_view.py |
| ViewSet | GithubLogin   | LearningAPI -> views -> github_login.py|

## 9. Serializers paired with their models

| Serializer    | Model   | Link |
|---------------|---------|------|
|capstone       |Capstone | converting all fields in Capstone model to JSON |
|cohort         |Cohort   | converting all fields in Cohort model to JSON     |
|nssuser_cohort |NssUserCohort | converting all fields in NssUserCohort to JSON      |
|nssuser        |NssUser | coverts url, slack handle, github handle, mentor, and user fields to JSON      |
|proposal_status|ProposalStatus | coverts url and status fields to JSON     |
|user           |User | coverts url, first & last name, username, email, and groups to JSON      |

## 10. What replaces the Templates and why?
client-side/front-end replaces the templates because the JavaScript framework handles 100% of the HTML rendering inside the user's browser. 