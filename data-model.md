# Data Model

## 1. Database Diagram

![Diagram](./data-model.png)

## 2. Database Info

**Database type:**

postgres:16

**ORM:**

## 3. Model to Table Mapping

| Model Name | Table Name         |
|------------|--------------------|
| Book       | LearningAPI_Book   |
| Course     | LearningAPI_Course |

| Property Name      | Column Name    | Data Type |
|--------------------|----------------|-----------|
| Learning_API Course|   id           | integer   |
|                    |   name         | varchar   |
|                    |   date_created | date      |
|                    |   active       | boolean   |


## 4. Relationship Examples

**One-to-one** (field name: )

| Model Name        | Table Name                      | PK Column | FK Column          |
|-------------------|---------------------------------|-----------|--------------------|
| SocialAccount     | social_account                  | id        |  user_id           |
| User              | auth_user                       | id        |                    |

**One-to-many** (field name: Capstonetimeline )

| Model Name | Table Name                                | PK Column | FK Column          |
|------------|-------------------------------------------|-----------|--------------------|
| CapstoneTimeline  | Learning_API capstonetimeline      | id        |                    |
| Capstone          | Learning_API capstone              | id        |    capstone_id     |
| ProposalStatus    | Learning_API proposalstatus        | id        |    status_id       |

**Many-to-many** (field name: ProjectTag )

| Model Name | Table Name                  | PK Column  | FK Column           |
|------------|-----------------------------|------------|---------------------|
| Project    | Learning_API_project        |   id       |   project_id        |
| Tag        | Learning_API_tag            |   id       |   tag_id            |
| ProjectTag | Learning_API_projecttag     |   id       |                     |