# Quarkus Security Identity Providers Configuration Reference

Use this file for the provider-specific properties that usually matter first.

## High-value properties by provider

| Provider | Property | Default | Use when |
|----------|----------|---------|----------|
| JPA | `quarkus.security-jpa.persistence-unit-name` | default persistence unit | The user entity lives in a non-default persistence unit |
| JDBC | `quarkus.security.jdbc.enabled` | `false` | The JDBC realm should be active |
| JDBC | `quarkus.security.jdbc.principal-query.sql` | none | You need the main authentication query |
| JDBC | `quarkus.security.jdbc.principal-query.datasource` | default datasource | User data comes from a named datasource |
| JDBC | `quarkus.security.jdbc.principal-query.bcrypt-password-mapper.enabled` | `false` | Passwords are stored as bcrypt hashes |
| JDBC | `quarkus.security.jdbc.principal-query.attribute-mappings.*` | none | SQL result columns should become Quarkus groups or attributes |
| LDAP | `quarkus.security.ldap.enabled` | `false` | The LDAP realm should be active |
| LDAP | `quarkus.security.ldap.dir-context.url` | none | You need to point Quarkus at the directory server |
| LDAP | `quarkus.security.ldap.dir-context.principal` / `password` | none | LDAP searches require a bind identity |
| LDAP | `quarkus.security.ldap.identity-mapping.search-base-dn` | none | User search must start from the correct subtree |
| LDAP | `quarkus.security.ldap.identity-mapping.attribute-mappings.*` | none | LDAP group or attribute lookup should map into Quarkus roles |
| LDAP | `quarkus.security.ldap.cache.enabled` | `false` | You want fewer LDAP roundtrips and can tolerate cache staleness |

## JPA

```properties
quarkus.security-jpa.persistence-unit-name=users
```

Use this only when the `@UserDefinition` entity is not in the default persistence unit. Most applications using a single datasource do not need any JPA-specific security property beyond the normal ORM setup.

## JDBC

### Minimal bcrypt-backed query

```properties
quarkus.security.jdbc.enabled=true
quarkus.security.jdbc.principal-query.sql=SELECT password, role FROM app_user WHERE username=?
quarkus.security.jdbc.principal-query.bcrypt-password-mapper.enabled=true
quarkus.security.jdbc.principal-query.bcrypt-password-mapper.password-index=1
quarkus.security.jdbc.principal-query.attribute-mappings.0.index=2
quarkus.security.jdbc.principal-query.attribute-mappings.0.to=groups
```

Notes:

- Query indexes are 1-based.
- `to=groups` is the usual target for role-based authorization.
- Keep the SQL narrow: password first, only the extra columns you really need.

### Named datasource or separate roles query

```properties
quarkus.security.jdbc.principal-query.datasource=users
quarkus.security.jdbc.principal-query.roles.sql=SELECT role_name FROM user_role WHERE username=?
quarkus.security.jdbc.principal-query.roles.attribute-mappings.0.index=1
quarkus.security.jdbc.principal-query.roles.attribute-mappings.0.to=groups
```

Use this when credentials and roles live in different tables or different datasources.

### Non-MCF bcrypt layout

```properties
quarkus.security.jdbc.principal-query.bcrypt-password-mapper.password-index=1
quarkus.security.jdbc.principal-query.bcrypt-password-mapper.salt-index=2
quarkus.security.jdbc.principal-query.bcrypt-password-mapper.iteration-count-index=3
```

Leave `salt-index` and `iteration-count-index` at `-1` when the password column already uses Modular Crypt Format.

## LDAP

### Core connection and identity mapping

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

The highest-value LDAP settings are the server URL, bind identity, user search base, and the group lookup filter.

### Performance and reliability settings

```properties
quarkus.security.ldap.cache.enabled=true
quarkus.security.ldap.cache.max-age=5m
quarkus.security.ldap.cache.size=1000
quarkus.security.ldap.dir-context.connect-timeout=5s
quarkus.security.ldap.dir-context.read-timeout=10s
quarkus.security.ldap.dir-context.referral-mode=ignore
```

Enable caching only when a short delay in reflecting directory changes is acceptable.

### Direct verification and recursive search

```properties
quarkus.security.ldap.direct-verification=true
quarkus.security.ldap.identity-mapping.search-recursive=true
```

- `direct-verification=true` is the normal choice when the LDAP directory should verify the supplied password.
- `search-recursive=true` helps when users are nested below the base DN rather than stored directly under it.

## Safe reading order

1. Confirm the store is enabled.
2. Confirm how Quarkus finds the user record: entity, SQL, or LDAP search base.
3. Confirm how Quarkus verifies the password.
4. Confirm how roles map into `groups`.
5. Add performance tuning only after the basic auth flow works.
