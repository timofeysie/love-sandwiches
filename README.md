# Love Sandwiches

This project contains the code from the [Code Institute](https://codeinstitute.net/global/) Diploma in Full Stack Software Development project 3.

The goal is to build a command line based Python program to handle data automation for a fictional sandwich company.  This CLI (Command Line Interface) will save the company staff time by automating repetitive tasks, and help reduce the surplus by better predicting sales for future markets.  It will interact with a Google Sheet so that we can push and pull data to and from the spreadsheet.

The story goes: *Love Sandwiches runs a local market stall, selling  a small range of sandwiches.  For each market day, their staff pre-make stock to sell. If they sell out of a particular sandwich, their staff make extra for their customers. And the unsold  ones are thrown away at the end of the day.
Our job is to write a Python program that will collect the company’s market day sales data, calculate the surplus for the day, and produce  recommendations for the number of each sandwich to make for the next market.*

The learning outcomes for the project are:

* help students become comfortable with Python functions
* understand how to build a command-line application in Python
* set up a Google Sheets API to send and receive data to an external application
* refactor code to streamline your projects and reduce repetition

## Original README

![CI logo](https://codeinstitute.s3.amazonaws.com/fullstack/ci_logo_small.png)

Welcome,

This is the Code Institute student template for deploying your third portfolio project, the Python command-line project. The last update to this file was: **May 14, 2024**

## Reminders

- Your code must be placed in the `run.py` file
- Your dependencies must be placed in the `requirements.txt` file
- Do not edit any of the other files or your code may not deploy properly

## Creating the Heroku app

When you create the app, you will need to add two buildpacks from the _Settings_ tab. The ordering is as follows:

1. `heroku/python`
2. `heroku/nodejs`

You must then create a _Config Var_ called `PORT`. Set this to `8000`

If you have credentials, such as in the Love Sandwiches project, you must create another _Config Var_ called `CREDS` and paste the JSON into the value field.

Connect your GitHub repository and deploy as normal.

## Constraints

The deployment terminal is set to 80 columns by 24 rows. That means that each line of text needs to be 80 characters or less otherwise it will be wrapped onto a second line.

---

Happy coding!
