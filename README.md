# TypeWeave

**Compile-time typesafe REST for Spring Boot 4 and Java 25.**

TypeWeave is a design-and-build effort for a Spring REST stack where you annotate controller methods with entity metadata, bind keys, and association trails. An annotation processor generates the repository, service, and mappers at **compile time** — wrong lookups, invalid trails, or DTO/graph mismatches fail the build, not production.

> **Status:** Design phase. No implementation modules yet. See [docs/design-spec.md](docs/design-spec.md) for the full specification.

---

## Problem

Typical Spring APIs repeat the same wiring for every endpoint: repository query, fetch plan, service method, entity→DTO mapping, and pagination boilerplate. Association loading is easy to get wrong (`LazyInitializationException`, N+1 queries), and response shapes often drift from what the persistence layer actually loaded.

## Approach

You keep **hand-written** domain types and API contracts; TypeWeave **generates** the plumbing beneath them.

| You write | TypeWeave generates |
|-----------|---------------------|
| JPA entities (`UserEntity`, …) | `UserRepository` (+ fetch variants per trail) |
| Response DTOs per graph shape | `UserService` / `UserServiceImpl` |
| Controller + annotations | Entity→DTO mappers |

Annotate a GET on your controller — the processor weaves repository and service code before the app runs.

```java
@WeaveRead(
    entity = UserEntity.class,
    bindKey = @BindKey(UserEntity_.id),
    defaultResponse = UserResponseDto.class,
    shapes = {
        @TrailShape(
            trail = @Trail(UserEntity_.orders), 
            response = UserDetailResponseDto.class
            )
    }
)
@GetMapping("/{id}")
public Object getUser(@PathVariable Long id, 
                    @RequestParam(required = false) String trail) {
    return userService.getUser(id, trail);
}
```

- **`GET /users/{id}`** → `UserResponseDto` (root entity only)
- **`GET /users/{id}?trail=user.orders`** → `UserDetailResponseDto` (orders loaded)
- Invalid trail → HTTP 400; invalid annotation/DTO pairing → compile error

List endpoints use **`@WeaveList`** with Spring Data **`Pageable`** / **`Page<>`** and validated sort fields.

---

## Key ideas

- **Controller-first** — metadata lives on the REST method you already own, not only on entities (unlike many CRUD generators).
- **Trail-driven DTOs** — each association graph shape maps to a **separate hand-written DTO** (`@OneToMany`, `@ManyToOne`, etc.); no single DTO that silently morphs at runtime.
- **Type-safe paths** — `bindKey` and `trail` use JPA static metamodel (or `FieldRef`), not stringly-typed JPQL in controllers.
- **Spring Boot 4 + Java 25** — constructor injection, `@Transactional` services, optional virtual threads.

---

## Planned layout

```
typesafe-api/
├── typeweave-annotations/   # @WeaveRead, @WeaveList, @TrailShape, @BindKey, …
├── typeweave-processor/     # Annotation processor (APT)
├── typeweave-runtime/       # Trail parsing, exceptions, optional auto-config
├── app/                     # Demo Spring Boot application
└── docs/
```

---

## Documentation

| Document | Description |
|----------|-------------|
| [docs/thoughts.md](docs/thoughts.md) | Original intent and examples |
| [docs/design-spec.md](docs/design-spec.md) | Full design: annotations, codegen, pagination, trail/DTO rules, phases |
| [docs/similar-frameworks.md](docs/similar-frameworks.md) | Related tools in Java and Scala (CrudCraft, Spring Data REST, Tapir, …) |

---

## Tech stack (target)

- Java 25
- Spring Boot 4.0.x (Spring MVC)
- Spring Data JPA + Hibernate 7
- Gradle 9 (Kotlin DSL)
- Java Annotation Processing API

---

## Comparison at a glance

| | TypeWeave | Entity CRUD generators | Spring Data REST |
|--|-----------|------------------------|------------------|
| Trigger | Controller annotations | JPA `@Entity` | `Repository` beans |
| Output | Repo + service + mappers | Often full stack incl. controller | Runtime HTTP on repos |
| Per-endpoint DTO/trail | Yes | Usually entity-centric | HAL / hypermedia |
| Compile-time safety | Bind key, trail, DTO shape | Entity-focused | Repository names |

See [docs/similar-frameworks.md](docs/similar-frameworks.md) for a broader survey.

---

## Roadmap (from design spec)

1. **Phase 1** — `@WeaveRead`, trail-driven DTO registry, generated repo/service/mappers, demo app
2. **Phase 2** — `@WeaveList` pagination, `@WeaveWrite`, MapStruct integration
3. **Phase 3** — springdoc, native image smoke tests, IDE hints

---

## License

TBD.
