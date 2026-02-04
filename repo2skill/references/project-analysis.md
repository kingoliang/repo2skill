# Project Analysis Guide

## General Analysis Steps

### 1. Identify Tech Stack

Check the following files by priority:

| Marker File | Tech Stack |
|-------------|------------|
| `pom.xml` | Java + Maven |
| `build.gradle` / `build.gradle.kts` | Java/Kotlin + Gradle |
| `package.json` | Node.js/TypeScript |
| `requirements.txt` / `pyproject.toml` / `setup.py` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `Gemfile` | Ruby |
| `composer.json` | PHP |
| `*.sln` / `*.csproj` | C#/.NET |

```
Glob "**/pom.xml"
Glob "**/package.json"
Glob "**/requirements.txt"
Glob "**/go.mod"
Glob "**/Cargo.toml"
```

Read found build files and extract:
- Language version
- Framework and version (Spring Boot, Express, Django, etc.)
- Key dependency list

### 2. Analyze Directory Structure

```bash
# Get top-level directory
ls -la {project-root}

# Get src directory structure (limited depth)
find {project-root}/src -type d -maxdepth 4 2>/dev/null | head -50
```

### 3. Identify Core Files

Search by file role:

#### API Layer
```
Glob "**/controller/**/*.java"
Glob "**/controllers/**/*.{ts,js}"
Glob "**/routes/**/*.{ts,js,py}"
Glob "**/views/**/*.py"
Glob "**/*Provider.java"
Glob "**/*Handler.{go,java}"
```

#### Business Layer
```
Glob "**/service/**/*.java"
Glob "**/services/**/*.{ts,js,py}"
Glob "**/usecases/**/*"
Glob "**/domain/**/*"
```

#### Data Layer
```
Glob "**/entity/**/*.java"
Glob "**/model/**/*.{java,ts,py}"
Glob "**/models/**/*.{ts,js,py}"
Glob "**/repository/**/*.java"
Glob "**/mapper/**/*.{java,xml}"
Glob "**/dao/**/*.java"
Glob "**/schema/**/*"
```

#### Configuration
```
Glob "**/config/**/*"
Glob "**/configuration/**/*"
Glob "**/*.yml"
Glob "**/*.yaml"
Glob "**/*.toml"
Glob "**/.env.example"
```

### 4. Analyze APIs

Read each API entry file and extract:
- Route/endpoint paths
- HTTP methods
- Request parameters (path params, query params, request body)
- Response structure
- Authentication requirements
- Called Service methods

### 5. Analyze Data Models

Read Entity/Model files and extract:
- Table/collection names
- Field names, types, constraints
- Relationships
- Index definitions

For Java projects, also check Mapper XML to confirm field mappings.

**Data source priority**: Entity classes > Mapper XML > DDL files

### 6. Extract Enums and Constants

```
Glob "**/enum/**/*.java"
Glob "**/enums/**/*.java"
Glob "**/constants/**/*.java"
Glob "**/types/**/*.ts"
Grep "export enum" --glob "*.ts"
Grep "class.*Enum" --glob "*.py"
```

### 7. Analyze Build and Development Workflow

Read the following files for development info:
- `Makefile` / `Taskfile.yml`
- `docker-compose.yml`
- `.github/workflows/*.yml`
- `scripts/` directory
- `README.md` (development-related sections)

---

## Tech Stack Specific Patterns

### Java + Spring Boot

```
# Entry class
Grep "@SpringBootApplication" --glob "*.java"

# REST controllers
Grep "@RestController" --glob "*.java"
Grep "@RequestMapping" --glob "*.java"

# Services
Grep "@Service" --glob "*.java"

# Data access
Grep "@Repository" --glob "*.java"
Grep "@Entity" --glob "*.java"
Grep "@Table" --glob "*.java"

# Configuration
Glob "**/application*.yml"
Glob "**/application*.properties"
```

### Node.js / TypeScript

```
Glob "**/index.ts"                              # Entry
Glob "**/app.ts"
Grep "router\." --glob "*.ts"                   # Routes
Grep "app\.(get|post|put|delete)" --glob "*.ts"
Grep "Schema\(" --glob "*.ts"                   # ORM
Grep "@Entity" --glob "*.ts"
```

### Python (Django/FastAPI/Flask)

```
Glob "**/urls.py"                               # Django routes
Glob "**/views.py"                              # Django views
Glob "**/models.py"                             # Models
Grep "@app\.(get|post|put|delete)" --glob "*.py" # FastAPI
Grep "@app\.route" --glob "*.py"                # Flask
```

### Go

```
Glob "**/main.go"                               # Entry
Grep "func.*Handler" --glob "*.go"              # Routes
Grep "type.*struct" --glob "*.go"               # Models
```
