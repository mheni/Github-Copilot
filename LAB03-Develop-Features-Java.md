# LAB 03 - Développer des fonctionnalités de code avec GitHub Copilot (Java)

## Description
Apprendre à utiliser GitHub Copilot pour développer des fonctionnalités complètes dans une application Java.

**Durée estimée** : 35 minutes

---

## Objectifs d'apprentissage

À la fin de cet exercice, vous serez capable de :
- Utiliser GitHub Copilot pour générer des classes Java complètes
- Créer des méthodes métier avec GitHub Copilot
- Générer des tests unitaires JUnit automatiquement
- Utiliser les différents modes de GitHub Copilot (suggestions inline, chat, agent)

---

## Prérequis

- JDK 17+, Maven 3.8+, Git installés
- Visual Studio Code avec Extension Pack for Java
- GitHub Copilot activé
- Environnement configuré (voir LAB 00)

---

## Scénario de l'exercice

Vous êtes développeur dans une équipe qui construit un système de gestion de bibliothèque. Votre tâche est de développer les fonctionnalités suivantes avec l'aide de GitHub Copilot :

1. Une classe `Book` pour représenter un livre
2. Une classe `Library` pour gérer une collection de livres
3. Des méthodes pour ajouter, rechercher et emprunter des livres
4. Des tests unitaires JUnit pour valider le comportement

---

## Partie 1 : Créer le projet Maven

### Étape 1 : Créer la structure du projet

```bash
mkdir library-management
cd library-management
```

### Étape 2 : Créer le fichier `pom.xml`

Créez un fichier `pom.xml` à la racine du projet :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>library-management</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.1</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.2</version>
            </plugin>
        </plugins>
    </build>
</project>
```

### Étape 3 : Créer la structure des dossiers

```bash
mkdir -p src/main/java/com/example/library
mkdir -p src/test/java/com/example/library
```

---

## Partie 2 : Développer la classe Book avec GitHub Copilot

### Étape 1 : Créer le fichier Book.java

1. Dans VS Code, créez `src/main/java/com/example/library/Book.java`

2. Commencez par un commentaire descriptif :

```java
package com.example.library;

// Classe représentant un livre avec ISBN, titre, auteur, année de publication et statut d'emprunt
```

3. Appuyez sur **Entrée** et laissez GitHub Copilot suggérer la classe complète

### Résultat attendu

GitHub Copilot devrait générer quelque chose comme :

```java
package com.example.library;

public class Book {
    private String isbn;
    private String title;
    private String author;
    private int publicationYear;
    private boolean isBorrowed;

    public Book(String isbn, String title, String author, int publicationYear) {
        this.isbn = isbn;
        this.title = title;
        this.author = author;
        this.publicationYear = publicationYear;
        this.isBorrowed = false;
    }

    // Getters
    public String getIsbn() {
        return isbn;
    }

    public String getTitle() {
        return title;
    }

    public String getAuthor() {
        return author;
    }

    public int getPublicationYear() {
        return publicationYear;
    }

    public boolean isBorrowed() {
        return isBorrowed;
    }

    // Méthodes métier
    public void borrow() {
        this.isBorrowed = true;
    }

    public void returnBook() {
        this.isBorrowed = false;
    }

