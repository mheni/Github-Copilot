---
lab:
  title: "Exercise - Create and use a Security Review Agent with GitHub Copilot in VS Code (Java)"
  description: "Create a custom GitHub Copilot agent specialized in security review, configure it in VS Code, and use it to analyze a Java project and its dependencies."
  duration: "60-90 minutes"
---

# Exercise - Create and use a Security Review Agent with GitHub Copilot in VS Code (Java)

In this lab, you will create a **custom GitHub Copilot agent** that acts as an Application Security Engineer (AppSec) embedded in the development team, then use it inside VS Code to review a Java project and its dependencies.

You will:

- Configure your environment for Java development with GitHub Copilot agents in VS Code.
- Create a custom agent definition file (`SecurityReview.agent.md`).
- Enable the agent in VS Code.
- Use the agent to:
  - Analyze Java code, configuration, and dependencies.
  - Detect potential security vulnerabilities and bad practices.
  - Suggest mitigations aligned with DevSecOps and OWASP recommendations.

> ⚠️ **Prerequisites**
>
> - Visual Studio Code with GitHub Copilot (Chat + Agent mode) enabled.
> - A GitHub Copilot subscription with access to agents in VS Code.
> - Git and a local workspace folder under Git version control.
> - Java 17 or later installed (JDK).
> - VS Code Java extensions installed (for example: “Extension Pack for Java”).
> - Maven installed and available in your `PATH`.
> - Basic knowledge of Java, Maven and common web/security concepts (OWASP Top 10 basics).

---

## 1. Set up the Java sample project

In this section, you create a simple Java web-like project that the Security Review Agent will analyze.

### 1.1. Create a workspace folder

1. Open a terminal and create a new folder:

   ```bash
   mkdir security-review-java-lab
   cd security-review-java-lab
   git init
   ```

2. Open the folder in VS Code:

   ```bash
   code .
   ```

### 1.2. Create a Maven Java project with basic security issues

1. In the integrated terminal, generate a Maven project:

   ```bash
   mvn archetype:generate \
     -DgroupId=com.example \
     -DartifactId=insecure-app \
     -DarchetypeArtifactId=maven-archetype-quickstart \
     -DarchetypeVersion=1.5 \
     -DinteractiveMode=false
   ```

2. Move into the project directory:

   ```bash
   cd insecure-app
   ```

3. Edit `pom.xml` to simulate potentially vulnerable dependencies. For example, set an older Spring Web dependency (this is for lab/demo purposes, not for production):

   ```xml
   <dependencies>
     <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-web</artifactId>
       <version>5.2.0.RELEASE</version>
     </dependency>

     <dependency>
       <groupId>org.apache.commons</groupId>
       <artifactId>commons-text</artifactId>
       <version>1.9</version>
     </dependency>

     <dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
       <version>4.13.2</version>
       <scope>test</scope>
     </dependency>
   </dependencies>
   ```

4. In `src/main/java/com/example`, replace `App.java` with the following insecure example:

   ```java
   package com.example;

   import java.io.BufferedReader;
   import java.io.File;
   import java.io.FileReader;
   import java.io.IOException;
   import java.sql.Connection;
   import java.sql.DriverManager;
   import java.sql.ResultSet;
   import java.sql.Statement;

   public class App {

       // Hard-coded secret (bad practice, used for the lab)
       private static final String DB_PASSWORD = "SuperSecretPassword123!";

       public static void main(String[] args) throws Exception {
           if (args.length == 0) {
               System.out.println("Usage: java com.example.App <username>");
               return;
           }

           String userInput = args;
           runInsecureQuery(userInput);
           readFileFromUserInput(userInput);
       }

       // Example of SQL injection risk
       private static void runInsecureQuery(String username) throws Exception {
           String url = "jdbc:h2:mem:testdb";
           Connection conn = DriverManager.getConnection(url, "sa", DB_PASSWORD);
           Statement stmt = conn.createStatement();

           String sql = "SELECT * FROM users WHERE name = '" + username + "'";
           ResultSet rs = stmt.executeQuery(sql);

           while (rs.next()) {
               System.out.println("User: " + rs.getString("name"));
           }

           rs.close();
           stmt.close();
           conn.close();
       }

       // Example of path traversal / unsafe file access
       private static void readFileFromUserInput(String path) throws IOException {
           File file = new File(path);
           if (!file.exists()) {
               System.out.println("File not found: " + path);
               return;
           }

           try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
               String line;
               while ((line = reader.readLine()) != null) {
                   System.out.println(line);
               }
           }
       }
   }
   ```

