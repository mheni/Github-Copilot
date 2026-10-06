---
lab:
  title: "Labo - Documenter et moderniser un module Java avec GitHub Copilot"
  description: "Utiliser GitHub Copilot dans VS Code pour comprendre, documenter, refactorer et tester un module Java existant, sans agents ni MCP."
  duration: "60-90 minutes"
---

# Labo - Documenter et moderniser un module Java avec GitHub Copilot

## Objectif du labo

Dans ce labo, vous allez apprendre à exploiter **GitHub Copilot** dans VS Code pour travailler sur un module Java existant :

- comprendre rapidement ce que fait un code "legacy" ou peu documenté ;
- générer de la **documentation** (Javadoc, README, synthèse architecturale) ;
- proposer et appliquer des **refactorings** ciblés ;
- générer des **tests unitaires JUnit 5** pour sécuriser les changements.

Contrairement aux labs orientés **Agents** ou **MCP**, ici vous n’utilisez que :

- les suggestions inline de Copilot (complétion dans l’éditeur) ;
- **GitHub Copilot Chat** dans VS Code.

L’objectif est de montrer comment Copilot s’intègre dans un flux de développement Java classique.

---

## 1. Pré-requis

Avant de commencer, assurez-vous d’avoir :

- Visual Studio Code à jour.
- Extension GitHub Copilot installée et connectée.
- Un abonnement GitHub Copilot actif.
- Java 17 ou plus (JDK).
- Maven installé et accessible dans le `PATH`.
- Extensions Java pour VS Code, par exemple :
  - "Extension Pack for Java".

Connaissances souhaitées :

- bases de Java (classes, méthodes, exceptions) ;
- bases de Maven ;
- notions simples de tests unitaires (JUnit).

---

## 2. Mise en place du projet Java "legacy"

Dans cette partie, vous allez créer un petit module Java volontairement imparfait, qui servira de base pour le travail avec Copilot.

### 2.1. Créer le projet

1. Ouvrez un terminal.
2. Créez un dossier de travail et initialisez Git :

   ```bash
   mkdir copilot-legacy-java-lab
   cd copilot-legacy-java-lab
   git init
   ```

3. Ouvrez le dossier dans VS Code :

   ```bash
   code .
   ```

4. Dans le terminal intégré, générez un projet Maven simple :

   ```bash
   mvn archetype:generate \
     -DgroupId=com.example \
     -DartifactId=legacy-order-service \
     -DarchetypeArtifactId=maven-archetype-quickstart \
     -DarchetypeVersion=1.4 \
     -DinteractiveMode=false
   ```

5. Déplacez-vous dans le sous-dossier du projet :

   ```bash
   cd legacy-order-service
   ```

### 2.2. Adapter le pom.xml pour JUnit 5

1. Ouvrez `pom.xml` et remplacez la section `<dependencies>` par :

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

2. Sauvegardez le fichier.

### 2.3. Créer une classe "legacy" à améliorer

Nous allons créer un service `OrderService` volontairement imparfait :

- responsabilités mélangées ;
- pas de Javadoc ;
- logique de validation peu claire.

1. Dans `src/main/java`, créez le package :

   ```text
   src/main/java/com/example/legacy
   ```

2. Créez la classe `OrderService.java` :

   ```java
   package com.example.legacy;

   import java.math.BigDecimal;
   import java.time.LocalDateTime;
   import java.util.ArrayList;
   import java.util.List;

   public class OrderService {

       private final List<Order> orders = new ArrayList<>();

       public Order createOrder(String customerId, BigDecimal amount) {
           if (customerId == null || customerId.isBlank()) {
               throw new IllegalArgumentException("customerId is required");
           }

           if (amount == null) {
               throw new IllegalArgumentException("amount is required");
           }

           if (amount.compareTo(BigDecimal.ZERO) <= 0) {
               throw new IllegalArgumentException("amount must be positive");
           }

           Order order = new Order();
           order.setId("ORD-" + (orders.size() + 1));
           order.setCustomerId(customerId.trim());
           order.setAmount(amount);
           order.setCreatedAt(LocalDateTime.now());
           order.setStatus("CREATED");

           orders.add(order);

           if (amount.compareTo(new BigDecimal("1000")) > 0) {
               order.setStatus("REVIEW");
           }

           return order;
       }

       public List<Order> findOrdersByCustomer(String customerId) {
           List<Order> result = new ArrayList<>();
           for (Order order : orders) {
               if (order.getCustomerId().equals(customerId)) {
                   result.add(order);
               }
           }
           return result;
       }

       public void cancelOrder(String orderId) {
           for (Order order : orders) {
               if (order.getId().equals(orderId)) {
                   if ("CANCELLED".equals(order.getStatus())) {
                       throw new IllegalStateException("Order already cancelled");
                   }
                   if ("SHIPPED".equals(order.getStatus())) {
                       throw new IllegalStateException("Cannot cancel shipped order");
                   }
                   order.setStatus("CANCELLED");
                   return;
               }
           }
           throw new IllegalArgumentException("Order not found: " + orderId);
       }
   }
   ```

