---
lab:
  title: "Labo - Connecter une application Java Spring Boot à GitHub Copilot via MCP"
  description: "Créer une application Spring Boot Java qui expose des actions métier comme outils MCP (Model Context Protocol), les connecter à GitHub Copilot dans VS Code et les invoquer en langage naturel."
  duration: "60-90 minutes"
---

# Labo - Connecter une application Java Spring Boot à GitHub Copilot via MCP

## Objectif du labo

Dans ce labo, vous allez découvrir comment **GitHub Copilot** peut consommer des services externes via le **Model Context Protocol (MCP)**.

Vous allez :

- créer une petite application Java Spring Boot de gestion de tâches ;
- y ajouter Spring AI MCP pour exposer des **tools MCP** ;
- démarrer un serveur MCP en SSE ;
- connecter ce serveur dans VS Code ;
- utiliser GitHub Copilot pour appeler ces tools en langage naturel.

L’objectif est de montrer la différence entre :

- “Copilot écrit du code”  
et  
- “Copilot invoque des fonctionnalités métier réelles via MCP”.

---

## 1. Pré‑requis

Avant de commencer :

- Visual Studio Code à jour.
- Extension GitHub Copilot installée et connectée.
- GitHub Copilot avec **Agent mode + support MCP** disponibles.
- Java 17 ou plus (JDK).
- Maven installé (`mvn -v` doit fonctionner).
- Extensions Java pour VS Code (par exemple “Extension Pack for Java”).
- Connaissances de base :
  - Java,
  - Spring Boot,
  - REST,
  - Maven.

---

## 2. Créer le projet Java Spring Boot

### 2.1. Créer le workspace

Dans un terminal :

```bash
mkdir mcp-java-lab
cd mcp-java-lab
git init
code .
```

### 2.2. Générer le projet Spring Boot

Vous pouvez utiliser Spring Initializr (site web) ou la CLI. Pour rester scriptable, on suppose que le projet a :

- Group: `com.example`
- Artifact: `task-mcp-app`
- Type: Maven
- Java: 17
- Dépendances :
  - Spring Web

Placez‑vous dans le dossier du projet (par ex. `task-mcp-app`) avant de continuer.

---

## 3. Configurer Maven et Spring AI MCP

Nous allons :

- ajouter Spring AI MCP server pour WebMVC (SSE),
- préparer la structure de code.

### 3.1. Mettre à jour `pom.xml`

Ouvrez `pom.xml` et ajustez le contenu comme suit (simplifié pour le labo) :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>task-mcp-app</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>task-mcp-app</name>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.5</version>
        <relativePath/>
    </parent>

    <properties>
        <java.version>17</java.version>
        <!-- Adaptez la version de Spring AI à votre environnement -->
        <spring-ai.version>1.0.0</spring-ai.version>
    </properties>

    <dependencies>
        <!-- Web MVC classique -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- MCP Server Spring AI (SSE + WebMVC) -->
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-starter-mcp-server-webmvc</artifactId>
            <version>${spring-ai.version}</version>
        </dependency>

        <!-- Tests -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

> Adaptation : dans un environnement d’entreprise, on utilisera souvent un BOM Spring AI. Pour un labo, garder `spring-ai.version` dans `<properties>` suffit.

---

## 4. Modèle et service Java

Nous allons coder une petite gestion de tâches en mémoire.

### 4.1. Structure de packages

Créez la structure :

```text
src/main/java/com/example/taskmcp/
  TaskMcpApplication.java
  model/Task.java
  service/TaskService.java
```

### 4.2. Application principale

`TaskMcpApplication.java` :

```java
package com.example.taskmcp;

import com.example.taskmcp.service.TaskService;
import org.springframework.ai.mcp.server.tool.ToolCallbackProvider;
import org.springframework.ai.tool.MethodToolCallbackProvider;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class TaskMcpApplication {

    public static void main(String[] args) {
        SpringApplication.run(TaskMcpApplication.class, args);
    }

    // Enregistrer les méthodes @Tool du service comme tools MCP
    @Bean
    public ToolCallbackProvider mcpTools(TaskService taskService) {
        return MethodToolCallbackProvider.builder()
                .toolObjects(taskService)
                .build();
    }
}
```

### 4.3. Modèle `Task`

`model/Task.java` :

```java
package com.example.taskmcp.model;

public class Task {

    private Long id;
    private String title;
    private boolean completed;

    public Task(Long id, String title, boolean completed) {
        this.id = id;
        this.title = title;
        this.completed = completed;
    }

    public Long getId() {
        return id;
    }

    public String getTitle() {
        return title;
    }

    public boolean isCompleted() {
        return completed;
    }

    public void setCompleted(boolean completed) {
        this.completed = completed;
    }
}
```

### 4.4. Service `TaskService` (avec annotations MCP)

`service/TaskService.java` :

```java
package com.example.taskmcp.service;

import com.example.taskmcp.model.Task;
import org.springframework.ai.tool.annotation.Tool;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicLong;

@Service
public class TaskService {

    private final List<Task> tasks = new ArrayList<>();
    private final AtomicLong counter = new AtomicLong(1);

    @Tool(
        name = "list_tasks",
        description = "Liste toutes les tâches existantes."
    )
    public List<Task> listTasksTool() {
        return tasks;
    }

    @Tool(
        name = "create_task",
        description = "Crée une nouvelle tâche avec un titre fourni."
    )
    public Task createTaskTool(String title) {
        Task task = new Task(counter.getAndIncrement(), title, false);
        tasks.add(task);
        return task;
    }

    @Tool(
        name = "complete_task",
        description = "Marque une tâche comme terminée à partir de son identifiant."
    )
    public Task completeTaskTool(Long id) {
        return tasks.stream()
                .filter(task -> task.getId().equals(id))
                .findFirst()
                .map(task -> {
                    task.setCompleted(true);
                    return task;
                })
                .orElseThrow(() -> new IllegalArgumentException("Task not found: " + id));
    }

    @Tool(
        name = "task_summary",
        description = "Retourne un résumé du nombre total, terminé et ouvert de tâches."
    )
    public String taskSummaryTool() {
        long total = tasks.size();
        long completed = tasks.stream().filter(Task::isCompleted).count();
        long open = total - completed;
        return "Total=" + total + ", completed=" + completed + ", open=" + open;
    }
}
```

