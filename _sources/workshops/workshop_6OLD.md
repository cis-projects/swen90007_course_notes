# Workshop 6

This workshop introduces continuous integration and continuous delivery (CI/CD), and walks you through building a pipeline for your team's repository with [GitHub Actions](https://docs.github.com/en/actions).

Everything here targets the stack you're building on: a Java back-end built with Maven, packaged as a Docker image and deployed to Render. The pipeline below works the same whether your build produces a WAR (the setup in the course notes) or an executable JAR — only the artifact filename changes, and that is flagged where it matters.

Specifically, it covers:

- why teams automate build, test and deployment steps
- the anatomy of a GitHub Actions workflow
- a taxonomy of pipeline stages — what a pipeline should actually do
- how to build a pipeline that fits inside the subject's shared Actions minutes budget
- a hands-on, step-by-step build of that pipeline in your own repository

```{admonition} By the end of this session
:class: important
Your team repository should have a working CI/CD pipeline that automatically checks code quality, runs tests, and (optionally) deploys your application every time someone pushes code or opens a pull request.
```

```{admonition} Tip
:class: tip
Your CI/CD pipeline is a living part of your project. Start with the basics today and evolve it as your project grows — add integration tests, security scanning, or automated releases when you're ready.
```

## Why CI/CD?

### The problem

Without automation, teams rely on manual processes: "Did you run the tests before merging?" "Is the formatting consistent?" "Does the build still work?" These questions lead to inconsistency, broken main branches, and stressful deployments.

### The solution

| Term | What it means | Analogy |
| --- | --- | --- |
| **Continuous Integration (CI)** | Automatically build, lint, and test code on every push or pull request. | A spell-checker that runs every time you save a document. |
| **Continuous Delivery (CD)** | Automatically prepare a release-ready artifact after CI passes. | The document is formatted and ready to print at any moment. |
| **Continuous Deployment** | Automatically deploy to production when CI/CD passes on the main branch. | The document is printed and mailed the moment it's approved. |

