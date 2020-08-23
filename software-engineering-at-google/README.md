# Software Engineering at Google

## Chapter 3 - Knowledge Sharing

- This chapter talks about how domain knowledge is collected and shared throughout Google.
- There are several problems that can be caused by a lack of knowledge sharing culture:
    **1. Lack of psychological safety**
        - People become afraid to take risks and make mistakes.
    **2. Information islands**
        - Knowledge becomes fragmented between developers/teams.
    **3. Single point of failure (SPOF)**
        - Only one person has information.
    **4. All-or-nothing expertise**
        - A split in a group of people where one side has all the knowledge and the other side has very little knowledge.
    **5. Parroting**
        - People use knowledge/information without really understanding it.
    **6. Haunted graveyards**
        - Casebases that people are afraid to touch due to a lack of knowledge.
        - Real world: Ansible Kafka jobs
- One-on-one knowledge sharing is useful, but it's important to be able to scale up knowledge. A team with only a
single expert on a particular subject may be put into a lurch when that particular expert goes on vacation or leaves
the team.Having a single expert on a particular subject.
- "Tribal knowledge exists in the gap between what individual team members know and what is documented."


## Chapter 6 - Leading at Scale

- "The three Always of leadership": Always Be Deciding, Always Be Leaving, Always Be Scaling.
- Always be deciding: identify blinders (things that prevent people who have been long steeped in a problem from seeing
the forest through the trees) and identify tradeoffs. The search latency problem is an example of managing tradeoffs at
Google: speed vs quality vs capacity (compute time).
- Always be leaving: aim to create a team that can function properly without you.
- Always be scaling: Maximize your influence, don't do things that you can delegate. Avoid the temptation to do a task
instead of delegating because you can get the task done more efficiently. By delegating, you help people grow. Think
about doing things that only *you* can do.
- Learn the difference between problems that are urgent vs important. Problems that are urgent are not important, and
problems that are important are not urgent. (Attributed to Eisenhower) Don't slip into reactive mode, where you are
only handling urgent problems. To make sure you hit the right balance, delegate urgent issues and consciously dedicate
time to solve important problems.

## Chapter 7 - Measuring Engineering Productivity

- It's important to first figure out if something is even worth measuring. Ask the following questions:
    - What result are you expecting, and why?
    - If the data supports your expected result, what action will be taken?
    - If we get a negative result, will appropriate action be taken?
    - Who is going to decide to take action on the result, and when would they do it?
- Google uses the Goals/Signals/Metrics (GSM) framework to guide metrics creation.
    - A goal is the end result, what you're trying to understand.
    - A signal is how you might you know you've achieved the end result. It's something we'd like to measure, but it
    might not be measurable.
    - A metric is a proxy for the signal, it's something we can measure.

## Chapter 8 - Style Guides and Rules

- Google puts in an enormous amount of effort to standardize code style.
- The main reason to standardize is for being able to scale.
    - Developers focus more on what to do rather than how it's done.
    - Automated tooling can be created to modify code en masse.
    - Increases mobility for engineers across teams.
- Google prevents developers from using bleeding edge language constructs until they can be properly evaluated.
- While the style guides are fairly rigid, they do allow for exceptions and a degree of pragmatism.

## Chapter 10 - Documentation

- Documentation helps with questions like:
    - Why were these design decisions made?
    - Why did we implement this code in this manner?
    - Why did I implement this code in this manner, if you’re looking at your own code two years later?
- Writing documentation provides these benefits:
    - Helps design an API. Just writing itself helps developers think more deeply about the design.
    - Provides a roadmap for maintenance and a historical record.
    - Documentation helps outside developers build confidence in the code.
    - Reduces the number of questions from other developers.
- Documentation should be treated like code.
    - Develop rules and best practices.Have internal policies or rules to be followed
    - Trace in version control.
    - Have clear ownsership.
    - Changes should be reviewed.
    - Track issues.
    - Periodically tested.
    - Measure accuracy, freshness.
- When writing a doc, know your audience and it make it clear in the beginning. Audience may vary by experience level,
domain knowledge, and purpose.
- Documentation writers should distinguish between seekers (readers who know what they want) vs stumblers (people who
are exploring/new to a topic). For seekers, documentation should be consistency. For stumblers, prioritize clarity.
- There are several types of documentation. Do not mix types of documentation—do one thing well.
    - Reference documentation, including code comments
    - Design documents
        - Covers the goals of the design, its implementation strategy, and design decisions with an emphasis on their
            individual trade-offs.
        - Should discuss design goals and alternative designs, with their tradeoffs.
    - Tutorials
        - Guides someone through the setup of a project, great for newcomers.
        - Make sure to explicitly number steps that the reader must do. Don't number the actions that the system takes
            in response to the user.
    - Conceptual documentation
        - Meant to impart understanding of a system, should augment reference documentation.
        - It's OK to have duplicate information in both reference and conceptual documentation in the name of clarity.
        - Emphasize clarity vs completeness. Make it broadly useful to both experts and novices.
    - Landing pages
        - A directory for documentation for a particular service. Just include links.
        - Landing pages should serve as a "traffic cop" for only one project. Don't mix team documentation with system
            documentation.
- The most documentation serves the need to know the "How." Documentation should also serve the 5 W's: who, what, when,
    where, why.
    - **Who** is the audience?
    - **What** is the purpose of the document?
    - **When** was the document created, updated, or reviewed?
    - **Where** does the doc live? 
    - **Why** does the doc exist? Summarize what you expect someone to take away from the document after reading it.