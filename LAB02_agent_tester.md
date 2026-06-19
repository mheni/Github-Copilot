---
lab:
  title: "Exercise - Create and use a Unit Tester Agent with GitHub Copilot in VS Code (Java)"
  description: "Create a custom GitHub Copilot agent specialized in unit testing, configure it in VS Code, and use it to generate and validate JUnit 5 tests for a Java project."
  duration: "45-60 minutes"
---

# Exercise - Create and use a Unit Tester Agent with GitHub Copilot in VS Code (Java)

In this lab, you will create a **custom GitHub Copilot agent** that acts as a paranoid unit-test QA assistant, then use it inside VS Code to generate and validate **JUnit 5** unit tests for an existing Java project.

You will:

- Configure your environment for Java development with GitHub Copilot agents in VS Code.
- Create a custom agent definition file (`UnitTester.agent.md`).
- Enable the agent in VS Code.
- Use the agent to:
  - Analyze Java code and propose a test strategy.
  - Generate JUnit 5 unit tests.
  - Run the Maven test suite and interpret failures.
  - Improve coverage and robustness of the code.

>  **Prerequisites**
>
> - Visual Studio Code with GitHub Copilot (Chat + Agent mode) enabled.
> - A GitHub Copilot subscription with access to agents in VS Code.
> - Git and a local workspace folder under Git version control.
> - Java 17 or later installed (JDK).
> - VS Code Java extensions installed (for example: “Extension Pack for Java”).
> - Maven installed and available in your `PATH`.
> - Basic knowledge of Java and JUnit 5.

---

## 1. Set up the Java sample project

In this section, you create a simple Maven project that the Unit Tester Agent will analyze and test.

### 1.1. Create a workspace folder

1. Open a terminal and create a new folder:

   ```bash
   mkdir unit-tester-java-lab
   cd unit-tester-java-lab
   git init
   ```

2. Open the folder in VS Code:

   ```bash
   code .
   ```

### 1.2. Create a Maven Java project

1. In the terminal inside VS Code, create a Maven project:

   ```bash
   mvn archetype:generate \
     -DgroupId=com.example \
     -DartifactId=calculator-app \
     -DarchetypeArtifactId=maven-archetype-quickstart \
     -DarchetypeVersion=1.5 \
     -DinteractiveMode=false
   ```

2. Move into the project directory:

   ```bash
   cd calculator-app
   ```

3. Open `pom.xml` and ensure it uses JUnit 5. Replace the `<dependencies>` section with:

   ```xml
   <dependencies>
     <dependency>
       <groupId>org.junit.jupiter</groupId>
       <artifactId>junit-jupiter</artifactId>
       <version>5.10.0</version>
       <scope>test</scope>
     </dependency>
   </dependencies>

   <build>
     <plugins>
       <plugin>
         <groupId>org.apache.maven.plugins</groupId>
         <artifactId>maven-surefire-plugin</artifactId>
         <version>3.1.2</version>
         <configuration>
           <useModulePath>false</useModulePath>
         </configuration>
       </plugin>
     </plugins>
   </build>
   ```

4. In `src/main/java/com/example`, create a `Calculator.java` class:

   ```java
   package com.example;

   public class Calculator {

       public int add(int a, int b) {
           return a + b;
       }

       public int subtract(int a, int b) {
           return a - b;
       }

       public int divide(int a, int b) {
           if (b == 0) {
               throw new IllegalArgumentException("Denominator cannot be zero.");
           }
           return a / b;
       }
   }
   ```

5. Commit the initial state:

   ```bash
   git add .
   git commit -m "Initial Java calculator app"
   ```

>  At this point, you have a minimal Java codebase with non-trivial behavior (division by zero, integer division), ideal for testing.

---

## 2. Create the Unit Tester Agent file

In this section, you create a **custom Copilot agent** file that defines the behavior and tools for a "Unit Tester Agent".

### 2.1. Create the agent file via VS Code

1. In VS Code, open the **Command Palette**:

   - Windows / Linux: `Ctrl+Shift+P`
   - macOS: `Cmd+Shift+P`

2. Run the command:

   ```text
   Chat: New Custom Agent
   ```

3. When prompted for the location, choose:

   - **User Profile** for a global agent, or
   - **Workspace** to make it specific to this project.

4. VS Code will create a new `.agent.md` file. Rename it to:

   ```text
   UnitTester.agent.md
   ```

   Make sure the extension is exactly `.agent.md`.

### 2.2. Define the Unit Tester Agent frontmatter

Replace the YAML frontmatter at the top of `UnitTester.agent.md` with:

