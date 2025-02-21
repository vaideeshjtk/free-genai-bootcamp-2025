# Technical Specs for Backend:

## Business Goal: 
A language learning school wants to build a prototype of learning portal which will act as three things:
- Inventory of possible vocabulary that can be learned
- Act as a  Learning record store (LRS), providing correct and wrong score on practice
vocabulary
- A unified launchpad to launch different learning apps

## Technical Requierments:
- The backend will be deployed in Python 
- The Data base will be SQLite3 
- The API will be built using Flask 
- The API will always return JSON

### Technical Requierments for my knowledge:
- The backend will be deployed in C#
- The Data base will be My SQL Server or MS SQL server or Oracle SQL
- The API will be built using .Net or ASP.Net
- The API will always return JSON


## Database Schema:

We have the following tables:
- study_activities
- groups
- study_sessions
- Words_groups
- word_review_items
- words

## Backend Structure:
language-portal/
├── backend-flask/
  ├── app.py
  ├── migrate.py
  ├── tasks.py
  ├── Readme.md
  ├── Requierments.txt
  ├── .gitignore
  ├── Technical-specs.md
  ├── lib/
  |   └── db.py
  ├── seed/
  │   └── data_adjectives.json
  |   └── data_verbs.json
  |   └── study_actvities.json
  ├── routes/
  │   └── dashboard.py  
  |   └── groups.py  
  |   └── study_activities.py  
  |   └── study_sessions.py  
  |   └── words.py
  ├── SQL/Setup
  │     └── create_table_groups.sql            
            create_table_word_groups.sql       
            create_table_words.sql
            create_table_study_activities.sql  
            create_table_word_review_items.sql  
            create_word_reviews.sql
            create_table_study_sessions.sql    
            create_table_word_reviews.sql       
            insert_study_activities.sql 

### Tables and Their Purpose
#### study_activities
Purpose: Stores information about different study activities (e.g., flashcards, quizzes).
``` sh
Columns:
id (Primary Key, PK): A unique identifier for each activity.
name: The name of the activity (e.g., "Flashcards").
url: A link to the activity (e.g., a web page or resource).
```
#### groups
Purpose: Stores groups of words (e.g., "Core Verbs", "Core Adjectives").
``` sh
Columns:
id (Primary Key, PK): A unique identifier for each group.
name: The name of the group (e.g., "Core Verbs").
words_count: The number of words in the group.
```

#### study_sessions
Purpose: Tracks study sessions where students engage with study activities.
```sh
Columns:
id (Primary Key, PK): A unique identifier for each session.
group_id: The ID of the group being studied.
study_activity_id: The ID of the activity being used.
created_at: The timestamp when the session was created.
```

#### word_group
Purpose: Links words to groups (many-to-many relationship).
``` sh
Columns:
word_id (Primary Key, PK): The ID of the word.
group_id (Primary Key, PK): The ID of the group.
```

#### word_review_items
Purpose: Tracks individual word reviews during study sessions.
``` sh
Columns:
id (Primary Key, PK): A unique identifier for each review item.
word_id: The ID of the word being reviewed.
study_session_id: The ID of the study session.
correct: A boolean (true/false) indicating whether the review was correct.
created_at: The timestamp when the review was created.
```
#### words
Purpose: Stores individual Spanish vocabulary words.

``` sh
Columns:
id (Primary Key, PK): A unique identifier for each words.
spanish: Spanish words
english: English transalation words
parts: Word components stored in JSON format
```

### Relationships Between Tables
#### study_sessions and groups and study_activities
A study session is associated with a specific group (e.g., "Core Verbs").
A study session is associated with a specific activity (e.g., "Flashcards").
The group_id in study_sessions links to the id in groups.The study_activity_id in study_sessions links to the id in study_activities.


##### word_group
This table connects words to groups.

A word can belong to multiple groups, and a group can contain multiple words.

The word_id links to the id in the words table (not shown in the schema but assumed to exist).

The group_id links to the id in the groups table.

##### word_review_items
This table tracks individual word reviews during study sessions.

The word_id links to the id in the words table.

The study_session_id links to the id in the study_sessions table.

#### Relationships

word belongs to groups through  word_groups
group belongs to words through word_groups
session belongs to a group
session belongs to a study_activity
session has many word_review_items
word_review_item belongs to a study_session
word_review_item belongs to a word

### Example Workflow
Let’s walk through an example to see how these tables work together:

Create a Group:

Add a group called "Core Verbs" to the groups table.

Example row: id: 1, name: "Core Verbs", words_count: 50.

Add Words to the Group:

Add words like "hablar" (to speak) and "comer" (to eat) to the words table.

Link these words to the "Core Verbs" group in the word_group table.

Example rows:

word_id: 1, group_id: 1 (for "hablar").

word_id: 2, group_id: 1 (for "comer").

Create a Study Activity:

Add a study activity called "Flashcards" to the study_activities table.

Example row: id: 1, name: "Flashcards", url: "https://example.com/flashcards".

Start a Study Session:

