# Software Engineering at Google

## Chapter 3 - Knowledge Sharing

- This chapter talks about how domain knowledge is collected and shared throughout Google.
- There are several problems that can be caused by a lack of knowledge sharing culture:
    1. **Lack of psychological safety**
        - People become afraid to take risks and make mistakes.
    2. **Information islands**
        - Knowledge becomes fragmented between developers/teams.
    3. **Single point of failure (SPOF)**
        - Only one person has information.
    4. **All-or-nothing expertise**
        - A split in a group of people where one side has all the knowledge and the other side has very little knowledge.
    5. **Parroting**
        - People use knowledge/information without really understanding it.
    6. **Haunted graveyards**
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

## Chapter 9 - Code Review

- Essentially every change is reviewed before being committed at Google.
- Three aspects of review that require approval for a change:
    - Correctness and comprehension check, often from team member.
    - Approval from one of the code owners, usually a tech lead or another engineer that is an expert of the codebase.
    - Approval with someone with language "readability", someone who is certified in Google's best practices for the language the change is written in.
- Benefits of code review:
    - Checks code correctness.
    - Makes sure change is comprehensible to other engineers.
    - Enforces consistency across the codebase.
    - Promotes team ownership.
    - Promotes knowledge sharing.
    - Provides a historical record of the change.
- Best practices
    - Be polite and professional. Keep all feedback and criticism professional.
        - Reviewers should defer to authors and onlypoint out alternatives if the author's approach is deficient.
        - Feedback should be prompt (returned within 24 working hours).
        - Understand that the code is not "yours" but the team's.
    - Write small change, generally 200 lines of code.
    - Write good change descriptions.
        - The summary should go in the first line.
        - The following description should detail what's being changed and *why*.
    - Keep reviewers to a minimum.
        - Google has determined that there are diminishing returns with more than 1 reviewer.
        - The most important LGTM is the first one.
- Automate where possivle with static analysis, tests, linters, and formatters.
- Code reviews for greenfield projects require a design review.
- New projects should contain an `OWNERS` file.

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

## Chapter 11 - Testing Overview

- Early Google did not have institutional adoption of engineer-driven testing. This chapter tells the story of Google 
Web Server. This system started out with no automated testing and continued that way until 80% of production pushes
contained bugs that needed to be rolled back. The tech lead introduced engineer-driven, automated testing, and the
number of emergency pushes dropped by half within the first year even though the number of changes to the project
increased every quarter.
- After experiencing pain with slow, unreliable system-scale tests, Google now prefers "small" tests. Tests are sized 
    according to characteristics other than the number of lines of code.
    - *Small*
        - Runs on a single process.
        - Can't sleep, perform I/O operations, or make any blocking calls.
        - These restrictions ensure that tests are fast and deterministic.
    - *Medium*
        - Can span multiple processes, use threads, make blocking calls, and
            use the localhost network.
        - Can run a database instance or implement UI tests.
    - *Large*
        - For testing across multiple machines.
        - Can run without `localhost` network restriction.
        - Typically run at building time/during release process in order to not slow down workflow.
        - Reserved for validating system configurations.
- The size of the tests are actually enforced by tooling.
- Google aims for 80% of tests to be narrow in scope, 15% medium, and 5% large.
- Two anti-patterns: "ice cream cone" and "hourglass".
    - *Ice cream cone*: Heavy on manual testing and large tess.
    - *Hourglass*: Heavy on unit and end-to-end tests, light on integration tests.
- Google discourages the code coverage metrics.
    - Only assumes the code was invoked, not that the code did anything useful.
    - Becomes its own goal. If code coverage requirement is 80%, why go above and beyond 80%?
    - A better way to think about test quality is to think about the behaviors that are tested. 
        - Can me make changes without breaking things for the customer?
        - Can we catch breaking changes in dependencies?
        - Are the tests stable and reliable?
- Google didn't get serious about testing until 2005-2006. Now the company offers testing classes during orientation test
    certification.
- ["Testing on the Toilet"](https://testing.googleblog.com/2007/01/introducing-testing-on-toilet.html) (a flyer distributed
in toilet stalls across the company) raised awareness of testing best practices.
- Automated Testing is not the best for all testing tasks, such as rating search result quality, audio/video quality,
    and security.
- "Exploratory Testing" is a creative type of testing used when problems are unknown from the start.

## Chapter 12 - Unit Testing

- The ideal test is unchanging: after it's written, a test never changes unless the requirements change.
- The test should remain the same despite refactorings, bug fixes, and new features.
    - Test via public APIs.
    - Test State, not Interactions.
        ```java
        // Brittle
        @Test
        ublic void shouldWriteToDatabase() {
         accounts.createUser("foobar");
         verify(database).put("foobar");
        }

        // Robust, tests using state
        @Test
        public void shouldCreateUsers() {
          accounts.createUser("foobar");
          assertThat(accounts.getUser("foobar")).isNotNull();
        }
        ```
- In order to test against state, we must use real objects instead of mock objects. I'm skeptical of this advice, but will learn more in chapter 13.
- Make sure tests are **complete**, meaning all relevant informaton necessary for the reader to understand the method is in the test body.
- Also make sure the tests are **concise**, meaning the tests do not contain unnecessary, distracting information.
- Test behaviors, not methods. A behavior is any guarantee the system makes for a given input. We can express behaviors
    using the words "given" (the setup) "when" (the action taken) and "then" (the result validation). Well-structed
    tests are organized around these three components.
    ```java
    @Test
    public void transferFundsShouldMoveMoneyBetweenAccounts() {
      // Given two accounts with initial balances of $150 and $20
      Account account1 = newAccountWithBalance(usd(150));
      Account account2 = newAccountWithBalance(usd(20));

      // When transferring $100 from the first to the second account
      bank.transferFunds(account1, account2, usd(100));

      // Then the new account balances should reflect the transfer
      assertThat(account1.getBalance()).isEqualTo(usd(50));
      assertThat(account2.getBalance()).isEqualTo(usd(120));
    }
    ```
- Name tests after behavior being testing (e.g. `shouldNotAllowWithdrawalsWhenBalanceIsEmpty`).
- Don't put logic in tests, such as operators, loops and conditionals. The chapter gives a confusing example where a
    test assertion uses a confusingly-concatenated string for its expected input.
- Wite clear failure messages. Favor 
    ```java
    `Expected an account in state CLOSED, but got account: <{name: "my-account", state: "OPEN"}`
    ```
    instead of
    ```java
    Test failed: account is closed
    ```
- The [truth](https://truth.dev/) is a Google assertion framework that helps make clear error messages.
- Don't be dogmatic about DRY (Don't Repeat Yourself) when it comes to testing, as maintaning DRY can lead to
    diminished test clarity. Instead focus on DAMP (Descriptive and Maningful Phrases).