Suppose you have a web app with an architecture like this

![App architecture](assets/app_arch.png)

The web server is the only thing that can query your internal API and the database, but if somebody finds a way to make your web server send requests to the internal API and database with anything that he wants, he will be **F**orging **R**equests on the **S**erver **S**ide, which, depending on your setup, could result in something very catastrophic.


