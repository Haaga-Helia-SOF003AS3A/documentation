<!-- Slide number: 1 -->

# Back End Programming: Using Postgre Database

Minna Pellikka

This document explains how we replace the runtime H2 database with an external database.

<!-- Slide number: 2 -->

## Install PostgreSQL

1. Install PostgreSQL on your own machine if you don't already have it there. Link is htps://www.postgresql.org/download/

   PostgreSQL installation instructions e.g. here
   htps://www.w3schools.com/postgresql/postgresql_install.php
   or here
   htps://www.postgresqltutorial.com/postgresql-ge ng-started/install-postgresql/
   (No need to install Stackbuilder)
   Save the password you entered during installation – you'll need it later.

2. Launch the pgAdmin application (htps://www.pgadmin.org/docs/)

3. Create a new database by hovering over the Databases symbol under PostgreSQL and right-clicking on it. Select Create > Database from the menu

4. Give your Database a name and finally click Save

5. Go to the code editor and create a "SQL script" for your Backend project, which contains the statements for creating tables. Save the database either to the root of your project or to the resources folder.

6. You can run the scrip in the previous section to the side of the ethos you have created, e.g. the following : a) select Schemas > Tables. b) On top of Tables, select PSQL Tool from the pop-up menu c) copy to the PSQL editor SQL commands from the database

7. Now you have an etho database on your own machine -> let's go to modify the Spring Boot application, which it would use instead of the PostgreSQL database in H2

8. Open the SB project application.properties database and define the address of the database you created and the username & password. It would be good to parameterize the correct url, username & password instead of exporting the database to version control, but let's put them in plain language in the database. Below is an example, the ones colored in yellow update according to the settings of your own database:

9. Take a new dependency for POM.XML development

10. Comment out of the main application category test data creation.

11. Check that entity classes names match of table names. Remember the caseSensitivity. If you notice a difference in the name, correct it by @Table annotation

Check also entity column names.

12. Start your Backend application and test the functionality.

13. If the application works, take it to version control. Note! maybe you want to use git branches to H2 version and PostgreSQL version.

14. Export also the db script to git.
