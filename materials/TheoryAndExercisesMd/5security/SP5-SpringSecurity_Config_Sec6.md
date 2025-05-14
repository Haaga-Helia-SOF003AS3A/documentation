<!-- Slide number: 1 -->
# WebSecurityConfig – muutoksetUusi Spring Security 6 (with lambdas)	  Vanha Spring Security 5 (videoissa)

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
     .formLogin()                .loginPage("/login")
                .defaultSuccessUrl("/studentlist”, true)
                .permitAll()
                .and()
     .logout()
                .permitAll();
}}

@Configuration
@EnableMethodSecurity(securedEnabled = true)
public class WebSecurityConfig {
@Bean
public SecurityFilterChain configure(HttpSecurity http)                    throws Exception {
 http
       .authorizeHttpRequests( authorize -> authorize
             .requestMatchers("/home").permitAll()
              .anyRequest().authenticated()
        )
       .formLogin(formlogin -> formlogin
                .loginPage("/login")
                .defaultSuccessUrl("/studentlist”, true)
                .permitAll()        )
       .logout(logout -> logout
                .permitAll()
        );
       return http.build();
    }}

1
4.10.2023
Back End Programming

### Notes:

<!-- Slide number: 2 -->

![3](Picture1.jpg)
# Spring Security
Create in-memory users
This is only for testing and demo purposes (Security configuration class)
You can add multiple user using Collection<UserDetails>
@Bean
@Override //!
public UserDetailsService userDetailsService() {
  UserDetails user = 	User.withDefaultPasswordEncoder()           	.username("user")
	.password("password")
     	.roles("USER")
     	.build();
	List<UserDetails> users = new ArrayList<UserDetails>();
	users.add(user);
  return new InMemoryUserDetailsManager(users);
}
2
Server Programming
4.10.2023