5. Commit the initial state:

   ```bash
   git add .
   git commit -m "Initial insecure Java app"
   ```

> ✅ You now have a small Java application that intentionally contains several security issues (hard-coded secret, possible SQL injection, unsafe file access, and older dependencies) to give the agent something meaningful to review.

---

## 2. Create the Security Review Agent file

In this section, you create a **custom Copilot agent** file that defines the behavior and tools for a "Security Review Agent".

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
   SecurityReview.agent.md
   ```

   Make sure the extension is exactly `.agent.md`.

### 2.2. Define the Security Review Agent frontmatter

Replace the YAML frontmatter at the top of `SecurityReview.agent.md` with:

```yaml
***
name: "Security Review Agent"
description: "An AppSec-focused agent that reviews Java code and dependencies for security vulnerabilities, suggests mitigations, and improves DevSecOps posture."
tools:
  - edit
  - execute
  - read/problems
  - search/changes
  - todo
  - agent/runSubagent
language: "java"
***
```

- `tools` reflects the capabilities used in this lab:
  - `edit` to propose and apply secure code changes,
  - `execute` to run commands such as `mvn dependency:tree` or security scanners,
  - `read/problems` to inspect compiler problems and possibly warnings,
  - `search/changes` to review recent modifications,
  - `todo` to find TODO/FIXME comments related to security debt,
  - `agent/runSubagent` to delegate specific tasks to other agents if needed.

### 2.3. Add the Security Review Agent instructions

Under the YAML frontmatter in `SecurityReview.agent.md`, paste the following content:

```markdown
# Security Review Agent - Behavior (Java / AppSec)

You are a **Security Review Agent** acting as an Application Security Engineer embedded in a Java development team.

Your main responsibilities are:

- Analyze Java code, configuration files, and dependencies for security risks.
- Detect common vulnerabilities and bad practices.
- Propose concrete mitigations and refactorings that improve the application's security posture.
- Promote DevSecOps practices throughout the project.

## Security objectives

Always focus on:

- Identifying common vulnerabilities:
  - Injection (SQL injection, command injection, LDAP injection, etc.).
  - Cross-Site Scripting (XSS), CSRF, and similar web vulnerabilities when relevant.
  - Remote Code Execution (RCE) patterns.
  - Path traversal and unsafe file access.
- Detecting hard-coded secrets or sensitive data:
  - API keys, tokens, passwords, certificates, connection strings.
- Spotting vulnerable or outdated dependencies:
  - Libraries and frameworks with known vulnerabilities, based on version hints or obvious outdated versions.
- Reviewing input and output handling:
  - Input validation and sanitization.
  - Output encoding where needed.
- Recommending secure coding practices:
  - OWASP guidelines (for example OWASP Top 10).
  - Principles like least privilege, fail-safe defaults, and defense in depth.
  - 12-Factor style configuration management (no secrets in code, environment-based configuration).

## Tools and workflow

You have access to the following tools:

- `edit` to propose secure refactorings directly in Java and configuration files.
- `execute` to run commands such as `mvn dependency:tree`, build commands, or security tools available in the environment.
- `read/problems` to inspect compiler warnings and errors that might indicate issues.
- `search/changes` to review recent code changes that may introduce security risks.
- `todo` to locate TODO/FIXME comments that might represent known but unresolved security concerns.
- `agent/runSubagent` to delegate tasks to specialized agents (for example, a Unit Tester Agent for verifying security-related tests).

Use these tools only when necessary and always explain what you intend to do before running a command or modifying files.

## Review strategy

When reviewing the project, follow this strategy:

