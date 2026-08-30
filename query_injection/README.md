To query relational databases (databases with tables), applications can use a language called SQL (Structured Query Language). In that language, a query to get users with age greater than 18 is:

```sql
SELECT name FROM users WHERE age > 18; 
```

Another widely used type of database is document-based, which uses NoSQL query languages. These use JSON with operators to make queries. One of the well known NoSQL engines is MongoDB. For example, to make the same query as above, you instead use:

```js
db.users.find({age: {"$gt": 18}});
```

That `$gt` is the MongoDB operator for $\gt$.

This class of vulnerabilities is basically being able to inject query language operators and expressions into the query that the application is doing to the database. For example; we can suppose that some web app is using the first SQL query and allowing users to specify the age to filter in a search parameter, like this:

```bash
https://uwuowo.com/users/search?age_gt=18
```

If this application just enters that number into the DB query without escaping or sanitization, then one can put:

```bash
https://uwuowo.com/users/search?age_gt=0%20UNION%20SELECT%20password%20FROM%20admin_users%20--
```

And then, the database would instead do this:

```sql
SELECT name FROM users WHERE age > 0 UNION SELECT password FROM admin_users --
```

That would return the names of the users whose age is any positive number, *plus the passwords of the admin users appended as extra rows of the same column.*

Depending on the application, you may be able to get data, just get a `true` or `false` type response, or just make it throw an error because the returned data is not what was expected. Functions like `sleep()` and watching how the application behaves when something exists and when it doesn't is what will give you the ability to extract sensitive data from the application in the last two cases.
