# Spring Boot + MySQL (Remote host softala.haaga-helia.fi)

_Updated: 2026-02-11_ Jukka Juslin

This is instruction on how to use an external "real" database instead of
no-setup-needed H2 db and deploy the application to the cloud. The database server is **softala.haaga-helia.fi** — a
Linux host maintained by Haaga-Helia. MySQL is already installed there;
**you do not need to install anything on the server side**.

Your teacher will provide you with:
- a **username** (e.g. `sp_student01`)
- a **password**
- a **database name** (typically the same as the username)

> **Important:** The remote host **blocks direct connections** to MySQL port
> 3306. You must use an **SSH tunnel** to forward the remote port to your
> local machine. All connections (CLI, GUI tools, Spring Boot) go through
> `localhost:3306` after the tunnel is established.

### Prerequisites — basic Linux commands

You will be working on a remote Linux server. You should know or learn the
basics of the following commands:

| Command | Purpose |
|---------|---------|
| `ls -al` | List files with details |
| `mkdir` | Create a directory |
| `rm` | Remove files |
| `nano filename` | Simple text editor — edit a file in the terminal. Save with `Ctrl+O`, exit with `Ctrl+X` |
| `less` | View file contents page by page |
| `tail -f` | Follow a log file in real time |
| Tab key | Auto-complete file and directory names |
| Up arrow | Recall previous commands |

> **Copy & paste in Git Bash (Windows): `Ctrl+C` and `Ctrl+V` do NOT work!
> To copy text, select it with the mouse — it is copied automatically.
> To paste, right-click the mouse or press `Shift+Insert`.**

### MySQL command line reference

Once connected to MySQL with `mysql -h localhost -u sp_student01 -p`, you
can use the following commands:

| Command | Purpose |
|---------|---------|
| `SHOW DATABASES;` | List all databases you have access to |
| `USE sp_student01;` | Switch to your database |
| `SHOW TABLES;` | List all tables in the current database |
| `DESCRIBE Person;` | Show column names, types and keys of a table |
| `SELECT * FROM Person;` | View all rows in a table |
| `SELECT * FROM Person LIMIT 10;` | View first 10 rows of a table |
| `SELECT COUNT(*) FROM Person;` | Count the number of rows in a table |
| `EXIT;` or `\q` | Quit the MySQL client |

## 1) Build the JAR file (on your local machine)

Before deploying to the server you need to package your Spring Boot
application into a JAR file. You may need to download the correct version of
Maven first. Run the following command in your project root:

```bash
./apache-maven-3.9.9/bin/mvn -U -Dmaven.test.skip=true package
```

This skips tests and creates a JAR file in the `target/` directory
(e.g. `target/myapp-0.0.1-SNAPSHOT.jar`).

## 2) Open an SSH tunnel

Before doing anything else, open **Git Bash** and start an SSH tunnel. This
forwards the remote MySQL port 3306 to your local machine's port 3306:

```bash
ssh -L 3306:localhost:3306 sp_student01@softala.haaga-helia.fi
```

You will be prompted for your SSH password (same credentials your teacher
gave you). **Keep this Git Bash terminal open** for as long as you need the database
connection. Once connected you also have a regular SSH shell on the server.

> **Tip:** If your local port 3306 is already in use (e.g. you have a local
> MySQL running), pick a different local port such as 3307:
> ```bash
> ssh -L 3307:localhost:3306 sp_student01@softala.haaga-helia.fi
> ```
> Then use `localhost:3307` everywhere below instead of `localhost:3306`.

## 3) Copy the JAR file to the server

Open another **Git Bash** terminal and use `scp` to copy the JAR file to
the remote server:

```bash
scp target/myapp-0.0.1-SNAPSHOT.jar sp_student01@softala.haaga-helia.fi:
```

The colon `:` at the end means the file will be placed in your home directory
on the server.

## 4) Log in to the server and run the application

Open an interactive SSH session to the server:

```bash
ssh sp_student01@softala.haaga-helia.fi
```

Once logged in, start the application:

```bash
java -jar myapp-0.0.1-SNAPSHOT.jar
```

You can follow the application log output in real time. Press `Ctrl+C` to
stop the application.

> **Tip:** To keep the application running after you close the terminal, use:
> ```bash
> nohup java -jar myapp-0.0.1-SNAPSHOT.jar &
> ```
> You can then follow the log with `tail -f nohup.out`.

## 5) Enable HTTPS (without nginx or Apache)

Spring Boot has built-in HTTPS support. You can generate a self-signed
certificate and configure Spring Boot to use it directly — no reverse proxy
needed.

### Generate a self-signed certificate

Run this on the server (or on your local machine before copying the JAR):

```bash
keytool -genkeypair -alias myapp -keyalg RSA -keysize 2048 \
  -storetype PKCS12 -keystore keystore.p12 -validity 365 \
  -storepass changeit -dname "CN=softala.haaga-helia.fi"
```

