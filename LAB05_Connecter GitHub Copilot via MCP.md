---
lab:
  title: "Labo - Connecter une application Java à GitHub Copilot via MCP dans VS Code"
  description: "Découvrir le Model Context Protocol (MCP) en connectant une application Java Spring Boot à GitHub Copilot dans VS Code, principalement par configuration (MCP servers) plutôt que par codage complexe."
  duration: "60-90 minutes"
---

# Labo - Connecter une application Java à GitHub Copilot via MCP dans VS Code

## Objectif du labo

Dans ce labo, vous allez découvrir comment **GitHub Copilot** peut consommer des services externes via le **Model Context Protocol (MCP)**, sans écrire beaucoup de code.

L’idée ici n’est pas de construire une application complexe, mais de :

- Démarrer une petite application Java Spring Boot qui expose une API très simple.
- La déclarer comme **serveur MCP** dans VS Code.
- Utiliser GitHub Copilot (via le bouton “Open in Agents” et la configuration des MCP servers) pour interagir avec cette application en langage naturel.

Ce labo montre comment transformer une application Java en **outil utilisable par Copilot**, et comment les développeurs peuvent tirer parti de MCP avec surtout de la **configuration** plutôt que du développement spécifique côté Copilot.

> Remarque : le code Java reste volontairement simple. Le focus pédagogique est sur la **configuration MCP dans VS Code** et l’usage côté développeur.

---

## 1. Pré-requis

Avant de commencer, assurez-vous d’avoir :

- Visual Studio Code à jour, avec :
  - GitHub Copilot,
  - le mode Agent activé,
  - le support MCP (MCP servers) disponible.
- Un abonnement GitHub Copilot actif.
- Java 17 ou plus (JDK).
- Maven installé et accessible dans le `PATH`.
- Extensions Java pour VS Code, par exemple :
  - “Extension Pack for Java”.
- Connaissances de base :
  - Java,
  - HTTP/REST,
  - usage basique de Git et Maven.

---

## 2. Créer l’application Java minimale

Dans cette partie, vous créez une petite application Java Spring Boot qui expose une API REST “Todo” très simple, que Copilot utilisera ensuite via MCP.

### 2.1. Créer le dossier de travail

1. Ouvrez un terminal.
2. Créez un nouveau dossier et initialisez Git :

   ```bash
   mkdir mcp-java-lab
   cd mcp-java-lab
   git init
   ```

3. Ouvrez le dossier dans VS Code :

   ```bash
   code .
   ```

### 2.2. Générer un projet Spring Boot minimal

Vous pouvez passer par Spring Initializr (web) si vous préférez, mais ici on reste sur Maven CLI pour garder un script reproductible.

1. Dans le terminal intégré VS Code, lancez :

   ```bash
   mvn archetype:generate \
     -DgroupId=com.example \
     -DartifactId=mcp-todo-app \
     -DarchetypeArtifactId=maven-archetype-quickstart \
     -DarchetypeVersion=1.5 \
     -DinteractiveMode=false
   ```

2. Déplacez-vous dans le projet :

   ```bash
   cd mcp-todo-app
   ```

3. Ouvrez `pom.xml` et remplacez le contenu par un pom Spring Boot minimal :

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>

       <groupId>com.example</groupId>
       <artifactId>mcp-todo-app</artifactId>
       <version>0.0.1-SNAPSHOT</version>
       <packaging>jar</packaging>

       <name>mcp-todo-app</name>

       <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>3.2.5</version>
           <relativePath/>
       </parent>

       <properties>
           <java.version>17</java.version>
       </properties>

       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>

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

4. Dans `src/main/java`, adaptez la structure de packages en créant :

   ```text
   src/main/java/com/example/mcp
   ```

5. Créez la classe `McpTodoApplication.java` :

   ```java
   package com.example.mcp;

   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;

   @SpringBootApplication
   public class McpTodoApplication {

       public static void main(String[] args) {
           SpringApplication.run(McpTodoApplication.class, args);
       }
   }
   ```

### 2.3. Ajouter une API REST Todo très simple

1. Créez `TodoController.java` :

   ```java
   package com.example.mcp;

   import org.springframework.web.bind.annotation.*;

   import java.util.ArrayList;
   import java.util.List;
   import java.util.concurrent.atomic.AtomicLong;

   @RestController
   @RequestMapping("/api/todos")
   public class TodoController {

       private final List<TodoItem> items = new ArrayList<>();
       private final AtomicLong idGenerator = new AtomicLong(1);

       @GetMapping
       public List<TodoItem> listTodos() {
           return items;
       }

       @PostMapping
       public TodoItem createTodo(@RequestBody TodoItemRequest request) {
           TodoItem item = new TodoItem();
           item.setId(idGenerator.getAndIncrement());
           item.setTitle(request.getTitle());
           item.setCompleted(false);
           items.add(item);
           return item;
       }

       @PostMapping("/{id}/complete")
       public TodoItem completeTodo(@PathVariable long id) {
           return items.stream()
               .filter(t -> t.getId() == id)
               .findFirst()
               .map(t -> {
                   t.setCompleted(true);
                   return t;
               })
               .orElseThrow(() -> new IllegalArgumentException("Todo not found: " + id));
       }
   }
   ```