A student starts a study session for the "Core Verbs" group using the "Flashcards" activity.

Add a row to the study_sessions table.

Example row: id: 1, group_id: 1, study_activity_id: 1, created_at: "2023-10-01 12:34:56".

Review Words:

During the session, the student reviews words like "hablar" and "comer".

Add rows to the word_review_items table to track the results.

Example rows:

id: 1, word_id: 1, study_session_id: 1, correct: true, created_at: "2023-10-01 12:35:00".

id: 2, word_id: 2, study_session_id: 1, correct: false, created_at: "2023-10-01 12:36:00".

4. Key Points
Primary Keys (PK): Unique identifiers for each row in a table.

Foreign Keys: Columns that link to primary keys in other tables (e.g., group_id in study_sessions links to id in groups).

Many-to-Many Relationships: Tables like word_group connect words to groups, allowing a word to belong to multiple groups and a group to contain multiple words.

Tracking Progress: The word_review_items table tracks how well students perform during study sessions.

5. Visual Representation
Here’s a simplified diagram of the relationships:

groups (id, name, words_count)
   |
   |-- study_sessions (id, group_id, study_activity_id, created_at)
   |       |
   |       |-- word_review_items (id, word_id, study_session_id, correct, created_at)
   |
   |-- word_group (word_id, group_id)
           |
           |-- words (id, spanish, english, parts)
## API Endpoints

Dashboard Page:
- GET /dashboard/last_study_session 
- GET /dashboard/study_progress
- GET /dashboard/quick-stats
- GET /api/study_actvities/:id
- GET /api/study_actvities/:id/study_session
- GET /api/study_actvities
- POST /api/study_activity
    - Required Parameters: GroupId, study_actvities_id
- GET /api/words - Get list of words with review statistics 
    - Paginated with 100 items per page
- GET /api/groups - Get list of word groups with word counts
    - Paginated with 100 items per page
- GET /api/groups/:id - Get words from a specific group 
- GET /api/groups/:id/words
- GET /api/groups/:id/study_sessions
- GET /api/study_sessions
- GET /api/study_sessions/:id
- GET /api/study_sessions/:id/words
- POST /api/reset_history
- POST /api/full_reset
- POST /api/study_sessions/:id/words/:word_id/review
    - Requried params: correct

### JSON Responses

#### GET /api/dashboard/last_study_session
```sh
{
  "id": 1,
  "activity_name": "Flashcards",
  "last_used": "2023-10-01T12:34:56Z",
  "correct_count": 5,
  "wrong_count": 2,
  "group_id": 123,
  "group_name": "Core Verbs"
}
```

#### GET /api/dashboard/study_progress
``` sh
{
  "total_words": 1000,
  "words_studied": 200,
}

```

#### GET /api/dashboard/quick-stats
``` sh
{
  "total_sessions": 20,
  "words_studied": 200,
  "mastered_words": 50,
  "success_rate": 0.75,
  "active_groups": 5,
  "current_streak": 10
}
```

#### GET /api/study_actvities
``` sh
{
  "activities": [
    {
      "id": 1,
      "name": "Flashcards",
      "thumbnail_url": "https://example.com/flashcards/thumbnail.jpg"
    },
    {
      "id": 2,
      "name": "Quizzes",
      "thumbnail_url": "https://example.com/quizzes/thumbnail.jpg"
    }
  ],
  "total_pages": 1,
  "current_page": 1
}
```

#### GET /api/study_actvities/:id

``` sh
{
  "id": 1,
  "name": "Flashcards",
  "description": "A fun way to learn new words using flashcards.",
  "thumbnail_url": "https://example.com/flashcards/thumbnail.jpg",
  "launch_url": "https://example.com/flashcards",
  "total_pages": 1,
  "current_page": 1
}
```

#### GET /api/study_actvities/:id/study_session

``` sh
{
  "study_sessions": [
    {
      "id": 1,
      "activity_name": "Flashcards",
      "group_name": "Core Verbs",
      "start_time": "2023-10-01T12:34:56Z",
      "end_time": "2023-10-01T12:45:00Z",
      "review_items_count": 10
    }
  ],
  "total_pages": 1,
  "current_page": 1
}
```

#### GET /api/study_actvities/:id/study_session

``` sh
{
  "study_sessions": [
    {
      "id": 1,
      "activity_name": "Flashcards",
      "group_name": "Core Verbs",
      "start_time": "2023-10-01T12:34:56Z",
      "end_time": "2023-10-01T12:45:00Z",
      "review_items_count": 10
    }
  ],
  "pagination":
    { 
        "total_pages": 1,
        "current_page": 1
    }

}
```

#### POST /api/study_activity
```sh
{
  "message": "Study activity created successfully",
  "activity_id": 1
  "group_id": 1
}
```

#### GET /api/words
```sh
{
  "words": [
    {
      "id": 1,
      "spanish": "hablar",
      "english": "to speak",
      "correct_count": 10,
      "wrong_count": 2
    },
    {
      "id": 2,
      "spanish": "comer",
      "english": "to eat",
      "correct_count": 8,
      "wrong_count": 3
    }
  ],
  "pagination":
    { 
        "total_pages": 1,
        "current_page": 1
    }
}
```

