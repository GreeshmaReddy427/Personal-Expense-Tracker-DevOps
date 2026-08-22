# Expense Tracker

A Flask-based web application for recording, managing, filtering,
visualizing, and exporting personal expenses. The application uses
SQLite for persistent storage, Flask-SQLAlchemy for database operations,
Jinja2 templates for server-side rendering, Tailwind CSS for the
interface, and Chart.js for spending visualizations.

## Features

-   Add expenses with:
    -   Description
    -   Amount
    -   Category
    -   Date
-   Store expenses in a SQLite database.
-   View all stored expenses in a table.
-   Delete individual expenses.
-   Filter expenses by:
    -   Start date
    -   End date
    -   Category
-   Validate expense input:
    -   Required description, amount, and category
    -   Amount must be a positive number
    -   Date must follow `YYYY-MM-DD`
    -   Invalid date ranges are rejected
-   Calculate the total amount for the currently filtered expenses.
-   Export the currently filtered expenses as a CSV file.
-   Visualize spending:
    -   Expenses by category using a pie chart
    -   Daily spending over time using a bar chart
-   Display success and error messages using Flask flash messages.
-   Responsive dark-themed dashboard interface.

## Tech Stack

### Backend

-   Python
-   Flask
-   Flask-SQLAlchemy
-   SQLAlchemy
-   SQLite

### Frontend

-   HTML
-   Jinja2 templates
-   Tailwind CSS
-   JavaScript
-   Chart.js

### Data Export

-   Python `csv` module
-   CSV download through a Flask response

## Project Structure

``` text
Expense-tracker/
│
├── instance/
│   └── expenses.db
│
├── templates/
│   ├── base.html
│   └── index.html
│
├── app.py
├── requirements.txt
├── .gitignore
└── venv/
```

> `venv/` is the local Python virtual environment and should not be
> committed to GitHub. `requirements.txt` records the Python
> dependencies needed to recreate the environment.

## Application Architecture

The application follows a simple Flask MVC-style flow:

``` text
                ┌─────────────────────┐
                │       Browser       │
                │   HTML Dashboard    │
                └──────────┬──────────┘
                           │
                    HTTP GET / POST
                           │
                           ▼
                ┌─────────────────────┐
                │      Flask App      │
                │       app.py        │
                └──────────┬──────────┘
                           │
            ┌──────────────┼──────────────┐
            │              │              │
            ▼              ▼              ▼
       Validation      Database       Calculations
            │              │              │
            │              ▼              │
            │       SQLite + ORM         │
            │              │              │
            └──────────────┼──────────────┘
                           │
                           ▼
                ┌─────────────────────┐
                │   Jinja2 Template   │
                │     index.html      │
                └──────────┬──────────┘
                           │
                           ▼
                ┌─────────────────────┐
                │   Dashboard Output  │
                │ Table + Total +     │
                │ Charts + Messages   │
                └─────────────────────┘
```

## Complete Application Workflow

### 1. Application Startup

When `app.py` starts:

1.  Flask creates the application.
2.  SQLite is configured through SQLAlchemy.
3.  A secret key is configured for Flask flash messages.
4.  SQLAlchemy is initialized.
5.  The `Expense` database model is defined.
6.  `db.create_all()` creates the required database table if it does not
    already exist.
7.  The application starts in debug mode.

The database configuration uses SQLite through Flask-SQLAlchemy.

## 2. Database Layer

The application defines an `Expense` model with four user-provided
fields plus an automatically generated ID:

  Field           Type      Purpose
  --------------- --------- ---------------------
  `id`            Integer   Primary key
  `description`   String    Expense description
  `amount`        Float     Expense amount
  `category`      String    Expense category
  `date`          Date      Expense date

The currently supported categories are:

``` text
Food
Utility
Health
Rent
Bills
```

SQLAlchemy is used as the ORM, so the application works with Python
objects rather than writing raw SQL for normal CRUD operations.

## 3. Home Page Workflow

The main dashboard is available at:

``` text
GET /
```

When the user opens the dashboard:

1.  Flask reads optional filter values from the URL.
2.  Start date, end date, and category are extracted.
3.  Date strings are converted into Python `date` objects.
4.  The date range is validated.
5.  A database query is created.
6.  Date filters are applied when provided.
7.  Category filtering is applied when provided.
8.  Matching expenses are retrieved.
9.  Expenses are ordered by date descending and then ID descending.
10. The total expense amount is calculated.
11. Daily spending data is grouped by date.
12. Category spending data is grouped by category.
13. All calculated data is passed to `index.html`.
14. Jinja2 renders the dashboard.

