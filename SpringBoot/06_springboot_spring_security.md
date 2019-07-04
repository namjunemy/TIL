# 6. 스프링 시큐리티

* 웹 시큐리티
* 메소드 시큐리티
* 다양한 인증 방법 지원
  * LDAP, 폼 인증, Basic 인증, OAuth, ...

## 6-1. 스프링 부트 시큐리티 자동 설정

* spring-boot-starter-security

* SecurityAutoConfiguration.class 설정파일 참조

  * 스프링 시큐리티가 의존성으로 등록되어 있으면,

  * **DefaultAuthenticationEventPublisher**가 @Bean으로 등록된다.

    ```java
    @Configuration
    @ConditionalOnClass(DefaultAuthenticationEventPublisher.class)
    @EnableConfigurationProperties(SecurityProperties.class)
    @Import({SpringBootWebSecurityConfiguration.class, WebSecurityEnablerConfiguration.class,
        SecurityDataConfiguration.class})
    public class SecurityAutoConfiguration {
    
        @Bean
        @ConditionalOnMissingBean(AuthenticationEventPublisher.class)
        public DefaultAuthenticationEventPublisher authenticationEventPublisher(
            ApplicationEventPublisher publisher) {
            return new DefaultAuthenticationEventPublisher(publisher);
        }
    
    }
    ```

  * 이 이벤트 퍼블리셔는

    * BadCredentialsException
    * UsernameNotFoundException
    * AccountExpiredException
    * 등의 

  * 이벤트를 매핑해서 발생시켜 준다. => **인증 관련 각종 이벤트 발생**

    * DefaultAuthenticationEventPublisher.java

    * 그러면 우리는 이벤트 에러 핸들러를 등록해서 유저의 상태를 변경하는 등의 특정 동작을 수행할 수 있다.

      ```java
      ...
        
      public DefaultAuthenticationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
      
        addMapping(BadCredentialsException.class.getName(),
                   AuthenticationFailureBadCredentialsEvent.class);
        addMapping(UsernameNotFoundException.class.getName(),
                   AuthenticationFailureBadCredentialsEvent.class);
        addMapping(AccountExpiredException.class.getName(),
                   AuthenticationFailureExpiredEvent.class);
        addMapping(ProviderNotFoundException.class.getName(),
                   AuthenticationFailureProviderNotFoundEvent.class);
        addMapping(DisabledException.class.getName(),
                   AuthenticationFailureDisabledEvent.class);
        addMapping(LockedException.class.getName(),
                   AuthenticationFailureLockedEvent.class);
        addMapping(AuthenticationServiceException.class.getName(),
                   AuthenticationFailureServiceExceptionEvent.class);
        addMapping(CredentialsExpiredException.class.getName(),
                   AuthenticationFailureCredentialsExpiredEvent.class);
        addMapping(
          "org.springframework.security.authentication.cas.ProxyUntrustedException",
          AuthenticationFailureProxyUntrustedEvent.class);
      }
      
      ...
      ```