#### GET /api/words/:id

```sh
{
  "id": 1,
  "spanish": "hablar",
  "english": "to speak",
  "correct_count": 10,
  "wrong_count": 2,
  "groups": [
    {
      "id": 1,
      "name": "Core Verbs"
    },
    {
      "id": 2,
      "name": "Common Verbs"
    }
  ]
}
```

#### GET /api/groups

```sh
{
  "groups": [
    {
      "id": 1,
      "name": "Core Verbs",
      "words_count": 50
    },
    {
      "id": 2,
      "name": "Core Adjectives",
      "words_count": 30
    }
  ],
  "pagination":
    { 
        "total_pages": 1,
        "current_page": 1
    }
}
```
#### GET /api/groups/:id

```sh
{
  "id": 1,
  "name": "Core Verbs",
  "words_count": 50,
  "words": [
    {
      "id": 1,
      "spanish": "hablar",
      "english": "to speak"
    },
    {
      "id": 2,
      "spanish": "comer",
      "english": "to eat"
    }
  ],
  "study_sessions": [
    {
      "id": 1,
      "activity_name": "Flashcards",
      "start_time": "2023-10-01T12:34:56Z",
      "end_time": "2023-10-01T12:45:00Z",
      "review_items_count": 10
    }
  ],
  "pagination":
    { 
        "total_pages": 1,
        "current_page": 1
    }
}
```

#### GET /api/groups/:id/words


```sh
{
  "words": [
    {
      "id": 1,
      "spanish": "hablar",
      "english": "to speak"
    },
    {
      "id": 2,
      "spanish": "comer",
      "english": "to eat"
    }
  ],
  "pagination":
    { 
        "total_pages": 1,
        "current_page": 1
    }
```
#### GET /api/groups/:id/study_sessions

``` sh
{
  "study_sessions": [
    {
      "id": 1,
      "activity_name": "Flashcards",
      "group_name": "Core Verbs",
      "start_time": "2023-10-01T12:34:56Z",
      "end_time": "2023-10-01T12:45:00Z",
      "review_items_count": 10
    }
  ],
  "pagination":
    { 
        "total_pages": 1,
        "current_page": 1
    }
}
```

#### GET /api/groups/:id/study_sessions

``` sh
{
  "study_sessions": [
    {
      "id": 1,
      "activity_name": "Flashcards",
      "group_name": "Core Verbs",
      "start_time": "2023-10-01T12:34:56Z",
      "end_time": "2023-10-01T12:45:00Z",
      "review_items_count": 10
    }
  ],
  "pagination":
    { 
        "total_pages": 1,
        "current_page": 1
    }
}
```
#### GET /api/study_sessions

``` sh
{
  "study_sessions": [
    {
      "id": 1,
      "activity_name": "Flashcards",
      "group_name": "Core Verbs",
      "start_time": "2023-10-01T12:34:56Z",
      "end_time": "2023-10-01T12:45:00Z",
      "review_items_count": 10
    }
  ],
  "pagination":
    { 
        "total_pages": 1,
        "current_page": 1
    }
}
```

#### GET /api/study_sessions/:id

``` sh
{
  "id": 1,
  "activity_name": "Flashcards",
  "group_name": "Core Verbs",
  "start_time": "2023-10-01T12:34:56Z",
  "end_time": "2023-10-01T12:45:00Z",
  "review_items_count": 10
}
```
#### GET /api/study_sessions/:id/words

``` sh
{
  "words": [
    {
      "id": 1,
      "spanish": "hablar",
      "english": "to speak",
      "correct": true,
      "created_at": "2023-10-01T12:35:00Z"
    },
    {
      "id": 2,
      "spanish": "comer",
      "english": "to eat",
      "correct": false,
      "created_at": "2023-10-01T12:36:00Z"
    }
  ],
  "pagination":
    { 
        "total_pages": 1,
        "current_page": 1
    }
}
```
#### POST /api/reset_history

``` sh
{
    "success": "True"
    "message": "History reset successfully"
}
```
#### POST /api/full_reset

``` sh
{
    "success": "True"
    "message": "Full reset completed successfully"
}
```

#### POST /api/study_sessions/:id/words/:word_id/review

##### Request Payload
``` sh
{
    "correct": true
}
```
##### Response Payload
``` sh
{
    "success": "True"
    "message": "Review recorded successfully",
    "created_at": "2023-10-01T12:36:00Z"
    "study_session_id": 1,
    "word_id": 1,
    "correct": true
}
```

### Initilize Database
We need to initilize a SQLlite3 DB, the migrate.py task will do that and create an words.db file.

### Seed Data 

There is folder called Seed and the files are there and they are in JSON. 
The files need to be loaded into DB.
That is JSON into target data into Data base
``` JSON
  {
    "spanish": "grande",
    "english": "big",
    "parts": "adjective"
  },
```