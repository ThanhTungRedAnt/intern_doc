# Ex06: Spring boot - Spring security JWT
Create by: TungBT - Bui Thanh Tung <<04/05/2021>>

![N|Solid](https://stackjava.com/wp-content/uploads/2018/03/SPRING-BOOT-SECURITY-LOGO.jpg)

# Mục tiêu bài học
- Biết cách dùng Spring security

# Giới thiệu
Như các bài trước chúng ta muốn lấy thông tin chúng ta chỉ cần gọi API là có, và ai cũng có thể gọi và sử dùng data.
Nhưng trong thực tế thì chúng ta muốn có thông tin thì phải có quyền được xem và phía API cần phải đảm bảo việc đó.

Chính vì vậy khi chúng ta thường thấy là phải đăng nhập thì mới sử dụng được

Trong bài hôm nay chúng ta sẽ tìm hiểu một phần cực kỳ quan trọng trong các hệ thống bảo mật ngày nay, đó là JWT.
JWT (Json web Token) là một chuỗi mã hóa được gửi kèm trong Header của client request có tác dụng giúp phía server xác thực request người dùng có hợp lệ hay không. Được sử dụng phổ biến trong các hệ thống API ngày nay.

![N|Solid](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/alkylumvji_3.png)

# Thực hành

Chúng ta tiếp tục phát triển trên project cũ từ các bài trước

## 1. Update pom.xml, thêm thư viện JWT, Spring security

```xml

<!--Begin Security-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>problem-spring-web</artifactId>
    <version>0.25.0</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.1</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.1</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.1</version>
</dependency>
<!--End Security-->

```

## 2. Tạo thêm bảng user trong database để lưu thông tin người dùng.

![N|CSDL](./image/db_img_tbl.PNG)

Đây là script tạo bảng và account mẫu

```sql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for user_tbl
-- ----------------------------
DROP TABLE IF EXISTS `user_tbl`;
CREATE TABLE `user_tbl`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `password_hash` varchar(60) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `role` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `first_name` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `last_name` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `email` varchar(191) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE INDEX `ux_user_login`(`username`) USING BTREE,
  UNIQUE INDEX `ux_user_email`(`email`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 11 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- Records of user_tbl
-- ----------------------------
INSERT INTO `user_tbl` VALUES (1, 'admin', '$2a$10$gSAhZrxMllrbgj/kkK9UceBPpChGWJA7SYIb1Mqo.n5aNLq1/oRrC', 'ROLE_ADMIN', 'Administrator', 'Administrator', 'admin@localhost');


SET FOREIGN_KEY_CHECKS = 1;

```

Trong database gồm các trường dữ liệu
id: id của user
username: username đăng nhập
password_hash: password đã đươc mã hóa của user (Chúng ta không được phép lưu password thật )

## 3. Tạo Entity, BO cho bảng user_tbl vừa tạo
Như chúng ta đã biết thì để thao tác với data chúng ta dùng JPA thì cần Entity và Repository

BO - Entity

```java

package com.intern.entity;

import javax.persistence.*;

/**
 * A user.
 */
@Entity
@Table(name = "user_tbl")
public class UserBO {

    @Id
    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "username")
    private String username;

    @Column(name = "password_hash")
    private String passwordHash;

    @Column(name = "role")
    private String role;

    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

    @Column(name = "email")
    private String email;

    public UserBO() {
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPasswordHash() {
        return passwordHash;
    }

    public void setPasswordHash(String passwordHash) {
        this.passwordHash = passwordHash;
    }

    public String getRole() {
        return role;
    }

    public void setRole(String role) {
        this.role = role;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}

```
  
Repository

```java

package com.intern.repository;

import com.intern.entity.UserBO;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface UserRepository extends JpaRepository<UserBO, Long> {
    @Query("SELECT u FROM UserBO u WHERE u.username = :username")
    Optional<UserBO> getUserByUsername(@Param("username") String username);
}

```


Các bước chuẩn bị vừa rồi chúng ta đã đều quen thuộc trong 2 bài trước rồi.
Bây giờ chúng ta bắt đầu đến phần Security

Chúng ta tạo thêm package security chuyên để xử lý về security 

## 1. Tạo DomainUserDetailsService

Khi người dùng đăng nhập thì Spring Security sẽ cần lấy các thông tin hiện có để kiểm tra. Vì vậy, bạn cần tạo ra một class kế thừa lớp UserDetailsService mà Spring Security cung cấp để làm nhiệm vụ này.
Chúng ta sẽ tạo DomainUserDetailsService kết thừa UserDetailsService để lấy thông tin của user theo username khi đăng nhập

```java
package com.intern.security;

import com.intern.entity.UserBO;
import com.intern.repository.UserRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import java.util.Arrays;
import java.util.Locale;

/**
 * Authenticate a user from the database.
 */
@Component("userDetailsService")
public class DomainUserDetailsService implements UserDetailsService {

    private final Logger log = LoggerFactory.getLogger(DomainUserDetailsService.class);

    private final UserRepository userRepository;

    public DomainUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    @Transactional
    public UserDetails loadUserByUsername(final String login) {
        log.debug("Authenticating {}", login);

        String lowercaseLogin = login.toLowerCase(Locale.ENGLISH);
        return userRepository.getUserByUsername(lowercaseLogin)
                .map(user -> createSpringSecurityUser(lowercaseLogin, user))
                .orElseThrow(() -> new UsernameNotFoundException("User " + lowercaseLogin + " was not found in the database"));

    }

    private User createSpringSecurityUser(String lowercaseLogin, UserBO user) {
        String role = user.getRole();
        GrantedAuthority authority = new SimpleGrantedAuthority(role);
        return new org.springframework.security.core.userdetails.User(user.getUsername(),
                user.getPasswordHash(),
                Arrays.asList(authority));
    }
}

```

- Đầu tiên chúng ta @Override lại method loadUserByUsername
Các bước xử lý như sau:

- Dùng Repository để tìm kiếm user theo username
 + Nếu tìm thấy user ==> Tạo ra Obj User mặc định của spring với 3 thông tin: username, password, role
 + Nếu không tìm thấy ===> Throw ra UsernameNotFoundException


## JWT

Đầu tiên ta tìm hiểu về cấu trúc của chuỗi JWT

JWT là một phương tiện đại diện cho các yêu cầu chuyển giao giữa hai bên Client – Server , các thông tin trong chuỗi JWT được định dạng bằng JSON . Trong đó chuỗi Token phải có 3 phần là header , phần payload và phần signature được ngăn bằng dấu “.”

![N|CSDL](https://topdev.vn/blog/wp-content/uploads/2017/12/jwt-la-gi.jpeg)

Sau khi có các thông tin về người dùng, chúng ta cần mã hóa thông tin người dùng thành chuỗi JWT và giải mã, verify mã JWT

```java
package com.intern.security;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtException;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.security.Keys;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import java.nio.charset.StandardCharsets;
import java.security.Key;
import java.util.Arrays;
import java.util.Collection;
import java.util.Date;
import java.util.stream.Collectors;

@Component
public class TokenProvider {

    private final Logger log = LoggerFactory.getLogger(TokenProvider.class);
    private static final String AUTHORITIES_KEY = "auth";

    private Key key;
    private long tokenValidityInMilliseconds;
    private long tokenValidityInMillisecondsForRememberMe;

    public TokenProvider() {
    }

    @PostConstruct
    public void init() {
        // Khai báo chuỗi bí mật key dùng để mã hóa và giải mã JWT
        String secret = "NmZhZmQwOGJjZDBiNjNmZDBjYzRjNGIxNmNlZWRhMDdkNTI0YTdiYWFmNDQ5NjVlMmM2OGZhYjlmOWJmNjA1YjRhMGVlZmRkOGZmMjM5Mjg5NmVhZDEwN2IwN2I4Y2ZlYzUzNDg4NTdkZGNhZjg1ZGVhNWQxZjUyZTQyNWY1YTk=";
        byte[] keyBytes = secret.getBytes(StandardCharsets.UTF_8);
        this.key = Keys.hmacShaKeyFor(keyBytes);

        // Khai báo lượng thời gian có hiệu lực của JWT
        this.tokenValidityInMilliseconds = 1000 * 60 * 60 * 24; // 24H = 1000 * 60 * 60 * 24 ms
    }

    /*Hàm tạo Token*/
    public String createToken(Authentication authentication) {
        String authorities = authentication.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority)
                .collect(Collectors.joining(","));

        long now = (new Date()).getTime();

        // Thời gian hết hạn của JWT = Hiện tại + Lượng thời gian hiệu lực
        Date validity = new Date(now + this.tokenValidityInMilliseconds);

        // Dùng Jwts để tạo ra chuỗi JWT
        return Jwts.builder()
                .setSubject(authentication.getName())
                .claim(AUTHORITIES_KEY, authorities)
                .signWith(key, SignatureAlgorithm.HS512)
                .setExpiration(validity)
                .compact();
    }

    // Giải mã JWT và Tạo Authentication: Dùng khi 1 request gửi lên chúng ta verify JWT
    public Authentication getAuthentication(String token) {
        Claims claims = Jwts.parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(token)
                .getBody();

        Collection<? extends GrantedAuthority> authorities =
                Arrays.stream(claims.get(AUTHORITIES_KEY).toString().split(","))
                        .map(SimpleGrantedAuthority::new)
                        .collect(Collectors.toList());

        User principal = new User(claims.getSubject(), "", authorities);

        return new UsernamePasswordAuthenticationToken(principal, token, authorities);
    }

    public boolean validateToken(String authToken) {
        try {
            Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(authToken);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            log.info("Invalid JWT token.");
            log.trace("Invalid JWT token trace.", e);
        }
        return false;
    }
}


```

## Tạo JWTFilter
Mục đích của filter sẽ lại khi 1 request bất kỳ gửi lên server sẽ phải đi qua Filter này

B1: Lấy thông tin JWT người dùng gửi lên qua 
B2: Verify JWT đó, nếu đúng cho đi tiếp, nếu sai trả về lỗi 401

```java

package com.intern.security.jwt;

import com.intern.security.TokenProvider;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.util.StringUtils;
import org.springframework.web.filter.GenericFilterBean;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * Filters incoming requests and installs a Spring Security principal if a header corresponding to a valid user is
 * found.
 */
public class JWTFilter extends GenericFilterBean {

    public static final String AUTHORIZATION_HEADER = "Authorization";

    private final TokenProvider tokenProvider;

    public JWTFilter(TokenProvider tokenProvider) {
        this.tokenProvider = tokenProvider;
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException {
        // Lấy ra request
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;

        // Lấy ra chuỗi JWT trong header của request
        String jwt = resolveToken(httpServletRequest);

        // Validate JWT Token
        if (StringUtils.hasText(jwt) && this.tokenProvider.validateToken(jwt)) {
            // Tạo Authentication từ chuỗi JWT
            Authentication authentication = this.tokenProvider.getAuthentication(jwt);

            // Set Authentication vào SecurityContextHolder: Thông báo với Spring Security là Authen thành công
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }

        // Tiếp tục thực hiện request
        filterChain.doFilter(servletRequest, servletResponse);
    }

    private String resolveToken(HttpServletRequest request) {
        String bearerToken = request.getHeader(AUTHORIZATION_HEADER);
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}

```


Tạo JWTConfigurer dùng để đăng ký JWTFilter vừa tạo bên trên


```java

package com.intern.security.jwt;

import com.intern.security.TokenProvider;
import org.springframework.security.config.annotation.SecurityConfigurerAdapter;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.DefaultSecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

public class JWTConfigurer extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity> {

    private final TokenProvider tokenProvider;

    public JWTConfigurer(TokenProvider tokenProvider) {
        this.tokenProvider = tokenProvider;
    }

    @Override
    public void configure(HttpSecurity http) {
        // Khởi tạo JWTFilter 
        JWTFilter customFilter = new JWTFilter(tokenProvider);
        
        // Thêm filter của mình vào HttpSecurity
        http.addFilterBefore(customFilter, UsernamePasswordAuthenticationFilter.class);
    }
}

```


## Cấu hình và phân quyền
Bây giờ, chúng ta bắt đầu cấu hình Spring Security bao gồm việc kích hoạt bằng @EnableWebSecurity.


```java

package com.intern.security;

import com.intern.security.jwt.JWTConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Import;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import org.springframework.security.web.header.writers.ReferrerPolicyHeaderWriter;
import org.springframework.web.filter.CorsFilter;
import org.zalando.problem.spring.web.advice.security.SecurityProblemSupport;

@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
@Import(SecurityProblemSupport.class)
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    private final TokenProvider tokenProvider;
    private final CorsFilter corsFilter;
    private final SecurityProblemSupport problemSupport;

    public SecurityConfiguration(TokenProvider tokenProvider, CorsFilter corsFilter, SecurityProblemSupport problemSupport) {
        this.tokenProvider = tokenProvider;
        this.corsFilter = corsFilter;
        this.problemSupport = problemSupport;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        // Khai báo thuật toán mã hóa cho password
        // Như trong data mẫu password đang là admin sau khi mã khóa bằng BCryptPasswordEncoder sẽ ra
        // $2a$10$gSAhZrxMllrbgj/kkK9UceBPpChGWJA7SYIb1Mqo.n5aNLq1/oRrC
        return new BCryptPasswordEncoder();
    }

    @Override
    public void configure(WebSecurity web) {
        web.ignoring()
            .antMatchers(HttpMethod.OPTIONS, "/**")
            .antMatchers("/app/**/*.{js,html}")
            .antMatchers("/i18n/**")
            .antMatchers("/content/**")
            .antMatchers("/swagger-ui/index.html")
            .antMatchers("/test/**");
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
        // @formatter:off
        http
            .csrf()
            .disable()
            .addFilterBefore(corsFilter, UsernamePasswordAuthenticationFilter.class)
            .exceptionHandling()
                .authenticationEntryPoint(problemSupport)
                .accessDeniedHandler(problemSupport)
        .and()
            .headers()
            .contentSecurityPolicy("default-src 'self'; frame-src 'self' data:; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://storage.googleapis.com; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self' data:")
        .and()
            .referrerPolicy(ReferrerPolicyHeaderWriter.ReferrerPolicy.STRICT_ORIGIN_WHEN_CROSS_ORIGIN)
        .and()
            .featurePolicy("geolocation 'none'; midi 'none'; sync-xhr 'none'; microphone 'none'; camera 'none'; magnetometer 'none'; gyroscope 'none'; speaker 'none'; fullscreen 'self'; payment 'none'")
        .and()
            .frameOptions()
            .deny()
        .and()
            .sessionManagement()
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
        .and()
            .authorizeRequests()
            .antMatchers("/api/v1/login").permitAll() // Cấu hình cho API với URL /api/v1/login là permitAll tức không cần phải Authen
            .antMatchers("/api/**").authenticated() // Còn cái API còn lại bắt đầu /api/** authenticated tức cần Authen
        .and()
            .httpBasic()
        .and()
            .apply(securityConfigurerAdapter());
        // @formatter:on
    }

    private JWTConfigurer securityConfigurerAdapter() {
        // Đăng ký HttpSecurity Filter 
        return new JWTConfigurer(tokenProvider);
    }
}

```

Việc cấu hình đã xong, tiếp đến chúng ta tạo ra 1 API Login

1 .Đầu tiên chúng ta tạo LoginDTO để người dùng truyền lên

```java

package com.intern.dto;

/**
 * View Model object for storing a user's credentials.
 */
public class LoginDTO {

    private String username;
    private String password;

    public LoginDTO() {
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}

```


Tạo 1 API login như sau


```java
package com.intern.rest;

import com.intern.dto.LoginDTO;
import com.intern.security.TokenProvider;
import com.intern.security.jwt.JWTFilter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.Valid;

@RestController
@RequestMapping("/api/v1")
public class LoginRest {
    @Autowired
    private TokenProvider tokenProvider;
    @Autowired
    private AuthenticationManagerBuilder authenticationManagerBuilder;

    @PostMapping("/login")
    public ResponseEntity<JWTToken> authorize(@Valid @RequestBody LoginDTO loginDTO) {

        try {
            // Tạo input đầu vào cho Spring security
            UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(loginDTO.getUsername(), loginDTO.getPassword());
            
            // Thực hiện Authen bằng authenticationManagerBuilder của Spring
            Authentication authentication = authenticationManagerBuilder.getObject().authenticate(authenticationToken);
            
            // Thông tin đăng nhập chính xác Spring trả về Authentication và mình set vào SecurityContextHolder để xác nhận 
            SecurityContextHolder.getContext().setAuthentication(authentication);
            
            // Tạo chuỗi JWT bằng tokenProvider
            String jwt = tokenProvider.createToken(authentication);
            HttpHeaders httpHeaders = new HttpHeaders();
            httpHeaders.add(JWTFilter.AUTHORIZATION_HEADER, "Bearer " + jwt);

            // Trả về kết quả
            return new ResponseEntity<JWTToken>(new JWTToken(jwt), httpHeaders, HttpStatus.OK);
        } catch (Exception ex) {
            // Khi thông tin đăng nhập sai Spring Security sẽ throw ra Exception và chúng ta catch Ex đó để thông báo lỗi đăng nhập sai
            return new ResponseEntity<>(new JWTToken(), HttpStatus.UNAUTHORIZED);
        }
    }

    static class JWTToken {

        private String idToken;
        private String role;

        public JWTToken() {
        }

        public JWTToken(String idToken) {
            this.idToken = idToken;
        }

        public String getIdToken() {
            return idToken;
        }

        public void setIdToken(String idToken) {
            this.idToken = idToken;
        }

        public String getRole() {
            return role;
        }

        public void setRole(String role) {
            this.role = role;
        }
    }
}

```


Chúng ta tạo thêm 1 class ExceptionTranslator với mục đích dùng để cập nhật lại HTTP Status Code và Headers khi mà 401 Unthor


```java

package com.intern.security;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.context.request.NativeWebRequest;
import org.zalando.problem.Problem;
import org.zalando.problem.spring.web.advice.ProblemHandling;
import org.zalando.problem.spring.web.advice.security.SecurityAdviceTrait;

import javax.annotation.Nullable;

/**
 * Controller advice to translate the server side exceptions to client-friendly json structures.
 * The error response follows RFC7807 - Problem Details for HTTP APIs (https://tools.ietf.org/html/rfc7807).
 */
@ControllerAdvice
public class ExceptionTranslator implements ProblemHandling, SecurityAdviceTrait {

    public ExceptionTranslator() {
    }

    /**
     * Post-process the Problem payload to add the message key for the front-end if needed.
     */
    @Override
    public ResponseEntity<Problem> process(@Nullable ResponseEntity<Problem> entity, NativeWebRequest request) {
        if (entity == null) {
            return entity;
        }
        return new ResponseEntity<>(entity.getHeaders(), entity.getStatusCode());
    }
}

```


Kết quả:
Như trong ví dụ trước chúng ta có API http://localhost:8080/api/v1/employee/employees

Và bây giờ chúng ta thực hiện gọi lại và nhận được lỗi 401

![N|CSDL](./image/401.PNG)

===> Tức là phần Security chúng ta đã chặn được request lại ko Authen


Gọi API Login

![N|CSDL](./image/login.PNG)


Kết quả


```json

{
    "idToken": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJhZG1pbiIsImF1dGgiOiJST0xFX0FETUlOIiwiZXhwIjoxNjIwMjExMTg0fQ.C1KEgGyVKrxRgsRqKgBVyWihVTVouSx6dwqGcnlXgph0rgcwJh5g235tSK0NPZmIaYtSdyeYZBBqgcK7fG9OdA",
    "role": null
}
```


Dùng idToken vừa nhận được cho request http://localhost:8080/api/v1/employee/employees trước để thực hiện gọi thanh công
Chúng ta set Author trong postman như sau

![N|CSDL](./image/Postman_Bear.PNG)



Bài này khá là phức tạp mọi người cố gắng đọc, tìm hiểu kỹ nhé!!!!!


Thannks,
TungBT.
