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

The user request loop is a while statement in the ```get_sales_data()``` function which calls the ```validate_data(values)``` function.

```py
    try:
        [int(value) for value in values]
        if len(values) != 6:
            raise ValueError(
                f"Exactly 6 values required, you provided {len(values)}"
            )
    except ValueError as e:
        print(f"Invalid data: {e}, please try again.\n")
        return False

    return True
```

This try/except block is actually called a try/catch block elsewhere.  I'm not sure why Python had to go it alone on changing catch to except.  I assume there was a good reason.  ChatGPT said: *Python was influenced by several languages, including ABC and Modula-3, which also use except for exception handling* so there you go.

## Updating the sales worksheet

We use the gspread worksheet() method to access the sales worksheet and append_row() and pass it the data to be inserted and add a new row to the end of the data in the worksheet selected:

```py
    sales_worksheet = SHEET.worksheet("sales")
    sales_worksheet.append_row(data)
```

### Requesting stock data from the spreadsheet

Add a main() function and call that at the bottom of the file.

```py
def main():
    """
    Run all program functions
    """
    data = get_sales_data()
    sales_data = [int(num) for num in data]
    update_sales_worksheet(sales_data)
```

This is now calling a new update_sales_worksheet(sales_data) with the formatted data from the user input.

The business logic we implement in the update_sales_worksheet() function is as follows:

The numbers of sandwiches made fresh or thrown away is the surplus data.  This data is calculated by taking the stock numbers and subtracting the number of sandwiches sold.

For example:

* If there were 8 egg salad sandwiches in stock, and they sold 10 that day, this  would result in a value of -2 (ie: 2 extra sandwiches were made when stock ran out).
* If they made stock of 15 egg salad sandwiches but they only sold 10, then the surplus number would be 5 (ie: 5 sandwiches were thrown away at the end of the day).

This is basically done like this:

```py
def calculate_surplus_data(sales_row):
    print("Calculating surplus data...\n")
    stock = SHEET.worksheet("stock").get_all_values()
    stock_row = stock[-1] # last row in the stock worksheet
```

The "stock" worksheet is a tab on the spreadsheet with the stock data.

### Calculating Surplus Data

Take each value in each list, and subtract the sales number from the stock number using the Python zip() method and a for loop.
Put these values in the surplus_data array.

```py
def calculate_surplus_data(sales_row):
    stock = SHEET.worksheet("stock").get_all_values()
    stock_row = stock[-1]
    
    surplus_data = []
    for stock, sales in zip(stock_row, sales_row):
        surplus = int(stock) - sales
        surplus_data.append(surplus)

    return surplus_data
```

As the new stock_row data is still in string format when retrieved from the sheets API, those values are converted with the int function one at a time.

### Updating the surplus worksheet

The update_surplus_worksheet() is pretty much the same as the update_sales_worksheet() function.

Get the surplus worksheet and append the row from the data argument:

```py
def update_surplus_worksheet(data):
    surplus_worksheet = SHEET.worksheet("surplus")
    surplus_worksheet.append_row(data)
```

This function is then refactored to a shared version and used like this:

```py
    update_worksheet(sales_data, "sales")
    new_surplus_data = calculate_surplus_data(sales_data)
    update_worksheet(new_surplus_data, "surplus")
```

## Getting sales data & calculating stock averages

The recommended stock numbers for the next market will be calculated using the average number of sandwiches sold in the last 5 markets and increase the result by 10% to encourage more sales.

This time we access a single column by using the col_values() gspread method which takes the column number starting at 1 not 0.

```py
def get_last_5_entries_sales():
    sales = SHEET.worksheet("sales")

    columns = []
    for ind in range(1, 7):                 # from 1 - 6
        column = sales.col_values(ind)
        columns.append(column[-5:])         # limit to the last five averages
                                            # we need a colon to slice multiple values from the list
    return columns
```

To calculate the average we use this function:

```py
def calculate_stock_data(data):
    """
    Calculate the average stock for each item type, adding 10%
    """
    print("Calculating stock data...\n")
    new_stock_data = []

    for column in data:
        int_column = [int(num) for num in column]
        average = sum(int_column) / len(int_column)
        stock_num = average * 1.1
        new_stock_data.append(round(stock_num))

    return new_stock_data
```

Then use it in the main function like this:

```py
    sales_columns = get_last_5_entries_sales()
    stock_data = calculate_stock_data(sales_columns)
    update_worksheet(stock_data, "stock")
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
