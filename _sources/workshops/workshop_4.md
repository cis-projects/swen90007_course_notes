# Architectural Decisions Records (ADRs)

## Introduction

ADRs are a lightweight and effective tool at documenting the decisions behind the adoption of an architectural pattern in software projects. ADRs document each architectural decision and state the context, reasons, and consequences behind the adoption of that architectural decision.

Individually, an ADR represents one architectural decisions. Collectively, ADRs form a decision log for a project, providing necessary project context and detailed design and implementation guidance.

When decisions aren't documented, the result is one or many anti-patterns:

- Teams have the same conversations over and over, no decision is written down, no-one takes action, and nothing is implemented nor achieved.
- Decisions are made, but the teams don't understand the benefits or consequences of the decision.
- Decisions are made, but documents aren't preserved for posterity so others (now or in the future) attempting to understand your code can't.

## Example ADR

```{admonition} Bare Minimum ADR
The example provided below is just for illustration purposes and does not contain sufficient discussion nor justification.
Your team is free to make use of this template, but make sure it is more fully fleshed out. Your discussion should follow the Course Notes.

An ADR should be created for each pattern required for the assessment.
```

### Title

Implementing the Data Mapper Pattern for ABC Application

### Context

ABC application requires a robust mechanism to handle data persistence and retrieval operations for its social media application. Our current approach tightly couples business logic with data access code, meaning business logic is often written into SQL queries in the object classes. This means that there are often many almost identical functions created to cater for each business logic, making the system difficult to maintain and scale. As business logic increases in the future, complexity will increase. To address this, we need a pattern that removes business logic from the class objects and provides a clear separation of concerns.

### Decision

We will implement the Data Mapper pattern for the ABC application.

### Implementation Strategy

- Data Access Layer: Introduce a Data Access Layer (DAL) that contains mapper classes for the objects User, Post, and Action, which will be responsible for transferring data between their corresponding database tables and business objects.

### Consequences

#### Positive

- Scalability: Given the growth projection of ABC Corporation's social media application, we expect to extend the functionality over the coming years. By implementing the Data Mapper Pattern, we can recylce database query methods and minimise the requirements of writing new code.

#### Negative

- Learning Curve: Team members will need to familiarize themselves with the Data Mapper pattern and the new architecture.

### Alternatives Considered

*Discuss alternatives considered*

#### Compliance

- Any new object with a corresponding database table must have a mapper class created.

### Notes

| Author | Version | Changelog |
| --- | --- | --- |
| Jane Doe | 0.1 | Initial proposed version |

## References

[1] [AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/welcome.html)
[2] [ADR GitHub](https://adr.github.io)
