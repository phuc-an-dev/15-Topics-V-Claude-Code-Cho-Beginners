# Topic 3: Best Claude Skills Cho Spring Boot Developer

> **Nguồn**: `Jeffallan/claude-skills`, `a-pavithraa/springboot-skills-marketplace`, `piomin/claude-ai-spring-boot` (Piotr's Blog), `claudioed/claude-skills`, `VoltAgent/awesome-claude-code-subagents`

---

## 🎯 Starter Pack cho Spring Boot Developer

### Must-Have (Top 5)

#### 1. **piomin/claude-ai-spring-boot** ⭐⭐⭐⭐⭐
**Repo**: https://github.com/piomin/claude-ai-spring-boot

**Template hoàn chỉnh cho Spring Boot project** - clone & dùng ngay.

**Cấu trúc (pre-built `.claude/`):**
```
.claude/
├── agents/                      # 8 specialized subagents
│   ├── code-reviewer.md
│   ├── devops-engineer.md
│   ├── docker-expert.md
│   ├── java-architect.md
│   ├── kubernetes-specialist.md
│   ├── security-engineer.md
│   ├── spring-boot-engineer.md
│   └── test-automator.md
└── skills/
    ├── api-contract-review/
    ├── clean-code/
    ├── design-patterns/
    ├── java-architect/         # với references/jpa-optimization, spring-security, ...
    ├── java-code-review/
    ├── jpa-patterns/
    ├── logging-patterns/
    └── spring-boot-engineer/
```

Author **Piotr Mińkowski** - tác giả sách "Hands-On Java with Kubernetes".

#### 2. **a-pavithraa/springboot-skills-marketplace** 🔥
**Repo**: https://github.com/a-pavithraa/springboot-skills-marketplace

**Architecture-first approach**: Hỏi vài câu về project, sau đó build đúng architecture.

**Skills chính**:
- `creating-springboot-projects`: Assessment-driven scaffolding, Spring Initializr setup
- `spring-data-jpa`: JPA code không slow down sau 6 tháng (query patterns, DTO projections, custom repos, CQRS)
- `java25-springboot4-reviewer`: Check architecture fit, performance gotchas (N+1, virtual thread pinning, entity leaks)

**5 Architecture Patterns supported**:
- Layered (cho CRUD đơn giản)
- Tomato architecture (cho business rules phức tạp)
- CQRS (Command Query Responsibility Segregation)
- Hexagonal / Ports & Adapters
- Modulith (monolith với boundaries rõ ràng)

**Install**:
```bash
/plugin marketplace add a-pavithraa/springboot-skills-marketplace
/plugin enable a-pavithraa/springboot-skills-marketplace
/plugin install springboot-architecture@springboot-skills-marketplace
```

#### 3. **Jeffallan/claude-skills** (Java Architect + Spring Boot Engineer)
**Repo**: https://github.com/Jeffallan/claude-skills

**2 skills quan trọng cho Spring dev**:

**A. `java-architect`**:
- Enterprise Java với Spring Boot 3.x, Java 21 LTS
- Microservices architecture
- Cloud-native development
- Tech stack: Spring WebFlux, Project Reactor, Spring Data JPA, Spring Security, OAuth2/JWT, Hibernate, R2DBC, Spring Cloud, Resilience4j, Micrometer
- Testing: JUnit 5, TestContainers, Mockito

**B. `spring-boot-engineer`**:
- Workflow: Design → Implement → Secure → Test (mỗi step verify trước khi proceed)
- Constructor injection, layered architecture
- Security: Spring Security, OAuth2, method security, CORS
- Testing: unit, integration, slice tests với `./mvnw test` verification

**Recommended combo**:
> "Modernize legacy Java monolith" → Legacy Modernizer + Java Architect + Microservices Architect

#### 4. **claudioed/claude-skills**
**Repo**: https://github.com/claudioed/claude-skills

**4 Java-specific skills cực hữu ích**:

| Skill | Mục đích |
|-------|---------|
| `spring-bootstrap-optimizer` | Tối ưu Spring Boot startup time |
| `java17-2-21-migration` | Migrate Java 17 → 21 |
| `java-dependency-auditor` | Scan vulnerable/outdated/unused deps, license compliance |
| `java-mutation-test` | Setup PITest, mutation testing |

Hỗ trợ: Spring Boot 2.x/3.x, Maven & Gradle

#### 5. **giuseppe-trisciuoglio/developer-kit** (Java Plugin)
**Repo**: https://github.com/giuseppe-trisciuoglio/developer-kit

Modular plugin marketplace:
- `developer-kit-core` (required base)
- `developer-kit-java` (Java ecosystem plugin)
- Plus: typescript, python, php, aws plugins

**Auto-activate rules** cho `*.java` files:
> "Always use constructor injection. Never use field injection with `@Autowired`."

**Install**:
```bash
/plugin marketplace add giuseppe-trisciuoglio/developer-kit
```

---

## 🏗️ Specialized Skills theo Needs

### For Microservices
- **Microservices Architect** (Jeffallan)
- **spring-boot-engineer** + Spring Cloud patterns
- **Circuit Breaker patterns** (Resilience4j)
- **Distributed tracing** (Micrometer)

### For Database/JPA
- **spring-data-jpa** (a-pavithraa) - N+1 prevention, DTO projections
- **jpa-patterns** (piomin) - relationship handling
- **Database Optimizer** skill

### For Reactive Programming
- **java-architect** với `references/reactive-webflux.md`
- **Spring WebFlux** + Project Reactor patterns
- **R2DBC** cho reactive DB access

### For Security
- **spring-security** references (piomin/java-architect)
- **OAuth2/JWT** setup
- **security-engineer** subagent
- **owasp-security** (cross-language)

### For Testing
- **test-automator** subagent (piomin)
- **java-mutation-test** (claudioed) cho PITest
- **TestContainers** patterns
- **Slice tests** (@WebMvcTest, @DataJpaTest, @SpringBootTest)

### For DevOps
- **devops-engineer** subagent
- **docker-expert** + **kubernetes-specialist** (piomin)
- Dockerfile generation cho Spring Boot
- K8s manifests (Deployment, Service, ConfigMap, Secret, HPA)

---

## 📋 CLAUDE.md Template Cho Spring Boot Project

```markdown
# Project: [Tên]

## Stack
- Spring Boot 3.2+
- Java 21 LTS
- Spring Data JPA with Hibernate
- PostgreSQL (production), H2 (tests)
- Maven (or Gradle)
- JUnit 5 + Mockito + TestContainers

## Architecture
- Layered architecture: Controller → Service → Repository
- DDD principles: Domain models với rich entities
- Constructor injection ONLY (never @Autowired on fields)
- Use DTOs for API, never expose entities

## Code Style
- Prefer `var` for local vars when type obvious
- Immutable POJOs where possible (records cho DTOs)
- Use Lombok sparingly (@RequiredArgsConstructor OK, @Data NO)
- Package by feature, not by layer

## Spring Boot Rules
- NO business logic trong controllers
- @Transactional ở service layer
- Use @Slice test annotations (@WebMvcTest, @DataJpaTest)
- Custom exceptions + @ControllerAdvice cho error handling
- Structured logging với MDC context

## JPA/Hibernate Rules
- NEVER use `findAll()` trong production paths
- Pagination mandatory cho list endpoints
- Use DTO projections cho read queries (avoid entity bloat)
- @EntityGraph cho relationships cần fetch
- JPA queries: prefer @Query over method naming (clarity)

## Commands
- `./mvnw spring-boot:run`: Dev server
- `./mvnw test`: Unit + integration tests
- `./mvnw verify`: Full build with coverage
- `./mvnw spring-boot:build-image`: Build OCI image

## IMPORTANT
- Run `./mvnw verify` AFTER every security change
- NEVER commit `application.yml` with real passwords
- Use `@ConfigurationProperties` over `@Value`
- Externalize config via Spring Cloud Config / env vars
```

---

## 🔧 Workflow Examples

### Example 1: Create CRUD Service
```
> Using java-architect skill, create a UserService CRUD:
> - REST controller with CRUD endpoints
> - Service layer with business logic
> - JPA repository with custom queries
> - DTOs for request/response
> - Validation with Bean Validation
> - Unit + integration tests
> - OpenAPI documentation
```
→ Triggers: java-architect + spring-boot-engineer + test-automator

### Example 2: Performance Investigation
```
> My product search API is slow.
> Use java25-springboot4-reviewer to check for:
> - N+1 queries
> - Missing caches
> - Virtual thread pinning
> - Entity leaks
```
→ Triggers: spring-data-jpa skill + Database Optimizer

### Example 3: Migration
```
> Migrate this project from Spring Boot 3.2 to Spring Boot 4.
> Use the java25-springboot4-reviewer migration scanner.
> Walk through dependencies → code → config in phases.
```
→ Triggers: java17-2-21-migration + spring-boot migration skill

### Example 4: Security Hardening
```
> Add Spring Security with JWT authentication.
> Implement role-based access control.
> Run ./mvnw verify after to confirm filter chain works.
```
→ Triggers: java-architect + security-engineer

---

## 🎯 Subagent Strategy

### Combos hiệu quả từ Jeffallan guide:
| Task | Subagents combo |
|------|-----------------|
| Modernize legacy Java monolith | Legacy Modernizer + Java Architect + Microservices Architect |
| Feature development | Feature Forge + Architecture Designer + Fullstack Guardian + Test Master + DevOps Engineer |
| Bug investigation | Debugging Wizard + Framework Expert + Test Master + Code Reviewer |
| Security hardening | Secure Code Guardian + Security Reviewer + Test Master |

### Piotr's Template subagents (có sẵn):
- `code-reviewer` - PR reviews
- `devops-engineer` - CI/CD, deployment
- `docker-expert` - Dockerfile optimization
- `java-architect` - Architecture decisions
- `kubernetes-specialist` - K8s manifests
- `security-engineer` - Security review
- `spring-boot-engineer` - Implementation
- `test-automator` - Test coverage

---

## 🚫 Common Anti-Patterns (Skills Giúp Tránh)

### 1. **Repository per Entity Hell**
❌ Tạo `UserRepository`, `OrderRepository`, `ProductRepository`... cho mọi entity
✅ Shared generic patterns, CQRS query services

### 2. **Method-name Query Hell**
❌ `findByFirstNameAndLastNameAndEmailAndActiveAndCreatedDateBetween...`
✅ Custom `@Query` với clear name

### 3. **Blind `save()` Calls**
❌ Call `repository.save()` mà không understand dirty checking
✅ Understand JPA persistence context, use `saveAndFlush()` khi cần

### 4. **N+1 Queries**
❌ `@OneToMany` lazy load trong loops
✅ `@EntityGraph` or DTO projections, explicit `JOIN FETCH`

### 5. **Entity Leaks qua API**
❌ Return `User` entity trực tiếp từ controller
✅ DTO mapping, never expose entities

### 6. **Field Injection**
❌ `@Autowired private UserService userService;`
✅ Constructor injection (Lombok `@RequiredArgsConstructor`)

### 7. **Business Logic trong Controller**
❌ Controllers làm CRUD + business rules + validation
✅ Thin controllers, fat services

### 8. **Missing Pagination**
❌ `GET /api/users` returns all 1M users
✅ `Pageable` parameter mandatory cho list endpoints

---

## 📦 Quick Install Order

### Tuần 1 - Foundations
1. Clone `piomin/claude-ai-spring-boot` làm template base
2. Install `a-pavithraa/springboot-skills-marketplace` plugin
3. Customize CLAUDE.md cho project

### Tuần 2 - Deep dive
4. Add `Jeffallan/claude-skills` java-architect + spring-boot-engineer
5. Add `claudioed/claude-skills` (dependency auditor rất useful)

### Tuần 3 - Advanced
6. Setup `giuseppe-trisciuoglio/developer-kit` Java plugin
7. Add mutation testing với java-mutation-test skill

### Tuần 4 - Operations
8. Subagents cho DevOps/K8s
9. Custom project-specific skills dựa trên patterns team bạn

---

## 📚 Key Repos Summary

| Repo | Focus |
|------|-------|
| [piomin/claude-ai-spring-boot](https://github.com/piomin/claude-ai-spring-boot) | Complete SB template với 8 subagents + 10 skills |
| [a-pavithraa/springboot-skills-marketplace](https://github.com/a-pavithraa/springboot-skills-marketplace) | Architecture-first, 5 patterns |
| [Jeffallan/claude-skills](https://github.com/Jeffallan/claude-skills) | java-architect + spring-boot-engineer |
| [claudioed/claude-skills](https://github.com/claudioed/claude-skills) | Dep audit, Java 21 migration, mutation test |
| [giuseppe-trisciuoglio/developer-kit](https://github.com/giuseppe-trisciuoglio/developer-kit) | Modular plugin kit |
| [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) | 100+ subagents incl. spring-boot-engineer |

## 📝 Blog Resources
- Piotr's TechBlog: https://piotrminkowski.com/2026/03/24/claude-code-template-for-spring-boot/
- Jeffallan Skills Guide: https://jeffallan.github.io/claude-skills/
