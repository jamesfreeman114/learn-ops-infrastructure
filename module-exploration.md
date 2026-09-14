# learn-ops-api: Service Exploration

## 1. Top-level folders in `learn-ops-api`

| Folder | Why does this folder need to exist? |
|--------|-------------------------------------|
|  .github/workflows      | sets up basic workflow and github settings                                 |
|  .vscode      |        vscode settings and launch configurations                             |
|   config     | config files for Digital Ocean                                     |
|   LearningAPI     |  this is the main App package, contains fixtures, migrations,models, serializers, test, and views                                   |
|   LearningPlatform     |  this contains our Django settings and also defines our urls/routes                                  |
|


## 2. Folders inside `LearningAPI`

| Folder | What responsibility does it own and why? |
|--------|----------------------------------------------------------|
| fixtures    | contains placeholder seed data (.json) to populate database until real values are put in                                                          |
| migrations  | uses predefined Models to create new Django objects from the .json in fixtures                                                          |
| models      | contains all the models for our project. Models are lists keys, their related names, data type, and whether or not the key is a Foreign Key to another Model                                                         |
| serializers |  converts  objects to json to be sent to client                                                  |
| tests       |  tests the model methods using simple html                                                       |
| views       |  where methods for our models are defined                                                        |

## 3. What is the Pipfile?

Pipenv's dependency file. It lists the packages the project depends on and the required Python version

## 4. Key packages in the Pipfile

| Package | What functionality does it provide and why? |
|---------|----------------------------------|
| django |  the configuration, the settings, the top-level entry point. version needs to be the same for all people working on the project. "*" means any |
| djangorestframework |  turns Django into a framework for building JSON APIs. adds serializers, Viewsets, and routers. removes the need for the Template Layer |
| django-allauth | integrated set of Django applications regarding authentication, regitration, and account management |

## 5. What does `decorators.py` do?

Defines our custom decorators for the project. These functions wrap other functions throughout the project without changing their original logic.


## 6. What is a serializer, and what serializers are defined here?

Converts python objects into plain text which is sent as .json. The client can take the .json plain text and convert it into a JavaScript object to use. Also works the other direction (converting incoming json information into a Python object). 

Serializers defined in this project include:

capstone_serializer.py
cohort_serializer.py
nssuser_cohort_serializer.py
nssuser_serializer.py
proposal_status_serializer.py
user_serializer.py




## 7. Models and what they represent

Models represent real world data and the rules around it. 

| Model | Real-world thing it represents |
|-------|-------------------------------|
| NssUser  | NSS Student with their name, cohort, scores, slack, github etc.  API needs this to information current for each student. NssUser objects need to all follow the same rules so they are created using the Model. |                    


## 8. Views vs. viewsets

| Type | Example class | File path |
|------|--------------|-----------|
| View | retrieve Book()| | /books/1 |
| ViewSet | BookViewSet() | /books |

## 9. Serializers paired with their models

| Serializer | Model | Link |
|------------|-------|------|
| BookSerializer()           |  Book     |  serializes Python Book object into json    |

## 10. What replaces the Templates and why?

Our React App replaces Templates in this project.