2. Créez `TodoItem.java` :

   ```java
   package com.example.mcp;

   public class TodoItem {
       private long id;
       private String title;
       private boolean completed;

       public long getId() {
           return id;
       }

       public void setId(long id) {
           this.id = id;
       }

       public String getTitle() {
           return title;
       }

       public void setTitle(String title) {
           this.title = title;
       }

       public boolean isCompleted() {
           return completed;
       }

       public void setCompleted(boolean completed) {
           this.completed = completed;
       }
   }
   ```

3. Créez `TodoItemRequest.java` :

   ```java
   package com.example.mcp;

   public class TodoItemRequest {
       private String title;

       public String getTitle() {
           return title;
       }

       public void setTitle(String title) {
           this.title = title;
       }
   }
   ```

4. Testez rapidement l’application :

   ```bash
   mvn spring-boot:run
   ```

   - `GET http://localhost:8080/api/todos` doit renvoyer une liste vide.
   - `POST http://localhost:8080/api/todos` avec `{ "title": "Préparer la démo MCP" }` doit créer une tâche.

Si tout fonctionne, vous avez une petite API REST prête à être utilisée comme backend pour MCP.

---

## 3. Comprendre MCP côté Copilot (contexte)

Avant de configurer quoi que ce soit, resituons MCP :

- **MCP (Model Context Protocol)** est un protocole standard qui permet à des modèles comme GitHub Copilot d’accéder à des **outils externes** (APIs, services, données) via une interface uniformisée.
- Dans VS Code, Copilot peut utiliser des **MCP servers** : chaque serveur expose des “tools” que Copilot peut appeler depuis la vue Agents / Chat.
- L’intérêt est de ne plus coder un plugin Copilot spécifique : une application Java existante peut devenir un outil accessible aux développeurs via Copilot.

Dans ce labo, on va rester sur un flux **centré UI** :

- ouvrir la vue Agents,
- utiliser le bouton/icone pour configurer les MCP servers,
- sans écrire à la main des fichiers de configuration complexes.

---

## 4. Ajouter le serveur MCP dans VS Code (via l’interface)

L’idée est d’enregistrer votre service Java en tant que MCP server dans VS Code, en passant par l’interface graphicale d’Agents.

> Attention : selon la version de VS Code, les menus peuvent légèrement varier, mais la logique reste la même.

### 4.1. Ouvrir la vue Agents / Copilot

1. Assurez-vous que votre application Java tourne toujours dans un terminal (`mvn spring-boot:run`).
2. Dans VS Code, ouvrez le panneau GitHub Copilot / Chat.
3. Basculez en **Agent mode** si ce n’est pas déjà le cas (via le sélecteur de mode dans la zone de saisie).
4. Cliquez sur l’icône ou le lien **Open in Agents** (ou équivalent) pour ouvrir la vue dédiée aux agents et aux outils (MCP inclus).

### 4.2. Ajouter un MCP server sans éditer de JSON à la main

1. Dans la vue Agents, repérez le bouton ou menu associé aux outils, par exemple une icône d’engrenage ou de clé à molette, proche de “Tools”.
2. Cliquez dessus et cherchez une option du type :

   - “Ajouter plus d’outils…”
   - puis “Ajouter un serveur MCP”,
   - ou “MCP: Add Server”.

3. Quand l’assistant vous le propose, choisissez le **type** de serveur :

   - HTTP / SSE (HTTP ou Server-Sent Events), si vous avez un endpoint MCP HTTP,
   - ou Command (stdio) si vous lancez un serveur MCP en local via une commande.

4. Dans ce labo, l’idée est de montrer le flux UI, donc vous pouvez :

   - soit utiliser un starter MCP Java/Spring Alert déjà fourni (si disponible dans votre environnement),
   - soit simplement simuler un serveur MCP en suivant un exemple existant (par exemple un starter Spring AI MCP).

Si vous utilisez un starter qui expose un endpoint du type :

```text
http://localhost:8080/sse
```

alors :

