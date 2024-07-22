# Workshop 4

## Architectural Decisions Records (ADRs)

### Introduction

ADRs are a lightweight and effective tool at documenting the decisions behind the adoption of an architectural pattern in software projects. ADRs document each architectural decision and state the context, reasons, and consequences behind the adoption of that architectural decision.

Individually, an ADR represents one architectural decisions. Collectively, ADRs form a decision log for a project, providing necessary project context and detailed design and implementation guidance.

When decisions aren't documented, the result is one or many anti-patterns:

- Teams have the same conversations over and over, no decision is written down, no-one takes action, and nothing is implemented nor achieved.
- Decisions are made, but the teams don't understand the benefits or consequences of the decision.
- Decisions are made, but documents aren't preserved for posterity so others (now or in the future) attempting to understand your code can't.

### References

[1] [AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/welcome.html)
[2] [ADR GitHub](https://adr.github.io)

```{admonition} ADR Templates
The examples provided below are just for illustration purposes and does not contain sufficient discussion nor justification.
```

### ADR - Part 2

#### Title

Implementing the Data Mapper Pattern for ABC Application

#### Context

ABC application requires a robust mechanism to handle data persistence and retrieval operations for its social media application. Our current approach tightly couples business logic with data access code, meaning business logic is often written into SQL queries in the object classes. This means that there are often many almost identical functions created to cater for each business logic, making the system difficult to maintain and scale. As business logic increases in the future, complexity will increase. To address this, we need a pattern that removes business logic from the class objects and provides a clear separation of concerns.

#### Decision

We will implement the Data Mapper pattern for the ABC application.

#### Implementation Strategy

*Make use of diagrams here*

- Data Access Layer: Introduce a Data Access Layer (DAL) that contains mapper classes for the objects User, Post, and Action, which will be responsible for transferring data between their corresponding database tables and business objects.

#### Consequences

##### Positive

- Scalability: Given the growth projection of ABC Corporation's social media application, we expect to extend the functionality over the coming years. By implementing the Data Mapper Pattern, we can recylce database query methods and minimise the requirements of writing new code.

##### Negative

- Complexity: Data Mapper Pattern adds complexity, as it requires separation of logic between domain model objects.

##### Compliance

- Any new object with a corresponding database table must have a mapper class created.

#### Alternatives Considered

##### Active Record Pattern

###### Consequences

It was decided by the team to not adopt the Active Record Pattern. While the pattern had many positive reasons for adoption, it was ultimately decided that the lower complexity and development time did not outweigh our desire and need for future scalability. Knowing that the Active Record Pattern would become a hinderance to the development of further functionality in the future, it was decided against.

###### Positive 

- Complexity: Low complexity and development time to implement.

###### Negative

- Scalability: It does not promote scalability in the way Data Mapper Pattern does.

#### Notes

| Author | Version | Changelog |
| --- | --- | --- |
| Jane Doe | 0.1 | Initial proposed version |

### ADR - Part 3

#### Title

Implementing the Optimistic Locking for concurrency issue 1.

#### Context

Concurrency issue 1: Students simultaneously RSVP to an event.



Our application requires concurrency controls for situations where multipple students RSVP to the same event simultaneously.

#### Decision

We will implement optimistic locking for this use case...

#### Implementation Strategy

*Make use of diagrams here*

- Data Access Layer: Requires creation of data locking classes for each object...

#### Consequences

##### Positive

- Complexity: It is less complex than...

##### Negative

- Loss of data: It may frustrate users due to...

##### Compliance

- Any new object with a corresponding database table must have a mapper class created.

#### Alternatives Considered

##### Pessimistic Locking

###### Consequences

###### Positive 

###### Negative

#### Notes

| Author | Version | Changelog |
| --- | --- | --- |
| Jane Doe | 0.1 | Initial proposed version |
