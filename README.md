# Project Setup Playbook (Arch + VS Code)

This document is a repeatable, “error-proof” setup guide per project type.

**Principles**
- Pin tool versions per project.
- Commit lockfiles.
- Provide one-command install/lint/test/run.
- Use **Dev Containers** when you need maximum reproducibility or multiple runtime versions.

---

## Table of Contents

- [0. Universal Checklist (any project)](#0-universal-checklist-any-project)
  - [Baseline files](#baseline-files)
  - [.editorconfig](#editorconfig)
  - [VS Code workspace settings](#vs-code-workspace-settings)
- [1. JS / TS (Node) template (Volta + npm)](#1-js--ts-node-template-volta--npm)
  - [Create project](#create-project)
  - [Pin Node + npm (Volta)](#pin-node--npm-volta)
  - [TypeScript (optional)](#typescript-optional)
  - [Lint / format](#lint--format)
  - [Recommended scripts](#recommended-scripts)
  - [Files to commit](#files-to-commit)
  - [Run checklist](#run-checklist)
  - [VS Code additions](#vs-code-additions)
- [2. Python template (uv)](#2-python-template-uv)
  - [Create project](#create-project-1)
  - [Add dependencies](#add-dependencies)
  - [Recommended pyproject settings](#recommended-pyproject-settings)
  - [Files to commit](#files-to-commit-1)
  - [Run checklist](#run-checklist-1)
  - [VS Code additions](#vs-code-additions-1)
- [3. Java template (Gradle wrapper or Maven wrapper)](#3-java-template-gradle-wrapper-or-maven-wrapper)
  - [Option A: Gradle](#option-a-gradle)
  - [Option B: Maven](#option-b-maven)
  - [VS Code notes](#vs-code-notes)
- [4. C# / .NET template (global.json + props)](#4-c--net-template-globaljson--props)
  - [Create project](#create-project-2)
  - [Pin SDK (global.json)](#pin-sdk-globaljson)
  - [Repository-wide defaults (Directory.Build.props)](#repository-wide-defaults-directorybuildprops)
  - [Run checklist](#run-checklist-2)
- [5. PHP template (Composer)](#5-php-template-composer)
  - [Create project](#create-project-3)
  - [Recommended structure](#recommended-structure)
  - [Autoloading (PSR-4)](#autoloading-psr-4)
  - [QA tools (optional)](#qa-tools-optional)
  - [Run checklist](#run-checklist-3)
- [6. Dev Containers (maximum reproducibility)](#6-dev-containers-maximum-reproducibility)
  - [When to use](#when-to-use)
  - [Minimal devcontainer.json](#minimal-devcontainerjson)
  - [Per-language notes](#per-language-notes)
- [7. Commit Checklist by language](#7-commit-checklist-by-language)
- [8. README contract (every repo)](#8-readme-contract-every-repo)

---

## 0. Universal Checklist (any project)

### Baseline files
Create these early:
- `README.md`
- `.editorconfig`
- `.gitignore`
- `.vscode/settings.json`

### .editorconfig
Copy as-is:

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
indent_style = space
indent_size = 2
trim_trailing_whitespace = true

[*.{py,cs,java,php}]
indent_size = 4

[*.md]
trim_trailing_whitespace = false
```

### VS Code workspace settings
Create `.vscode/settings.json`:

```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll": "explicit",
    "source.organizeImports": "explicit"
  },
  "files.insertFinalNewline": true,
  "files.trimTrailingWhitespace": true
}
```

---

## 1. JS / TS (Node) template (Volta + npm)

### Create project
```bash
mkdir my-node-app && cd my-node-app
git init
npm init -y
```

### Pin Node + npm (Volta)
```bash
volta pin node@lts npm@latest
```
This writes a `volta` section into `package.json`. Commit it.

### TypeScript (optional)
```bash
npm i -D typescript ts-node @types/node
npx tsc --init
```

### Lint / format
```bash
npm i -D eslint prettier eslint-config-prettier eslint-plugin-import
```

### Recommended scripts
In `package.json`:

```json
{
  "scripts": {
    "dev": "node src/index.js",
    "build": "echo "no build step"",
    "test": "echo "no tests" && exit 0",
    "lint": "eslint .",
    "format": "prettier . --write",
    "typecheck": "tsc -p tsconfig.json --noEmit"
  }
}
```

### Files to commit
- `package.json`
- lockfile (`package-lock.json` / `pnpm-lock.yaml` / `yarn.lock`)
- `tsconfig.json` (if TS)
- ESLint/Prettier config files (if you create them)

### Run checklist
```bash
npm ci
npm run lint
npm run typecheck   # TS only
npm run dev
```

### VS Code additions
Add to `.vscode/settings.json`:

```json
{
  "eslint.validate": ["javascript", "typescript"],
  "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```

---

## 2. Python template (uv)

### Create project
```bash
mkdir my-py-app && cd my-py-app
git init
uv init
uv venv
```

### Add dependencies
```bash
uv add requests
uv add --dev ruff pytest
```

### Recommended pyproject settings
In `pyproject.toml`:

```toml
[tool.ruff]
line-length = 100

[tool.ruff.format]
quote-style = "double"

[tool.pytest.ini_options]
testpaths = ["tests"]
```

### Files to commit
- `pyproject.toml`
- `uv.lock`
- `.venv/` (DO NOT commit; add to `.gitignore`)
- `src/` or your package folder
- `tests/`

### Run checklist
```bash
uv run ruff check .
uv run ruff format .
uv run pytest
uv run python -m your_module
```

### VS Code additions
`.vscode/settings.json`:

```json
{
  "python.defaultInterpreterPath": ".venv/bin/python",
  "python.analysis.typeCheckingMode": "basic",
  "editor.defaultFormatter": "charliermarsh.ruff"
}
```

---

## 3. Java template (Gradle wrapper or Maven wrapper)

### Option A: Gradle
```bash
mkdir my-java-app && cd my-java-app
git init
gradle init
./gradlew wrapper
```

Pin Java toolchain (example: 21) in `build.gradle(.kts)`.

**Groovy (`build.gradle`)**
```groovy
java {
  toolchain {
    languageVersion = JavaLanguageVersion.of(21)
  }
}
```

Run checklist:
```bash
./gradlew test
./gradlew run
```

Commit:
- `gradlew`, `gradlew.bat`, `gradle/wrapper/*`
- `build.gradle*`, `settings.gradle*`

### Option B: Maven
```bash
mkdir my-java-app && cd my-java-app
git init
mvn -q archetype:generate   -DgroupId=com.example   -DartifactId=my-java-app   -DarchetypeArtifactId=maven-archetype-quickstart   -DinteractiveMode=false
cd my-java-app
mvn -N wrapper:wrapper
```

Run checklist:
```bash
./mvnw test
./mvnw package
```

Commit:
- `mvnw`, `.mvn/wrapper/*`, `pom.xml`

### VS Code notes
Prefer running builds via `./gradlew ...` or `./mvnw ...` so the project controls tooling.

---

## 4. C# / .NET template (global.json + props)

### Create project
```bash
mkdir my-dotnet-app && cd my-dotnet-app
git init
dotnet new sln -n MySolution
dotnet new console -n MyApp
dotnet sln add MyApp/MyApp.csproj
```

### Pin SDK (global.json)
Check installed SDK version:
```bash
dotnet --version
```

Create `global.json` using that exact version:
```json
{
  "sdk": { "version": "10.0.100" }
}
```

### Repository-wide defaults (Directory.Build.props)
Create `Directory.Build.props` at repo root:

```xml
<Project>
  <PropertyGroup>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
    <LangVersion>latest</LangVersion>
  </PropertyGroup>
</Project>
```

### Run checklist
```bash
dotnet restore
dotnet build
dotnet test   # if you have tests
dotnet run --project MyApp
```

---

## 5. PHP template (Composer)

### Create project
```bash
mkdir my-php-app && cd my-php-app
git init
composer init
```

### Recommended structure
```
src/
tests/
composer.json
composer.lock
```

### Autoloading (PSR-4)
In `composer.json`:

```json
{
  "autoload": { "psr-4": { "App\\": "src/" } },
  "autoload-dev": { "psr-4": { "Tests\\": "tests/" } }
}
```

Then:
```bash
composer dump-autoload
```

### QA tools (optional)
```bash
composer require --dev phpunit/phpunit phpstan/phpstan friendsofphp/php-cs-fixer
```

Add scripts to `composer.json`:

```json
{
  "scripts": {
    "test": "phpunit",
    "lint": "phpstan analyse -l 6 src",
    "format": "php-cs-fixer fix"
  }
}
```

### Run checklist
```bash
composer install
composer run lint
composer run test
composer run format
```

---

## 6. Dev Containers (maximum reproducibility)

### When to use
Use a Dev Container when:
- You must pin a runtime version tightly (e.g., PHP 8.1 vs 8.3).
- You want identical environments across machines/teammates.
- You don’t want host updates to ever break builds.

### Minimal devcontainer.json
Create `.devcontainer/devcontainer.json`:

```json
{
  "name": "Dev Container",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {},
  "postCreateCommand": "echo 'ready'",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-azuretools.vscode-docker"
      ]
    }
  }
}
```

### Per-language notes
- Node: use a Node devcontainer image and still keep Volta pins in `package.json`.
- Python: use Python devcontainer image + `uv venv` in `postCreateCommand`.
- .NET: use a .NET devcontainer image + keep `global.json`.
- Java: use a Java devcontainer image + keep Gradle/Maven wrapper.
- PHP: devcontainers are the easiest way to manage multiple PHP versions reliably.

---

## 7. Commit Checklist by language

### JS/TS
- `package.json`
- lockfile
- `tsconfig.json` (TS)
- ESLint/Prettier config (if present)

### Python
- `pyproject.toml`
- `uv.lock`

### Java
- Gradle: `gradlew`, `gradle/wrapper/*`, `build.gradle*`, `settings.gradle*`
- Maven: `mvnw`, `.mvn/wrapper/*`, `pom.xml`

### .NET
- `global.json`
- `Directory.Build.props`
- solution + projects

### PHP
- `composer.json`
- `composer.lock`
- `src/`, `tests/`

---

## 8. README contract (every repo)

At the top of `README.md`, document these commands (and make sure they work):

- **Install / Restore**
- **Lint**
- **Test**
- **Run / Dev**

Example:

```text
Install:  <command>
Lint:     <command>
Test:     <command>
Run:      <command>
```