* **스프링 부트의 시큐리티 설정파일**인 SpringBootWebSecurityConfiguration.class를 보면,

  * WebSecurityConfigurerAdapter를 상속받아서 그대로 사용하고 있다.

    ```java
    /**
     * The default configuration for web security. It relies on Spring Security's
     * content-negotiation strategy to determine what sort of authentication to use. If the
     * user specifies their own {@link WebSecurityConfigurerAdapter}, this will back-off
     * completely and the users should specify all the bits that they want to configure as
     * part of the custom security configuration.
     *
     * @author Madhura Bhave
     * @since 2.0.0
     */
    @Configuration
    @ConditionalOnClass(WebSecurityConfigurerAdapter.class)
    @ConditionalOnMissingBean(WebSecurityConfigurerAdapter.class)
    @ConditionalOnWebApplication(type = Type.SERVLET)
    public class SpringBootWebSecurityConfiguration {
    
        @Configuration
        @Order(SecurityProperties.BASIC_AUTH_ORDER)
        static class DefaultConfigurerAdapter extends WebSecurityConfigurerAdapter {
    
        }
    
    }
    ```

  * 스프링 시큐리티가 제공하는 **기본설정을 그대로 쓴다**는 의미가 된다.

    * @ConditionalOnMissingBean(WebSecurityConfigurerAdapter.class)가 의미하는 것은
    * **만약, WebSecurityConfigurerAdapter.class 를 상속받은 빈을 만들면**
    * **커스텀하게 우리의 시큐리티 설정을 가져갈 수 있다는 이야기가 된다.** 

  * WebSecurityConfigurerAdapter.class 안에서 특히 configure() 메서드를 보면, 

    * 기본적으로 사용자 인증이 된 요청에 대해서만 요청을 허용하고.
    * 로그인 폼과, HttpBasic을 사용하도록 되어 있는것을 볼 수 있다.
      * formLogin - 사용자가 폼기반 로그인으로 인증 할 수 있다.
      * httpBasic - 사용자가 Http기반 인증으로 인증 할 수 있다.

  * **우리가 부트에서 시큐리티 스타터를 등록만 했는데 모든 인증 없는 컨트롤러 테스트가 깨지고, 웹에선 로그인 페이지로 리다이렉트 되는 이유가 여기에 있다.**

    ```java
    ...
    protected void configure(HttpSecurity http) throws Exception {
      logger.debug("Using default configure(HttpSecurity). If subclassed this will potentially override subclass configure(HttpSecurity).");
      
      http
        .authorizeRequests()
        .anyRequest().authenticated()
        .and()
        .formLogin().and()
        .httpBasic();
    }
    ...
    ```

* 부트에서 시큐리티 스타터 의존성을 추가하고, application을 run하면 아래와 같은 로그가 찍힌다.

  ```shell
  2019-06-28 14:45:37.001  INFO 18612 --- [  main] .s.s.UserDetailsServiceAutoConfiguration : 
  
  Using generated security password: 960d55de-dd62-415c-b7ff-8f5083e78498
  ```

  * spring-boot-autoconfigure의 security쪽 자동 설정중 하나인 UserDetailsServiceAutoConfiguration의 역할이다.

  * 코드를 보면

    * @ConditionalOnMissingBean({ AuthenticationManager.class, AuthenticationProvider.class,
      		UserDetailsService.class }) 에 의해서 3개의 클래스가 없을때 아래의 설정 파일이 적용되고,
    * inMemoryUserDetailsManager() 에 의해서 인메모리로 UserDetailsManager를 만들어서, 랜덤하게 유저가 만들어지게 된다.
    * 만약, UserDetailsService.class를 상속받아서 재정의하거나 하면 랜덤 유저는 생성되지 않는다.

    ```java
    @Configuration
    @ConditionalOnClass(AuthenticationManager.class)
    @ConditionalOnBean(ObjectPostProcessor.class)
    @ConditionalOnMissingBean({AuthenticationManager.class, AuthenticationProvider.class,
        UserDetailsService.class})
    public class UserDetailsServiceAutoConfiguration {
    
        private static final String NOOP_PASSWORD_PREFIX = "{noop}";
    
        private static final Pattern PASSWORD_ALGORITHM_PATTERN = Pattern
            .compile("^\\{.+}.*$");
    
        private static final Log logger = LogFactory
            .getLog(UserDetailsServiceAutoConfiguration.class);
    
        @Bean
        @ConditionalOnMissingBean(type = "org.springframework.security.oauth2.client.registration.ClientRegistrationRepository")
        @Lazy
        public InMemoryUserDetailsManager inMemoryUserDetailsManager(
            SecurityProperties properties,
            ObjectProvider<PasswordEncoder> passwordEncoder) {
            SecurityProperties.User user = properties.getUser();
            List<String> roles = user.getRoles();
            return new InMemoryUserDetailsManager(User.withUsername(user.getName())
                .password(getOrDeducePassword(user, passwordEncoder.getIfAvailable()))
                .roles(StringUtils.toStringArray(roles)).build());
        }
    
        private String getOrDeducePassword(SecurityProperties.User user,
            PasswordEncoder encoder) {
            String password = user.getPassword();
            if (user.isPasswordGenerated()) {
                logger.info(String.format("%n%nUsing generated security password: %s%n",
                    user.getPassword()));
            }
            if (encoder != null || PASSWORD_ALGORITHM_PATTERN.matcher(password).matches()) {
                return password;
            }
            return NOOP_PASSWORD_PREFIX + password;
        }
    
    }
    ```

