# Quarkus Security Identity Providers Usage Patterns

Use these patterns to choose and apply the right username/password identity store.

## Pattern: Keep auth data in your ORM model with JPA

When to use:

- The application already uses Hibernate ORM.
- User records belong to the same relational model as the rest of the app.
- You want entity mapping rather than SQL configuration.

Example:

```java
import jakarta.persistence.Entity;
import jakarta.persistence.ManyToMany;
import jakarta.persistence.Table;

import io.quarkus.security.jpa.Password;
import io.quarkus.security.jpa.Roles;
import io.quarkus.security.jpa.RolesValue;
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

    @ManyToMany
    @Roles
    public java.util.List<Role> roles;
}

@Entity
public class Role {
    @RolesValue
    public String name;
}
```

This is the most natural fit when authentication data is application-owned and the team already understands JPA.

## Pattern: Authenticate against an existing schema with JDBC

When to use:

- The schema already exists and you should not remodel it as entities.
- Authentication needs only one or two SQL queries.
- You want the smallest integration surface.

Configuration:

```properties
quarkus.security.jdbc.enabled=true
quarkus.security.jdbc.principal-query.sql=SELECT password, role FROM legacy_users WHERE login=?
quarkus.security.jdbc.principal-query.bcrypt-password-mapper.enabled=true
quarkus.security.jdbc.principal-query.bcrypt-password-mapper.password-index=1
quarkus.security.jdbc.principal-query.attribute-mappings.0.index=2
quarkus.security.jdbc.principal-query.attribute-mappings.0.to=groups
```

Prefer JDBC when the identity schema is stable, simple, and already managed outside the application model.

## Pattern: Delegate identity to LDAP

When to use:

- Users and groups are managed centrally in a directory.
- The application must not store passwords locally.
- Group membership should be resolved from LDAP rather than relational tables.

Configuration:

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

Prefer LDAP when identity lifecycle belongs to enterprise infrastructure, not the app database.

## Pattern: Normalize external roles to application roles

When to use:

- Store-specific roles differ from the role names used in code.
- You want stable `@RolesAllowed` values even if LDAP groups or DB role names change.

Configuration:

```properties
quarkus.http.auth.roles-mapping."standardRole"=user
quarkus.http.auth.roles-mapping."admins"=admin
```

This keeps authorization annotations stable and avoids leaking directory or schema naming into resource code.

## Pattern: Seed local users for JPA or JDBC development

When to use:

- You want a fast local loop with disposable user data.
- The provider is JPA or JDBC and local testing should not depend on shared infrastructure.

Approach:

1. Use `%dev` or `%test` datasource settings plus `import.sql` or startup fixtures.
2. Store hashed passwords, typically bcrypt.
3. Keep `%prod` pointed at the real user store.

This gives repeatable local auth tests without weakening production password handling.

## Pattern: Cache LDAP lookups carefully

When to use:

- LDAP latency is noticeable.
- The app does many repeated authentications for the same users.

Configuration:

```properties
quarkus.security.ldap.cache.enabled=true
quarkus.security.ldap.cache.max-age=60s
quarkus.security.ldap.cache.size=500
```

Start with a short max age. Increase it only after confirming the staleness tradeoff is acceptable.

## See Also

- [`../data-orm/README.md`](../data-orm/README.md) - ORM setup details behind JPA-backed identity entities
- [`../data-panache/README.md`](../data-panache/README.md) - Panache variants for JPA-backed user models