```yaml
***
name: "Unit Tester Agent"
description: "A paranoid QA agent specialized in unit tests for Java: analyzes code, designs test plans, generates JUnit tests, runs them, and explains failures."
tools:
  - edit
  - runCommands
  - runTasks
  - problems
  - changes
  - testFailure
  - todos
  - runSubagent
  - runTests
language: "java"
***
```

- `tools` lists the capabilities the agent can use in VS Code (edit files, run commands/tasks, inspect problems, run tests, etc.).
- This lab focuses on **Java only**, with JUnit 5 as the testing framework.

### 2.3. Add the Unit Tester Agent instructions

Under the YAML frontmatter in `UnitTester.agent.md`, paste the following content:

```markdown
#  Unit Tester Agent - Behavior (Java / JUnit 5)

You are a **paranoid QA agent specialized in unit testing for Java**.
Your main responsibilities are:

- Analyze the existing Java code and identify behaviors, edge cases, and error conditions.
- Design a clear and prioritized **test plan**.
- Generate **JUnit 5 unit tests** for the current project.
- Run tests when allowed, interpret failures, and suggest fixes.
- Challenge the code to improve robustness, not just reach minimal coverage.
- Aim for **logical coverage > 90%** where reasonable.

##  Tools and environment

You have access to the following tools in VS Code:

- `edit`: propose and apply changes to Java source files and test files.
- `runCommands`: run shell or CLI commands (for example `mvn test`) when the user approves.
- `runTasks`: run VS Code tasks defined for building/testing.
- `problems`: inspect compiler diagnostics.
- `changes`: inspect recent changes made in the workspace.
- `testFailure`: inspect failed tests and their stack traces.
- `todos`: search for TODO/FIXME comments that might require tests.
- `runSubagent`: delegate subtasks to other specialized agents when appropriate.
- `runTests`: run the project's test suite or a subset of tests.

Only use these tools when necessary and always explain what you intend to do before running a command or modifying files.

##  Testing strategy

Always follow this workflow:

1. **Understand the code**
   - Ask the user which Java package or class to focus on if it's ambiguous.
   - Summarize the responsibilities, inputs, outputs, and error conditions of the target code.
   - Identify obvious edge cases and invalid inputs.

2. **Propose a test plan**
   - List test categories:
     - Happy path (nominal behavior)
     - Edge cases (bounds, limits)
     - Error conditions and exceptions
     - Invalid or unexpected inputs
   - For each category, list concrete test cases.
   - Ask the user to confirm or adjust the test plan.

3. **Generate tests**
   - Use **JUnit 5** exclusively for this lab.
   - Follow standard Maven / JUnit conventions:
     - Place tests under `src/test/java`.
     - Use package names mirroring the main source (for example `com.example`).
     - Name test classes `<ClassName>Test` or `<ClassName>Tests`.
   - Ensure tests are deterministic and isolated.

4. **Run tests and interpret results**
   - When the user approves, run the Maven test command:
     - `mvn test`
   - Use `testFailure` and `problems` to inspect failing tests or compilation errors.
   - Explain failures in plain language and propose fixes in the code or tests.

5. **Strengthen robustness and coverage**
   - Identify untested branches or conditions in the Java code.
   - Propose additional tests for:
     - Extreme inputs
     - Boundary values
     - Invalid arguments (for example negative numbers, zero denominators)
   - Suggest refactorings that make the code more testable and easier to reason about.

6. **Report to the developer**
   - Provide a concise, structured report including:
     - Summary of the tested Java component.
     - Test plan and coverage focus.
     - List of added JUnit tests.
     - Known gaps or limitations.
     - Recommendations for further improvements.

##  Output style

- Always start by clarifying the **scope** (which Java file/class/method).
- Use headings and bullet lists for test plans and reports.
- When generating code, show only the relevant files or diff sections.
- Ask for confirmation before applying large changes.

Stay focused on **JUnit 5 unit tests** (not integration or end-to-end tests) unless the user explicitly requests otherwise.
```

Save the file.

>  If the agent does not appear later in the UI, reload VS Code:
>
> - Command Palette → **Developer: Reload Window**.

---

## 3. Make sure VS Code can see custom agents

Depending on your VS Code version and configuration, custom agent files may need to be in specific locations or referenced in settings.

1. If prompted when you created the agent, VS Code should have placed the file in a discoverable location (user profile or workspace).  
2. If you customized the location manually:
   - Open **Settings** (JSON view).
   - Search for `chat.agentFilesLocations`.
   - Ensure the path to the folder containing `UnitTester.agent.md` is listed and uses an absolute path.
3. Reload the window again if you changed settings.

---

## 4. Use the Unit Tester Agent in Agent mode (Java)

Now you will use the agent to analyze and test the Java calculator code.

