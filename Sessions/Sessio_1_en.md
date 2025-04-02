# Session 1

In this session, we will cover the basic technologies that we will be working with during this project.

## Objectives

- Create a project with [Django](https://www.djangoproject.com/) [Backend]
- Create a project with [Vue](https://vuejs.org/) [Frontend]

## Introduction to the Frontend

We will define the frontend using the JavaScript framework [Vue](https://vuejs.org/), which will allow us to visualize the game state at all times and interact with it.  
Below you can see some screenshots of the expected final result:

<figure>
  <img
  src="../Images/front_result_1.jpeg"
  alt="Initial Board.">
  <figcaption>Board at the start of the game</figcaption>
</figure>
<figure>
  <img
  src="../Images/front_result_2.jpeg"
  alt="Board with ships placed.">
  <figcaption>Board with ships placed</figcaption>
</figure>
<figure>
  <img
  src="../Images/front_result_3.jpeg"
  alt="Board during the game.">
  <figcaption>Board during the game</figcaption>
</figure>

Keep in mind that the frontend acts as a **Client** in our application and runs on the user's machine inside a web browser such as Chrome, Mozilla, Edge, or Safari.

## Introduction to the Backend

We will define the backend using the Python framework [Django](https://www.djangoproject.com/), which allows rapid development of web applications by incorporating modules to add functionality.  
In our case, the main modules we will include are:

- [Django Auth](https://docs.djangoproject.com/en/5.1/topics/auth/): Django's default authentication system. It provides the **User** model, along with authentication and authorization mechanisms.
- [Django Rest Framework](https://www.django-rest-framework.org/): A framework that makes it easy to build a REST API.
- [drf-spectacular](https://drf-spectacular.readthedocs.io/en/latest/): A module that will allow us to automatically generate API documentation.

One of the responsibilities of the backend is access to the database, which is used to persist the data models. To manage the application’s data model, we will use Django’s ORM (Object Relational Mapping) system, based on the [Django Model class](https://docs.djangoproject.com/en/5.1/topics/db/models/).  
This system allows us to synchronize the data models defined in the application (in Python code) with the database (instead of using Data Definition Language or DDL) via [migrations](https://docs.djangoproject.com/en/5.1/topics/migrations/).  
The idea of migrations is that every time we change the data models in the application, scripts are generated explaining the changes needed in the database to keep it synchronized with the models. These migrations must then be applied to update the structure and content of the database.  
By default, we will use **SQLite** as the database, which is a file-based database.

## Exercise 1

Follow the instructions in the [introduction to Vue guide](../Guies/inici_Vue_en.md) to create a Vue project from scratch and run it.

## Exercise 2

Follow the instructions in the [introduction to Django guide](../Guies/inici_DJango_en.md) to create a Django project from scratch and run it.