There's a direct assessment argument too. Parts 2 and 3 are assessed from a **tagged release that is deployed and running**, and you demonstrate your pipeline at the interactive oral. A pipeline that builds and deploys the tag for you turns "did someone remember to redeploy before the deadline?" into a question nobody has to ask.

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
| **Workflow** | The entire YAML file. A repo can have multiple workflows. |
| **Trigger (`on`)** | When the workflow runs: `push`, `pull_request`, `schedule`, `workflow_dispatch` (manual), etc. |
| **Job** | A unit of work that runs on a fresh virtual machine. Jobs run in parallel unless you add `needs:`. |
| **Step** | A single task inside a job — either a shell command (`run:`) or a reusable action (`uses:`). |
| **Action** | A reusable, community-maintained step (e.g., `actions/checkout@v4`). Browse them at [GitHub Marketplace → Actions](https://github.com/marketplace?type=actions). |
| **Artifact** | A file produced by a job (e.g., a build output, test report) that can be downloaded or passed to another job. |

```{admonition} Every job starts from nothing
:class: note
A job runs on a **fresh** virtual machine. It has no JDK configured, no local Maven repository, and no copy of your code until you check it out. This is exactly why CI catches "works on my machine" problems — and, as the next section shows, it's also why every extra job costs you real time.
```

## Pipeline taxonomy — what should your pipeline do?

A well-designed pipeline is made up of **stages**, each with a clear purpose. Below is a taxonomy of common pipeline stages. You do not need all of them on day one — start small and grow.

::::{grid} 1 1 2 2
:gutter: 3

:::{grid-item-card}
:class-card: sd-shadow-sm sd-rounded-3 sd-border-0
:class-header: sd-bg-secondary sd-text-white sd-font-weight-bold

1. Code quality
^^^
Checkstyle, formatting, static analysis.
:::

:::{grid-item-card}
:class-card: sd-shadow-sm sd-rounded-3 sd-border-0
:class-header: sd-bg-secondary sd-text-white sd-font-weight-bold

2. Test
^^^
JUnit unit tests, integration tests, coverage.
:::

:::{grid-item-card}
:class-card: sd-shadow-sm sd-rounded-3 sd-border-0
:class-header: sd-bg-secondary sd-text-white sd-font-weight-bold

3. Build
^^^
Compile, package the deployable artifact, upload it.
:::

:::{grid-item-card}
:class-card: sd-shadow-sm sd-rounded-3 sd-border-0
:class-header: sd-bg-secondary sd-text-white sd-font-weight-bold

4. Review gates
^^^
Require passing checks on PRs, auto-assign reviewers.
:::

:::{grid-item-card}
:class-card: sd-shadow-sm sd-rounded-3 sd-border-0
:class-header: sd-bg-secondary sd-text-white sd-font-weight-bold

5. Deploy (CD)
^^^
Deploy to Render on merge, or on a release tag.
:::
::::

Stages 1 to 4 typically run on **every push and pull request**; stage 5 usually runs on the **main branch only**, or on a tag.

Conveniently, the first three map onto phases of the Maven build lifecycle, so a single `mvn verify` runs all of them in one pass. Keep that in mind as you read — it's the key to a cheap pipeline.

### Code quality — "is the code clean?"

**Purpose:** Enforce consistent style and catch common mistakes before anyone reviews the code.

| Tool | What it does | Maven goal |
| --- | --- | --- |
| **Checkstyle** | Style and convention checking against a rule set (Google or Sun checks make a good starting point) | `mvn -B checkstyle:check` |
| **Spotless** | Auto-formatting, with a `check` goal that fails CI when files aren't formatted | `mvn -B spotless:check` |
| **PMD** | Static analysis for common bad practice, dead code, and copy-paste | `mvn -B pmd:check` |
| **SpotBugs** | Bytecode analysis for likely bugs — null dereferences, unclosed resources, ignored return values | `mvn -B spotbugs:check` |

Pick **one** to start with. Checkstyle is the usual choice because the rules are easy to agree on and the failures are unambiguous. SpotBugs earns its place later in the semester — it's good at spotting the resource leaks that show up once you're managing your own JDBC connections.

```{admonition} Bind the check to a phase
:class: tip
If you bind these plugins to a lifecycle phase in your `pom.xml` (Checkstyle to `validate`, SpotBugs to `verify`) they run as part of `mvn verify` — no separate CI step, and the same command works on your laptop.
```

### Automated testing — "does the code work?"

**Purpose:** Run your test suite automatically so broken code never reaches the main branch.

| Test type | When to use | How it runs |
| --- | --- | --- |
| **Unit tests** | Test individual classes in isolation — mappers, domain logic, your Unit of Work. *Every team should have these.* | JUnit 5 via Surefire, `mvn -B test` |
| **Integration tests** | Test components together against a real PostgreSQL database. | JUnit 5 + [Testcontainers](https://java.testcontainers.org/) via Failsafe, named `*IT.java`, `mvn -B verify` |
| **Performance tests** | Measure throughput and latency under load (Part 3). | k6 — see Workshop 8. Run these on a schedule or manually, **not** on every push |

**Code coverage** measures what percentage of your code is exercised by tests. [JaCoCo](https://www.jacoco.org/jacoco/trunk/doc/maven.html) is the standard choice: bind `prepare-agent` and `report` in your `pom.xml` and a report appears in `target/site/jacoco/` on every `mvn verify`.

```{admonition} Testcontainers costs minutes
:class: caution
Testcontainers is excellent — it spins up a real PostgreSQL container so your mappers are tested against the database they'll actually run on. It also pulls a Docker image on every CI run. Keep integration tests in a separate Failsafe run and consider limiting them to pull requests against `main` rather than every push to a feature branch.
```

### Build verification — "does the project compile and package?"

**Purpose:** Make sure the project can be successfully built from a clean environment — not just on your laptop.

| Task | Command | Why it matters |
| --- | --- | --- |
| Compile and test | `mvn -B verify` | Catches missing dependencies, and anything that only compiles because of stale state in your IDE. |
| Package the artifact | mvn package | Produces the WAR (or executable JAR) that your Dockerfile copies into the image Render runs. If this breaks, you cannot deploy. | 
| Upload the artifact | actions/upload-artifact@v4 | Lets you download the exact artefact a run produced, which is invaluable when the deployed build misbehaves but your local one doesn't. |

### Review gates — "is this PR safe to merge?"

**Purpose:** Use GitHub features to prevent merging code that hasn't passed your pipeline.

| Feature | How to set it up |
| --- | --- |
| **Required status checks** | Settings → Branches → Branch protection rules → Require status checks to pass. Select your CI job names. |
| **Required reviews** | Same page → Require approvals (set to 1 or 2). |
| **Auto-assign reviewers** | Use the [auto-assign action](https://github.com/kentaro-m/auto-assign-action) or GitHub's built-in CODEOWNERS file. |
| **PR labelling** | Use [actions/labeler](https://github.com/actions/labeler) to auto-label PRs by file path. |

This is also where CI starts paying for itself as a *team* practice. With five people committing to one repository, a protected `main` is what stops a broken build from blocking everyone else's work an hour before a deadline.

### Continuous deployment — "ship it automatically"

**Purpose:** Once CI passes, get the new version onto Render without anyone clicking anything.

You have three options, and the cheapest one is probably the right one:

| Approach | Actions minutes | When to use it |
| --- | --- | --- |
| **Render auto-deploy on commit** | None — Render watches the branch itself | The sensible default while you're building |
| **Render auto-deploy after CI checks pass** | None | Once you have branch protection: Render waits for your green checks, then deploys |
| **Deploy hook called from a workflow** | A few seconds | When you need to deploy something Render can't detect on its own — most usefully, a **release tag** |

The second option is worth setting up properly: it's configured on the service's Settings page in the Render dashboard, and it means a red pipeline never reaches your deployed URL. See [Render's deploy documentation](https://render.com/docs/deploys) for the setting, and [deploy hooks](https://render.com/docs/deploy-hooks) for the third option.

## Your Actions minutes budget

```{admonition} A shared, finite pool
:class: warning
GitHub Actions minutes are a **shared pool across the whole subject** — roughly 200 minutes per team per month. Your repositories are private, so every minute a workflow runs is drawn from it. If the pool is exhausted, deployments stop **for every team in the subject**, and wasteful CI can be raised at your oral assessment.
```

This constraint is not an obstacle to good CI — it's the reason to write good CI. Consider a naive pipeline with three jobs (`lint`, `test`, `build`). Each one gets a fresh runner, so each one checks out the repository, installs a JDK, and downloads your entire dependency tree from Maven Central before doing any useful work. Call it three minutes per run. Five teammates pushing fifteen times a week comes to about 45 minutes a week — and you're out of minutes before the end of the month.

The same checks in a single cached job take well under a minute. Six levers, roughly in order of impact:

::::{grid} 1 1 2 2
:gutter: 3

:::{grid-item-card} 1. One job, not three
Maven's lifecycle already runs quality checks, tests and packaging in order. `mvn -B verify` does all three in one JVM on one runner — one checkout, one JDK setup, one dependency resolution.
:::

:::{grid-item-card} 2. Cache `~/.m2`
`actions/setup-java` will do it for you with `cache: maven`. Downloading dependencies is most of a cold build; caching turns minutes into seconds.
:::

:::{grid-item-card} 3. Cancel superseded runs
A `concurrency` group with `cancel-in-progress: true` stops the previous run when you push a fix thirty seconds later. Nobody needs CI results for a commit that's already been replaced.
:::

:::{grid-item-card} 4. Don't run twice per change
If you trigger on both `push` and `pull_request` for the same branch, a PR from a branch in your own repo runs the pipeline **twice**. Trigger `push` on `main` only, and let `pull_request` cover everything else.
:::

:::{grid-item-card} 5. Filter paths
Wiki edits, `README` changes and documentation don't need a Java build. `paths-ignore` skips the run entirely.
:::

:::{grid-item-card} 6. Let Render do the deploying
Render can watch your branch itself, at no cost in Actions minutes. Reserve workflow-triggered deploys for the cases Render can't handle, like tags.
:::
::::

You can see what you've spent under **Settings → Billing** on the organisation, and each run shows its duration in the Actions tab. Check it occasionally — a pipeline that quietly doubled in cost is much easier to fix in Week 6 than in Week 12.

## Hands-on: build your pipeline step by step

Work through the steps below in your team repository. Steps 1 to 5 build the pipeline up one stage at a time so you can see each part fail and pass; Step 6 then consolidates them into the single efficient job you'll actually keep.

### Step 1 — Hello World workflow

**Goal:** Verify that GitHub Actions is working in your repo.

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

```{admonition} ✅ Checkpoint
:class: note
Open the **Actions** tab on GitHub. You should see a green check mark and the log message.
```

### Step 2 — Code quality (Checkstyle)

**Goal:** Add automated style checking to your pipeline.

First, add the plugin to your `pom.xml` and bind it to the `validate` phase, so it runs early and fails fast:

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-checkstyle-plugin</artifactId>
  <version>3.6.0</version>
  <configuration>
    <configLocation>checkstyle.xml</configLocation>
    <consoleOutput>true</consoleOutput>
    <failOnViolation>true</failOnViolation>
    <violationSeverity>warning</violationSeverity>
  </configuration>
  <executions>
    <execution>
      <id>checkstyle-validate</id>
      <phase>validate</phase>
      <goals>
        <goal>check</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

Add a `checkstyle.xml` rule set at the root of the repository — starting from Google's or Sun's published checks and deleting the rules your team disagrees with is a perfectly respectable approach, and a faster route to consensus than writing one from scratch.

Then replace the `hello` job in `ci.yml`:

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
          cache: maven
      - name: Run Checkstyle
        run: mvn -B --no-transfer-progress checkstyle:check
```

```{admonition} What the flags do
:class: note
`-B` (batch mode) stops Maven emitting interactive progress, and `--no-transfer-progress` suppresses the download chatter. Together they turn thousands of lines of CI log into something you can actually read when a build fails.
```

**Try it:**

- [ ] Push code with a deliberate style violation → watch the pipeline **fail**.
- [ ] Fix the violation → push again → pipeline turns **green**.
- [ ] Look at the run time of the second run compared with the first. The Maven cache should have made it noticeably faster.

### Step 3 — Automated testing

**Goal:** Run your JUnit suite on every push and pull request.

```yaml
  test:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
          cache: maven
      - name: Run tests
        run: mvn -B --no-transfer-progress test
```

Surefire picks up anything named `*Test.java` automatically. If you've added Failsafe for integration tests (`*IT.java`), those run under `mvn verify` instead — which is what Step 6 will use.

**Try it:**

- [ ] Write a simple test that passes → push → green.
- [ ] Break the test intentionally → push → red.
- [ ] Fix it → push → green again.

### Step 4 — Build the deployable artifact

**Goal:** Ensure the deployable artifact builds in a clean environment.

Your application is packaged by Maven and then copied into a Docker image, which is what Render runs. On the setup in the course notes that artefact is a .war, produced by the default package phase. No extra plugin configuration is required.

:class: dropdown
You will need the Shade plugin to produce a single self-contained JAR. Everything else in this workshop is unchanged; substitute `*.jar` for `*.war` in the artifact paths below.

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-shade-plugin</artifactId>
  <version>3.6.0</version>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
        <goal>shade</goal>
      </goals>
      <configuration>
        <transformers>
          <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <mainClass>au.edu.unimelb.swen90007.Main</mainClass>
          </transformer>
        </transformers>
      </configuration>
    </execution>
  </executions>
</plugin>
```

And the job that builds and keeps it:

```yaml
  build:
    name: Package
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
          cache: maven
      - name: Package
        run: mvn -B --no-transfer-progress package -DskipTests
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-artifact
          path: target/*.war        # or target/*.jar if you build an executable JAR
          retention-days: 5
```

```{admonition} ✅ Checkpoint
:class: note
Download the artefact from the run summary page. It is the same file your Dockerfile copies into the image: build the image locally with docker build, run it, and what starts up is what Render will run.
```

### Step 5 — Branch protection & PR checks

**Goal:** Prevent merging code that hasn't passed the pipeline.

This step is done in **GitHub Settings**, not in a YAML file:

1. Go to your repository → **Settings** → **Branches**.
2. Click **Add branch protection rule**.
3. Set **Branch name pattern** to `main` (and optionally `develop`).
4. Enable the following:
    - ✅ **Require a pull request before merging**
    - ✅ **Require approvals** — set to at least 1
    - ✅ **Require status checks to pass before merging** — search for and select your CI job names
    - ✅ **Require branches to be up to date before merging**
5. Click **Create** / **Save changes**.

```{admonition} ✅ Checkpoint
:class: note
Create a PR with a failing test. You should see the merge button blocked with a message indicating the required checks have not passed.
```

### Step 6 — Consolidate into one job

**Goal:** Get the same coverage for a fraction of the minutes.

You now have three jobs doing three checkouts, three JDK installs and three dependency resolutions. Because Checkstyle is bound to `validate`, tests run at `test`, and Shade runs at `package`, a single `mvn verify` performs all of it in order — and stops at the first failure, exactly as the separate jobs did.

Replace the whole of `ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
    paths-ignore: ['**.md', 'docs/**']
  pull_request:
    paths-ignore: ['**.md', 'docs/**']

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  verify:
    name: Checkstyle, test & package
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
          cache: maven
      - name: Verify
        run: mvn -B --no-transfer-progress verify
      - name: Upload JAR
        uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: |
            target/*.jar
            !target/original-*.jar
          retention-days: 5
```

Note the trigger change: `push` now fires on `main` only, while `pull_request` covers every branch. Previously a PR from a branch in your own repository ran the pipeline twice for the same commit.

```{admonition} ✅ Checkpoint
:class: important
Compare the total run time against the three-job version in the Actions tab, then update your branch protection rule — the required status check is now `verify`, and your old job names no longer exist. **A protection rule pointing at a deleted job blocks every merge**, so fix this before you go home.
```

**Try it:**

- [ ] Push a change to a Markdown file only. No workflow should run at all.
- [ ] Push twice in quick succession. The first run should be cancelled automatically.

### Step 7 — (Optional) Deploy on a release tag

**Goal:** Have your submission tag deploy itself.

Your submissions are release tags of the form `SWEN90007_2026_Part1A_<team name>`. A workflow that deploys when such a tag is pushed means the deployed application and the tagged code can't drift apart.

First, create a deploy hook: in the Render dashboard, open your service → **Settings** → **Deploy Hook**, and copy the URL. Then add it to GitHub under **Settings → Secrets and variables → Actions → New repository secret**, named `RENDER_DEPLOY_HOOK_URL`.

```yaml
name: Deploy

on:
  push:
    tags: ['SWEN90007_2026_Part*']

jobs:
  deploy:
    name: Trigger Render deploy
    runs-on: ubuntu-latest
    steps:
      - name: Call deploy hook
        env:
          DEPLOY_HOOK: ${{ secrets.RENDER_DEPLOY_HOOK_URL }}
        run: curl -fsS -X POST "$DEPLOY_HOOK"
```

```{admonition} Anyone with the hook URL can deploy your service
:class: warning
The deploy hook needs no other authentication, so treat it like a password: it belongs in GitHub Secrets and nowhere else — not in `ci.yml`, not in your Wiki, not in a screenshot in your report. Passing it through `env:` rather than interpolating it directly into the `run:` line keeps it out of the workflow logs.
```

## Quick reference

::::{admonition} Complete `ci.yml` — Java / Maven
:class: dropdown

```yaml
name: CI

on:
  push:
    branches: [main]
    paths-ignore: ['**.md', 'docs/**']
  pull_request:
    paths-ignore: ['**.md', 'docs/**']

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  verify:
    name: Checkstyle, test & package
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
          cache: maven
      - name: Verify
        run: mvn -B --no-transfer-progress verify
      - name: Upload JAR
        uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: |
            target/*.jar
            !target/original-*.jar
          retention-days: 5
```
::::

::::{admonition} Additional job — React front-end
:class: dropdown

Only for teams that chose React. This assumes the front-end lives in a `frontend/` directory; the `paths` filter means it runs only when front-end code actually changes, and the Java job's `paths-ignore` can be extended with `'frontend/**'` so the two don't trigger each other.

```yaml
  frontend:
    name: Front-end lint, test & build
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    defaults:
      run:
        working-directory: frontend
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
          cache-dependency-path: frontend/package-lock.json
      - run: npm ci
      - run: npx eslint .
      - run: npm test -- --run
      - run: npm run build
```

JSP teams need nothing here — your views are part of the Maven build and are covered by `mvn verify`.
::::

::::{admonition} Deploy on tag — Render
:class: dropdown

```yaml
name: Deploy

on:
  push:
    tags: ['SWEN90007_2026_Part*']

jobs:
  deploy:
    name: Trigger Render deploy
    runs-on: ubuntu-latest
    steps:
      - name: Call deploy hook
        env:
          DEPLOY_HOOK: ${{ secrets.RENDER_DEPLOY_HOOK_URL }}
        run: curl -fsS -X POST "$DEPLOY_HOOK"
```
::::

```{admonition} Plugin versions
:class: note
The plugin versions in this workshop were current when it was written. Pin a version for every plugin — an unpinned plugin resolves differently over time and produces builds that fail for one teammate and not another — and check [Maven Central](https://central.sonatype.com) for the current release when you add one.
```

## Expected outcome & checklist

By the end of this workshop, your team repository should have:

- [ ] A `.github/workflows/ci.yml` file committed to your repo.
- [ ] **Code quality stage** — Checkstyle runs on every push/PR and fails the build on violations.
- [ ] **Test stage** — at least one JUnit test runs automatically.
- [ ] **Build stage** — the deployable artefact packages successfully in a clean CI environment.
- [ ] **Branch protection** — `main` requires passing CI checks before merge.
- [ ] **A consolidated job** with Maven caching, a `concurrency` group, and no duplicate `push`/`pull_request` runs.
- [ ] PRs show ✅ green checks or ❌ red crosses providing instant feedback.
- [ ] *(Stretch goal)* Deployment to Render on merge to `main` or on a release tag.

Record in your Wiki which of these you've done and why — the pipeline is part of what's assessed in Parts 2 and 3, and you'll be asked to walk through it at the oral.

### Expected repository structure

```text
your-repo/
├── .github/
│   └── workflows/
│       ├── ci.yml              ← your pipeline definition
│       └── deploy.yml          ← optional, deploy on tag
├── src/
│   ├── main/java/              ← application code
│   └── test/java/              ← JUnit tests
├── frontend/                   ← React teams only
├── checkstyle.xml              ← your agreed rule set
├── pom.xml                     ← dependencies and plugins
└── README.md
```

## Further reading

### Official documentation

| Resource | Link |
| --- | --- |
| GitHub Actions documentation | <https://docs.github.com/en/actions> |
| Workflow syntax reference | <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions> |
| `actions/setup-java` (including Maven caching) | <https://github.com/actions/setup-java> |
| GitHub Marketplace (Actions) | <https://github.com/marketplace?type=actions> |

### Maven plugins

| Plugin | Link |
| --- | --- |
| Checkstyle | <https://maven.apache.org/plugins/maven-checkstyle-plugin/> |
| Surefire (unit tests) | <https://maven.apache.org/surefire/maven-surefire-plugin/> |
| Failsafe (integration tests) | <https://maven.apache.org/surefire/maven-failsafe-plugin/> |
| Shade (fat JAR - optional) | <https://maven.apache.org/plugins/maven-shade-plugin/> |
| JaCoCo (coverage) | <https://www.jacoco.org/jacoco/trunk/doc/maven.html> |

### Deployment and testing

| Resource | Link |
| --- | --- |
| Render — deploying and auto-deploy settings | <https://render.com/docs/deploys> |
| Render — deploy hooks | <https://render.com/docs/deploy-hooks> |
| Testcontainers for Java | <https://java.testcontainers.org/> |
| GitHub's official starter workflows | <https://github.com/actions/starter-workflows> |
