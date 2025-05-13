**Chapter 6**

## Bookstore: Securing the web application

Use your own bookstore application from the GitHub.

Check SecureStudentList example from the moodle

* 1. Add Spring Security to your project by adding dependency
  2. Add Spring Security configuration class which will secure all URLs (requires authentication in all URLs) and add two in-memory users with roles USER and ADMIN
  3. Create the login page and add login method to controller
  4. Add logout functionality to your booklist page
  5. Show authenticated username in your booklist page
  6. Restrict Delete book functionality only to users with ADMIN role

Push all changes to GitHub. Create in github a release of the steps before you overwrite them with other changes.

## Bookstore: Securing the web application Part II

Use your own bookstore application from the GitHub.

* 1. Create user entity for your application endusers
     1. Attributes: username (unique), password, email, role
  2. Create crud repository for user entity
  3. Implement UserDetailService
  4. Add some users to your database and hash the passwords with BCrypt hashing algorithm.
  5. Change Spring Security configuration to use user entities instead of in-memory users in authentication

Push all changes to GitHub