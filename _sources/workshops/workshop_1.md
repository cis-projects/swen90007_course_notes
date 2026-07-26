# Workshop 1

The purpose of workshop 1 is to form teams and get to know the subject and your tutor.

## Today's Agenda

<!-- TODO: adjust timings to your tutorial length -->

| Time | Activity |
| --- | --- |
| 0–10 min | Welcome and meet your tutor |
| 10–25 min | Overview of the subject and the project/assessments |
| 25–50 min | Team formation |
| 50–end | Register team on Canvas, swap contacts, agree a weekly meeting time |

## Meet Your Tutor

Take a few minutes to introduce yourselves.

- Your tutor will introduce themselves and share how they prefer to be contacted.
- Go around the room and share your name, degree, and one thing you're hoping to get out of the subject.
<!-- TODO: add an icebreaker prompt if you'd like one -->

## Overview of SWEN90007

### Aims

One of the main challenges in developing enterprise-wide distributed systems is in choosing the right software architectures. In this subject, students will study software architectures in depth and the principles, techniques, and tools for creating, developing and evaluating software architectures.

### Indicative Content

Topics covered in this subject will be drawn from: design styles and architectural patterns; design strategies; domain specific architectures; evaluation of designs; architectural design for non-functional requirements; and modelling architectures.

### Learning Outcomes

On completion of this subject the student is expected to:

- Analyse large scale and distributed systems and select appropriate architectures for them
- Evaluate architectures both qualitatively and quantitatively
- Make suitable trade-offs between different architectures

## Assessments

This semester's project is to build an **Event Ticketing Platform** — an enterprise application where organisers publish events and attendees reserve and pay for seats. It is delivered in 4 parts (100 marks total), building on the same codebase across the semester.

::::{grid} 1 1 2 2
:gutter: 3

:::{grid-item-card} Part 1A · 8 marks
Domain model diagram and description (GitHub Wiki).
+++
**Due:** Week 3 — Fri 14 Aug 2026
:::

:::{grid-item-card} Part 1B · 12 marks
First use case deployed end-to-end + individual video.
+++
**Due:** Week 5 — Mon 24 Aug 2026
:::

:::{grid-item-card} Part 2 · 40 marks
Core patterns, functionality, observability. Oral demo in Week 9.
+++
**Due:** Week 9 — Mon 21 Sep 2026
:::

:::{grid-item-card} Part 3 · 40 marks
Concurrency, Unit of Work, performance. Oral demo in Week 12.
+++
**Due:** Week 12 — Mon 19 Oct 2026
:::
::::

Dates are authoritative from the spec but should be confirmed against the final Semester 2, 2026 academic calendar. **Only Part 1B is submitted to Canvas** (the individual videos) — everything else is assessed from your tagged GitHub release and Wiki. See the LMS / project specification for the complete requirements and marking rubrics.

```{admonition} Hurdle requirement
:class: warning
To pass the subject you must obtain **at least 50% in _each_ of the three project submissions**: Part 1 (1A + 1B combined, ≥10/20), Part 2 (≥20/40), and Part 3 (≥20/40). It is not enough to reach 50/100 overall.
```

### Key things to know about the project

::::{grid} 1 1 2 2
:gutter: 3

