---
layout: post
title: Simplicity vs Clarity
date: 2022-01-08 18:20:00 +0800
categories: architecture
---
Hi, it's been a while since my last post, I've been so busy this week, don't have much time for myself. [:(](# "jk :D, it is my hobby to create programs.")

Anyway, for the brief introduction I will share how I failed to design a Nodejs backend just because I want to simplify everything.

While trying to experiment by mixing logic and existing source code implementations,
I tried to simplify things by using a parameter naming convention. `Use same parameter names in Backend, Database and Front-end`.

The coding is easy and really fast, **just copy and paste the parameter names from Database to Backend upto Front-end is like coding in breeze**, rescued some of my braincells overthinking about parameter names and the keyboard thing.

Yet. it is a parameter naming issue if we failed to monitor the implementation.

Given the below examples:
```sql
-- SQL
CREATE PROCEDURE sp_process_employees(
    p_id INT,
    p_name VARCHAR(50),
    p_middle_name VARCHAR(50),
    p_last_name VARCHAR(50),
    p_nick_name VARCHAR(20),
    p_company_id INT,
    p_user_id INT
) 
BEGIN
    -- some process here...
END
|
```
```js
// NodeJS backend/models/employee.js
const process = async ({
    id,
    name,
    middle_name,
    last_name,
    nick_name,
    company_id,
    user_id,
}, transaction) => {
    // some process here to execute
}
```
```dart
/// models/employee.dart
class Employee {
    int id;
    String name;
    String middle_name;
    String last_name;
    String nick_name;
    int company_id;
    int user_id;

    Employee({
        this.id,
        this.name,
        this.middle_name,
        this.last_name,
        this.nick_name,
        this.company_id,
        this.user_id,
    });
}
```

the above code was clear and straight, until there's a bug.

-
-
-

```dart
/// models/employee.dart
class Employee {
    int id;
    String name;
    String middle_name;
    String last_name;
    String nick_name;
    int company_id;     // <-- this belongs to employee
    int user_id;        // <-- also this
}
```
```js
// NodeJS backend/models/employee.js
const process = async ({
    id,
    name,
    middle_name,
    last_name,
    nick_name,
    company_id,     // <-- this belongs to employee
    user_id,        // <-- while this should be the current system user, it is intended for created/modified_by_id
}, transaction) => {
    // some process here to execute
}
```

instead of simplicity, I made the backend convuluted enough to mistrust the origin of parameters.

-
-
-


![spiders.jpg](/assets/images/spidermans.jpg)


### Fix
always separate the session related parameters from domain logic parameters. we can use IoC function hierarchy to achieve this. Also avoid using the generic term `user_id` alone, we have to include at least the origin or purpose when dealing with generic parameter names.

```js
// revised backend/models/employee.js
const process = async (
    { transaction },                      // <-- dependency injections
    { user_id: session_user_id },         // <-- session related parameters
) => {
    return async ({
        id,              // <-- business logic parameters
        name,
        middle_name,
        last_name,
        nick_name,
        company_id,
        user_id,         // <-- belongs to employee
    }) => {
        // process here
    }
}
```

a much better version
```js
// revised backend/models/employee.js
const process = async (
    { transaction },
    session,             // <-- a backend common term, can be `token` or `claims`
) => {
    return async ({
        id,
        name,
        middle_name,
        last_name,
        nick_name,
        company_id,
        user_id,
    }) => {
        // process here, use session.user_id
    }
}
```
```js
// NodeJS backend/controllers/employee.js
const process = async (req, res) => {
    let transaction = await createTransaction();
    let command = await Employees.process({transaction}, req.session);
    let result = await command(req.body);
    await commit(transaction);
    return res.status(200).json(result);
}
```

## Conclusion
When there is need to simplify things, a ruling convention is not always the solution.
Most of the time we overlooked the current architecture the reason why we lose clarity.

<center>- end -</center>
