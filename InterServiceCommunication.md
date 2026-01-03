1️⃣ RestTemplate (Legacy – Blocking)
✅ Maven Dependencies
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>


spring-web contains RestTemplate

🔹 Bean Creation
@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder) {
        return builder.build();
    }
}

🔹 Injection
@Service
public class UserService {

    private final RestTemplate restTemplate;

    public UserService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
}

🎯 Interview Highlight

Blocking

Deprecated in Spring 6

Still widely used in legacy systems

2️⃣ RestClient (Spring 6.1+ – Blocking, Modern)
✅ Maven Dependencies
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>


RestClient is part of spring-web (6.1+)

🔹 Bean Creation
@Configuration
public class RestClientConfig {

    @Bean
    RestClient restClient(RestClient.Builder builder) {
        return builder
                .baseUrl("http://user-service")
                .build();
    }
}

🔹 Injection
@Service
public class UserService {

    private final RestClient restClient;

    public UserService(RestClient restClient) {
        this.restClient = restClient;
    }
}

🎯 Interview Highlight

“RestClient is the recommended blocking HTTP client in Spring 6+.”

3️⃣ WebClient (Reactive – Non-Blocking)
✅ Maven Dependencies
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>

🔹 Bean Creation
@Configuration
public class WebClientConfig {

    @Bean
    WebClient webClient(WebClient.Builder builder) {
        return builder
                .baseUrl("http://user-service")
                .build();
    }
}

🔹 Injection
@Service
public class UserService {

    private final WebClient webClient;

    public UserService(WebClient webClient) {
        this.webClient = webClient;
    }
}

🎯 Interview Highlight

Non-blocking

Uses Netty event-loop

Returns Mono / Flux

4️⃣ OpenFeign (Declarative – Spring Cloud)
✅ Maven Dependencies
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

(Also add Spring Cloud BOM)
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2023.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

🔹 Enable Feign
@EnableFeignClients
@SpringBootApplication
public class Application {}

🔹 Declare Client
@FeignClient(name = "user-service")
public interface UserFeignClient {

    @GetMapping("/users/{id}")
    User getUser(@PathVariable Long id);
}

🔹 Injection
@Service
public class OrderService {

    private final UserFeignClient userFeignClient;

    public OrderService(UserFeignClient userFeignClient) {
        this.userFeignClient = userFeignClient;
    }
}

🎯 Interview Highlight

“Feign clients are created as dynamic proxy beans by Spring Cloud at startup.”

5️⃣ HttpInterfaces (Spring 6+ – Declarative & Lightweight)
✅ Maven Dependencies
Blocking
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

Reactive (optional)
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>

🔹 Define Interface
@HttpExchange("/users")
public interface UserHttpClient {

    @GetExchange("/{id}")
    User getUser(@PathVariable Long id);
}

🔹 Create Proxy Bean (Blocking)
@Configuration
public class HttpClientConfig {

    @Bean
    UserHttpClient userHttpClient(RestClient restClient) {

        HttpServiceProxyFactory factory =
                HttpServiceProxyFactory
                        .builderFor(RestClientAdapter.create(restClient))
                        .build();

        return factory.createClient(UserHttpClient.class);
    }
}

🔹 Injection
@Service
public class UserService {

    private final UserHttpClient userHttpClient;

    public UserService(UserHttpClient userHttpClient) {
        this.userHttpClient = userHttpClient;
    }
}
