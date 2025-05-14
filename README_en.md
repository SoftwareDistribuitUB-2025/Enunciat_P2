---
author: Xevi Baró, Kaisar Kushibar, Eloi Puertas
date: Abril 2024
title: Pràctica 2 - FastAPI and Vue application (Software distribuït)
---

# Practice 2 Statement

# Introduction

Following Practice 1, in this practice we will implement the Battleship game, but now using standard web protocols and a database.  
We will work with two completely separate modules:

- **Backend**: Will implement a REST-API to securely access and manage application data. It will handle user authentication and database access.
- **Frontend**: Consists of a web application that will run locally in the user’s browser, allowing interaction with the application.

## Important

This guided exercise assumes you already have some version of Python (minimum version `3.10`) installed and are familiar with installing packages using `pip`.

Note that the `pip` command shown in this tutorial may correspond to `pip3` depending on how many versions of Python are installed on your machine.

Using the IDE [PyCharm Professional](https://www.jetbrains.com/pycharm/) is highly recommended for developing this practice. If you don’t have it installed, there is a student version you can get by registering with your UB email.

## Submission of Exercises

The work will follow the same method as in Practice 1, using a `GitHub Classroom` repository as the base. The repository must include:

- The code implementing all required functionalities.
- Weekly `Pull Requests` with:
  - a description of completed tasks and pending ones,
  - organization of team members for task execution,
  - problems encountered during the week,
  - a summary of tests conducted.
- During the development of the practice, you must document your progress in the report found in the `docs` directory of the practice template.

**NOTE:** Remember that `Pull Requests` must be initiated by one team member and approved by the other, confirming the content by both.

## Practice 2 Evaluation

- 20% Weekly pull requests
- 80% Final Delivery:
  - 20% Test session,
  - 20% Unit tests
  - 50% Final code,
  - 10% Progress report.

## Practice Objectives

#### Back-End:

- Create a CRUD application with authentication to manage the application model (CRUD: Create, Read, Update, and Delete)
- Handle game logic

#### Front-End:

- User interaction, both in terms of data visualization and capturing actions.
- Secure communication with the back-end

# Schedule

| Session | Date              | Topic                                    | Guide                                | Links                                                                             |
| ------- | ----------------- | ---------------------------------------- | ------------------------------------ | --------------------------------------------------------------------------------- |
| 1       | 02/04/25          | Local deployment of backend and frontend | [Session 1](Sessions/Sessio_1_en.md) | [Intro to Vue](Guies/inici_Vue_en.md) [Intro to Django](Guies/inici_DJanog_en.md) |
| 2       | 23/04/25-30/04/25 | Backend and database                     | [Sessió 2](Sessions/Sessio_2_en.md)  |                                                                                   |
| 3       | 07/05/25          | **Quiz 2** and Frontend                  | [Sessió 3](Sessions/Sessio_3_en.md)  |                                                                                   |
| 4       | 14/05/25          | Authentication and authorization         | [Sessió 4](Sessions/Sessio_4_en.md)  |                                                                                   |
| 5       | 21/05/25          | Leaderboard                              |                                      |                                                                                   |
| 6       | 28/05/25          | Testing Session                          |                                      |                                                                                   |