This creates a `keystore.p12` file. Place it in the same directory as your
JAR file on the server, or inside `src/main/resources/` in your project
before building.

### Configure Spring Boot to use the certificate

Add the following to your `application.properties`:

```properties
# HTTPS
server.port=8443
server.ssl.key-store=keystore.p12
server.ssl.key-store-password=changeit
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=myapp
```

If you placed the keystore inside `src/main/resources/`, use:
```properties
server.ssl.key-store=classpath:keystore.p12
```

If the keystore is in the same directory as the JAR on the server, use:
```properties
server.ssl.key-store=file:./keystore.p12
```

Now start the application and access it at `https://softala.haaga-helia.fi:8443`.

> **Note:** Since this is a self-signed certificate, browsers will show a
> security warning. This is normal for development and testing. Click
> "Advanced" → "Proceed" to continue.

> **Tip:** If you also want to redirect HTTP to HTTPS, you can add a
> configuration class in your project. However, for a simple student project
> using only HTTPS on port 8443 is sufficient.

## 6) Run the application as a systemd service

Instead of running the JAR manually with `java -jar` or `nohup`, you can
set it up as a **systemd service** so it starts automatically and restarts
on failure.

### Create a service file

On the server, create a service file (replace `sp_student01` and file names
with your own):

```bash
sudo nano /etc/systemd/system/myapp.service
```

Paste the following content:

```ini
[Unit]
Description=My Spring Boot Application
After=network.target

[Service]
User=sp_student01
WorkingDirectory=/home/sp_student01
ExecStart=/usr/bin/java -jar /home/sp_student01/myapp-0.0.1-SNAPSHOT.jar
SuccessExitStatus=143
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

> **Note:** Creating the service file with `sudo` may require administrator
> privileges. Ask your teacher if you do not have `sudo` access. As an
> alternative you can use the `nohup` approach described in step 4.

### Enable and start the service

```bash
# Reload systemd to pick up the new service file
sudo systemctl daemon-reload

# Enable the service to start on boot
sudo systemctl enable myapp

# Start the service now
sudo systemctl start myapp

# Check that it is running
sudo systemctl status myapp
```

### Useful service commands

| Command | Purpose |
|---------|---------|
| `sudo systemctl start myapp` | Start the application |
| `sudo systemctl stop myapp` | Stop the application |
| `sudo systemctl restart myapp` | Restart after deploying a new JAR |
| `sudo systemctl status myapp` | Check if the application is running |
| `journalctl -u myapp -f` | Follow the application log in real time |

After deploying a new JAR file, simply run:
```bash
sudo systemctl restart myapp
```

## 7) Verify access to MySQL through the tunnel

With the SSH tunnel running, open a **second** terminal and verify your
MySQL credentials. You can use any of the following:

### Option A — MySQL CLI client (Linux / macOS)
```bash
# Connect through the SSH tunnel
mysql -h localhost -u sp_student01 -p
```
You will be prompted for your password. After a successful login you will see
the `mysql>` prompt. Type `SHOW DATABASES;` to see your database.

### Option B — MySQL Workbench or DBeaver (Windows / macOS / Linux)
Use any graphical database tool. Create a new connection with:
- **Host:** `localhost`
- **Port:** `3306` (or the local port you chose)
- **Username:** your MySQL username (e.g. `sp_student01`)
- **Password:** your MySQL password
- **Database:** your database name (e.g. `sp_student01`)

### Option C — IntelliJ / VS Code database extension
Most IDEs have a built-in or plugin-based database browser that can connect
the same way using `localhost:3306`.

## 8) Explore your database

Your teacher has already created a database and a user for you. Typically the
database name is the same as your username. After logging in:

```sql
-- Check which databases you can see
SHOW DATABASES;

-- Switch to your database
USE sp_student01;

