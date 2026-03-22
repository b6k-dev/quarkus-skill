# Quarkus Security Identity Providers API Reference

Use this file for copy-ready JPA, JDBC, and LDAP identity-store examples.

## Extension entry points

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-security-jpa</artifactId>
</dependency>

<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-elytron-security-jdbc</artifactId>
</dependency>

<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-elytron-security-ldap</artifactId>
</dependency>
```

Pick one store first unless you have a clear need for multiple realms.

## JPA identity store

```java
import jakarta.persistence.Entity;
import jakarta.persistence.Table;

import io.quarkus.elytron.security.common.BcryptUtil;
import io.quarkus.security.jpa.Password;
import io.quarkus.security.jpa.Roles;
import io.quarkus.security.jpa.UserDefinition;
import io.quarkus.security.jpa.Username;

@Entity
@Table(name = "app_user")
@UserDefinition
public class User {
    @Username
    public String username;

    @Password
    public String password;

    @Roles
    public String roles;

    public static User create(String username, String rawPassword, String roles) {
        User user = new User();
        user.username = username;
        user.password = BcryptUtil.bcryptHash(rawPassword);
        user.roles = roles;
        return user;
    }
}
```

- `@UserDefinition` must appear on exactly one entity.
- `@Roles` can map a comma-separated `String` or a role collection.
- Default password handling expects bcrypt-style hashed values.

## JDBC identity store

```properties
quarkus.security.jdbc.enabled=true
quarkus.security.jdbc.principal-query.sql=SELECT password, role FROM app_user WHERE username=?
quarkus.security.jdbc.principal-query.bcrypt-password-mapper.enabled=true
quarkus.security.jdbc.principal-query.bcrypt-password-mapper.password-index=1
quarkus.security.jdbc.principal-query.attribute-mappings.0.index=2
quarkus.security.jdbc.principal-query.attribute-mappings.0.to=groups
```

The principal query must take exactly one parameter and return the password plus any role or attribute columns you want to map.

## LDAP identity store

```properties
quarkus.security.ldap.enabled=true
quarkus.security.ldap.dir-context.url=ldaps://ldap.example.com:636
quarkus.security.ldap.dir-context.principal=uid=admin,ou=system
quarkus.security.ldap.dir-context.password=secret
quarkus.security.ldap.identity-mapping.rdn-identifier=uid
quarkus.security.ldap.identity-mapping.search-base-dn=ou=Users,dc=example,dc=com
quarkus.security.ldap.identity-mapping.attribute-mappings."cn".from=cn
quarkus.security.ldap.identity-mapping.attribute-mappings."cn".filter=(member=uid={0},ou=Users,dc=example,dc=com)
quarkus.security.ldap.identity-mapping.attribute-mappings."cn".filter-base-dn=ou=Roles,dc=example,dc=com
```

LDAP needs a bind context plus identity mapping. `{0}` in the group filter is substituted with the resolved user identifier.

## Using loaded roles

```java
import jakarta.annotation.security.RolesAllowed;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.core.Context;
import jakarta.ws.rs.core.SecurityContext;

@Path("/me")
public class MeResource {
    @GET
    @RolesAllowed("user")
    public String me(@Context SecurityContext securityContext) {
        return securityContext.getUserPrincipal().getName();
    }
}
```

All three providers end up populating the authenticated principal and role set used by Quarkus authorization.
