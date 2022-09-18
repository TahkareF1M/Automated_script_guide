# Automating your SQL scripts

If you're reading this, I'm assuming that you have a basic knowledge of SQL and writing Python scripts to execute SQL queries on a database.
In this guide, I will explain how to automate the execution of your scripts by writing them as triggers instead of SQL queries.

### 1. What is a trigger ?

A trigger is a block of SQL code structured as follows:  
```
CREATE TRIGGER [trigger_name]
[BEFORE | AFTER | INSTEAD OF] [INSERT | DELETE | UPDATE] ON [table_name]
WHEN [conditions]
BEGIN
  [instruction];
  ...
  [instruction];
END;
```
Every time the specified event happens on the chosen table and the conditions are met, all the instructions between `BEGIN` and `END` are executed.
If your Python script executes a query that creates a trigger, the instructions inside will be executed automatically.  
Seems quite simple ? Yes but the real difficulties come from the fact that, unlike your old Python code, SQLite doesn't support variables, loops and conditions in the instruction block. In the two following parts, I will show you a way to "simulate" variables in triggers and walk you through my thought process when writing one of my scripts.

### 2. Snippets of code to "simulate" variables in SQLite

To store variables in SQLite, a possibility is to create tables in your Python code before creating your trigger:
```
cursor.execute("CREATE TABLE IF NOT EXISTS Variables_Text(Name TEXT PRIMARY KEY, Value TEXT, Persistent INT);")
cursor.execute("CREATE TABLE IF NOT EXISTS Variables_Int(Name TEXT PRIMARY KEY, Value INT, Persistent INT);")
cursor.execute("CREATE TABLE IF NOT EXISTS Variables_Real(Name TEXT PRIMARY KEY, Value REAL, Persistent INT);")
```
`Name` and `Value` are pretty self-explanatory column names. `Persistent` contains a 0 or a 1 indicating whether we can delete the variable at the end of the trigger or not.

There are two ways of storing variables in these tables.  
To store a known value, use:
```
REPLACE INTO Variables_[Text | Int | Real] (Name, Value, Persistent) VALUES ('[variable_name]', [variable_value], [0 | 1]);
```  
To retrieve a value in a column `[column]` of the table `[table]` and store it in a variable, use:
```
REPLACE INTO Variables_[Text | Int | Real] (Name, Value, Persistent) SELECT '[variable_name]', [column], [0 | 1] FROM [table] WHERE [conditions];
```

The value of a variable can be retrieved by using the following subquery:
```
(SELECT Value FROM Variables_[Text | Int | Real] WHERE Name = '[variable_name]')
```
Note that if you want to add a condition where a column is equal to the value of a variable, you need to use `IN` instead of `=`:
```
SELECT ... FROM ... WHERE [column] IN (SELECT Value FROM Variables_[Text | Int | Real] WHERE Name = '[variable_name]');
```

### 3. My thought process when writing a script

There are as many ways to tackle a problem as there are people in the world but here's how I wrote the `powertrain_evolution` trigger.

#### A. When do the script needs to be run ?

This one's easy, the powertrain stats must be updated at the start of every season (or said differently, when the season changes).
The season is stored in the "Player_State" table in the "CurrentSeason" column.
Thus, the trigger should look like:
```
CREATE TRIGGER [trigger_name]
AFTER UPDATE ON Player_State
WHEN OLD.CurrentSeason <> NEW.CurrentSeason
BEGIN
  ...
END;
```

#### B. What should my script do ?

TODO