-- List existing tables (probably empty at this point)
SHOW TABLES;
```

You do **not** need to create a database or a user — that has been done for you.

## 9) SQL-scripts to create tables (optional, but confirms that your access works)

Create in your Spring Boot project test SQL files
`src/main/resources/schema.sql` and `data.sql`. Example:

**schema.sql:**
```sql
CREATE TABLE IF NOT EXISTS Person (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**data.sql:**
```sql
INSERT INTO Person(name) VALUES ('Jukka');
```

You can run these manually to verify they work:

```bash
# run at project root (SSH tunnel must be open)
mysql -h localhost -u sp_student01 -p sp_student01 < src/main/resources/schema.sql
mysql -h localhost -u sp_student01 -p sp_student01 < src/main/resources/data.sql
```

## 10) Spring Boot configuration (application.properties)

`src/main/resources/application.properties`:
```properties
# Connects through the SSH tunnel on localhost
spring.datasource.url=jdbc:mysql://localhost:3306/sp_student01
spring.datasource.username=sp_student01
spring.datasource.password=your_password_here
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# Hibernate/JPA
spring.jpa.hibernate.ddl-auto=update       # use none/validate/update as needed
spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect
spring.jpa.open-in-view=false

# Optional: connection pool (Hikari)
spring.datasource.hikari.maximum-pool-size=5
spring.datasource.hikari.minimum-idle=2
spring.datasource.hikari.idle-timeout=600000
```

**Do not** commit clear text passwords. Store them in environment variables
and use Spring's variable substitution:
```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASS}
```

## 11) Add MySQL driver

**Maven (`pom.xml`)**:
```xml
<dependency>
  <groupId>com.mysql</groupId>
  <artifactId>mysql-connector-j</artifactId>
</dependency>
```
Spring Boot manages the version automatically via the parent POM, so you do
not need to specify a `<version>` tag.

Remove or comment out H2 dependency. Commenting out is preferable because H2
can still be useful during early development. In Maven `pom.xml`:
```xml
<!--
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <scope>runtime</scope>
</dependency>
-->
```

## 12) Entity class names and table names

MySQL on Linux is **case-sensitive** for table names (unlike on Windows).
Some words are reserved by MySQL, for example **User**, **Order**, **Group**.
You can use `@Table` and `@Column` annotations to rename entities in the
database, but it is simpler to avoid reserved words altogether. For example
`User` → `AppUser`, `Order` → `CustomerOrder`.

```java
@Entity
@Table(name = "persontable")
public class Person {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  @Column(name = "firstname")
  private String name;
}
```

> **Note:** With MySQL use `GenerationType.IDENTITY` instead of the default
> `GenerationType.AUTO`, because MySQL uses `AUTO_INCREMENT` columns for
> primary keys.

## 13) Start and test

```bash
./mvnw spring-boot:run
```

When the project is starting, follow the command line output closely for
errors. Common problems are:
- **Access denied** — wrong username or password
- **Unknown database** — wrong database name in the URL
- **Communications link failure** — server not reachable, check network / VPN
- **Driver not found** — MySQL dependency missing from `pom.xml`

Fix errors, save files and try to run again.

## 14) Version control

- Commit your project to GitHub. **Never** commit clear text passwords.
  It is easier during development to first hard-code the password locally,
  but always switch to environment variables before committing.
- Also commit SQL scripts (`schema.sql`, `data.sql`).
- Modify `.gitignore` if needed. Check `.vscode/` for hard-coded file paths
  that will not work for others.

## 15) JPA tests with MySQL (if you do not use H2 in tests)

Add a test profile `src/test/resources/application-test.properties`:
```properties
# SSH tunnel must be running
spring.datasource.url=jdbc:mysql://localhost:3306/sp_student01
spring.datasource.username=sp_student01
spring.datasource.password=your_password_here
spring.jpa.hibernate.ddl-auto=create-drop
spring.test.database.replace=none
```

> **Note:** Because you share the database with your running application,
> using `create-drop` in tests **will drop all tables** when tests finish.
> Consider using a separate test database if your teacher provides one, or
> use H2 for tests instead.

Start tests with the test profile:
```bash
# Maven
./mvnw test -Dspring.profiles.active=test
```

---

### Troubleshooting

- **Connection refused on localhost:3306:** make sure the SSH tunnel is
  running in another terminal. Re-run:
  `ssh -L 3306:localhost:3306 sp_student01@softala.haaga-helia.fi`
- **Access denied for user:** double-check username, password and database
  name. Remember that the database name in the JDBC URL must match exactly.
- **Port 3306 already in use locally:** you may have a local MySQL running.
  Use a different local port (e.g. 3307) in the SSH tunnel command and update
  the JDBC URL accordingly.
- **SSH connection drops / times out:** some networks drop idle SSH
  connections. Add `-o ServerAliveInterval=60` to the SSH command to send
  keep-alive packets.
- **SSL errors:** if the server requires SSL, add `?useSSL=true` to the JDBC
  URL. If it does not, you can add `?useSSL=false` to suppress warnings.
- **Time zone error:** add `?serverTimezone=Europe/Helsinki` to the JDBC URL
  or combine: `?useSSL=false&serverTimezone=Europe/Helsinki`

---

## TL;DR
1. Build the JAR: `./apache-maven-3.9.9/bin/mvn -U -Dmaven.test.skip=true package`
2. Teacher gives you a MySQL username, password and database name on
   `softala.haaga-helia.fi`.
3. Open an SSH tunnel: `ssh -L 3306:localhost:3306 sp_student01@softala.haaga-helia.fi`
4. Copy the JAR to the server: `scp target/myapp.jar sp_student01@softala.haaga-helia.fi:`
5. Log in and run: `java -jar myapp.jar`
6. Optional: enable HTTPS with a self-signed certificate in `application.properties`.
7. Optional: set up a systemd service for automatic start/restart.
8. Verify access with a MySQL client connecting to `localhost:3306`.
9. Add `com.mysql:mysql-connector-j` dependency.
10. Set `spring.datasource.*` properties pointing to `localhost:3306`.
11. Comment out H2 when running against the real database.
