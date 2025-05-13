# Chapter 4

## Bookstore: One-to-many

Use your own bookstore application from the GitHub.

Check StudentList example from the moodle (Student – Department relationship).

1. Add new entity called Category which has attributes id and name
2. Add one-to-many relationship between Book and Category
   1. Add new category attribute to the Book entity
   2. Add new books attribute to the Category entity (List of books)
   3. Add getters and setters for new attributes
   4. Add annotations for one-to-many relationship
3. Add crud repository class for the Category entity
4. Add Category dropdown list to the book creation form
   1. Add categories to controller model
   2. Show category name in the dropdown list
5. Add Category into book listpage to show category name of the book
6. Insert few categories to database by using CommandLineRunner

Push all changes to GitHub.

## MySQL: Booklist (Optional)

This exercise will be returned to Moodle

Check StudentListJDBC example from the moodle.

1. Install local MySQL database using XAMPP (default username: root, password empty and name of the database is test
2. Create similar software as booklist from previous week, but make it use MySQL
