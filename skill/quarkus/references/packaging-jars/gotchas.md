# Quarkus JAR Packaging Gotchas

Common jar packaging pitfalls, symptoms, and fixes.

## Fast-jar and mutable-jar delivery mistakes

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `quarkus-run.jar` exists but startup fails with missing classes/resources | Only the runner jar was copied, not the full `quarkus-app/` directory | Ship the entire `quarkus-app/` directory, including `app/`, `lib/`, and `quarkus/` |
| Container starts with `java -jar app.jar` but the file does not exist | The image entrypoint assumes an uber jar while the build produced `fast-jar` | Align the entrypoint with the selected jar type and actual output layout |

## Re-augmentation surprises

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Build-time property changes appear ignored after packaging | The app was built as `fast-jar` or `uber-jar`, not `mutable-jar` | Rebuild as `mutable-jar` before trying `quarkus.launch.rebuild=true` |
| Re-augmentation runs but the app never starts | `quarkus.launch.rebuild=true` rebuilds augmentation output and exits by design | Run the rebuild step first, then launch normally without the rebuild flag |
| Re-augmentation cannot use a new feature or driver | The needed extension was not present at original build time | Include the extension during the original build; re-augmentation cannot add new extension capabilities later |
| Re-augmentation stops working after image slimming | `quarkus-app/lib/deployment/` was deleted to save space | Keep `lib/deployment/` if future re-augmentation is required |

## Mutable-jar security and config pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Sensitive values appear inside mutable-jar contents | Build-time `quarkus.*` values were put in `pom.xml` or `gradle.properties` | Keep secrets out of build tool config and inject sensitive values through safer runtime or deployment mechanisms |
| Runtime launch succeeds with the wrong build-time behavior | Build-time config drift only produced a warning or was silently ineffective | Use `quarkus.config.build-time-mismatch-at-runtime=fail` and make re-augmentation explicit |

## Uber-jar pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Uber-jar build fails or runtime verification breaks | Dependency signature/resource entries conflict when merged | Exclude conflicting entries with `quarkus.package.ignored-entries` or build-tool-specific ignored-entry options |
| Downstream script cannot find the expected jar name | The default `-runner` suffix changed the final filename | Keep the suffix documented, customize `quarkus.package.runner-suffix`, or disable the added suffix intentionally |

## Optional dependency filtering pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Expected optional driver/provider is missing from the package | `filter-optional-dependencies=true` was enabled but the dependency was not listed | Add the dependency to `quarkus.package.jar.included-optional-dependencies` using full GACT form |
| Package still contains all optional dependencies | The include list was set but filtering itself was not enabled | Set both `quarkus.package.jar.filter-optional-dependencies=true` and the include list |

## Legacy-jar expectations

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Older scripts expect a pre-1.12-style layout | The build now defaults to `fast-jar` | Update the scripts for `fast-jar` or switch explicitly to `legacy-jar` only when compatibility truly requires it |

## See Also

- [./configuration.md](./configuration.md) - The properties behind these failure modes
- [./patterns.md](./patterns.md) - Safer package selection and delivery workflows
