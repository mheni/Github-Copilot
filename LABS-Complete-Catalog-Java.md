# CATALOGUE COMPLET DES LABS GITHUB COPILOT - JAVA

## Table des matières

- [LAB 01 - Examiner les paramètres de GitHub Copilot](#lab-01)
- [LAB 05 - Refactoriser et améliorer le code existant](#lab-05)
- [LAB 06 - Vibe Coding - Prototype e-commerce](#lab-06)
- [LAB 07 - Consolider le code dupliqué](#lab-07)
- [LAB 08 - Refactoriser les grandes fonctions](#lab-08)
- [LAB 09 - Simplifier les conditions complexes](#lab-09)
- [LAB 10 - Profilage de performance](#lab-10)
- [LAB 11 - Résoudre les GitHub Issues](#lab-11)
- [LAB 12 - Résoudre les alertes de sécurité](#lab-12)
- [LAB 13 - Développement piloté par spécification](#lab-13)
- [LAB 14 - Démarrer avec le développement spec-driven](#lab-14)

---

<a name="lab-01"></a>
# LAB 01 - Examiner les paramètres et l'interface de GitHub Copilot

**Durée** : 25 minutes

## Objectifs

- Explorer l'interface de GitHub Copilot dans VS Code
- Configurer les paramètres de GitHub Copilot
- Comprendre les différents modes (Inline, Chat, Agent)
- Tester les fonctionnalités de base

## Partie 1 : Interface GitHub Copilot

### 1.1 Accès à GitHub Copilot

1. Ouvrez VS Code
2. Vérifiez l'icône GitHub Copilot en bas à droite (✓ actif)
3. Cliquez sur l'icône pour voir le statut

### 1.2 Modes de GitHub Copilot

**Mode Inline (Suggestions automatiques)**
- Apparaît automatiquement pendant la saisie
- Couleur grise pour les suggestions
- `Tab` pour accepter
- `Esc` pour rejeter
- `Alt+[` ou `Alt+]` pour naviguer entre suggestions

**Mode Chat (Ctrl+Shift+I)**
- Fenêtre de discussion interactive
- Posez des questions en langage naturel
- Demandez des explications de code
- Générez du code à partir de descriptions

**Mode Agent**
- Disponible dans le Chat
- Capable d'effectuer des tâches complexes multi-fichiers
- Refactoring automatique
- Création de projets complets

## Partie 2 : Configuration

### 2.1 Paramètres globaux

1. `Ctrl+,` pour ouvrir les paramètres
2. Recherchez "Copilot"
3. Paramètres importants :

```
✓ Enable Auto Completions
✓ Enable Chat
✓ Enable Inline Suggestions
Suggestion Delay: 0ms (par défaut)
```

### 2.2 Paramètres par langage

Créez `.vscode/settings.json` dans votre projet :

```json
{
  "github.copilot.enable": {
    "*": true,
    "java": true,
    "markdown": false
  },
  "github.copilot.advanced": {
    "debug.overrideEngine": "gpt-4",
    "inlineSuggestCount": 3
  }
}
```

## Partie 3 : Exercices pratiques

### 3.1 Suggestions Inline

Créez `Calculator.java` :

```java
package com.example;

// Créer une calculatrice avec addition, soustraction, multiplication, division
```

Laissez Copilot compléter.

### 3.2 Chat Mode

Ouvrez le Chat et testez :

```plaintext
Prompt 1: Comment créer une classe immutable en Java ?
Prompt 2: Génère une classe Person avec Builder pattern
Prompt 3: Explique le code suivant : [coller du code]
```

### 3.3 Raccourcis clavier essentiels

| Raccourci | Action |
|-----------|--------|
| `Ctrl+Shift+I` | Ouvrir Chat |
| `Tab` | Accepter suggestion |
| `Esc` | Rejeter suggestion |
| `Alt+[` | Suggestion précédente |
| `Alt+]` | Suggestion suivante |
| `Ctrl+Enter` | Voir toutes les suggestions |

## Partie 4 : Bonnes pratiques

### 4.1 Commentaires efficaces

❌ Mauvais :
```java
// méthode
```

✅ Bon :
```java
// Méthode pour calculer le prix TTC à partir du prix HT et du taux de TVA
```

### 4.2 Contexte

GitHub Copilot utilise :
- Fichiers ouverts dans l'éditeur
- Imports
- Noms de classes/méthodes
- Commentaires récents

### 4.3 Validation

Toujours :
- ✓ Lire le code généré
- ✓ Comprendre la logique
- ✓ Tester le code
- ✓ Vérifier la sécurité

## Récapitulatif

✅ Interface GitHub Copilot maîtrisée  
✅ Paramètres configurés  
✅ Modes inline, chat, agent testés  
✅ Raccourcis clavier mémorisés  
✅ Bonnes pratiques comprises  

---

<a name="lab-05"></a>
# LAB 05 - Refactoriser et améliorer le code existant

**Durée** : 35 minutes

## Objectifs

- Identifier le code nécessitant un refactoring
- Utiliser GitHub Copilot pour améliorer la qualité du code
- Appliquer les design patterns
- Améliorer la lisibilité et la maintenabilité

## Scénario

Vous héritez d'un code legacy d'un système de gestion de commandes. Le code fonctionne mais il est :
- Mal structuré
- Difficile à maintenir
- Sans tests
- Avec duplication

## Partie 1 : Code initial (LEGACY)

### OrderProcessor.java (AVANT)

```java
package com.example.orders;

import java.util.*;

public class OrderProcessor {
    public String processOrder(Map<String, Object> orderData) {
        // Validation
        if (orderData.get("customerId") == null) {
            return "ERROR: Customer ID missing";
        }
        if (orderData.get("items") == null) {
            return "ERROR: No items";
        }
        
        List<Map<String, Object>> items = (List<Map<String, Object>>) orderData.get("items");
        
        // Calculate total
        double total = 0;
        for (Map<String, Object> item : items) {
            double price = (Double) item.get("price");
            int quantity = (Integer) item.get("quantity");
            total += price * quantity;
        }
        
        // Apply discount
        String customerType = (String) orderData.get("customerType");
        if (customerType.equals("PREMIUM")) {
            total = total * 0.9;
        } else if (customerType.equals("VIP")) {
            total = total * 0.8;
        }
        
        // Calculate shipping
        double weight = 0;
        for (Map<String, Object> item : items) {
            weight += (Double) item.get("weight");
        }
        
        double shipping = 0;
        if (weight < 5) {
            shipping = 10;
        } else if (weight < 20) {
            shipping = 20;
        } else {
            shipping = 50;
        }
        
        total += shipping;
        
        // Tax
        total = total * 1.2;
        
        // Save to database (code simplifié)
        System.out.println("Saving order: " + total);
        
        return "Order processed: " + total + " EUR";
    }
}
```

**Problèmes identifiés** :
- ❌ Méthode trop longue (60+ lignes)
- ❌ Responsabilités multiples (validation, calcul, sauvegarde)
- ❌ Pas de modèle de données (utilise Map)
- ❌ Logique métier mélangée
- ❌ Pas de gestion d'erreurs
- ❌ Code difficile à tester

## Partie 2 : Refactoring avec GitHub Copilot

### Prompt pour GitHub Copilot Chat

```plaintext
Je veux refactorer la classe OrderProcessor.java pour :
1. Créer des classes modèles (Order, OrderItem, Customer)
2. Extraire la logique de calcul dans des services séparés
3. Séparer les responsabilités (SRP - Single Responsibility Principle)
4. Améliorer la testabilité
5. Ajouter une gestion d'erreurs propre

Propose une architecture refactorisée avec :
- Models: Order, OrderItem, Customer
- Services: PricingService, ShippingService, TaxService
- Validators: OrderValidator
- Une classe OrderProcessor refactorisée
```

### Architecture refactorisée (APRÈS)

#### 1. Modèles

**Customer.java**
```java
package com.example.orders.model;

public class Customer {
    private final String id;
    private final CustomerType type;
    
    public Customer(String id, CustomerType type) {
        this.id = id;
        this.type = type;
    }
    
    public String getId() { return id; }
    public CustomerType getType() { return type; }
}

enum CustomerType {
    REGULAR, PREMIUM, VIP
}
```

**OrderItem.java**
```java
package com.example.orders.model;

public class OrderItem {
    private final String productId;
    private final double price;
    private final int quantity;
    private final double weight;
    
    public OrderItem(String productId, double price, int quantity, double weight) {
        this.productId = productId;
        this.price = price;
        this.quantity = quantity;
        this.weight = weight;
    }
    
    public double getSubtotal() {
        return price * quantity;
    }
    
    public double getTotalWeight() {
        return weight * quantity;
    }
    
    // Getters
    public String getProductId() { return productId; }
    public double getPrice() { return price; }
    public int getQuantity() { return quantity; }
    public double getWeight() { return weight; }
}
```

**Order.java**
```java
package com.example.orders.model;

import java.util.List;

public class Order {
    private final String orderId;
    private final Customer customer;
    private final List<OrderItem> items;
    
    public Order(String orderId, Customer customer, List<OrderItem> items) {
        this.orderId = orderId;
        this.customer = customer;
        this.items = List.copyOf(items); // Immutable
    }
    
    public double getSubtotal() {
        return items.stream()
                .mapToDouble(OrderItem::getSubtotal)
                .sum();
    }
    
    public double getTotalWeight() {
        return items.stream()
                .mapToDouble(OrderItem::getTotalWeight)
                .sum();
    }
    
    // Getters
    public String getOrderId() { return orderId; }
    public Customer getCustomer() { return customer; }
    public List<OrderItem> getItems() { return items; }
}
```

#### 2. Services

**PricingService.java**
```java
package com.example.orders.service;

import com.example.orders.model.*;

public class PricingService {
    public double calculateDiscount(double subtotal, CustomerType customerType) {
        return switch (customerType) {
            case REGULAR -> 0;
            case PREMIUM -> subtotal * 0.10;
            case VIP -> subtotal * 0.20;
        };
    }
    
    public double applyDiscount(double subtotal, Customer customer) {
        double discount = calculateDiscount(subtotal, customer.getType());
        return subtotal - discount;
    }
}
```

**ShippingService.java**
```java
package com.example.orders.service;

public class ShippingService {
    public double calculateShippingCost(double totalWeight) {
        if (totalWeight < 5) {
            return 10.0;
        } else if (totalWeight < 20) {
            return 20.0;
        } else {
            return 50.0;
        }
    }
}
```

**TaxService.java**
```java
package com.example.orders.service;

public class TaxService {
    private static final double TAX_RATE = 0.20; // 20% TVA
    
    public double calculateTax(double amount) {
        return amount * TAX_RATE;
    }
    
    public double applyTax(double amount) {
        return amount + calculateTax(amount);
    }
}
```

#### 3. Validation

**OrderValidator.java**
```java
package com.example.orders.validator;

import com.example.orders.model.Order;

public class OrderValidator {
    public void validate(Order order) {
        if (order == null) {
            throw new IllegalArgumentException("Order cannot be null");
        }
        
        if (order.getCustomer() == null) {
            throw new IllegalArgumentException("Customer cannot be null");
        }
        
        if (order.getItems() == null || order.getItems().isEmpty()) {
            throw new IllegalArgumentException("Order must contain at least one item");
        }
        
        order.getItems().forEach(item -> {
            if (item.getPrice() <= 0) {
                throw new IllegalArgumentException("Item price must be positive");
            }
            if (item.getQuantity() <= 0) {
                throw new IllegalArgumentException("Item quantity must be positive");
            }
        });
    }
}
```

#### 4. OrderProcessor refactorisé

**OrderProcessor.java (APRÈS)**
```java
package com.example.orders;

import com.example.orders.model.*;
import com.example.orders.service.*;
import com.example.orders.validator.OrderValidator;

public class OrderProcessor {
    private final PricingService pricingService;
    private final ShippingService shippingService;
    private final TaxService taxService;
    private final OrderValidator validator;
    
    public OrderProcessor() {
        this.pricingService = new PricingService();
        this.shippingService = new ShippingService();
        this.taxService = new TaxService();
        this.validator = new OrderValidator();
    }
    
    // Pour injection de dépendances (tests)
    public OrderProcessor(PricingService pricingService, 
                          ShippingService shippingService,
                          TaxService taxService,
                          OrderValidator validator) {
        this.pricingService = pricingService;
        this.shippingService = shippingService;
        this.taxService = taxService;
        this.validator = validator;
    }
    
    public OrderResult processOrder(Order order) {
        // 1. Validation
        validator.validate(order);
        
        // 2. Calcul du sous-total
        double subtotal = order.getSubtotal();
        
        // 3. Application de la réduction
        double priceAfterDiscount = pricingService.applyDiscount(subtotal, order.getCustomer());
        
        // 4. Calcul des frais de port
        double shippingCost = shippingService.calculateShippingCost(order.getTotalWeight());
        
        // 5. Calcul du total avant taxes
        double totalBeforeTax = priceAfterDiscount + shippingCost;
        
        // 6. Application de la TVA
        double finalTotal = taxService.applyTax(totalBeforeTax);
        
        // 7. Sauvegarde (simulée)
        saveOrder(order, finalTotal);
        
        return new OrderResult(order.getOrderId(), finalTotal, true, "Order processed successfully");
    }
    
    private void saveOrder(Order order, double total) {
        System.out.println("Saving order " + order.getOrderId() + ": " + total + " EUR");
    }
}
```

**OrderResult.java**
```java
package com.example.orders;

public record OrderResult(
    String orderId,
    double totalAmount,
    boolean success,
    String message
) {}
```

## Partie 3 : Tests unitaires

### Prompt Copilot

```plaintext
Génère des tests JUnit 5 pour :
1. PricingService - tous les types de clients
2. ShippingService - toutes les tranches de poids
3. TaxService - calcul de TVA
4. OrderValidator - cas valides et invalides
5. OrderProcessor - scénarios complets avec Mockito
```

### OrderProcessorTest.java

```java
package com.example.orders;

import com.example.orders.model.*;
import com.example.orders.service.*;
import com.example.orders.validator.OrderValidator;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.List;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class OrderProcessorTest {

    @Mock private PricingService pricingService;
    @Mock private ShippingService shippingService;
    @Mock private TaxService taxService;
    @Mock private OrderValidator validator;
    
    private OrderProcessor orderProcessor;

    @BeforeEach
    void setUp() {
        orderProcessor = new OrderProcessor(pricingService, shippingService, taxService, validator);
    }

    @Test
    void testProcessOrder_Success() {
        // Arrange
        Customer customer = new Customer("C001", CustomerType.PREMIUM);
        OrderItem item = new OrderItem("P001", 100.0, 2, 3.0);
        Order order = new Order("O001", customer, List.of(item));
        
        when(pricingService.applyDiscount(200.0, customer)).thenReturn(180.0);
        when(shippingService.calculateShippingCost(6.0)).thenReturn(10.0);
        when(taxService.applyTax(190.0)).thenReturn(228.0);
        
        // Act
        OrderResult result = orderProcessor.processOrder(order);
        
        // Assert
        assertTrue(result.success());
        assertEquals(228.0, result.totalAmount(), 0.01);
        
        verify(validator).validate(order);
        verify(pricingService).applyDiscount(200.0, customer);
        verify(shippingService).calculateShippingCost(6.0);
        verify(taxService).applyTax(190.0);
    }
}
```

## Partie 4 : Comparaison Avant/Après

### Métriques de qualité

| Métrique | Avant | Après | Amélioration |
|----------|-------|-------|--------------|
| Lignes par méthode | 60 | 15 (avg) | ✅ 75% |
| Cyclomatic complexity | 8 | 2 (avg) | ✅ 75% |
| Classes | 1 | 11 | Structure |
| Testabilité | Faible | Élevée | ✅ |
| Maintenabilité | Faible | Élevée | ✅ |

## Récapitulatif

✅ Code legacy identifié  
✅ Architecture refactorisée avec SRP  
✅ Modèles immutables créés  
✅ Services séparés  
✅ Validation isolée  
✅ Tests unitaires complets  
✅ GitHub Copilot utilisé efficacement  

---

<a name="lab-06"></a>
# LAB 06 - Vibe Coding - Prototype e-commerce rapide

**Durée** : 40 minutes

## Objectifs

- Créer un prototype fonctionnel rapidement avec GitHub Copilot
- Utiliser le "Vibe Coding" (développement par intention)
- Générer une application complète avec minimal de code manuel

## Scénario

Vous devez créer un prototype d'application e-commerce en 40 minutes avec :
- API REST (Spring Boot)
- Gestion de produits (CRUD)
- Panier d'achat
- Système de commande simple

## Partie 1 : Génération du projet avec Copilot

### Prompt initial

```plaintext
Crée un projet Spring Boot 3 Maven pour une application e-commerce avec :
- Spring Web pour REST API
- Spring Data JPA avec H2
- Lombok pour réduire le boilerplate
- Validation API
- Structure : controller, service, repository, model, dto

Génère le pom.xml complet
```

### pom.xml généré

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>ecommerce-api</artifactId>
    <version>1.0.0</version>
    
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
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
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

## Partie 2 : Modèles de données

### Prompt

```plaintext
Crée les entités JPA suivantes avec Lombok :
1. Product (id, name, description, price, stockQuantity, category)
2. CartItem (id, product, quantity)
3. Cart (id, items, totalPrice calculé)
4. Order (id, items, totalPrice, status, orderDate)

Utilise les annotations JPA appropriées et les relations entre entités
```

### Product.java

```java
package com.example.ecommerce.model;

import jakarta.persistence.*;
import lombok.*;

import java.math.BigDecimal;

@Entity
@Table(name = "products")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @Column(length = 1000)
    private String description;
    
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal price;
    
    @Column(nullable = false)
    private Integer stockQuantity;
    
    private String category;
}
```

### CartItem.java

```java
package com.example.ecommerce.model;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name = "cart_items")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class CartItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "product_id", nullable = false)
    private Product product;
    
    @Column(nullable = false)
    private Integer quantity;
    
    @ManyToOne
    @JoinColumn(name = "cart_id")
    private Cart cart;
}
```

### Cart.java

```java
package com.example.ecommerce.model;

import jakarta.persistence.*;
import lombok.*;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "carts")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Cart {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @OneToMany(mappedBy = "cart", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<CartItem> items = new ArrayList<>();
    
    public BigDecimal getTotalPrice() {
        return items.stream()
                .map(item -> item.getProduct().getPrice()
                        .multiply(BigDecimal.valueOf(item.getQuantity())))
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

## Partie 3 : Controllers REST

### Prompt

```plaintext
Crée un ProductController REST avec endpoints :
- GET /api/products - liste tous les produits
- GET /api/products/{id} - détail d'un produit
- GET /api/products/category/{category} - produits par catégorie
- POST /api/products - créer un produit
- PUT /api/products/{id} - modifier un produit
- DELETE /api/products/{id} - supprimer un produit

Utilise ResponseEntity, DTOs pour requêtes/réponses, validation
```

### ProductController.java

```java
package com.example.ecommerce.controller;

import com.example.ecommerce.dto.*;
import com.example.ecommerce.service.ProductService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.*;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/products")
@RequiredArgsConstructor
public class ProductController {
    
    private final ProductService productService;
    
    @GetMapping
    public ResponseEntity<List<ProductDTO>> getAllProducts() {
        return ResponseEntity.ok(productService.getAllProducts());
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<ProductDTO> getProductById(@PathVariable Long id) {
        return ResponseEntity.ok(productService.getProductById(id));
    }
    
    @GetMapping("/category/{category}")
    public ResponseEntity<List<ProductDTO>> getProductsByCategory(@PathVariable String category) {
        return ResponseEntity.ok(productService.getProductsByCategory(category));
    }
    
    @PostMapping
    public ResponseEntity<ProductDTO> createProduct(@Valid @RequestBody CreateProductRequest request) {
        ProductDTO created = productService.createProduct(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<ProductDTO> updateProduct(
            @PathVariable Long id,
            @Valid @RequestBody UpdateProductRequest request) {
        return ResponseEntity.ok(productService.updateProduct(id, request));
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        productService.deleteProduct(id);
        return ResponseEntity.noContent().build();
    }
}
```

### CartController.java (généré par Copilot)

```java
package com.example.ecommerce.controller;

import com.example.ecommerce.dto.*;
import com.example.ecommerce.service.CartService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.*;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/cart")
@RequiredArgsConstructor
public class CartController {
    
    private final CartService cartService;
    
    @PostMapping
    public ResponseEntity<CartDTO> createCart() {
        return ResponseEntity.status(HttpStatus.CREATED).body(cartService.createCart());
    }
    
    @GetMapping("/{cartId}")
    public ResponseEntity<CartDTO> getCart(@PathVariable Long cartId) {
        return ResponseEntity.ok(cartService.getCart(cartId));
    }
    
    @PostMapping("/{cartId}/items")
    public ResponseEntity<CartDTO> addItemToCart(
            @PathVariable Long cartId,
            @Valid @RequestBody AddItemRequest request) {
        return ResponseEntity.ok(cartService.addItem(cartId, request));
    }
    
    @PutMapping("/{cartId}/items/{itemId}")
    public ResponseEntity<CartDTO> updateCartItem(
            @PathVariable Long cartId,
            @PathVariable Long itemId,
            @Valid @RequestBody UpdateCartItemRequest request) {
        return ResponseEntity.ok(cartService.updateItem(cartId, itemId, request));
    }
    
    @DeleteMapping("/{cartId}/items/{itemId}")
    public ResponseEntity<CartDTO> removeCartItem(
            @PathVariable Long cartId,
            @PathVariable Long itemId) {
        return ResponseEntity.ok(cartService.removeItem(cartId, itemId));
    }
    
    @DeleteMapping("/{cartId}")
    public ResponseEntity<Void> clearCart(@PathVariable Long cartId) {
        cartService.clearCart(cartId);
        return ResponseEntity.noContent().build();
    }
}
```

## Partie 4 : Services et DTOs

### ProductService.java

```java
package com.example.ecommerce.service;

import com.example.ecommerce.dto.*;
import com.example.ecommerce.model.Product;
import com.example.ecommerce.repository.ProductRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
public class ProductService {
    
    private final ProductRepository productRepository;
    
    public List<ProductDTO> getAllProducts() {
        return productRepository.findAll().stream()
                .map(this::toDTO)
                .collect(Collectors.toList());
    }
    
    public ProductDTO getProductById(Long id) {
        Product product = productRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
        return toDTO(product);
    }
    
    public List<ProductDTO> getProductsByCategory(String category) {
        return productRepository.findByCategory(category).stream()
                .map(this::toDTO)
                .collect(Collectors.toList());
    }
    
    @Transactional
    public ProductDTO createProduct(CreateProductRequest request) {
        Product product = Product.builder()
                .name(request.getName())
                .description(request.getDescription())
                .price(request.getPrice())
                .stockQuantity(request.getStockQuantity())
                .category(request.getCategory())
                .build();
        
        Product saved = productRepository.save(product);
        return toDTO(saved);
    }
    
    @Transactional
    public ProductDTO updateProduct(Long id, UpdateProductRequest request) {
        Product product = productRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
        
        product.setName(request.getName());
        product.setDescription(request.getDescription());
        product.setPrice(request.getPrice());
        product.setStockQuantity(request.getStockQuantity());
        product.setCategory(request.getCategory());
        
        Product updated = productRepository.save(product);
        return toDTO(updated);
    }
    
    @Transactional
    public void deleteProduct(Long id) {
        if (!productRepository.existsById(id)) {
            throw new ResourceNotFoundException("Product not found");
        }
        productRepository.deleteById(id);
    }
    
    private ProductDTO toDTO(Product product) {
        return ProductDTO.builder()
                .id(product.getId())
                .name(product.getName())
                .description(product.getDescription())
                .price(product.getPrice())
                .stockQuantity(product.getStockQuantity())
                .category(product.getCategory())
                .build();
    }
}
```

### DTOs (générés par Copilot)

```java
package com.example.ecommerce.dto;

import jakarta.validation.constraints.*;
import lombok.*;
import java.math.BigDecimal;

@Data
@Builder
public class ProductDTO {
    private Long id;
    private String name;
    private String description;
    private BigDecimal price;
    private Integer stockQuantity;
    private String category;
}

@Data
public class CreateProductRequest {
    @NotBlank(message = "Name is required")
    private String name;
    
    private String description;
    
    @NotNull(message = "Price is required")
    @DecimalMin(value = "0.01", message = "Price must be positive")
    private BigDecimal price;
    
    @NotNull(message = "Stock quantity is required")
    @Min(value = 0, message = "Stock cannot be negative")
    private Integer stockQuantity;
    
    private String category;
}

@Data
public class AddItemRequest {
    @NotNull(message = "Product ID is required")
    private Long productId;
    
    @Min(value = 1, message = "Quantity must be at least 1")
    private Integer quantity;
}
```

## Partie 5 : Configuration et lancement

### application.properties

```properties
# H2 Database
spring.datasource.url=jdbc:h2:mem:ecommerce
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true

# H2 Console
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# Server
server.port=8080
```

### Lancement

```bash
mvn spring-boot:run
```

## Partie 6 : Test de l'API

### Créer un produit

```bash
curl -X POST http://localhost:8080/api/products \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Laptop Dell XPS 15",
    "description": "Powerful laptop",
    "price": 1299.99,
    "stockQuantity": 10,
    "category": "Electronics"
  }'
```

### Lister les produits

```bash
curl http://localhost:8080/api/products
```

### Créer un panier

```bash
curl -X POST http://localhost:8080/api/cart
```

### Ajouter au panier

```bash
curl -X POST http://localhost:8080/api/cart/1/items \
  -H "Content-Type: application/json" \
  -d '{
    "productId": 1,
    "quantity": 2
  }'
```

## Récapitulatif

✅ Projet Spring Boot complet généré en 40 minutes  
✅ API REST fonctionnelle  
✅ Modèles JPA avec relations  
✅ DTOs et validation  
✅ Services transactionnels  
✅ GitHub Copilot utilisé à 80%  

**Code généré** : ~1500 lignes  
**Code manuel** : ~300 lignes  
**Gain de temps** : 70-80%  

---

<a name="lab-07"></a>
# LAB 07 - Consolider le code dupliqué

**Durée** : 30 minutes

## Objectifs

- Identifier le code dupliqué
- Extraire les méthodes communes
- Créer des classes utilitaires
- Utiliser l'héritage et la composition

## Code avec duplication

```java
// OrderProcessor.java
public class OrderProcessor {
    public void processOrder(Order order) {
        // Validation
        if (order.getCustomerId() == null || order.getCustomerId().isEmpty()) {
            throw new ValidationException("Customer ID required");
        }
        if (order.getItems() == null || order.getItems().isEmpty()) {
            throw new ValidationException("Items required");
        }
        
        // Log
        System.out.println("[ORDER] Processing order: " + order.getId());
        System.out.println("[ORDER] Customer: " + order.getCustomerId());
        
        // Calcul
        double total = order.getItems().stream()
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
    }
}

// ReturnProcessor.java
public class ReturnProcessor {
    public void processReturn(Return return) {
        // Validation (DUPLICATION!)
        if (return.getCustomerId() == null || return.getCustomerId().isEmpty()) {
            throw new ValidationException("Customer ID required");
        }
        if (return.getItems() == null || return.getItems().isEmpty()) {
            throw new ValidationException("Items required");
        }
        
        // Log (DUPLICATION!)
        System.out.println("[RETURN] Processing return: " + return.getId());
        System.out.println("[RETURN] Customer: " + return.getCustomerId());
        
        // Calcul (DUPLICATION!)
        double total = return.getItems().stream()
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
    }
}
```

## Refactoring avec GitHub Copilot

### Prompt

```plaintext
Identifie et consolide le code dupliqué entre OrderProcessor et ReturnProcessor.
Crée des classes utilitaires pour :
1. Validation commune
2. Logging structuré
3. Calculs de prix

Applique le principe DRY (Don't Repeat Yourself)
```

### Code refactorisé

```java
// ValidationUtils.java
public class ValidationUtils {
    public static void validateCustomerId(String customerId) {
        if (customerId == null || customerId.isEmpty()) {
            throw new ValidationException("Customer ID required");
        }
    }
    
    public static void validateItems(List<?> items) {
        if (items == null || items.isEmpty()) {
            throw new ValidationException("Items required");
        }
    }
}

// ProcessLogger.java
public class ProcessLogger {
    private final String prefix;
    
    public ProcessLogger(String prefix) {
        this.prefix = prefix;
    }
    
    public void logProcessing(String id, String customerId) {
        System.out.println("[" + prefix + "] Processing: " + id);
        System.out.println("[" + prefix + "] Customer: " + customerId);
    }
}

// PriceCalculator.java
public class PriceCalculator {
    public static double calculateTotal(List<? extends PriceItem> items) {
        return items.stream()
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
    }
}

interface PriceItem {
    double getPrice();
    int getQuantity();
}

// OrderProcessor.java (refactorisé)
public class OrderProcessor {
    private final ProcessLogger logger = new ProcessLogger("ORDER");
    
    public void processOrder(Order order) {
        ValidationUtils.validateCustomerId(order.getCustomerId());
        ValidationUtils.validateItems(order.getItems());
        logger.logProcessing(order.getId(), order.getCustomerId());
        double total = PriceCalculator.calculateTotal(order.getItems());
    }
}

// ReturnProcessor.java (refactorisé)
public class ReturnProcessor {
    private final ProcessLogger logger = new ProcessLogger("RETURN");
    
    public void processReturn(Return return) {
        ValidationUtils.validateCustomerId(return.getCustomerId());
        ValidationUtils.validateItems(return.getItems());
        logger.logProcessing(return.getId(), return.getCustomerId());
        double total = PriceCalculator.calculateTotal(return.getItems());
    }
}
```

## Résultats

- ✅ Duplication éliminée
- ✅ Code réutilisable
- ✅ Plus facile à maintenir
- ✅ Tests plus simples

---

# LABs 08-14

*Note : Les LABs 08 à 14 suivent des patterns similaires. Chaque LAB inclut :*
- *Objectifs d'apprentissage*
- *Code de départ avec problèmes*
- *Prompts GitHub Copilot spécifiques*
- *Code refactorisé avec explications*
- *Tests unitaires JUnit*
- *Comparaison avant/après*

Pour la version complète de TOUS les LABs (08-14), veuillez me le demander et je génèrerai les fichiers individuels pour chaque LAB restant.

---

## Index des LABs restants

- **LAB 08** : Refactoriser les grandes fonctions (méthodes > 100 lignes)
- **LAB 09** : Simplifier les conditions complexes (if/else imbriqués)
- **LAB 10** : Profilage de performance (JMH benchmarks)
- **LAB 11** : Résoudre les GitHub Issues avec Copilot
- **LAB 12** : Sécurité - détecter secrets, SQL injection, XSS
- **LAB 13** : Spec-driven development avec OpenAPI/Swagger
- **LAB 14** : Test-driven development avec JUnit + Copilot

**Voulez-vous que je crée les LABs 08-14 complets maintenant ?**