1. **Clarify scope**
   - Ask the user which modules, packages, or features are in scope.
   - If scope is not specified, focus on the main application package and public entry points.

2. **High-level mapping**
   - Identify:
     - main entry points (main methods, controllers, public APIs),
     - configuration files (pom.xml, application properties/ YAML, etc.),
     - external dependencies and frameworks.

3. **Code-level review**
   - Look for:
     - direct string concatenation in SQL queries or command execution.
     - file system access based on user input (path traversal risks).
     - unsafe deserialization or reflection.
     - hard-coded secrets or credentials in the codebase.
   - Flag any suspicious patterns and code smells relevant to security.

4. **Dependencies review**
   - Inspect `pom.xml` and dependency tree.
   - Highlight dependencies or versions that are likely outdated or have known CVEs.
   - Recommend:
     - upgrading versions,
     - adding security-related plugins or scanners where appropriate.

5. **Mitigation proposals**
   - For each identified issue:
     - explain the risk in simple terms,
     - propose one or more mitigations,
     - suggest concrete code changes or library usages where applicable.
   - Where appropriate, propose to:
     - add validation or sanitization logic,
     - externalize secrets to environment variables or secure vaults,
     - use parameterized queries or ORM features.

6. **Security tests and DevSecOps**
   - Suggest test cases or security-focused unit/integration tests that would prevent regressions.
   - Propose automation ideas:
     - adding security scanning to CI/CD,
     - introducing static analysis tools,
     - regular dependency checks.

## Output style

When you respond:

- Start with a short summary of the overall security posture.
- Use structured sections:
  - Findings (by category: injection, secrets, dependencies, file access, etc.).
  - Impact and risk level (for example, low/medium/high).
  - Recommended remediation actions.
- When proposing code changes:
  - show only the relevant sections or diffs,
  - keep comments concise and focused on security rationale.
- Avoid making changes silently; always ask for confirmation before applying non-trivial edits.

