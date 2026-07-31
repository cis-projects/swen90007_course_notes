# Workshop 6

This workshop introduces continuous integration and continuous delivery (CI/CD), and walks you through building a pipeline for your team's repository with [GitHub Actions](https://docs.github.com/en/actions).

Specifically, it covers:

- why teams automate build, test and deployment steps
- the anatomy of a GitHub Actions workflow
- a taxonomy of pipeline stages - what a pipeline should actually do
- a hands-on, step-by-step build of a pipeline in your own repository
- a quick reference of complete workflows for common technology stacks

```{admonition} By the end of this workshop
:class: important
Your team repository should have a working CI/CD pipeline that automatically checks code quality, runs tests, and (optionally) deploys your application every time someone pushes code or opens a pull request.
```

Your pipeline is a living part of your project. Start with the basics today and evolve it as the project grows - add end-to-end tests, security scanning, or automated releases when you're ready.

## Why CI/CD?

Without automation, teams rely on manual processes and good intentions: *did you run the tests before merging?* *Is the formatting consistent?* *Does the build still work?* These questions lead to inconsistency, broken main branches, and stressful deployments. CI/CD replaces the questions with checks that run the same way every time, for everyone.

| Term | What it means | Analogy |
| --- | --- | --- |
| **Continuous Integration (CI)** | Automatically build, lint, and test code on every push or pull request. | A spell-checker that runs every time you save a document. |
| **Continuous Delivery (CD)** | Automatically prepare a release-ready artifact after CI passes. | The document is formatted and ready to print at any moment. |
| **Continuous Deployment** | Automatically deploy to production when CI/CD passes on the main branch. | The document is printed and mailed the moment it's approved. |

## Anatomy of a GitHub Actions workflow

Every pipeline lives in a YAML file inside `.github/workflows/`. Here is the skeleton:

```yaml
name: CI                         # Display name in the Actions tab

on:                              # Trigger events
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:                            # One or more jobs (run in parallel by default)
  build:
    runs-on: ubuntu-latest       # The virtual machine type
    steps:
      - uses: actions/checkout@v4          # Step 1: clone the repo
      - name: Run a command                # Step 2: any shell command
        run: echo "Pipeline is running!"
```

### Key concepts

