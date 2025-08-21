# Workshop 2

```{admonition} By Now You Should Have
:class: important
- Accepted the GitHub invitation.
- Created a team and repository in GitHub.
- Begun work on part 1 submission.
- Begun set up of development environment (see GitHub resources detailing steps to set up local environment).
```

```{admonition} Reminder
:class: important
You should be taking minutes of weekly group meetings.
```

```{admonition} Today's Workshop
Purpose of today's workshop is to continue working on part 1 of the project.
```

## GitHub tags

To submit part 1, there are a number of things you need to do:

- Save all documents for submission in your repository.
- Create a release tag *before the deadline*.
```{caution}
Teaching team can see when you create a tag and we will check to make sure it is before the deadline.
```
```{note}
To learn how to create a tag, please see GitHub documentation [here](https://docs.github.com/en/desktop/contributing-and-collaborating-using-github-desktop/managing-commits/managing-tags).
```

## Expected Knowledge

Software modelling and design is a prerequisite for this subject, so it is expected that students are comfortable and familiar with:

- object oriented programming, and
- creating use cases and domain models.

If you require more help, we recommend:

- course notes (available on LMS)
- Writing Effective Use Cases by Alistair Cockburn
- Applying UML Patterns by Craig Larman

## Diagrams

To aid you in artefacts for part 1, today we will learn Markdown syntax for
creating diagrams.

There are a number of different tools:

- [PlantUML](https://plantuml.com)
- [MermaidUML](https://mermaid-js.github.io/mermaid/#/)

```{tip}
MermaidUML is JavaScript based so good for those with JavaScript knowledge.
```

They allow creating the following diagrams:

- Sequence diagram
- Use case diagram
- Class diagram
- Object diagram
- Activity diagram
- Component diagram
- Deployment diagram
- State diagram
- Timing diagram

### Set Up PlantUML

First, you will need to install [GraphViz](https://graphviz.org) - an open source graph visualisation software 
that your computer will use to convert the text into graphics.

Second, install the [PlantUML for IntelliJ plugin](https://plugins.jetbrains.com/plugin/7017-plantuml-integration).

Third, open IntelliJ and create a new *.puml file:

![](resources/new_file.png)

![](resources/puml.png)

Fourth, copy the below into a new markdown file.

::::{admonition} Domain Model *.puml File
:class: dropdown

```
@startuml

Exam "1" -- "1..*" Question : contains >
Exam "1" -- "*" Grade : relates to >
Submissions "*" -- "*" Question : answers >
User "1" -- "*" Exam : creates >
User "1..*" -- "0..*" Subject : teaches >
User "1..*" -- "0..*" Subject : takes >
User "*" -- "1" Submissions : submits >
Subject "1" -- "0..*" Exam : includes >
Submissions "0..*" -- "1" Exam : belongs to >
Question <|-- MultipleChoiceQuestion
MultipleChoiceQuestion "1" -- "2..*" Option : includes >

class User {
  id
  firstName
  lastName
  email
}

class Grade {
  marks
}

class User {
}

class User {
}

class User {
}

class Subject {
  id
  name
  code
  year
}

class Exam {
  id
  title
}

class Question {
}

class MultipleChoiceQuestion {
}

class Option {
}

class Submissions {
}

@enduml
```
::::

```{tip}
Click [here](https://plantuml.com/class-diagram) to learn more about class diagrams in PlantUML.
```
