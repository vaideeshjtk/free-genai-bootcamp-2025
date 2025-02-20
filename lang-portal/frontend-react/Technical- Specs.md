# Technical Specs for Frontend:

## Pages

### Dashboard

#### Components
This Pages contains, its URL is `/dashboard` and its the default
- Last Study Session
        shows last activity used
        shows when last activity used
        summarized correct vs wrong from  last activty
        can view groups
- Study Progress
        total words studied out of all out of all words
        display of progress
- Quick Stats
        success rate
        total study sessions
        total active group
        study streak
- Start Studying button
        goes to study activites page
#### API Endpoints
We will need the following endpoints to view this page

- GET /api/dashboard/last_study_session 
- GET /api/dashboard/study_progress
- GET /api/dashboard/quick-stats

---

### Study_Activity `/Study_Activity`

#### Components
The purpose of this page is shows serious of collection of study_activtites with a thumbnail either to launch or view the the study actvity. This Page Contains

- Study Activity card
        shows a thumbnail of the study_activity
        shows name of the study activity
        launch buttopm to launch the study activty that is to go into the particular study actvity page
        can view the information about the study activty

#### Needed API Endpoints
We will need the following endpoints to view this page

- GET /api/study_actvities
        Pagination
---
### Study_Activity show `/Study_Activity/:id`

#### Components
The purpose of this page is to show the details of the study_activities and its past history 

- Name of the study_activities
- thumbnail of the study_activity
- Descrption of the study_activity
- Launch button
- Study_Activity Paginated list
        - id 
        - activity name
        - group name
        - start time 
        - end time (from last word submmited time)
        - number of review items 

#### Needed API Endpoints
- GET /api/study_actvities/:id
- GET /api/study_actvities/:id/study_session
---
### Study_Activity launch page `/Study_Activity/:id/launch`

#### Components
The purpose of this launch page is to launch a study activity

- Name of the Study Activity
- Launch form 
        - select field for group 
        - launch now button

After the form is submitted an new tab opens based on URL in the database. Also after study activity is done the page will redirect to to study_session page.

#### Needed API Endpoints
- POST /api/study_activity
---
### Words `/words`

#### Components
The purpose of this page is to display all words in the database

- Paginated word list 
        - Columns
                - Spanish
                - English
                - Correct words count
                - Wrong words count
        - Pagination with 100 items per page
        - Clicking the Spanish word will take us to word show page

#### Needed API Endpoints
- GET /api/words
---
### Words show page `/words:id`

#### Components
The Purpose of this is to show a information about a word

- Spanish word
- English word
- Study Statistics
        - Correct words count
        - wrong words count
- Word Groups  
        - As series of pills EG. tags
        - when groups name is clicked it will redirect to te group show page

#### Needed API Endpoints
- Get /api/words/:id

---
### Words groups `/groups`

#### Components
The purpose of this page is to display the list of groups in our database

- Paginated words groups list
        - Columns
                - words count
                - Groups name
        - when groups name is clicked it will redirect to te group show page

#### Needed API Endpoints
- GET /api/groups

### Group Show `/groups/:id`

#### Components
The purpose of this page is to show information about a specific group.

- Group Name
- Group Statistics
        - Total Word Count
- Words in Group (Paginated List of Words)
        - Should use the same component as the words index page
        - Study Sessions (Paginated List of Study Sessions)
        - Should use the same component as the study sessions index page

#### Needed API Endpoints
- GET /api/groups/:id (the name and groups stats)
- GET /api/groups/:id/words
- GET /api/groups/:id/study_sessions
---

### Study Sessions Index `/study_sessions`

#### Components
The purpose of this page is to show a list of study sessions in our database.
- Paginated Study Session List
        - Columns
                - Id
                - Activity Name
                - Group Name
                - Start Time
                - End Time
                - Number of Review Items
        - Clicking the study session id will take us to the study session show page
        
#### Needed API Endpoints
GET /api/study_sessions


### Study Session Show `/study_sessions/:id`

#### Components
The purpose of this page is to show information about a specific study session.

- Study Sesssion Details
        - Activity Name
        - Group Name
        - Start Time
        - End Time
        - Number of Review Items
- Words Review Items (Paginated List of Words)
        - Should use the same component as the words index page
#### Needed API Endpoints   
- GET /api/study_sessions/:id
- GET /api/study_sessions/:id/words

### Settings Page `/settings`
#### Components
The purpose of this page is to make configurations to the study portal.

- Theme Selection eg. Light, Dark, System Default
- Language Selection eg. English and Spanish (optional)
- Reset History Button
        - this will delete all study sessions and word review items
- Full Reset Button
        - this will drop all tables and re-create with seed data

#### Needed API Endpoints
- POST /api/reset_history
- POST /api/full_reset