| Concept | Explanation |
| --- | --- |
| **Workflow** | The entire YAML file. A repository can have multiple workflows. |
| **Trigger (`on`)** | When the workflow runs: `push`, `pull_request`, `schedule`, `workflow_dispatch` (manual), etc. |
| **Job** | A unit of work that runs on a fresh virtual machine. Jobs run in parallel unless you add `needs:`. |
| **Step** | A single task inside a job - either a shell command (`run:`) or a reusable action (`uses:`). |
| **Action** | A reusable, community-maintained step (e.g. `actions/checkout@v4`). Browse them at the [GitHub Marketplace](https://github.com/marketplace?type=actions). |
| **Artifact** | A file produced by a job (e.g. a build output or test report) that can be downloaded or passed to another job. |

## Pipeline taxonomy - what should your pipeline do?

A well-designed pipeline is made up of **stages**, each with a clear purpose. Below is a taxonomy of common stages; you do not need all of them on day one - start small and grow.

::::{grid} 1 1 2 2
:gutter: 3

:::{grid-item-card}
:class-card: sd-shadow-sm sd-rounded-3 sd-border-0
:class-header: sd-bg-secondary sd-text-white sd-font-weight-bold

1. Code quality
^^^
Linting, format checks, static analysis.
:::

:::{grid-item-card}
:class-card: sd-shadow-sm sd-rounded-3 sd-border-0
:class-header: sd-bg-secondary sd-text-white sd-font-weight-bold

2. Test
^^^
Unit tests, integration tests, coverage.
:::

:::{grid-item-card}
:class-card: sd-shadow-sm sd-rounded-3 sd-border-0
:class-header: sd-bg-secondary sd-text-white sd-font-weight-bold

3. Build
^^^
Compile, bundle, build a Docker image, upload the artifact.
:::

:::{grid-item-card}
:class-card: sd-shadow-sm sd-rounded-3 sd-border-0
:class-header: sd-bg-secondary sd-text-white sd-font-weight-bold

4. Review gates
^^^
Require passing checks on pull requests, auto-assign reviewers.
:::

:::{grid-item-card}
:class-card: sd-shadow-sm sd-rounded-3 sd-border-0
:class-header: sd-bg-secondary sd-text-white sd-font-weight-bold

5. Deploy (CD)
^^^
Staging preview, production deploy, rollback strategy.
:::
::::

Stages 1 to 4 typically run on **every push and pull request**; stage 5 usually runs on the **main branch only**.

### Code quality - "is the code clean?"

**Purpose:** enforce consistent style and catch common mistakes before anyone reviews the code.

| Tool | Language / stack | What it does | GitHub Action |
| --- | --- | --- | --- |
| **ESLint** | JavaScript / TypeScript | Linting: catches bugs, enforces style rules | `eslint` via `npm` / `npx` |
| **Prettier** | JS / TS / CSS / HTML / Markdown | Formatting: consistent whitespace, quotes, etc. | `npx prettier --check .` |
| **Flake8** | Python | PEP 8 linting | `pip install flake8 && flake8 .` |
| **Black** | Python | Opinionated auto-formatter | `pip install black && black --check .` |
| **Ruff** | Python | Very fast linter and formatter (replaces Flake8 + Black) | `pip install ruff && ruff check .` |
| **Checkstyle** | Java | Style and convention checker | [checkstyle/checkstyle-action](https://github.com/checkstyle/checkstyle-action) |
| **Spotless** | Java / Kotlin (Gradle) | Formatting via a Gradle plugin | `./gradlew spotlessCheck` |
| **SwiftLint** | Swift (iOS) | Swift style enforcement | [norio-nomura/action-swiftlint](https://github.com/norio-nomura/action-swiftlint) |

### Automated testing - "does the code work?"

**Purpose:** run your test suite automatically so broken code never reaches the main branch.

| Test type | When to use | Examples |
| --- | --- | --- |
| **Unit tests** | Test individual functions or classes in isolation. *Every team should have these.* | `pytest`, `jest`, JUnit |
| **Integration tests** | Test how multiple components work together (e.g. API + database). | `supertest` (Node), `pytest` with fixtures, Spring Boot `@SpringBootTest` |
| **End-to-end (E2E) tests** | Simulate real user interaction through a browser. *Optional but powerful.* | Cypress, Playwright, Selenium |

**Code coverage** measures what percentage of your code is exercised by tests. Consider adding a coverage report to track progress over time.

| Coverage tool | Stack | GitHub Action / command |
| --- | --- | --- |
| **coverage.py + pytest-cov** | Python | `pytest --cov=src --cov-report=xml` |
| **Jest (built-in)** | JS / TS | `npx jest --coverage` |
| **JaCoCo** | Java | Gradle/Maven plugin, generates `jacoco.xml` |

### Build verification - "does the project compile and bundle?"

**Purpose:** make sure the project can be successfully built from a clean environment - not just on your laptop.

| Task | Example command | Why it matters |
| --- | --- | --- |
| Compile / transpile | `npm run build`, `./gradlew build`, `dotnet build` | Catches missing dependencies or type errors. |
| Docker image build | `docker build -t myapp .` | Ensures the container can be assembled. |
| Upload artifact | `actions/upload-artifact@v4` | Saves the build output for deployment or inspection. |

### Review gates - "is this pull request safe to merge?"

**Purpose:** use GitHub features to prevent merging code that hasn't passed your pipeline.

| Feature | How to set it up |
| --- | --- |
| **Required status checks** | Settings → Branches → Branch protection rules → Require status checks to pass. Select your CI job names. |
| **Required reviews** | Same page → Require approvals (set to 1 or 2). |
| **Auto-assign reviewers** | Use the [auto-assign action](https://github.com/kentaro-m/auto-assign-action) or GitHub's built-in `CODEOWNERS` file. |
| **Pull request labelling** | Use [actions/labeler](https://github.com/actions/labeler) to auto-label pull requests by file path. |

### Continuous deployment - "ship it automatically"

**Purpose:** once CI passes on the main branch, deploy to a live environment without manual steps.

| Platform | Action / approach | Notes |
| --- | --- | --- |
| **Vercel** | Automatic on push (zero config for Next.js / React) | Also generates deploy previews on pull requests. |
| **Netlify** | Automatic on push (similar to Vercel) | Great for static sites and SPAs. |
| **Railway / Render** | Automatic deploys from a connected branch | Good for full-stack apps with databases. |
| **AWS (S3 + CloudFront)** | [aws-actions/configure-aws-credentials](https://github.com/aws-actions/configure-aws-credentials) | For static hosting on AWS. |
| **Docker → Cloud Run / ECS** | Build image → push to registry → deploy | More advanced; a good stretch goal. |
| **GitHub Pages** | [actions/deploy-pages](https://github.com/actions/deploy-pages) | Free hosting for static sites and docs. |

```{admonition} Choosing a platform
:class: tip
If your client has not specified a hosting platform, Vercel, Netlify or Railway are the easiest starting points and have generous free tiers.
```

## Hands-on: build your pipeline step by step

Work through the steps below in your team repository, adapting the snippets to your technology stack using the tables above and the complete workflows in **Quick reference**, later in this workshop.

### Step 1 - Hello World workflow

**Goal:** verify that GitHub Actions is working in your repository.

1. Create the folder `.github/workflows/` in your repository.
2. Add a file called `ci.yml` with the following content:

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Hello CI
        run: echo "CI pipeline is running on branch ${{ github.ref_name }}"
```

3. Commit and push.

```{admonition} Checkpoint
:class: note
Open the **Actions** tab on GitHub. You should see a green check mark and the log message.
```

### Step 2 - Code quality (linting and formatting)

**Goal:** add automated style and quality checks to your pipeline.

Pick the snippet that matches your stack, or adapt one from the taxonomy tables above.

::::{tab-set}

:::{tab-item} JavaScript / TypeScript

```yaml
jobs:
  lint:
    name: Lint & Format Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - name: Run ESLint
        run: npx eslint . --ext .js,.jsx,.ts,.tsx
      - name: Check Prettier formatting
        run: npx prettier --check .
```

**Prerequisite:** make sure `eslint` and `prettier` are in your `devDependencies` and that you have config files (`.eslintrc.*`, `.prettierrc`).
:::

:::{tab-item} Python

```yaml
jobs:
  lint:
    name: Lint & Format Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install ruff
      - name: Run Ruff linter
        run: ruff check .
      - name: Check Ruff formatting
        run: ruff format --check .
```

Ruff combines linting and formatting in a single, very fast tool.
:::

:::{tab-item} Java

```yaml
jobs:
  lint:
    name: Checkstyle
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
      - name: Run Checkstyle via Gradle
        run: ./gradlew checkstyleMain checkstyleTest
```

**Prerequisite:** add the Checkstyle plugin to your `build.gradle` and include a `checkstyle.xml` config file.
:::

::::

**Try it:**

- [ ] Push code with a deliberate style violation, and watch the pipeline **fail**.
- [ ] Fix the violation, push again, and watch the pipeline turn **green**.

### Step 3 - Automated testing

**Goal:** run your test suite on every push and pull request.

::::{tab-set}

:::{tab-item} JavaScript / TypeScript

```yaml
jobs:
  test:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - name: Run tests
        run: npx jest --ci --coverage
```
:::

:::{tab-item} Python

```yaml
jobs:
  test:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements.txt
      - name: Run tests with coverage
        run: pytest --cov=src --cov-report=term-missing
```
:::

:::{tab-item} Java

```yaml
jobs:
  test:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
      - name: Run tests
        run: ./gradlew test
```
:::

::::

**Try it:**

- [ ] Write a simple test that passes, push, and watch the pipeline turn green.
- [ ] Break the test intentionally, push, and watch it turn red.
- [ ] Fix it, push, and watch it turn green again.

### Step 4 - Build verification

**Goal:** ensure your project compiles or bundles successfully in a clean environment.

::::{tab-set}

:::{tab-item} JavaScript / TypeScript

```yaml
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run build
```
:::

:::{tab-item} Java

```yaml
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
      - name: Build with Gradle
        run: ./gradlew build -x test   # skip tests here; they ran in the test job
```
:::

:::{tab-item} Docker

```yaml
jobs:
  build:
    name: Docker Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
```
:::

::::

### Step 5 - Branch protection and pull request checks

**Goal:** prevent merging code that hasn't passed the pipeline.

This step is done in **GitHub Settings**, not in a YAML file:

1. Go to your repository → **Settings** → **Branches**.
2. Click **Add branch protection rule**.
3. Set **Branch name pattern** to `main` (and optionally `develop`).
4. Enable the following:
    - **Require a pull request before merging**
    - **Require approvals** - set to at least 1
    - **Require status checks to pass before merging** - search for and select your CI job names (e.g. `lint`, `test`, `build`)
    - **Require branches to be up to date before merging**
5. Click **Create** / **Save changes**.

```{admonition} Checkpoint
:class: note
Create a pull request with a failing test. The merge button should be blocked with a message indicating that the required checks have not passed.
```

### Step 6 - Continuous deployment (optional)

**Goal:** automatically deploy your app when code is merged to `main`.

::::{tab-set}

:::{tab-item} GitHub Pages

For static sites and docs:

```yaml
  deploy:
    name: Deploy to GitHub Pages
    needs: [lint, test, build]      # only deploy after CI passes
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci && npm run build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist          # adjust to your build output folder
      - uses: actions/deploy-pages@v4
```
:::

:::{tab-item} Vercel

Vercel connects directly to your GitHub repository and deploys automatically - no YAML needed. It also creates **preview deployments** on every pull request.

1. Go to [vercel.com](https://vercel.com/) and import your GitHub repository.
2. Vercel will auto-detect your framework and deploy.
3. Every pull request gets a unique preview URL for testing.
:::

:::{tab-item} Railway

Similar to Vercel, Railway connects to your repository and auto-deploys. It supports backend services, databases, and background workers.

1. Go to [railway.app](https://railway.app/) → New Project → Deploy from GitHub repo.
2. Configure environment variables in the Railway dashboard.
:::

::::

```{admonition} Deployment requirements
:class: caution
If your client has specific deployment requirements, follow those. The options above are suggestions for teams choosing their own hosting.
```

## Quick reference: complete workflows

Below is a consolidated reference for common GitHub Actions configurations - copy the relevant pieces into your `ci.yml`.

::::{admonition} Full workflow - Node.js
:class: dropdown

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    name: Lint & Format
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npx eslint . --ext .js,.jsx,.ts,.tsx
      - run: npx prettier --check .

  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npx jest --ci --coverage

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run build
```
::::

::::{admonition} Full workflow - Python
:class: dropdown

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    name: Lint & Format
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install ruff
      - run: ruff check .
      - run: ruff format --check .

  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements.txt
      - run: pytest --cov=src --cov-report=term-missing

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements.txt
      - run: python -m py_compile src/main.py   # adjust to your entry point
```
::::

::::{admonition} Full workflow - Java / Gradle
:class: dropdown

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    name: Checkstyle
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
      - run: ./gradlew checkstyleMain checkstyleTest

  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
      - run: ./gradlew test

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
      - run: ./gradlew build -x test
```
::::

## Expected outcome

By the end of this workshop, your team repository should have:

- [ ] A `.github/workflows/ci.yml` file committed to your repository.
- [ ] A **code quality stage** - a linter and/or formatter runs on every push and pull request.
- [ ] A **test stage** - at least one unit test runs automatically.
- [ ] A **build stage** - the project compiles or bundles in a clean CI environment.
- [ ] **Branch protection** - `main` requires passing CI checks before merge.
- [ ] Pull requests showing green checks or red crosses, providing instant feedback.
- [ ] *(Stretch goal)* Continuous deployment to a preview or staging environment.

Your repository should end up structured something like this:

```text
your-repo/
├── .github/
│   └── workflows/
│       └── ci.yml              ← your pipeline definition
├── src/                        ← source code
├── tests/                      ← test files
├── package.json / requirements.txt / build.gradle   ← dependencies
└── README.md
```

## Further reading

### Official documentation

| Resource | Link |
| --- | --- |
| GitHub Actions documentation | <https://docs.github.com/en/actions> |
| Workflow syntax reference | <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions> |
| GitHub Marketplace (Actions) | <https://github.com/marketplace?type=actions> |

### Stack-specific guides

| Stack | Guide |
| --- | --- |
| Python | <https://realpython.com/github-actions-python/> |
| Java / Spring Boot | <https://kscodes.com/spring-boot/spring-boot-ci-cd-with-github-actions/> |
| React / Next.js | <https://nextjs.org/docs/app/building-your-application/deploying> |
| Flutter / mobile | <https://docs.flutter.dev/deployment/cd#github-actions> |

### Example repositories

| Repository | Description |
| --- | --- |
| <https://github.com/actions/starter-workflows> | GitHub's official starter workflow templates for many languages |
| <https://github.com/alexmalins/github-actions-cicd-example> | A simple Python CI/CD example |