3. Créez la classe `Order.java` :

   ```java
   package com.example.legacy;

   import java.math.BigDecimal;
   import java.time.LocalDateTime;

   public class Order {

       private String id;
       private String customerId;
       private BigDecimal amount;
       private String status;
       private LocalDateTime createdAt;

       public String getId() {
           return id;
       }

       public void setId(String id) {
           this.id = id;
       }

       public String getCustomerId() {
           return customerId;
       }

       public void setCustomerId(String customerId) {
           this.customerId = customerId;
       }

       public BigDecimal getAmount() {
           return amount;
       }

       public void setAmount(BigDecimal amount) {
           this.amount = amount;
       }

       public String getStatus() {
           return status;
       }

       public void setStatus(String status) {
           this.status = status;
       }

       public LocalDateTime getCreatedAt() {
           return createdAt;
       }

       public void setCreatedAt(LocalDateTime createdAt) {
           this.createdAt = createdAt;
       }
   }
   ```

4. Vérifiez que le projet compile :

   ```bash
   mvn -q test
   ```

---

## 3. Comprendre le code avec GitHub Copilot Chat

Objectif : utiliser Copilot pour **comprendre** ce module, comme si vous arriviez sur un projet legacy.

### 3.1. Expliquer la classe `OrderService`

1. Ouvrez `OrderService.java` dans VS Code.
2. Sélectionnez l’ensemble de la classe.
3. Ouvrez **GitHub Copilot Chat**.
4. Vérifiez que la sélection est bien prise en compte comme contexte.
5. Posez la question suivante :

   ```text
   Explique ce que fait cette classe OrderService :
   - quelles sont ses responsabilités principales ?
   - comment les commandes sont créées, recherchées et annulées ?
   - quels sont les cas d'erreur possibles ?
   ```

6. Lisez la réponse et comparez-la à votre propre compréhension.

### 3.2. Générer un résumé technique

Toujours dans le chat, demandez :

```text
Rédige un court résumé technique (5 à 8 lignes) de ce module OrderService/Order,
comme si tu devais le documenter dans un fichier ARCHITECTURE.md.
Inclue : rôle, données manipulées, règles métier clés.
```

Créez un fichier `ARCHITECTURE.md` à la racine du projet et collez le résumé généré, en l’ajustant si besoin.

---

## 4. Générer de la documentation (Javadoc et README)

### 4.1. Générer la Javadoc avec Copilot

1. Placez le curseur au-dessus de la méthode `createOrder` dans `OrderService`.
2. Tapez `/**` puis Entrée pour laisser Copilot proposer un bloc Javadoc.
3. Si Copilot ne propose rien, utilisez Copilot Chat :

   ```text
   Génère une Javadoc complète pour la méthode createOrder, en décrivant
   les paramètres, la valeur de retour, les exceptions levées et les règles métier.
   ```

4. Vérifiez et ajustez la Javadoc proposée.
5. Répétez pour :
   - `findOrdersByCustomer` ;
   - `cancelOrder` ;
   - la classe `Order` (rôle, champs principaux).

### 4.2. Créer un README technique minimal

1. Créez un fichier `README.md` à la racine du projet.
2. Dans Copilot Chat, demandez :

   ```text
   À partir du code de OrderService et de ta compréhension, rédige un README technique concis pour ce module :
   - Contexte et rôle du service
   - Principales opérations
   - Contraintes et règles métier importantes
   ```