## 4. Adding an Expense

The Add Expense form sends a POST request to:

``` text
POST /add
```

The workflow is:

``` text
User enters expense
       │
       ▼
HTML form submits POST
       │
       ▼
Flask reads form data
       │
       ▼
Validate description,
amount and category
       │
       ▼
Validate amount
       │
       ▼
Validate/parse date
       │
       ▼
Create Expense object
       │
       ▼
db.session.add()
       │
       ▼
db.session.commit()
       │
       ▼
Flash "Expense added"
       │
       ▼
Redirect to dashboard
```

### Input Validation

The backend checks that:

-   Description is not empty.
-   Amount is not empty.
-   Category is not empty.
-   Amount can be converted to a number.
-   Amount is greater than zero.
-   Date follows the expected format.

If validation fails, a flash error message is displayed and the user is
redirected back to the dashboard.

## 5. Filtering Expenses

The dashboard allows the user to filter expenses using:

``` text
Start Date
End Date
Category
```

The filter values are sent as URL query parameters.

Example:

``` text
/?start=2026-08-01&end=2026-08-22&category=Food
```

The backend dynamically builds the database query based on the filters.

The same filtering logic is used for:

-   Expense table
-   Total expense calculation
-   Category chart
-   Daily spending chart
-   CSV export

This keeps the dashboard consistent with the selected filter.

## 6. Date Range Validation

The application checks:

``` text
end_date < start_date
```

If the end date occurs before the start date:

1.  A Flask error message is displayed.
2.  The invalid dates are cleared.
3.  The application falls back to the unfiltered date range.

This prevents an invalid date range from being used in the expense
query.

## 7. Total Expense Calculation

After retrieving the filtered expenses, the application calculates:

``` text
Total = sum of all filtered expense amounts
```

The result is rounded to two decimal places and displayed on the
dashboard.

For example:

``` text
Total: ₹1,250.50
```

The total therefore represents the currently selected filter rather than
necessarily the total of every record in the database.

## 8. Category Analysis

The backend performs a database aggregation:

``` text
Category → SUM(amount)
```

For example:

``` text
Food      → ₹2,500
Utility   → ₹1,200
Health    → ₹800
Rent      → ₹8,000
Bills     → ₹1,500
```

These values are sent to the frontend as:

``` text
cat_labels
cat_values
```

The frontend uses Chart.js to create a pie chart titled:

``` text
Expenses by Category
```

## 9. Daily Spending Analysis

The backend groups expenses by date:

``` text
Date → SUM(amount)
```

The resulting values are sent to the frontend as:

``` text
day_labels
day_values
```

Chart.js uses these values to generate a bar chart titled:

``` text
Spending Over Time
```

This allows the user to see how spending changes from day to day.

## 10. Deleting an Expense

The delete operation is handled by:

``` text
POST /delete/<expense_id>
```

Workflow:

``` text
User clicks Delete
       │
       ▼
Browser asks for confirmation
       │
       ▼
POST request sent
       │
       ▼
Flask finds expense by ID
       │
       ▼
Expense is deleted
       │
       ▼
Database commit
       │
       ▼
Success message
       │
       ▼
Redirect to dashboard
```

The frontend includes a confirmation dialog before submitting the delete
request.

## 11. CSV Export

The application provides:

``` text
GET /export_csv
```

The export follows the currently selected:

-   Start date
-   End date
-   Category

The backend:

1.  Reads the filter values.
2.  Converts date values.
3.  Builds the same filtered expense query.
4.  Retrieves matching expenses.
5.  Creates an in-memory CSV using `io.StringIO`.
6.  Writes a header row.
7.  Writes each expense as a CSV row.
8.  Generates a filename based on the selected date range.
9.  Returns the CSV as a downloadable HTTP response.

CSV columns:

``` text
date
description
category
amount
```

## 12. Frontend Workflow

The dashboard is implemented in `index.html`.

The page contains:

``` text
Dashboard
│
├── Flash Messages
│
├── Header
│
├── Filter Expenses
│   ├── Start Date
│   ├── End Date
│   ├── Category
│   ├── Apply
│   ├── Reset
│   └── Total
│
├── Export CSV
│
├── Add Expense
│   ├── Description
│   ├── Amount
│   ├── Date
│   ├── Category
│   └── Add Expense
│
├── Expense Table
│   ├── Date
│   ├── Description
│   ├── Category
│   ├── Amount
│   └── Delete
│
└── Visualizations
    ├── Expenses by Category
    └── Spending Over Time
```

