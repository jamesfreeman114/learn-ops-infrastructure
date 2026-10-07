# Data Model AI Prompts

## 1. Find the Database Connection Details

I need to connect to the database for this system using DataGrip.
Find the host, port, database name, username, and password.
Look in the environment files and docker-compose configuration.
Tell me which file each value comes from.

## 2. Identify the Database Type

What database engine does this application use?
Find where that is declared in the project files and tell me the version.

## 3. Map the ORM to the Database

How does this application communicate with the database? Find the ORM it uses
and where the database connection is configured. Include the ENGINE value from settings.
I don't need long descriptions for each. Just the name of the ORM and the ENGINE value, and one sentence explaining the configuration.

Then open LearningAPI/models/ and pick one model. Show me the Python field names
next to the SQL column names and their data types. Create a table using this format:

| Model Name | Table Name |
|------------|------------|
|            |            |

| Property Name | Column Name | Data Type |
|---------------|-------------|-----------|
|               |             |           |

Finally, open LearningAPI/views/book_view.py and find the create method.
What does book.save() actually do - what SQL statement does the ORM generate? Keep this summary direct and to the point.
The raw SQL Statement plus 2 sentences summarizing.


## 4. Generate a Database Diagram

Generate an entity-relationship diagram of the database as a Mermaid erDiagram.
Look at all the model files in LearningAPI/models/.
Include every table, every field, and every relationship between tables.
Output only the Mermaid code block. Use "Crows Foot Notation/Symbols" for the different relationship types.



## 5. Find Relationship Examples

Find one example each of a one-to-one, one-to-many, and many-to-many relationship
in the Django models in LearningAPI/models/.
For each example give me the file path and the name of the field that defines the relationship.
Format tables for each using the template below

One-to-one (field name: )

Model Name    Table Name    PK Column    FK Column
Model 1
Model 2

One-to-many (field name: )

Model Name    Table Name    PK Column    FK Column
Model 1

Model 2
Many-to-many (field name: )

Model Name    Table Name    PK Column    FK Column
Model 1
Model 2
(junction)