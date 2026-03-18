# Language-Specific Reviewer Agents

This document covers all language-specific reviewer and build-resolver agents in ECC. Each follows the same structural pattern but applies language-specific idioms and best practices.

---

## Agent Inventory

| Agent | Language/Stack | Model | Source |
|---|---|---|---|
| go-reviewer | Go | sonnet | `agents/go-reviewer.md` |
| go-build-resolver | Go | sonnet | `agents/go-build-resolver.md` |
| python-reviewer | Python | sonnet | `agents/python-reviewer.md` |
| java-reviewer | Java + Spring Boot | sonnet | `agents/java-reviewer.md` |
| java-build-resolver | Java/Maven/Gradle | sonnet | `agents/java-build-resolver.md` |
| kotlin-reviewer | Kotlin/Android/KMP | sonnet | `agents/kotlin-reviewer.md` |
| kotlin-build-resolver | Kotlin/Gradle | sonnet | `agents/kotlin-build-resolver.md` |
| rust-reviewer | Rust | sonnet | `agents/rust-reviewer.md` |
| rust-build-resolver | Rust/Cargo | sonnet | `agents/rust-build-resolver.md` |
| cpp-reviewer | C++ | sonnet | `agents/cpp-reviewer.md` |
| cpp-build-resolver | C++/CMake | sonnet | `agents/cpp-build-resolver.md` |

---

## go-reviewer

**Purpose:** Expert Go code reviewer specializing in idiomatic Go, concurrency patterns, error handling, and performance.

**When to Use:** For all Go code changes. Activated automatically in Go projects.

**Key Review Areas:**
- Idiomatic Go (error return patterns, interface design)
- Goroutine and channel safety
- Context propagation (`context.Context` usage)
- Error wrapping with `fmt.Errorf("...: %w", err)`
- Missing `defer` for resource cleanup
- Race conditions (`go test -race`)
- Proper use of `sync.Mutex` vs `sync/atomic`

**Review Process:**
1. Run `git diff -- '*.go'` to see changes
2. Run `go vet ./...` for static analysis
3. Run `golangci-lint run` for comprehensive linting
4. Review concurrency patterns manually

**Build Resolver (go-build-resolver):**
```bash
go build ./...      # Identify build errors
go mod tidy         # Fix dependency issues
go mod verify       # Verify module integrity
```

---

## python-reviewer

**Purpose:** Expert Python code reviewer specializing in PEP 8, Pythonic idioms, type hints, security, and performance.

**When to Use:** For all Python code changes. MUST BE USED for Python projects.

**Key Review Areas:**
- PEP 8 compliance (`flake8`, `black` formatting)
- Type annotations (`mypy` strictness)
- Pythonic patterns (list comprehensions, generators, context managers)
- Security: `eval()`, `pickle`, shell injection via `subprocess`
- Dependency injection and testability
- Missing `__init__.py` or package structure issues

**Review Process:**
1. Run `git diff -- '*.py'` to see changes
2. Run `flake8 .` for style and error checking
3. Run `mypy .` for type checking
4. Run `bandit -r .` for security analysis

**Common Issues:**
```python
# BAD: mutable default argument
def process(items=[]):  # Bug: shared across calls
    items.append(1)

# GOOD: use None sentinel
def process(items=None):
    if items is None:
        items = []
    items.append(1)
```

---

## java-reviewer

**Purpose:** Expert Java and Spring Boot code reviewer specializing in layered architecture, JPA patterns, security, and concurrency.

**When to Use:** For all Java code changes. MUST BE USED for Spring Boot projects.

**Key Review Areas:**
- Spring Boot layered architecture (Controller → Service → Repository)
- JPA/Hibernate N+1 queries and lazy loading
- Transaction boundaries (`@Transactional` placement)
- Exception handling (`@ControllerAdvice`, proper exception hierarchy)
- Security: Spring Security config, CSRF, CORS
- Concurrent access and thread safety

**Review Process:**
1. Run `git diff -- '*.java'` to see changes
2. Run `mvn verify -q` or `./gradlew check`
3. Review architecture layer violations manually

**Build Resolver (java-build-resolver):**
```bash
mvn compile -e      # Maven — see full error
./gradlew build --stacktrace  # Gradle — full stack
mvn dependency:tree  # Dependency conflict analysis
```

---

