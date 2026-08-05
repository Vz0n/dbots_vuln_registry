To query relational databases (database with tables), applications can use a language called SQL (Structured Query Language). On that language, a query to get users with age greater 18 is:

```sql
SELECT name FROM users WHERE age > 18; 
```

Another widely used type of database is document-based, which uses the NoSQL query language. This language uses JSON with operators to make queries. One of the well known NoSQL engines is MongoDB. For example, to make the same query as above, you instead use:

```js
db.users.find({age: {"$gt": 18}});
```

That `$gt` is the MongoDB operator for $\gt$.

This class of vulnerabilities are basically being able to inject query language's operators and expressions into the query that the application is doing to the database. For example; we can suppose that some web is using the first SQL query and allowing users to specify the age to filter in a search parameter, like this:

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

That would return the name from users whose age is any positive number *and the password from users that are administrators.*

Depending of the application, you may be able to get data, just get a `true` or `false` type response, or just make it throw an error because the returned data is not the expected. Functions like `sleep()` and watching how the application behaves when something exists and when no is what will give you the ability to extract sensitive data from the application in the last two cases.