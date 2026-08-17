<!-- Slide number: 1 -->
# WebSecurityConfig – muutokset

**Spring Security 6 (newer version) (with lambdas)**

```java
@Configuration
@EnableMethodSecurity(securedEnabled = true)
public class WebSecurityConfig {
	@Bean
	public SecurityFilterChain configure(HttpSecurity http) throws Exception {
		http
			.authorizeHttpRequests( authorize -> authorize
				.requestMatchers("/home").permitAll()
				.anyRequest().authenticated()
			)
			.formLogin(formlogin -> formlogin
				.loginPage("/login")
				.defaultSuccessUrl("/studentlist”, true)
				.permitAll()
			)
			.logout(logout -> logout
				.permitAll()
			);
		return http.build();
	}
}
```

**Old Spring Security 5 version (on videos)**

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.authorizeRequests()
				.antMatchers("/home").permitAll()
				.anyRequest().authenticated()
				.and()
			.formLogin()
				.loginPage("/login")
				.defaultSuccessUrl("/studentlist”, true)
				.permitAll()
				.and()
			.logout()
				.permitAll();
	}
}
```

<!-- Slide number: 2 -->
# Spring Security
- Create in-memory users
	- This is only for testing and demo purposes (Security configuration class)
	- You can add multiple user using `Collection<UserDetails>`

```java
@Bean
// @Override 	REMOVED
public UserDetailsService userDetailsService() {
	UserDetails user = User.withDefaultPasswordEncoder()
		.username("user")
		.password("password")
		.roles("USER")
		.build();
	List<UserDetails> users = new ArrayList<UserDetails>();
	users.add(user);
	return new InMemoryUserDetailsManager(users);
}
```