3. Collez et ajustez le texte généré dans `README.md`.

Objectif : montrer que Copilot peut produire une première base documentaire en quelques minutes, que vous pouvez ensuite relire et enrichir.

---

## 5. Identifier des améliorations et refactorings

Maintenant que le code est mieux compris, vous allez demander à Copilot de vous aider à l’améliorer.

### 5.1. Demander un plan de refactoring

Dans Copilot Chat (avec `OrderService.java` ouvert) :

```text
Analyse la classe OrderService et propose un plan de refactoring :
- points qui nuisent à la lisibilité ou à la maintenabilité
- améliorations possibles (extraction de méthodes, renommage, validations regroupées, etc.)
- risques à surveiller pendant le refactoring
Présente le plan sous forme de liste structurée.
```

Lisez la proposition de Copilot et choisissez **une ou deux améliorations simples** à mettre en œuvre (par exemple, extraire une méthode de validation des paramètres).

### 5.2. Appliquer un refactoring avec l’aide de Copilot

1. Sélectionnez le bloc de validation dans `createOrder`.
2. Demandez à Copilot (inline ou via chat) :

   ```text
   Propose une méthode privée pour extraire cette logique de validation des paramètres,
   avec un nom clair et des exceptions cohérentes.
   ```

3. Acceptez ou adaptez la proposition (nouvelle méthode privée `validateCreateOrderInput(...)`, par exemple).
4. Répétez l’exercice pour un autre petit refactoring si le temps le permet (par exemple, rendre la logique de changement de statut plus explicite).

> Important : insistez auprès des participants sur le fait que **Copilot propose**, mais que c’est au développeur de décider.

---

## 6. Générer des tests unitaires JUnit 5

Objectif : utiliser Copilot pour créer rapidement une première suite de tests unitaires, puis la renforcer.

### 6.1. Créer la structure de tests

1. Dans `src/test/java`, créez le package :

   ```text
   src/test/java/com/example/legacy
   ```

2. Créez la classe de test `OrderServiceTest.java` (avec un squelette vide) :

   ```java
   package com.example.legacy;

   import org.junit.jupiter.api.Test;

   class OrderServiceTest {

       @Test
       void dummy() {
           // sera remplacé par de vrais tests
       }
   }
   ```

### 6.2. Demander à Copilot de générer des tests de base

1. Ouvrez `OrderServiceTest.java` et `OrderService.java` côte à côte.
2. Dans Copilot Chat, demandez :

   ```text
   Génère une suite de tests JUnit 5 pour OrderService qui couvre au minimum :
   - la création de commande valide
   - le rejet des montants négatifs ou nuls
   - le rejet d'un customerId vide ou null
   - le passage en statut REVIEW pour les montants > 1000
   - la recherche de commandes par client
   - l'annulation de commande dans les cas valides et invalides
   Utilise une nouvelle instance d'OrderService par test.
   ```

3. Copilot devrait proposer un contenu pour `OrderServiceTest`. Copiez-le dans le fichier, relisez et ajustez si nécessaire.
4. Exécutez les tests :

   ```bash
   mvn test
   ```

5. Corrigez les éventuelles erreurs avec l’aide de Copilot Chat, par exemple :

   ```text
   Voici l'erreur de test. Explique-la et propose une correction du test ou du code.
   ```

### 6.3. Ajouter des cas limites supplémentaires

Toujours dans Copilot Chat :

```text
Propose des cas de test supplémentaires pour couvrir des scénarios limites ou d'erreur
qui ne sont pas encore testés (par exemple, annulation répétée, client avec plusieurs commandes, etc.).
Ajoute ces tests dans OrderServiceTest.
```

Copiez les tests proposés et ajustez-les si besoin.

Relancez les tests pour vérifier que tout est vert.

---

## 7. Synthèse et discussion

Prenez quelques minutes pour faire le point :

1. Qu’est-ce que Copilot a apporté sur :
   - la compréhension du code ;
   - la documentation ;
   - le refactoring ;
   - les tests ?
2. Où Copilot a-t-il été très utile ?
3. Où faut-il rester particulièrement vigilant (logique métier, règles de sécurité, performance) ?
4. Comment intégrer ce type d’usage dans votre flux de travail quotidien (revues de code, tâches de refactoring, amélioration continue) ?



---