:::{grid-item-card} The application
An **Event Ticketing Platform** built as **two independently deployable services**: your main Java application, plus a small Payment Service the teaching team provides (you deploy and integrate it, but it isn't assessed for patterns).
:::

:::{grid-item-card} Tech constraints
**Java back-end, no heavyweight frameworks** — you implement the patterns (Controller, Data Mapper, Unit of Work, Identity Field, …) yourselves. **Spring Boot is _not_ permitted** (no DI / IoC / ORM); Spring Security may be used _in isolation_ for auth only. Front-end is your choice (JSP + React templates provided).
:::

:::{grid-item-card} Operate, don't just deploy
Modern practices are assessed too: **embedded Tomcat fat-JAR deployed to Render**, CI/CD with GitHub Actions, Infrastructure as Code (Terraform), observability with OpenTelemetry, and performance/concurrency testing with k6.
:::

:::{grid-item-card} Documentation
Lives in your **GitHub Wiki**, organised with a provided **arc42** skeleton and **C4** diagrams. Detailed design is in UML.
:::

:::{grid-item-card} Individual accountability
Assessed via the Part 1B video and the **interactive oral demos** in Parts 2 & 3 — you walk through your _own_ code, diagrams, and pipeline. A FeedbackFruits peer review runs before each deadline; individual marks can be scaled up or down.
:::

:::{grid-item-card} Integrity & GenAI
GenAI is **allowed but must be disclosed and documented** in your Wiki. Work is checked with Turnitin and MOSS; collaboration between teams is not allowed.
:::

:::{grid-item-card} GitHub Actions
**GitHub Actions minutes are a shared, finite pool** across the whole subject (~200 min/month per team), so wasteful CI can be raised at your oral assessment and, if the pool is exhausted, stops deployments for every team.
:::

:::{grid-item-card} Late penalty
The **late penalty is 10% per day** (or part thereof) on the release tag / deployment, unless an extension is in place.
:::
::::

## Semester timeline

Here's how the semester lays out — the four deliverables, the two oral-assessment weeks, and the mid-semester break.

| Week | Week of | Workshop / focus | Key dates |
| :---: | --- | --- | --- |
| **1** | 27 Jul | **Workshop 1** — teams, subject & project overview | Semester begins |
| **2** | 3 Aug | **Workshop 2** — domain modelling & diagram-as-code | GitHub org invitations sent |
| **3** | 10 Aug | **Workshop 3** — dev stack: Docker, PostgreSQL, Tomcat, Servlets/JSP, MVC, React | 📌 **Part 1A due** — Fri 14 Aug |
| **4** | 17 Aug | **Workshop 4** — arc42 & C4 documentation (Part 1B) | |
| **5** | 24 Aug | Project work — finish & deploy Part 1B | 📌 **Part 1B due** — Mon 24 Aug (+ individual videos to Canvas) |
| **6** | 31 Aug | Project work — Part 2 | |
| **7** | 7 Sep | Project work — Part 2 | |
| **8** | 14 Sep | **Workshop 7** — observability with OpenTelemetry | |
| **9** | 21 Sep | **Part 2 demonstrations** (interactive oral) | 📌 **Part 2 due** — Mon 21 Sep · demos this week |
| — | 28 Sep | 🏖️ **Mid-semester non-teaching week** | No classes |
| **10** | 5 Oct | Project work — Part 3 | |
| **11** | 12 Oct | **Workshop 8** — performance testing with k6 | |
| **12** | 19 Oct | **Part 3 demonstrations** (interactive oral) | 📌 **Part 3 due** — Mon 19 Oct · demos this week |

Teaching weeks and the non-teaching week above are **derived from the spec's due dates** and should be confirmed against the final Semester 2, 2026 academic calendar; the **due dates themselves are authoritative**. Workshops run every week — from Week 4 they're your team's dedicated project time with your tutor, and lectures introduce the patterns you'll need for each part before it is due.

## Tutorials

### Structure of Tutorials

Tutorials will primarily focus on group work. All subject material is introduced in lectures and tutorials are your chance to implement the content learned with your team with the support of your tutor. Please come to tutorials prepared to do work with your teammates and with questions for your tutor.

### Contacting Tutors

- **Ask all questions on the discussion board.** Other students learn from the answer, teaching staff don't answer the same question 100 times, and you'll get a quicker response.
- **Sharing code? Post a private question on the discussion board.** This is better than email — any staff member can pick it up, so you'll get a quicker response.
- **Avoid email** for subject questions where possible.

## Teams

Teams are **5 members, no exceptions**, and **everyone must be in the same workshop** — workshops are structured around team time with your tutor. Students without a team will be randomly allocated.

If you do not already have a group, we will use the remainder of the class to form teams.

If you already have a team, now is a good time to exchange emails or numbers and decide times outside of class to meet (prior to next week's tutorial so you can come prepared with questions).

Once your team is set, **submit your team details via the Team Registration survey on Canvas**. The teaching team will then create your private repository under the `SWEN90007-2026` GitHub Organisation — you'll receive an invitation in **Week 2** (do not create your own repositories).

### Team Meetings

You should meet weekly as a team. In project teams, weekly (and, in fact, we recommend more frequent than weekly) meetings are important opportunities to sync on team progress and a way to hold each other accountable.

### Team Reviews

For each assessment submission, you will be asked to review the contributions of each team member. Team members believed to not be equally contributing over the semester may have their grades penalised, so it is important to communicate with your team, and contribute equally.

### Task Tracking

Use a Kanban board to track and assign team tasks:

- [GitHub Projects](https://docs.github.com/en/issues/planning-and-tracking-with-projects/learning-about-projects/about-projects) — integrates with your repository's issues and pull requests.

### Collaborative Documentation

Your **GitHub Wiki** is the official home for your project documentation, and it is assessed at every submission. It comes with a provided **arc42** skeleton (with **C4** and UML diagrams); your team fills it in and keeps it current all semester — design decisions, architecture, and your weekly meeting minutes all live there. At each submission point you export the Wiki as a single PDF into your repository and create a release tag.

Your Wiki is created with your repository in Week 2. Until then, a shared space like Google Docs is fine for early drafting — but the assessed version must end up in the Wiki.

## Before Next Week

Make sure your team has:

- [ ] Formed a team of 5 (all in this workshop)
- [ ] Submitted the **Team Registration survey on Canvas**
- [ ] Exchanged contact details (email or phone)
- [ ] Agreed a regular weekly meeting time
- [ ] Read the project spec, especially the Part 1A requirements (due Week 3)
- [ ] Set up a shared space for collaborative documents
- [ ] Come to next week's tutorial with questions for your tutor

Once your Wiki exists, record **minutes for each workshop team meeting** in it — date/week, who attended, and a few bullet points on what the team did.