    @Override
    public String toString() {
        return String.format("Book{isbn='%s', title='%s', author='%s', year=%d, borrowed=%b}",
                isbn, title, author, publicationYear, isBorrowed);
    }
}
```

###  Vérification

- [ ] La classe a tous les attributs nécessaires
- [ ] Les getters sont générés
- [ ] Les méthodes `borrow()` et `returnBook()` existent
- [ ] La méthode `toString()` est implémentée

---

## Partie 3 : Développer la classe Library avec GitHub Copilot

### Étape 1 : Utiliser GitHub Copilot Chat

1. Ouvrez GitHub Copilot Chat (Ctrl+Shift+I)

2. Tapez le prompt suivant :

```plaintext
Crée une classe Library en Java qui :
- Stocke une liste de livres (ArrayList<Book>)
- Permet d'ajouter un livre (addBook)
- Permet de rechercher un livre par ISBN (findByIsbn)
- Permet de rechercher des livres par auteur (findByAuthor)
- Permet d'emprunter un livre par ISBN (borrowBook)
- Permet de retourner un livre par ISBN (returnBook)
- Gère les exceptions si le livre n'est pas trouvé ou déjà emprunté
```

### Étape 2 : Créer le fichier Library.java

Créez `src/main/java/com/example/library/Library.java` et copiez le code généré.

### Résultat attendu

```java
package com.example.library;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

public class Library {
    private List<Book> books;

    public Library() {
        this.books = new ArrayList<>();
    }

    public void addBook(Book book) {
        if (book == null) {
            throw new IllegalArgumentException("Book cannot be null");
        }
        books.add(book);
    }

    public Book findByIsbn(String isbn) {
        return books.stream()
                .filter(book -> book.getIsbn().equals(isbn))
                .findFirst()
                .orElse(null);
    }

    public List<Book> findByAuthor(String author) {
        return books.stream()
                .filter(book -> book.getAuthor().equalsIgnoreCase(author))
                .collect(Collectors.toList());
    }

    public void borrowBook(String isbn) {
        Book book = findByIsbn(isbn);
        if (book == null) {
            throw new IllegalArgumentException("Book with ISBN " + isbn + " not found");
        }
        if (book.isBorrowed()) {
            throw new IllegalStateException("Book is already borrowed");
        }
        book.borrow();
    }

    public void returnBook(String isbn) {
        Book book = findByIsbn(isbn);
        if (book == null) {
            throw new IllegalArgumentException("Book with ISBN " + isbn + " not found");
        }
        if (!book.isBorrowed()) {
            throw new IllegalStateException("Book is not currently borrowed");
        }
        book.returnBook();
    }

    public int getTotalBooks() {
        return books.size();
    }

    public int getAvailableBooks() {
        return (int) books.stream()
                .filter(book -> !book.isBorrowed())
                .count();
    }
}
```

---

## Partie 4 : Générer des tests unitaires avec GitHub Copilot

### Étape 1 : Créer la classe de test BookTest

1. Créez `src/test/java/com/example/library/BookTest.java`

2. Utilisez GitHub Copilot Chat avec ce prompt :

```plaintext
Génère des tests JUnit 5 pour la classe Book qui testent :
- Le constructeur
- Les getters
- Les méthodes borrow() et returnBook()
- Le comportement de isBorrowed()
```

### Résultat attendu

```java
package com.example.library;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class BookTest {

    @Test
    void testConstructor() {
        Book book = new Book("978-0134685991", "Effective Java", "Joshua Bloch", 2018);
        
        assertEquals("978-0134685991", book.getIsbn());
        assertEquals("Effective Java", book.getTitle());
        assertEquals("Joshua Bloch", book.getAuthor());
        assertEquals(2018, book.getPublicationYear());
        assertFalse(book.isBorrowed());
    }

    @Test
    void testBorrow() {
        Book book = new Book("978-0134685991", "Effective Java", "Joshua Bloch", 2018);
        
        book.borrow();
        
        assertTrue(book.isBorrowed());
    }

    @Test
    void testReturnBook() {
        Book book = new Book("978-0134685991", "Effective Java", "Joshua Bloch", 2018);
        book.borrow();
        
        book.returnBook();
        
        assertFalse(book.isBorrowed());
    }

    @Test
    void testToString() {
        Book book = new Book("978-0134685991", "Effective Java", "Joshua Bloch", 2018);
        
        String result = book.toString();
        
        assertTrue(result.contains("978-0134685991"));
        assertTrue(result.contains("Effective Java"));
        assertTrue(result.contains("Joshua Bloch"));
    }
}
```

### Étape 2 : Créer la classe de test LibraryTest

Créez `src/test/java/com/example/library/LibraryTest.java` et utilisez ce prompt :

```plaintext
Génère des tests JUnit 5 pour la classe Library qui testent :
- L'ajout de livres
- La recherche par ISBN
- La recherche par auteur
- L'emprunt et le retour de livres
- Les exceptions lancées
- Le comptage des livres disponibles
```

### Résultat attendu

```java
package com.example.library;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import java.util.List;
import static org.junit.jupiter.api.Assertions.*;

