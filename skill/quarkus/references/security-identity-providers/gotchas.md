# Quarkus Security Identity Providers Gotchas

Common identity-store pitfalls, symptoms, and fixes.

## JPA identity store

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Application fails to initialize the JPA identity provider | More than one entity is annotated with `@UserDefinition` | Keep `@UserDefinition` on exactly one entity |
| Authentication works inconsistently across databases | User table is named `user` | Use a safer table name such as `app_user` |
| Password verification always fails | Stored values are plain text but `@Password` uses the default hashed behavior | Store bcrypt hashes or explicitly use a clear/custom password type for controlled cases |
| Roles are missing | Role entity collection is mapped without a `@RolesValue` field | Mark the string role field in the role entity with `@RolesValue` |

## JDBC identity store

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Login always fails even though the SQL seems correct | Password or attribute indexes are off by one | Remember JDBC realm indexes are 1-based |
| Roles never satisfy `@RolesAllowed` | SQL columns were not mapped to `groups` | Add `attribute-mappings.*.to=groups` for role columns |
| Bcrypt verification fails against an existing schema | Salt and iteration count are stored outside the password column but mapper still assumes MCF | Set `salt-index` and `iteration-count-index`, or move to MCF storage |
| Startup succeeds but no users can authenticate | `quarkus.security.jdbc.enabled` was not set | Enable the JDBC realm explicitly |

## LDAP identity store

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Bind works but users are never found | `search-base-dn` or `rdn-identifier` does not match the directory layout | Verify the user subtree and identifier attribute |
| Users authenticate but have no roles | Group filter or filter base DN does not match the directory structure | Recheck `attribute-mappings.*.filter` and `filter-base-dn` |
| Role changes take time to appear | LDAP cache is enabled | Reduce `cache.max-age` or disable the cache |
| Auth is slow under load | Every request hits LDAP directly | Enable cache and tune connect/read timeouts after correctness is confirmed |

## Cross-provider choices

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| The setup feels overly complex for a simple user table | LDAP or custom SQL was chosen too early | Use JPA when auth data is app-owned, or JDBC when only a simple existing schema matters |
| Identity setup leaks infrastructure-specific role names into code | Resource annotations mirror DB or LDAP names directly | Normalize with `quarkus.http.auth.roles-mapping.*` |
| Local testing depends on external infrastructure | Production identity source is reused in dev without a lighter profile | Add `%dev` or `%test` fixtures for JPA/JDBC, or a dedicated LDAP test server |
