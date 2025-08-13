
# Spring Boot + PostgreSQL (Linux-ohje)

_Updated: 2025-08-13_ Jukka Juslin

This is instruction on how to use external "real" database instead of
no-setup-needed H2 db. 

On mac use brew program instead. This instruction is for WSL2 on
Windows or for a Linux computer. 

## 1) Install PostgreSQL

Install needed new database:

### Ubuntu WSL2
```bash
sudo apt update
sudo apt install -y postgresql postgresql-contrib
# Käynnistys ja käynnistyksen varmistus
sudo systemctl enable --now postgresql
# Luo postgres-sisäinen superuser-sessio
sudo -u postgres psql
```

Last command opens you Postgre CLI, so you can do all operations there.
If you type \h you get list of all commands.

If you have used MySQL before you do know that each database has a name.
In here you change to a specific database with \c database-name


## 2) Create database and a user

Run in the shell started like the last command in
previous step showed. Needs elevated privileges, so therefore the sudo there
is one way to get that.
```sql
-- After sign in you will see psql prompt: postgres=#
CREATE DATABASE myappdb ENCODING 'UTF8';
CREATE USER myapp WITH ENCRYPTED PASSWORD 'strong_enough_password_here';
GRANT ALL PRIVILEGES ON DATABASE myappdb TO myapp;
```
This allows connecting to your database, but actually this works until the end
only after next instructions, where database owner is changed. 

Here, like often with databases, the username is same
as the the database name. Database is on your local computer in port 5432 - default of postgresql.

After ownership is changed Spring Boot can also create the tables and fill with dummy data, if wanted.
Also is possible to manually create tables when using JPA. Matter of taste, but on this course most work
is done so that we let Spring Boot create tables.

```sql
\c myappdb
CREATE SCHEMA IF NOT EXISTS public AUTHORIZATION myapp;
ALTER DATABASE myappdb OWNER TO myapp;
```
Now the database should be up, but should not be possible to connect from other computers than your own.
Network security wise that is essential that that is the situation. This could be checked for example
by using **telnet** program.

## 3) SQL-scripts to create databases (in a way optional, but confirms step by step that you have
needed rights to execute SQL statements and that they work)

Create in your project test SQL table creation `src/main/resources/schema.sql` and `data.sql`. Example:
```sql
CREATE TABLE IF NOT EXISTS Person (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

Previous was schema.sql and next is data.sql.


```sql
INSERT INTO Person(name) VALUES ('Jukka');

```


Run those files using psql like with this syntax:

```bash
# run at project root
psql "postgresql://myapp:strong_password_here@localhost:5432/myappdb" -f src/main/resources/schema.sql
```

## 4) Spring Boot -configuration (application.properties or application.yml)

`src/main/resources/application.properties`:
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/myappdb
spring.datasource.username=myapp
spring.datasource.password=strong_password_here
spring.datasource.driver-class-name=org.postgresql.Driver

# Hibernate/JPA
spring.jpa.hibernate.ddl-auto=none        # käytä none/validate/update tarpeesi mukaan
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.open-in-view=false

# Valinnainen: connection pool (Hikari)
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=2
spring.datasource.hikari.idle-timeout=600000
```

**Do** not commit clear text passwords. Hard code on your computer to environment variables and use käytä Springin variable naming convention:
```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASS}
```

## 5) Add PostgreSQL-driver

**Maven (`pom.xml`)**:
```xml
<dependency>
  <groupId>org.postgresql</groupId>
  <artifactId>postgresql</artifactId>
  <version>42.7.3</version>
</dependency>
```

**Gradle (Kotlin DSL)**:
```kotlin
dependencies {
  implementation("org.postgresql:postgresql:42.7.3")
}
```

Remove or comment out with <!-- --> H2 dependency. Preferably is commenting out because can be useful with projects to start with simple H2
though it is not for production environments. In maven pom.xml that looks like:
```xml
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <scope>test</scope>
</dependency>
```

## 6) Names of Entity files and table names

PostgreSQL is case sensitive. Some words are reserved by PostgreSQL like **User**. In case you are a student with Finnish language skill
reserved words is not a problem in case you make your program in Finnish language. Otherwise you can check list of reserved words that none
of your entity name is the same in lowercase of uppercase. You can use annotations @Table and @Column to rename entities in the actual database,
but would be more straightforward if names pass the way they are. For example User can be AppUser etc.

```java
@Entity
@Table(name = "persontable")
public class Person {
  @Id @GeneratedValue
  private Long id;

  @Column(name = "firstname")
  private String name;
}
```

## 7) Start and test

```bash
# Maven
./mvnw spring-boot:run

# Gradle
./gradlew bootRun
```

When project is starting follow always closely the command line, also in the cloud, for errors. Framework tells quite clearly what is wrong if something
is wrong - it is just mandatory that you always have clear visibility to the output - also in the cloud. Common problems are signin failures, database
is not existing or driver is not existing. Fix errors, save files and try to run again.

## 8) Version control

- Commit your project with changes to github. Do not include commits where passwords are in clear text. It is when implementing software easier to have
first local version that has clear text password, but never commit that to github.
- Also commit to github **also** SQL scripts (`schema.sql`, `data.sql`).
- Modify .gitignore if needed. Check .vscode if there are hard coded file locations that will not work for others, so in that case add the file to .gitignore.

## 9) JPA-tests for PostgreSQL:ää (if you do not use H2 in tests)

Add tester profile `src/test/resources/application-test.properties`:
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/myappdb_test
spring.datasource.username=myapp
spring.datasource.password=test_password
spring.jpa.hibernate.ddl-auto=create-drop
spring.test.database.replace=none
```

Start tests with test profile:
```bash
# Maven
./mvnw test -Dspring.profiles.active=test

# Gradle
./gradlew test -Dspring.profiles.active=test
```

---

### Troubleshooting (WSL2)

- **Service does not start:** `systemctl status postgresql`, `journalctl -u postgresql -e`
- **Port 5432 reserved for other process :** `telnet 5432`
- **Connection failure:** check `pg_hba.conf` and `postgresql.conf`. path: `/etc/postgresql/<versio>/main/`.
- **UTF-8/locale:** make sure DB files are UTF-8 encoded

---

## TL;DR
1) Install PostgreSQL and start service
2) Create database `myappdb` + user `myapp`. Add dummy data.
3) Add `org.postgresql:postgresql` dependency.
4) Set `spring.datasource.*` for PostgreSQL
5) Comment out H2 when running on a real production database like PostgreSQL or MySQL