> Les annotations `@Tool` indiquent quelles méthodes doivent être exposées comme **tools MCP**.  
> Le bean `mcpTools` dans `TaskMcpApplication` enregistre ces méthodes auprès du serveur MCP Spring AI.

---

## 5. Configuration Spring Boot / MCP

### 5.1. Fichier `application.yml`

Créez `src/main/resources/application.yml` :

```yaml
spring:
  application:
    name: task-mcp-app
  ai:
    mcp:
      server:
        enabled: true
        protocol: SSE
        name: task-mcp-server
        version: 1.0.0
        # selon la version, ces propriétés peuvent exister :
        # sse-message-endpoint: /sse
```

> Selon la version du starter, l’endpoint SSE par défaut est souvent `/sse` sur le port 8080.  
> Si besoin, on peut forcer un chemin via `sse-message-endpoint`.

### 5.2. Démarrer l’application

Dans le terminal :

```bash
mvn clean package
mvn spring-boot:run
```

- Vérifiez que l’application démarre sans erreur.
- Si vous testez `http://localhost:8080/actuator/health`, vous devriez voir un statut `UP` (si vous avez activé Actuator).

---

## 6. Ajouter le serveur MCP dans VS Code / GitHub Copilot

L’idée maintenant est de dire à VS Code : “il existe un serveur MCP à `http://localhost:8080/sse` nommé `task-mcp-server`”.

### 6.1. Ouvrir Agent mode

1. Assurez‑vous que l’application tourne encore (`mvn spring-boot:run`).
2. Dans VS Code, ouvrez **GitHub Copilot Chat**.
3. Passez en **Agent mode**.
4. Cliquez sur **Open in Agents** (ou l’équivalent) pour ouvrir la vue Agents / Tools / MCP servers.

### 6.2. Ajouter le serveur MCP via l’UI

1. Dans la vue Agents, repérez la section **Tools** ou **MCP servers**.
2. Cliquez sur un bouton de type :
   - “Add more tools…”
   - puis “Add MCP server” / “MCP: Add Server”.
3. Choisissez le type **HTTP / SSE**.
4. Renseignez :
   - Name / ID : `task-mcp-server`
   - URL : `http://localhost:8080/sse` (ou l’URL indiquée par votre version de Spring AI).
5. Sauvegardez au niveau **workspace**.

VS Code devrait créer/mettre à jour un fichier `.vscode/mcp.json` avec une entrée pour `task-mcp-server`.

### 6.3. Vérifier la découverte des tools

Dans Copilot Chat :

1. Assurez‑vous que `task-mcp-server` est coché/activé comme source d’outils pour la session.
2. Demandez :

   ```text
   Liste les outils disponibles sur le serveur MCP "task-mcp-server".
   ```

Vous devriez voir au moins :

- `list_tasks`
- `create_task`
- `complete_task`
- `task_summary`

Si vous voyez “discovered zero tools”, la cause est généralement :

- le starter MCP n’est pas chargé,
- les méthodes ne sont pas annotées `@Tool`,
- ou le bean `ToolCallbackProvider` n’est pas présent.

---

## 7. Utiliser GitHub Copilot pour appeler les tools Java

Quelques prompts type à utiliser dans le chat (en s’assurant que le serveur MCP est actif et autorisé) :

```text
Utilise le serveur MCP "task-mcp-server" pour créer une tâche intitulée
"Préparer la revue de sprint".
```

```text
Utilise les outils MCP pour lister toutes les tâches.
```

```text
Utilise le serveur MCP pour marquer la tâche avec l'id 1 comme terminée.
```

```text
Donne-moi un résumé de l'état des tâches (total, terminées, ouvertes)
en appelant l'outil MCP approprié.
```

Pendant ces interactions, observez que :

- Copilot interprète la demande,
- choisit le tool MCP correspondant (`create_task`, `list_tasks`, etc.),
- appelle l’app Java via MCP,
- et revient avec le résultat structuré + un texte explicatif.

---

## 8. Extensions possibles (facultatif)

Pour aller plus loin :

- Ajouter un tool `delete_task(Long id)` et le tester via Copilot.
- Introduire des validations (titre obligatoire, longueur max) et voir comment Copilot gère les erreurs.
- Remplacer la liste en mémoire par une base H2 ou un repository Spring Data.
- Discuter des aspects de sécurité (auth, permissions, logs) quand on expose des tools internes via MCP.

---

## 9. Points de débrief pour le formateur

Questions possibles en fin de lab :

- Qu’est‑ce qui change par rapport à un usage “Copilot = autocomplétion de code” ?
- Quels types de services Java internes pourraient être exposés comme tools MCP dans votre SI ?
- Quels risques / questions de gouvernance cela soulève (droits, secrets, audit) ?
- Comment combiner agents Copilot + MCP servers dans une architecture globale ?

Ce labo montre qu’une équipe Java peut transformer une simple application Spring Boot en **outil standardisé** pour GitHub Copilot, sans écrire de plugin spécifique, simplement en ajoutant Spring AI MCP et quelques annotations.