## kotlin-reviewer

**Purpose:** Kotlin and Android/KMP code reviewer for idiomatic patterns, coroutine safety, Compose best practices, and clean architecture.

**When to Use:** For all Kotlin code changes, including Android and Kotlin Multiplatform.

**Key Review Areas:**
- Idiomatic Kotlin (data classes, sealed classes, extension functions)
- Coroutine safety (dispatcher selection, structured concurrency)
- Jetpack Compose best practices (recomposition, remember, LaunchedEffect)
- Clean Architecture violations (leaking domain into UI)
- Null safety (avoid `!!` operator)
- Flow vs StateFlow/SharedFlow appropriate usage

**Build Resolver (kotlin-build-resolver):**
```bash
./gradlew build --stacktrace
./gradlew dependencies  # Dependency conflict resolution
./gradlew clean build   # Clean build to rule out cache issues
```

---

## rust-reviewer

**Purpose:** Expert Rust code reviewer specializing in ownership, lifetimes, error handling, unsafe usage, and idiomatic patterns.

**When to Use:** For all Rust code changes.

**Key Review Areas:**
- Ownership and borrowing correctness
- Lifetime annotations (when necessary vs can be elided)
- Error handling (`?` operator, `thiserror`/`anyhow` crates)
- `unsafe` blocks — justification and minimal scope
- Async correctness (`Send + Sync` bounds, `.await` placement)
- Performance: unnecessary `.clone()`, allocation patterns
- Idiomatic iterators vs manual loops

**Build Resolver (rust-build-resolver):**
```bash
cargo build 2>&1       # Full error output
cargo check            # Fast type checking without linking
cargo clippy -- -D warnings  # Linting
```

---

## cpp-reviewer

**Purpose:** C++ code reviewer specializing in modern C++ (C++17/20), memory safety, RAII, and performance.

**When to Use:** For all C++ code changes.

**Key Review Areas:**
- Memory management (RAII, smart pointers over raw)
- Modern C++ idioms (range-based for, auto, structured bindings)
- Undefined behavior (signed overflow, use-after-free, etc.)
- Exception safety (strong/basic/no-throw guarantees)
- Const correctness
- Thread safety and race conditions

**Build Resolver (cpp-build-resolver):**
```bash
cmake --build build 2>&1  # CMake build errors
make -j4 2>&1             # Make errors
clang-tidy src/ --      # Static analysis
```

---

## Common Pattern Across All Language Reviewers

Every language reviewer follows this workflow:

1. **Run diff** — See what changed in language-specific files
2. **Run language toolchain** — Build, lint, type-check
3. **Apply review checklist** — Language-specific issues
4. **Report findings** — Severity-organized, confidence-filtered

---

## Activation Pattern in ECC

Language reviewers are **activated based on the file extensions in the diff**:

| Files Changed | Activated Agent |
|---|---|
| `*.go` | go-reviewer |
| `*.py` | python-reviewer |
| `*.java` | java-reviewer |
| `*.kt`, `*.kts` | kotlin-reviewer |
| `*.rs` | rust-reviewer |
| `*.cpp`, `*.cc`, `*.cxx`, `*.h`, `*.hpp` | cpp-reviewer |
| `*.ts`, `*.tsx`, `*.js`, `*.jsx` | code-reviewer |

---

## Platform Adaptation

### GitHub Copilot
Add language-specific review guidelines to `copilot-instructions.md`. Supplement with language-specific GitHub Actions linters (golangci-lint, mypy, clippy, etc.).

### IntelliJ AI Assistant
IntelliJ has native language support for Java, Kotlin, Python, Go, and Rust. Create language-specific AI Actions that apply the ECC review checklist for each language.

### Monorepo Usage
Place language-specific reviewer guidance in each service subdirectory:
```
services/
├── api/               # Java Spring Boot
│   └── CLAUDE.md      # "Use java-reviewer for all changes"
├── worker/            # Go
│   └── CLAUDE.md      # "Use go-reviewer for all changes"
└── ml-service/        # Python
    └── CLAUDE.md      # "Use python-reviewer for all changes"
```

---

## Dependencies

All language reviewers:
- **Extend:** code-reviewer (language-specific issues layered on universal quality review)
- **Paired with:** language-specific build-resolver for their language
- **Report to:** security-reviewer for security-relevant findings
