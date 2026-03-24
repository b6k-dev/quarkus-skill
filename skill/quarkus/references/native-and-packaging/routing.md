# Native and Packaging Routing

Use this file when a packaging-related request is still ambiguous and you need to route to the right leaf reference quickly.

## Route to `native-image`

Start with `native-image` when the main issue is one of these:

- Build or test commands using `-Dnative`, `quarkus.native.enabled`, or builder images
- Reflection, resources, proxies, `@RegisterForReflection`, or class-initialization failures
- Native-only SSL, trust store, heap, GC, diagnostics, or runtime differences from JVM mode
- Native executable validation with `@QuarkusIntegrationTest`

Typical prompts:

- "Why does my Quarkus app fail only in native mode?"
- "How do I include resources or reflection metadata in the native build?"
- "Should I build native locally or in a container?"

## Route to `packaging-jars`

Start with `packaging-jars` when the main issue is artifact shape rather than native compilation:

- Choosing between `fast-jar`, `uber-jar`, `mutable-jar`, or `legacy-jar`
- Understanding `quarkus-app/` layout and what must be shipped together
- Runner JAR naming, output directory customization, or optional dependency filtering
- Re-augmentation and mutable-jar operational tradeoffs

Typical prompts:

- "Should I ship `fast-jar` or `uber-jar`?"
- "Why does `java -jar target/quarkus-app/quarkus-run.jar` fail after copying only one file?"
- "When do I need `mutable-jar`?"

## Route to `packaging-containers`

Start with `packaging-containers` when the main issue is image construction or publishing:

- Choosing Jib vs Docker vs Podman vs OpenShift vs Buildpacks
- Registry naming, tagging, pushing, and multi-platform build behavior
- Dockerfile-based packaging, base image selection, or native runtime base images
- Image entrypoint/layout issues caused by `fast-jar`, `uber-jar`, or native outputs

Typical prompts:

- "Which Quarkus container-image extension should I use?"
- "How do I build and push a multi-arch image?"
- "What base image should I use for a native executable?"

## Route to `tooling` instead

Use `tooling` instead of this area when the packaging choice is already known and the user mostly needs:

- CLI vs Maven vs Gradle command equivalents
- Project creation, extension management, or dev-mode commands
- General build automation guidance that is not packaging-specific

## Mixed Requests

- If the request starts with native compilation and ends with containerizing the native binary, start in `native-image`, then cross-link to `packaging-containers`.
- If the request starts with `mutable-jar` or re-augmentation and later mentions Docker, start in `packaging-jars`, then cross-link to `packaging-containers`.
- If the request is "what should I choose?", keep the first answer decision-oriented and avoid dumping all three modules at once.