* [commit log](https://github.com/namjunemy/spring-boot-concept-and-utilization/commit/83ab8f4ff7095a94390ec3684edafae93dabea5f)

## 6.2 시큐리티 설정 커스터마이징

* 1 - 웹 시큐리티 설정

  * 6.1에서 설명했던것 처럼 SpringBootWebSecurityConfiguration는 WebSecurityConfigurerAdapter가 빈으로 등록되어 있지 않을 경우에 등록된다. 
  * 따라서, 아래와 같이 WebSecurityConfigurerAdapter를 상속받은 커스텀 설정을 빈으로 등록하면 스프링 부트의 기본 시큐리티 설정은 사용하지 않게 된다.
    * "/", "/hello"는 모든 사용자에게 허용하고, 
    * 그 외의 요청은 인증이 필요하도록 설정하는 코드이다.

  ```java
  @Configuration
  public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
  
      @Override
      protected void configure(HttpSecurity http) throws Exception {
          http.authorizeRequests()
              .antMatchers("/", "/hello").permitAll()
              .anyRequest().authenticated()
              .and()
              .formLogin()
              .and()
              .httpBasic();
      }
  }
  ```

* 2 - UserDetailsService 구현

  * https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#jc-authentication-userdetailsservice

  * 스프링 시큐리티가 랜덤생성해주는 유저 말고, 우리가 원하는 유저로 인증

  * 보통 Account를 관리하는 서비스 계층에서 UserDetailsService를 implements 한다.

  * UserDetailsService 타입의 빈이 등록이 되어있어야지 부트가 기본으로 생성해주는 유저가 더 이상 등록되지 않는다.

  * password 검증을 위해서 passwordEncoder가 꼭 필요하다.

    * 스프링 시큐리티에서 권장하는 인코더를 시큐리티 설정에 빈으로 등록해주면 된다.

    * https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#core-services-password-encodin

      ```java
      @Configuration
      public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
      
          @Override
          protected void configure(HttpSecurity http) throws Exception {
              http.authorizeRequests()
                  .antMatchers("/", "/hello").permitAll()
                  .anyRequest().authenticated()
                  .and()
                  .formLogin()
                  .and()
                  .httpBasic();
          }
      
          @Bean
          public PasswordEncoder passwordEncoder() {
              return PasswordEncoderFactories.createDelegatingPasswordEncoder();
          }
      }
      
      ```

  * AccountService.class

    ```java
    @Service
    @RequiredArgsConstructor
    public class AccountService implements UserDetailsService {
    
        private final AccountRepository accountRepository;
        private final PasswordEncoder passwordEncoder;
    
        public Account createAccount(String username, String password) {
            Account account = new Account();
            account.setUsername(username);
            account.setPassword(passwordEncoder.encode(password));
            return accountRepository.save(account);
        }
    
        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            Account account = accountRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException(username));
    
            return new User(account.getUsername(), account.getPassword(), authorities());
        }
    
        private Collection<? extends GrantedAuthority> authorities() {
            return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
        }
    }
    ```

* [commit log](https://github.com/namjunemy/spring-boot-concept-and-utilization/commit/1330e36e5d2ed20523e5f5c17b80844855b8c610)

## Reference

* https://spring.io/projects/spring-security
* [백기선 - 스프링 부트 개념과 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/)