class LibraryTest {

    private Library library;
    private Book book1;
    private Book book2;

    @BeforeEach
    void setUp() {
        library = new Library();
        book1 = new Book("978-0134685991", "Effective Java", "Joshua Bloch", 2018);
        book2 = new Book("978-0596009205", "Head First Java", "Kathy Sierra", 2005);
    }

    @Test
    void testAddBook() {
        library.addBook(book1);
        
        assertEquals(1, library.getTotalBooks());
    }

    @Test
    void testAddNullBook() {
        assertThrows(IllegalArgumentException.class, () -> library.addBook(null));
    }

    @Test
    void testFindByIsbn() {
        library.addBook(book1);
        library.addBook(book2);
        
        Book found = library.findByIsbn("978-0134685991");
        
        assertNotNull(found);
        assertEquals("Effective Java", found.getTitle());
    }

    @Test
    void testFindByIsbnNotFound() {
        library.addBook(book1);
        
        Book found = library.findByIsbn("999-9999999999");
        
        assertNull(found);
    }

    @Test
    void testFindByAuthor() {
        library.addBook(book1);
        library.addBook(book2);
        
        List<Book> books = library.findByAuthor("Joshua Bloch");
        
        assertEquals(1, books.size());
        assertEquals("Effective Java", books.get(0).getTitle());
    }

    @Test
    void testBorrowBook() {
        library.addBook(book1);
        
        library.borrowBook("978-0134685991");
        
        assertTrue(book1.isBorrowed());
    }

    @Test
    void testBorrowNonExistentBook() {
        assertThrows(IllegalArgumentException.class, 
            () -> library.borrowBook("999-9999999999"));
    }

    @Test
    void testBorrowAlreadyBorrowedBook() {
        library.addBook(book1);
        library.borrowBook("978-0134685991");
        
        assertThrows(IllegalStateException.class, 
            () -> library.borrowBook("978-0134685991"));
    }

    @Test
    void testReturnBook() {
        library.addBook(book1);
        library.borrowBook("978-0134685991");
        
        library.returnBook("978-0134685991");
        
        assertFalse(book1.isBorrowed());
    }

    @Test
    void testGetAvailableBooks() {
        library.addBook(book1);
        library.addBook(book2);
        library.borrowBook("978-0134685991");
        
        assertEquals(1, library.getAvailableBooks());
    }
}
```

---

## Partie 5 : Exécuter les tests

### Étape 1 : Compiler et tester

```bash
mvn clean test
```

### Résultat attendu

```plaintext
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.example.library.BookTest
[INFO] Tests run: 4, Failures: 0, Errors: 0, Skipped: 0
[INFO] Running com.example.library.LibraryTest
[INFO] Tests run: 10, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 14, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] BUILD SUCCESS
```

###  Vérification

- [ ] Tous les tests passent (14/14)
- [ ] Aucune erreur de compilation
- [ ] Le code est bien structuré et lisible

---

## Partie 6 : Créer une application de démonstration

### Étape 1 : Utiliser GitHub Copilot pour générer Main

Créez `src/main/java/com/example/library/LibraryApp.java` et tapez :

```java
package com.example.library;

// Application de démonstration pour la bibliothèque
// Créer une bibliothèque, ajouter des livres, effectuer des opérations
```

Laissez Copilot compléter.

### Résultat attendu

```java
package com.example.library;

