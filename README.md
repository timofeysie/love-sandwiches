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

## Connecting to the GCP API with Python

The steps include using pip3 to install packages:

```sh
pip3 install gspread google-auth
```

I usually just use pip, but have also used the venv method to install packages.

```sh
python --version
Python 3.10.11
pip install gspread google-auth
```

The [venv](https://realpython.com/python-virtual-environments-a-primer/) tool version looks something like this:

```sh
PS> python -m venv venv
PS> venv\Scripts\activate
… install with pip
(venv) PS> deactivate
PS>
```

Those commands are all specific to the operating system and shell being used.

In case you're wondering, this tool is used to create and manage separate virtual environments for your Python projects. Each environment can use different versions of package dependencies and Python.

The libraries being installed are:

* google-auth will use our creds.json file from the Google Cloud project.
* gspread used to access and update data in our spreadsheet.

The [repo](https://github.com/Code-Institute-Solutions/love-sandwiches-p5-sourcecode/tree/master/01-getting-set-up/02-connecting-oto-our-api-with-python) for this step.

IAM stands for Identity and Access Management. This configuration specifies what the user has access to.

It is used for the credentials in the run.py to access the spreadsheet and print out the data from it like this:

```py
import gspread
from google.oauth2.service_account import Credentials

SCOPE = [
    "https://www.googleapis.com/auth/spreadsheets",
    "https://www.googleapis.com/auth/drive.file",
    "https://www.googleapis.com/auth/drive"
    ]

CREDS = Credentials.from_service_account_file('creds.json')
SCOPED_CREDS = CREDS.with_scopes(SCOPE)
GSPREAD_CLIENT = gspread.authorize(SCOPED_CREDS)
SHEET = GSPREAD_CLIENT.open('love_sandwiches')

sales = SHEET.worksheet('sales')

data = sales.get_all_values()

print(data)
```

## Program Structure

The stock worksheet has one more line of data in it than the sales and surplus ones.  The app needs to first get the days sales data.

The high level overview of what is being built looks like this:

* request the  market day sales data from our user,
* check if the data provided is valid (if it isn’t, request the data again)
* add our sales data to the sales worksheet
* calculate and update the surplus data
* calculate the sales averages and  make our stock recommendations

## Validating user input

A *list comprehension* is used to loop through the values and convert the strings into integers like this:

```py
[int(value) for value in values]
```

## Reminders

* Your code must be placed in the `run.py` file
* Your dependencies must be placed in the `requirements.txt` file
* Do not edit any of the other files or your code may not deploy properly

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
