# Tech Stack AI Prompts

## 1. Run Questions

### 1a. Config Files

Look at the file, tech-stack-ai.md in the root of this repository. Find what Config files exist in the system and fill in the table for question 1a. Pick 3 values from each and explain what each value  is for and how it is used by the system. For the " Config Value" column record only the config name, not its value. Don't include any sensitive information like passwords. Then I want you to paste this prompt in the tech-stack-prompts.md file under 1a.

### 1b. How to Start It

Moving on to 1b, look at the Makefile in the root of the learn-ops-infrastructure. Identify the targets relevant to starting the system and explain how they differ. There should be multiple targets.

### 1c. Where to Access It

For 1c: Find each service's port and URL and put them in the services table with columns Service, Port, URL. Start in learn-ops-infrastructure/docker-compose.yml and work your way out from there.

### 1d. Service Dependencies

Map the service dependencies for the table 1d. For the "Why" column explain the reason for this dependency. Look at how the service uses the other service it is dependent on and use that for your explanation.

### 1e. Main Entry Points

For each service, find the startup file and the routes/URL config file and use this info to fill out Table 1e. These should be two separate files

## 2. Services

For #2 fill out the table with Service Name, Tech Stack (including version), and Purpose - one row per service.

## 3. System Overview

Write 3 paragraphs. In the first, describe what kind of application this is and what problem it solves. In the second, describe its main features from the perspective of someone using it. In the third, describe who uses it and how - are there different roles, and do they interact with the system differently?
