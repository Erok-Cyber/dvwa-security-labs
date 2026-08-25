## Evidence

### Normal input

Input:

`1`

![Normal input](screenshots/01-normal-input.png)

### SQL syntax error

Input:

`'`

![SQL error](screenshots/02-sql-error.png)

### Security level

DVWA was configured to use the Low security level for the controlled lab exercise.


### Successful SQL Injection

Input:

`1' OR '1'='1`

The application returned multiple database records.

![Successful SQL Injection](Successful-sql-injection.png)