The page uses Tailwind CSS for styling and Chart.js for charts.

## 13. Data Flow

### Add Expense

``` text
HTML Form
   ↓
POST /add
   ↓
Flask
   ↓
Validation
   ↓
Expense Model
   ↓
SQLAlchemy
   ↓
SQLite
   ↓
Redirect /
   ↓
Updated Dashboard
```

### Filter Expenses

``` text
Filter Form
   ↓
GET /
   ↓
Read query parameters
   ↓
Parse dates
   ↓
Build SQLAlchemy query
   ↓
Filter SQLite records
   ↓
Calculate total
   ↓
Calculate chart data
   ↓
Render index.html
```

### Export CSV

``` text
Export CSV
   ↓
GET /export_csv
   ↓
Read current filters
   ↓
Query filtered expenses
   ↓
Generate CSV in memory
   ↓
HTTP Response
   ↓
Browser downloads CSV
```

## 14. Error Handling

The application uses Flask flash messages to communicate operation
results to the user.

Examples:

``` text
Success:
"Expense added"
"Expense deleted"

Error:
"Please fill description, amount, and category"
"Amount must be a positive number"
"Invalid date. Using today's date."
"End date cannot be before start"
```

The frontend displays these messages with different visual styles for
success and error states.

## 15. Installation and Setup

### Prerequisites

-   Python 3.x
-   pip
-   Git

### Clone the Repository

``` bash
git clone https://github.com/YOUR_USERNAME/expense-tracker.git
cd expense-tracker
```

### Create a Virtual Environment

Windows:

``` powershell
python -m venv venv
```

### Activate the Environment

Windows PowerShell:

``` powershell
.\venv\Scripts\activate
```

### Install Dependencies

``` bash
pip install -r requirements.txt
```

### Run the Application

``` bash
python app.py
```

Open the local application in your browser at:

``` text
http://127.0.0.1:5000/
```

## 16. Git and Virtual Environment

The virtual environment is intentionally excluded from version control.

Instead of uploading:

``` text
venv/
```

the project uploads:

``` text
requirements.txt
```

Another developer can recreate the environment with:

``` bash
python -m venv venv
pip install -r requirements.txt
```

This keeps the GitHub repository small and portable.

## 17. Database Notes

The application uses SQLite for local persistence.

The database is created through:

``` text
db.create_all()
```

The application can therefore create the required database table when it
starts.

The local database file should generally not be committed when it
contains personal or test data.

## 18. Technologies Demonstrated

This project demonstrates practical use of:

-   Python
-   Flask
-   Flask routing
-   HTTP GET and POST requests
-   Jinja2 templating
-   Flask-SQLAlchemy
-   SQLAlchemy ORM
-   SQLite
-   CRUD operations
-   Form handling
-   Server-side validation
-   Query filtering
-   Database aggregation
-   CSV generation
-   HTTP file downloads
-   JavaScript
-   Tailwind CSS
-   Chart.js
-   Git and GitHub
-   Python virtual environments
-   Dependency management with `requirements.txt`

## 19. Future Improvements

Possible improvements for future versions include:

-   User authentication and separate expense data per user
-   Edit/update expense functionality
-   Pagination for large expense lists
-   Monthly and yearly spending summaries
-   Budget tracking and budget alerts
-   Recurring expenses
-   More advanced reports
-   Downloadable PDF reports
-   Improved server-side security configuration
-   CSRF protection for POST forms
-   Environment variables for configuration and secrets
-   Production deployment with a production WSGI server
-   Automated tests
-   Database migrations using Flask-Migrate

## 20. Project Learning Outcome

This project demonstrates how a Python backend can be connected to a
database and a web interface to build a complete CRUD application.

The overall concept is:

``` text
Frontend
   ↓
HTTP Request
   ↓
Flask Route
   ↓
Validation / Business Logic
   ↓
SQLAlchemy ORM
   ↓
SQLite Database
   ↓
Query / Aggregation
   ↓
Flask Response
   ↓
Jinja2 + JavaScript
   ↓
Dashboard
```

The project therefore combines backend development, database management,
frontend rendering, data visualization, and file export into one
complete web application.