### 4.1. Open Agent mode in VS Code

1. Open the **Chat view** in VS Code.
2. In the chat session type dropdown, select **Agent** (or the agent-capable session type in your version).
3. In the **agent picker dropdown**, select **Unit Tester Agent**.

> If you do not see your agent:
> - Verify the `.agent.md` extension and YAML frontmatter.
> - Confirm the file path matches any `chat.agentFilesLocations` configuration.
> - Reload the window.

### 4.2. Ask the agent to analyze the Java code

With the agent selected:

1. Open `src/main/java/com/example/Calculator.java` in the editor.
2. Select the `Calculator` class in the editor.
3. In the Chat view, ensure the selection is added as context (or drag-and-drop the file into the chat input).
4. Send the following prompt to the **Unit Tester Agent**:

   ```text
   Analyze this Calculator class and propose a prioritized JUnit 5 unit test plan.
   Highlight normal cases, edge cases, and error conditions. Assume goal: >90% logical coverage.
   ```

5. Review the proposed test plan.
6. If needed, ask follow-up questions, for example:

   ```text
   Refine the test plan to focus on divide() and potential failure scenarios, including division by zero and negative values.
   ```

---

## 5. Ask the agent to generate JUnit 5 tests

1. In the Chat view, send:

   ```text
   Generate JUnit 5 tests for the Calculator class based on your test plan.
   Use Maven conventions: put tests under src/test/java/com/example and name the test class CalculatorTest.
   ```

2. The agent should:
   - Create or modify `src/test/java/com/example/CalculatorTest.java`.
   - Implement tests covering:
     - normal addition and subtraction,
     - division with valid denominators,
     - division by zero throwing IllegalArgumentException,
     - possibly negative and boundary values.

3. Ask the agent to apply the changes using its `edit` tool.
4. Inspect the generated test file `CalculatorTest.java` pour vérifier :
   - l’utilisation correcte de JUnit 5 (`@Test`, `Assertions.assertEquals`, `Assertions.assertThrows`, etc.),
   - la clarté et l’indépendance des tests.

5. When satisfied, ask the agent to run the tests:

   ```text
   Run the Maven test suite (mvn test) and report which tests fail and why.
   ```

   The agent should use `runCommands` or `runTests` to execute `mvn test`, then analyze any failures using `testFailure` and `problems`.

---

## 6. Improve robustness and coverage for Java

In this section, you will push the agent to behave like a “paranoid QA” and look for untested paths in the Java code.

### 6.1. Ask for coverage gaps

1. In the Chat view, ask:

   ```text
   Based on the current JUnit tests and the Calculator implementation, identify untested branches or edge cases.
   Propose additional unit tests to increase coverage and robustness.
   ```

2. Review the suggestions (for example, large integers, negative combinations, different sign combinations, multiple calls).

### 6.2. Generate additional tests

1. Ask the agent:

   ```text
   Implement the additional JUnit 5 tests you proposed and update CalculatorTest.java.
   Avoid duplicating existing test logic.
   ```

2. Let the agent update `src/test/java/com/example/CalculatorTest.java` via its `edit` tool.
3. Run the tests again:

   ```text
   Re-run mvn test and confirm that all tests pass.
   Highlight what kinds of failures you would expect if there were bugs in the Calculator implementation.
   ```

### 6.3. Request a final QA report

1. Finally, ask:

   ```text
   Produce a concise QA report for the Calculator component:
   - Summary of the tested behaviors
   - Test plan and what we actually implemented
   - Known gaps or non-covered behaviors
   - Recommendations for further improvements
   ```

2. Review the report and compare it to your expectations as a Java developer / QA engineer.

---

## 7. Clean up

If this was a temporary lab environment, you can remove the repository and the custom agent file:

1. Close VS Code.
2. Delete the `unit-tester-java-lab` folder.
3. If you created the agent at user-profile level and you no longer want it, remove `UnitTester.agent.md` from the corresponding folder and reload VS Code.

---

## Summary

In this exercise, you:

- Created a **Unit Tester Agent** as a custom GitHub Copilot agent in VS Code using a `.agent.md` file.
- Configured the agent with tools such as `edit`, `runCommands`, `runTests`, `testFailure`, and others to analyze and test Java code.
- Used the agent in **Agent mode** to:
  - Analyze a Java class (`Calculator`).
  - Propose a JUnit 5 test plan with edge cases and error conditions.
  - Generate and run JUnit 5 tests with Maven.
  - Interpret failures and improve robustness and coverage.

This pattern can be reused to build more specialized Java QA agents for your own projects (par exemple pour les services Spring, les repositories JPA, ou les couches métier plus complexes).
