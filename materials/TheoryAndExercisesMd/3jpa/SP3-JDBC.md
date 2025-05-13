<!-- Slide number: 1 -->
# Server Programming: JDBC, Databases
Juha Hinkula

<!-- Slide number: 2 -->
# Spring Boot: JDBC
- Spring provides template class called `JdbcTemplate`
- `JdbcTemplate` takes care of connection handling and exception handling

```
Import org.springframework.jdbc.core.JdbcTemplate
```

- Dependency (Spring Boot automatically creates JdbcTemplate)

```
<dependency>
	 <groupId>org.springframework.boot</groupId>
	 <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

- Datasource configuration is controlled by using `application.properties` file

- Example:

```
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/ruichao?useUnicode=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC
spring.datasource.username=ruichao
spring.datasource.password=password
spring.datasource.initialization-mode=always
```

<!-- Slide number: 4 -->
- JdbcTemplate can be injected to your classes

```
@Autowired
private JdbcTemplate jdbcTemplate;
```

- Excecute single SQL statement (typically DDL) by using excecute method
```
jdbcTemplate.execute("CREATE TABLE customers...");
```

- You can also use `schema.sql` and `data.sql` files in resources folder which Spring Boot will automatically use to initialize database

<!-- Slide number: 5 -->
- Create a repository class and add methods
- Example

```
@Repository
public class StudentRepository
{
	@Autowired
	private JdbcTemplate jdbcTemplate;
	
	@Transactional(readOnly=true)
	public List<Student> findAll() {
		return jdbcTemplate.query("select * from student", new StudentRowMapper());
	}
}
```

<!-- Slide number: 6 -->
- Spring framework has comprehensive transaction support
- If method is tagged with `@Transactional`, meaning that any failure causes the entire operation to roll back to its previous state, and to throw the original exception

<!-- Slide number: 7 -->
- RowMapper is interface for mapping rows of a ResultSet
- Example

```
class StudentRowMapper implements RowMapper<Student> {
  @Override
  public Student mapRow(ResultSet rs, int rowNum) throws SQLException {
	Student student = new Student();
	student.setId(rs.getInt("id"));
	student.setName(rs.getString("name"));
	student.setEmail(rs.getString("email"));
	return student;
  }
}
```

<!-- Slide number: 8 -->
- It is recommended to use `?` for arguments to avoid SQL injections,
- Example

```
jdbcTemplate.update("INSERT INTO customers(first_name, last_name) VALUES (?,?)", "John", "West");
```
