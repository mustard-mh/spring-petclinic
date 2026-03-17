---
name: code-quality
description: Spring PetClinic code quality conventions. Use when writing, reviewing, or modifying Java code in this project to ensure consistency with established patterns.
---

# Spring PetClinic Code Quality Guidelines

## Project Overview

Spring Boot 3.5 / Java 17 MVC web application using Thymeleaf, Spring Data JPA, and H2 (default).

## Build & Test

```bash
./mvnw package                    # compile, test, checkstyle, format validation
./mvnw test                       # unit tests only
./mvnw spring-javaformat:apply    # auto-format to Spring Java Format
./mvnw spring-boot:run            # run on port 8080
```

The build enforces:
- **Spring Java Format** (`spring-javaformat-maven-plugin`)
- **Checkstyle** with nohttp — no `http://` URLs where `https://` should be used
- **JaCoCo** code coverage reporting

Always run `./mvnw spring-javaformat:apply` before committing. Tests must pass before any commit.

## Architecture

```
src/main/java/org/springframework/samples/petclinic/
├── model/    # Base entities: BaseEntity, NamedEntity, Person
├── owner/    # Owner, Pet, PetType, Visit entities + controllers + repositories
├── vet/      # Vet, Specialty entities + controller + repository
└── system/   # Configuration (cache, web) + system controllers
```

### Layering Rules

- **Controllers** are package-private classes annotated with `@Controller`. They receive repository dependencies via constructor injection.
- **Repositories** extend Spring Data interfaces. Query methods use Spring Data naming conventions.
- **Entities** extend `BaseEntity` (id + `isNew()`), `NamedEntity` (adds name), or `Person` (adds firstName/lastName). Use JPA annotations and Jakarta Validation.
- No service layer — controllers interact with repositories directly.

## Code Conventions

### Formatting

- **Tabs** for Java and XML indentation (tab width 4). See `.editorconfig`.
- Line continuations: align with opening parenthesis or use single tab indent.

### Java Style

- Package-private visibility for controllers (no `public` modifier on controller classes).
- Constructor injection only — no field injection, no `@Autowired` on fields.
- `Optional` return types from repositories; handle with `.orElseThrow()`.
- `List.of()` for immutable lists, `new ArrayList<>()` for mutable collections.
- `@Valid` on controller method parameters for bean validation.
- `RedirectAttributes.addFlashAttribute()` for user-facing messages after redirects.
- Apache 2.0 license header (2012-2025) on all source files.

### Naming

- Entity classes: singular nouns (`Owner`, `Pet`, `Vet`).
- Repository interfaces: plural nouns (`OwnerRepository`, `VetRepository`).
- Controller classes: `{Entity}Controller`.
- Test classes: `{Class}Tests` (not `Test` singular).
- View constants: `VIEWS_` prefix for view name strings.

### Testing

- `@WebMvcTest` for controller tests with `@MockitoBean` for repository mocking.
- `@DisabledInNativeImage` and `@DisabledInAotMode` on `@WebMvcTest` classes.
- BDD-style Mockito: `given(...).willReturn(...)`.
- Static imports for Hamcrest matchers and MockMvc methods.
- Test method names: `test{Action}{Scenario}` (e.g., `testProcessCreationFormSuccess`).
- Private helper methods for test fixtures (e.g., `george()`).

### JPA / Database

- `@Entity` and `@Table(name = "...")` on entity classes.
- `@Column(name = "...")` explicitly for all mapped fields.
- Relationships use `@OneToMany`, `@ManyToOne` with explicit `cascade` and `fetch` settings.
- H2 default; MySQL and PostgreSQL via profiles and `docker-compose.yml`.

### Validation

- Jakarta Validation annotations on entity fields (`@NotBlank`, `@Pattern`).
- Custom validators implement `org.springframework.validation.Validator` (see `PetValidator`).
- Bind validation errors via `BindingResult` in controller methods.

## Anti-Patterns to Avoid

- Do not add a service layer unless there is cross-cutting business logic.
- Do not use `@Autowired` on fields — use constructor injection.
- Do not use Lombok — this project uses explicit getters/setters.
- Do not use `public` on controller classes — keep them package-private.
- Do not introduce new dependencies without verifying compatibility with Spring Boot 3.5 and Java 17.
- Do not use `http://` URLs — the checkstyle build will fail.
- Do not skip the license header on new Java files.