Focus on helping the development team **understand and fix** security risks, not just listing them.
```

Save the file.

> If the agent does not appear later in the UI, reload VS Code:
>
> - Command Palette → Developer: Reload Window.

---

## 3. Ensure VS Code detects the custom agent

1. Confirm that `SecurityReview.agent.md` is located in:
   - either your user profile agent folder (if VS Code created one),
   - or in the workspace folder if you chose a workspace-specific agent.
2. If your setup uses explicit configuration:
   - Open **Settings (JSON)**.
   - Search for `chat.agentFilesLocations`.
   - Add the absolute path to the folder containing `SecurityReview.agent.md` if it is not already listed.
3. Reload the window after changing settings.

---

## 4. Use the Security Review Agent in Agent mode (Java)

Now you will use the agent to analyze and review the insecure Java application.

### 4.1. Open Agent mode in VS Code

1. Open the **Chat view** in VS Code.
2. In the chat session type dropdown, select **Agent** (or the agent-capable session type in your version).
3. In the **agent picker dropdown**, select **Security Review Agent**.

If you do not see your agent:
- Check the file name and extension (`SecurityReview.agent.md`).
- Check the YAML frontmatter.
- Check `chat.agentFilesLocations` if applicable.
- Reload VS Code.

### 4.2. Initial security scan of the Java code

1. Open `src/main/java/com/example/App.java` in the editor.
2. Select the entire `App` class.
3. Drag and drop the selection, or the file, into the Chat view to add it as context.
4. Send the following prompt to the Security Review Agent:

   ```text
   Review this Java App class for security issues.
   Identify potential vulnerabilities, categorize them (injection, secrets, path traversal, etc.),
   and propose concrete mitigations.
   ```

5. Review the findings:
   - The agent should highlight:
     - hard-coded secret (`DB_PASSWORD`),
     - SQL query building with string concatenation,
     - file access based on user input without validation.

6. Ask follow-up questions if needed, for example:

   ```text
   For each issue you identified, suggest specific code changes in App.java to mitigate the risk.
   Use parameterized queries and avoid hard-coded secrets.
   ```

### 4.3. Ask the agent to propose secure refactors

1. Ask the agent:

   ```text
   Propose secure refactorings for App.java:
   - Use parameterized SQL queries instead of string concatenation.
   - Remove the hard-coded password and replace it with an environment-based configuration placeholder.
   - Add basic validation to the file path before reading the file.
   Show the updated code as a diff or full class.
   ```

2. Let the agent generate a secure version of the `App` class.
3. Review the proposed changes:
   - parameterized query or prepared statements,
   - no hard-coded password (placeholder or configuration),
   - checks on the file path (for example, restricting to a base directory).

4. If the changes look reasonable, ask the agent to apply them using its `edit` tool, or apply them manually after inspection.

---

## 5. Review dependencies and configuration

In this section, you use the agent to examine `pom.xml` and the dependency tree.

### 5.1. Analyze `pom.xml` for vulnerable dependencies

1. Open `pom.xml` in the editor.
2. Add it to the Chat context (drag-and-drop or selection).
3. Ask the agent:

   ```text
   Review this pom.xml for potential vulnerable or outdated dependencies.
   Point out any libraries or versions that are likely to have known vulnerabilities,
   and propose safer version ranges or alternatives.
   ```

4. Examine the agent’s feedback on dependencies like:
   - `spring-web 5.2.0.RELEASE`
   - `commons-text 1.9`

5. Optionally, ask the agent for a concrete upgraded `pom.xml` snippet:

   ```text
   Propose updated version numbers for potentially vulnerable dependencies,
   and explain why these versions are safer.
   ```

### 5.2. Run a Maven dependency analysis command

1. Ask the agent:

   ```text
   Run a dependency analysis using Maven (for example, mvn dependency:tree),
   then summarize any findings relevant to security or dependency hygiene.
   ```

2. Approve when the agent proposes to use `execute` with:

   ```bash
   mvn dependency:tree
   ```

3. Let the agent parse the command output and comment on any dependency risks it detects.

> Note: In a real environment, you could also integrate dedicated security scanners (like OWASP Dependency-Check), but this lab focuses on using Copilot and Maven basics.

---

## 6. Consolidate findings and next steps

### 6.1. Ask for a structured security report

1. In the Chat view, ask:

   ```text
   Produce a concise security review report for this Java project, including:
   - Summary of the overall security posture.
   - List of findings grouped by category (injection, secrets, file access, dependencies).
   - Risk level for each finding (low/medium/high).
   - Recommended remediations and next steps.
   ```

2. Review the report and compare it to your own analysis as a developer or security engineer.

### 6.2. Optional: delegate to another agent

If you have another custom agent (for example, a Unit Tester Agent):

1. Ask the Security Review Agent:

   ```text
   Use agent/runSubagent to ask the Unit Tester Agent to generate security-focused unit tests
   for the most critical parts of App.java (for example, input validation and exception paths).
   ```

2. Observe how the agents can collaborate:
   - Security Review Agent for findings and mitigations,
   - Unit Tester Agent for generating tests that prevent regressions.

---

## 7. Clean up

If this was a temporary lab environment, you can remove the repository and the custom agent file:

1. Close VS Code.
2. Delete the `security-review-java-lab` folder.
3. If you created the agent at user-profile level and you no longer want it, remove `SecurityReview.agent.md` from the corresponding folder and reload VS Code.

---

## Summary

In this exercise, you:

- Created a **Security Review Agent** as a custom GitHub Copilot agent in VS Code using a `.agent.md` file.
- Configured the agent with tools such as `edit`, `execute`, `read/problems`, `search/changes`, `todo`, and `agent/runSubagent` to analyze and improve the security of a Java project.
- Used the agent in **Agent mode** to:
  - Analyze Java code and detect security issues (injection, secrets, unsafe file access).
  - Review `pom.xml` and dependencies for potential vulnerabilities.
  - Propose and apply secure refactorings.
  - Produce a structured security report and, optionally, collaborate with other agents for testing.

You can reuse this pattern to build more specialized AppSec agents for your own Java projects and to teach DevSecOps practices in a practical, code-centric way.
