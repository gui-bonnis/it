You can set up multiple Spring profiles with sub-profiles to achieve this flexibility by leveraging Spring Boot's profile inheritance and the concept of composite profiles. Here's how you can configure and use them effectively:

### 1. **Create Profile-Specific Property Files**

You can organize profiles in the following hierarchy:

- Base profiles for environments:
    
    - `application-DEV.yml`
    - `application-UAT.yml`
    - `application-PRD.yml`
- Sub-profiles for specific configurations:
    
    - `application-dev_db.yml`
    - `application-dev_api.yml`
    - `application-uat_db.yml`
    - `application-uat_api.yml`
    - `application-prd_db.yml`
    - `application-prd_api.yml`

### 2. **Define Configuration in Property Files**

#### Example for `application-DEV.yml`:

```yaml
spring:
  profiles:
    active: dev_db, dev_api
```

#### Example for `application-dev_db.yml`:

```yaml
database:
  url: jdbc:postgresql://localhost/dev
  username: dev_user
  password: dev_password
```

#### Example for `application-dev_api.yml`:

```yaml
api:
  base-url: http://localhost:8080/dev-api
```

### 3. **Set the Default Profile**

In `application.yml`, define the default profile to use when none is specified:

```yaml
spring:
  profiles:
    active: DEV
```

### 4. **Switch Profiles Dynamically**

You can override profiles at runtime using:

- **Command-line arguments**:
    
    ```bash
    java -Dspring.profiles.active=UAT,uat_api -jar app.jar
    ```
    
- **Environment variables**:
    
    ```bash
    export SPRING_PROFILES_ACTIVE=UAT,uat_db
    ```
    
- **Application arguments**:
    
    ```bash
    java -jar app.jar --spring.profiles.active=PRD,prd_api
    ```
    

### 5. **Ensure Profile Activation Order**

Profiles specified in `spring.profiles.active` are loaded in the order they are defined. For example:

```bash
java -Dspring.profiles.active=DEV,dev_api -jar app.jar
```

This ensures `DEV` is loaded first, followed by `dev_api`, allowing `dev_api` properties to override `DEV`.

### 6. **Testing Profiles**

To test profile behavior:

- Run your application with different profile combinations.
- Ensure property resolution is working correctly using logs or explicitly printing properties.

### 7. **Using Profiles in Code**

You can inject profile-specific properties using `@Value` or `@ConfigurationProperties`. To check active profiles programmatically:

```java
@Autowired
private Environment environment;

public void printActiveProfiles() {
    String[] activeProfiles = environment.getActiveProfiles();
    System.out.println("Active profiles: " + Arrays.toString(activeProfiles));
}
```

### 8. **Composite Profile Example**

If you want a single composite profile for certain combinations (e.g., `UAT_db_api`), you can create an additional profile:

#### `application-uat_db_api.yml`

```yaml
spring:
  profiles:
    include: uat_db, uat_api
```

Now, you can activate it with:

```bash
java -Dspring.profiles.active=uat_db_api -jar app.jar
```

This setup gives you maximum flexibility to switch between environment-level profiles (`DEV`, `UAT`, `PRD`) and fine-tune configurations with sub-profiles like `*_db` and `*_api`.