- renseignez cette URL comme **URL du serveur**,
- donnez un **nom/ID de serveur** parlant, par exemple `java-todo-mcp`,
- choisissez de sauvegarder la configuration au **niveau workspace**, ce qui créera un `.vscode/mcp.json` sans que vous ayez à le modifier.

5. Validez. VS Code va alors :

- enregistrer ce serveur MCP,
- l’afficher dans la liste des serveurs disponibles pour les agents,
- et vous demander, lors du premier appel, de confirmer les permissions.

---

## 5. Utiliser le serveur MCP depuis Copilot (focus sur l’UI)

Maintenant que le serveur MCP est configuré, vous allez l’utiliser directement depuis Copilot, en langage naturel.

### 5.1. Vérifier que le serveur MCP est actif

1. Dans la vue Agents / Chat :

   - assurez-vous d’être en **Agent mode**,
   - ouvrez le sélecteur de “Tools” (outils) associé à la zone de saisie.

2. Vérifiez que votre serveur, par exemple `java-todo-mcp`, est listé parmi les MCP servers :

   - si ce n’est pas le cas, vérifiez que :
     - l’application Java tourne,
     - la configuration MCP a bien été enregistrée,
     - vous êtes dans le bon workspace.

3. Activez les outils de ce serveur pour la session courante (case à cocher ou équivalent).

### 5.2. Scénarios de requêtes naturelles

Dans la zone de saisie de Copilot (Agent mode) :

1. Envoyez une première requête :

   ```text
   Utilise le serveur MCP \"java-todo-mcp\" pour lister toutes les tâches disponibles.
   ```

2. Observez le comportement :

   - Copilot devrait détecter qu’il doit appeler le tool MCP associé,
   - exécuter un appel vers votre serveur,
   - et afficher la réponse (une liste de tâches, souvent un tableau JSON ou équivalent formaté).

3. Créez une nouvelle tâche :

   ```text
   Crée une nouvelle tâche via le serveur MCP \"java-todo-mcp\" avec le titre \"Préparer le rapport de sprint\".
   ```

4. Vérifiez ensuite l’état :

   ```text
   Affiche à nouveau la liste des tâches via le serveur MCP \"java-todo-mcp\".
   ```

5. Marquez une tâche comme terminée :

   ```text
   Marque la tâche avec l'identifiant 1 comme terminée en utilisant le serveur MCP \"java-todo-mcp\".
   ```

Le but est que les stagiaires perçoivent bien :

- qu’ils ne tapent aucune commande HTTP ou curl,
- qu’ils pilotent un service Java via Copilot et MCP,
- et que tout cela se fait depuis la même interface de chat.

---

## 6. Discussion : pourquoi MCP est utile pour une équipe Java

Prenez quelques minutes pour discuter (ou faire réfléchir les participants) sur les points suivants :

- En quoi ce pattern est différent d’un simple “client HTTP + API REST” ?
- Quels types de services internes d’une entreprise Java pourraient être exposés via MCP :
  - référentiels internes,
  - systèmes de tickets,
  - services de configuration,
  - monitoring, etc. ?
- Quels gains pour :
  - les développeurs,
  - les SRE / ops,
  - le support ?

L’idée à faire passer :

- MCP permet de **factoriser l’intégration** d’un service interne : une fois exposé, il devient disponible pour tous les agents Copilot, dans VS Code, sans recoder une intégration pour chaque équipe.
- Les développeurs peuvent rester dans leur IDE, en bénéficiant d’un assistant qui sait parler à leurs outils internes.

---

## 7. Nettoyage

Si ce labo est réalisé sur un environnement temporaire :

1. Arrêtez l’application Java :

   - dans le terminal où tourne `mvn spring-boot:run`, tapez `Ctrl+C`.

2. Fermez VS Code si vous avez terminé.

3. Supprimez le dossier `mcp-java-lab` si vous n’en avez plus besoin.

4. Dans VS Code, vous pouvez supprimer le serveur MCP configuré si vous ne voulez plus qu’il apparaisse dans la liste (via la même interface “Tools / MCP servers”).

---

## Récapitulatif

Dans ce labo, vous avez :

- Créé une petite application Java Spring Boot avec une API REST Todo.
- Enregistré cette application comme **serveur MCP** dans VS Code en utilisant principalement l’interface (Open in Agents, ajout de MCP servers).
- Utilisé GitHub Copilot pour appeler votre application en langage naturel, sans écrire de code spécifique côté Copilot.
- Compris comment MCP permet d’exposer des services Java existants comme outils standardisés pour les agents Copilot.

Ce modèle est réutilisable dans vos projets : toute application Java exposée via MCP peut devenir un composant accessible et orchestrable par Copilot, pour construire de véritables workflows d’ingénierie augmentée.