public class LibraryApp {
    public static void main(String[] args) {
        Library library = new Library();

        // Ajouter des livres
        library.addBook(new Book("978-0134685991", "Effective Java", "Joshua Bloch", 2018));
        library.addBook(new Book("978-0596009205", "Head First Java", "Kathy Sierra", 2005));
        library.addBook(new Book("978-0132350884", "Clean Code", "Robert C. Martin", 2008));

        System.out.println("=== Bibliothèque initialisée ===");
        System.out.println("Total des livres : " + library.getTotalBooks());
        System.out.println("Livres disponibles : " + library.getAvailableBooks());

        // Rechercher un livre
        Book book = library.findByIsbn("978-0134685991");
        if (book != null) {
            System.out.println("\nLivre trouvé : " + book);
        }

        // Emprunter un livre
        System.out.println("\n=== Emprunt d'un livre ===");
        library.borrowBook("978-0134685991");
        System.out.println("Livre emprunté : " + book);
        System.out.println("Livres disponibles : " + library.getAvailableBooks());

        // Retourner un livre
        System.out.println("\n=== Retour d'un livre ===");
        library.returnBook("978-0134685991");
        System.out.println("Livre retourné : " + book);
        System.out.println("Livres disponibles : " + library.getAvailableBooks());

        // Rechercher par auteur
        System.out.println("\n=== Recherche par auteur ===");
        var booksByAuthor = library.findByAuthor("Joshua Bloch");
        System.out.println("Livres de Joshua Bloch : " + booksByAuthor.size());
    }
}
```

### Étape 2 : Exécuter l'application

```bash
mvn compile exec:java -Dexec.mainClass="com.example.library.LibraryApp"
```

### Sortie attendue

```plaintext
=== Bibliothèque initialisée ===
Total des livres : 3
Livres disponibles : 3

Livre trouvé : Book{isbn='978-0134685991', title='Effective Java', author='Joshua Bloch', year=2018, borrowed=false}

=== Emprunt d'un livre ===
Livre emprunté : Book{isbn='978-0134685991', title='Effective Java', author='Joshua Bloch', year=2018, borrowed=true}
Livres disponibles : 2

=== Retour d'un livre ===
Livre retourné : Book{isbn='978-0134685991', title='Effective Java', author='Joshua Bloch', year=2018, borrowed=false}
Livres disponibles : 3

=== Recherche par auteur ===
Livres de Joshua Bloch : 1
```

---

## Récapitulatif

### Ce que vous avez appris

 Générer des classes Java complètes avec GitHub Copilot  
 Utiliser les commentaires pour guider Copilot  
 Utiliser GitHub Copilot Chat pour des tâches complexes  
 Générer des tests unitaires JUnit automatiquement  
 Créer une application fonctionnelle complète  

### Bonnes pratiques

1. **Commentaires descriptifs** : Plus vos commentaires sont précis, meilleures sont les suggestions
2. **Validation** : Toujours vérifier et tester le code généré
3. **Tests** : Générer les tests en même temps que le code métier
4. **Iteratif** : Affiner les suggestions avec des prompts plus précis si nécessaire

---

## Défis supplémentaires (optionnel)

Si vous avez terminé, essayez ces extensions :

1. Ajouter une classe `Member` pour représenter les membres de la bibliothèque
2. Implémenter un système de réservation de livres
3. Ajouter une limite de livres empruntés par membre
4. Créer un rapport des livres les plus empruntés
5. Implémenter la sérialisation JSON des livres avec Jackson

**Astuce** : Utilisez GitHub Copilot Chat pour chaque nouvelle fonctionnalité !

---

## Ressources

- Documentation JUnit 5 : https://junit.org/junit5/docs/current/user-guide/
- Maven Getting Started : https://maven.apache.org/guides/getting-started/
- GitHub Copilot Java : https://github.blog/developer-skills/github/how-to-use-github-copilot-in-your-java-ide/
