# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Run the application
```bash
./mvnw spring-boot:run        # Maven
./gradlew bootRun             # Gradle
```
App available at http://localhost:8080.

### Build and test
```bash
./mvnw verify                 # build + tests + checkstyle + format validation
./mvnw test                   # tests only
./gradlew test                # tests only (Gradle)
```

### Run a single test class
```bash
./mvnw test -Dtest=OwnerControllerTests
./mvnw test -Dtest=ClinicServiceTests#testFindOwnersByLastName
```

### Code formatting (Spring Java Format)
```bash
./mvnw spring-javaformat:apply   # auto-format all Java sources
```
Format is validated on every `validate` phase; apply it before committing to avoid build failures.

### Recompile CSS from SCSS
```bash
./mvnw package -P css        # Maven only; no Gradle equivalent
```
Only needed when editing `src/main/scss/petclinic.scss` or upgrading Bootstrap.

### Container image
```bash
./mvnw spring-boot:build-image
```

## Architecture

### Package layout
```
org.springframework.samples.petclinic
├── model/          # JPA base classes (BaseEntity, NamedEntity, Person)
├── owner/          # Owner, Pet, PetType, Visit domain + controllers + repositories
├── vet/            # Vet, Specialty domain + controller + repository
└── system/         # CacheConfiguration, WelcomeController, CrashController, WebConfiguration
```

### Domain model inheritance
All entities extend `BaseEntity` (provides `id`, `isNew()`). Named entities extend `NamedEntity`. People (Owner, Vet) extend `Person` (firstName, lastName). The hierarchy is `BaseEntity → NamedEntity → ...` and `BaseEntity → Person → Owner/Vet`.

### Data access
Repositories are plain Spring Data JPA interfaces (`OwnerRepository`, `PetTypeRepository`, `VetRepository`) — no service layer sits between controllers and repositories. Controllers wire directly to repositories.

### Views
Thymeleaf templates live in `src/main/resources/templates/`. The shared page chrome is in `templates/fragments/layout.html`. Form field partials (`inputField.html`, `selectField.html`) are referenced by owner/pet/visit forms.

### Databases
Default is in-memory H2 (schema + data loaded from `src/main/resources/db/h2/`). Switch profiles for persistent databases:
- `spring.profiles.active=mysql` — requires MySQL on port 3306
- `spring.profiles.active=postgres` — requires PostgreSQL on port 5432

Start databases via Docker Compose: `docker compose up mysql` or `docker compose up postgres`.

### Testing
- `*Tests` classes — unit/slice tests (MockMvc, no real DB)
- `PetClinicIntegrationTests` — full Spring context with H2
- `MySqlIntegrationTests` — uses Testcontainers to spin up MySQL
- `PostgresIntegrationTests` — uses Docker Compose to spin up Postgres

### i18n
All UI strings go through `src/main/resources/messages/messages*.properties`. `I18nPropertiesSyncTest` asserts that all locale files stay in sync with the default `messages.properties` — adding a key to one file requires adding it to all.

### Coding conventions
- Keep all application classes under `org.springframework.samples.petclinic`
- Name tests `*Tests` (unit/slice) or `*IT` (integration)
- Commits require a `Signed-off-by` trailer (DCO)