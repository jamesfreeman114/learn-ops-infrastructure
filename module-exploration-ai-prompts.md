# learn-ops-api: AI Prompts

## 1. Project structure

I would like you to list all the Top-level folders in the learn-ops-api directory and provide a brief explanation why each one needs to exist and then fill in table 1 in module-exploration-ai-md (found in learn-ops-infrastructure.) Then look inside of the LearningAPI directory and list each folder found there and explain why each one needs to exist. Fill in table 2 with this information. Be sure to only look at folders in the parent learn-ops-api  directory for part 1, and only at the folders in the LearningAPI folder for part 2.

## 2. Dependencies

Next, take a look at the Pipfile in the learn-ops-api directory. Provide a brief explanation for what it is and why it exists and fill in part three for module-exporation-ai.md. While you are looking at the Pipfile, focus on three packages: django, djangorestframework, and django-allauth. What does each package provide and why does this project depend on it? Then, fill out the table for part 4 in the .md file

## 3. Decorators, serializers, and models

Take a look at LearningAPI/decorators.py. Provide a brief explanation for what a decorator is and how it is used in this specific file. Then fill in part 5 in the module-exploration-ai.md file. Next, open LearningAPI/serializers, explain what serializers are, and describe how they fit into the request/response cycle (part 6) Finally, open the models folder and explain what a Django model is. I'd like you to look at the NssUser Model, name the real-world thing it represents, and explain why the API needs to track that data (part 7)

## 4. Views, viewsets, and the MTV pattern

Find one example of a plain view and one example of a viewset in this project. For each, show me the class name and file path. Explain the difference between a view and a viewset and when you would choose one over the other. Fill in the table for part 8 with this information.

Django uses a Model-Template-View pattern. This project has no HTML templates. What takes the template's role here, and why does that make sense for a REST API?  Paste your explanation under part 9.

