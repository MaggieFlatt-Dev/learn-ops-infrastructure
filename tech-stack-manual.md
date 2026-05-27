# Tech Stack Manual

## 1. System Overview
This system is a application used by NSS instructors to track students progress throughout the boot camp. 

Instructors can join a cohort to better track their assigned cohort they are leading.

Instructors can see when a student has finished their assessments and mark them as complete/incomplete after meeting with student to review

Instructors can move students to the column/section they are working on and add notes about their progress or mental health

Instructors can create teams for group projects 

Instructors can track Average Start Delay for each book and the average time it takes to complete each project 
---

## 2. Services

| Service Name | Tech Stack (including version) | Purpose |
|---|---|---|
| database | postgres:16 | Stores and retrieves data |
| api | Django  | Handles requests from the frontend (or other services) and returns data. |
| client | React | Serves HTML, CSS, and JavaScript to the browser. |
| prometheus | PromQL | |

---

## 3. Run Questions

### 3a. Config Files

| Config File | Location | Config Value | What it's for | How it's used |
|---|---|---|---|---|
| .env | .env/root | POSTGRES_User | confirming who the user is | passed to login to show user their data|
| .env | .env/root | POSTGRES_DB | letting system know what database to use | |
| .env | .env/root | POSTGRES_PASSWORD | confirming the user is correct by having a secure login | |

### 3b. How to Start It
make up - starts both api and client
make up-api - starts just the api
make up-client - starts just the client


### 3c. Where to Access It

| Service | Port | URL |
|---|---|---|
| | | |
| | | |
| | | |

### 3d. Service Dependencies

| Service | Depends On | Why |
|---|---|---|
| | | |
| | | |
| | | |

### 3e. Main Entry Points

| Service | Startup File | Routes / URL Config File |
|---|---|---|
| | | |
| | | |
| | | |