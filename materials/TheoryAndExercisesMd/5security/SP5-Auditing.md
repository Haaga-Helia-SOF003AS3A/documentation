<!-- Slide number: 1 -->
# Server ProgrammingAuditing
Juha Hinkula

### Notes:

<!-- Slide number: 2 -->
# Auditing
Database auditing means logging the persistent entities. In the business applications we are often interested in who created the record and when.
JPA auditing allows you to record who, what and when for each entity object.
To enble JPA auditing in existing Spring Boot project (with JPA + Spring Security) you need following steps

2
Server Programming
25.6.2018

<!-- Slide number: 3 -->
#
1.) Add following annotations to entity classes
@Entity
@EntityListeners(AuditingEntityListener.class)
public class Todo {
@Id
@GeneratedValue(strategy=GenerationType.AUT)
private long id;
private String description, status;

@CreatedDate
private Date createdDate;

@CreatedBy
private String createdBy;

@LastModifiedDate
private Date lastModifiedDate;

@LastModifiedBy
private String lastModifiedBy;

3
25.6.2018
Server Programming

<!-- Slide number: 4 -->
# Auditing
@CreatedBy, @CreatedDate, @ModifiedBy and @ModifiedDate are used to annotate entity attributes that records auditing data.
Specify AuditingEntityListener with @EntityListeners annotation

4
Server Programming
25.6.2018

<!-- Slide number: 5 -->
1.) Create auditing configuration class
@Configuration
@EnableJpaAuditing
class AuditConfig {
@Bean
public AuditorAware<String> createAuditorProvider() {
  return new SecurityAuditor();
}

@Bean
public AuditingEntityListener createAuditingListener() {
  return new AuditingEntityListener();
}

public static class SecurityAuditor implements AuditorAware<String> {
@Override
public String getCurrentAuditor() {
  Authentication auth = SecurityContextHolder.getContext().getAuthentication();
  String username = auth.getName();
  return username;
}}
}

5
25.6.2018
Server Programming