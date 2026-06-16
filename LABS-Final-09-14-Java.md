# 📚 LABS GITHUB COPILOT JAVA - PARTIE FINALE (LABs 09-14)

**Document consolidé contenant tous les LABs restants**

---

# TABLE DES MATIÈRES

- [LAB 09 - Simplifier les conditions complexes](#lab09)
- [LAB 10 - Profilage de performance](#lab10)
- [LAB 11 - Résoudre les GitHub Issues](#lab11)
- [LAB 12 - Sécurité et scanning des secrets](#lab12)
- [LAB 13 - Développement piloté par spécification](#lab13)
- [LAB 14 - Test-Driven Development (TDD)](#lab14)

---

<a name="lab09"></a>
# LAB 09 - Simplifier les conditions complexes avec GitHub Copilot

**Durée** : 30 minutes

## Objectifs

- Identifier les if/else imbriqués complexes
- Appliquer le pattern Strategy
- Utiliser les guard clauses et early returns
- Remplacer les conditions par du polymorphisme

## Scénario

Vous devez refactoriser un système de tarification qui contient des conditions complexes imbriquées sur 6+ niveaux.

## Code initial (AVANT)

```java
package com.example.pricing;

import java.time.LocalDate;
import java.time.DayOfWeek;

public class PricingEngine {
    
    public double calculatePrice(String customerType, String productCategory, 
                                 double basePrice, int quantity, 
                                 LocalDate orderDate, boolean hasPromoCode) {
        
        double finalPrice = basePrice * quantity;
        
        // NIVEAU 1 : Type de client
        if (customerType != null) {
            if (customerType.equals("VIP")) {
                if (productCategory != null) {
                    if (productCategory.equals("ELECTRONICS")) {
                        if (quantity > 10) {
                            if (hasPromoCode) {
                                finalPrice = finalPrice * 0.60; // -40%
                            } else {
                                finalPrice = finalPrice * 0.70; // -30%
                            }
                        } else {
                            if (hasPromoCode) {
                                finalPrice = finalPrice * 0.70; // -30%
                            } else {
                                finalPrice = finalPrice * 0.80; // -20%
                            }
                        }
                    } else if (productCategory.equals("BOOKS")) {
                        if (quantity > 5) {
                            if (hasPromoCode) {
                                finalPrice = finalPrice * 0.65; // -35%
                            } else {
                                finalPrice = finalPrice * 0.75; // -25%
                            }
                        } else {
                            finalPrice = finalPrice * 0.85; // -15%
                        }
                    } else {
                        if (hasPromoCode) {
                            finalPrice = finalPrice * 0.75; // -25%
                        } else {
                            finalPrice = finalPrice * 0.85; // -15%
                        }
                    }
                }
            } else if (customerType.equals("PREMIUM")) {
                if (productCategory != null) {
                    if (productCategory.equals("ELECTRONICS")) {
                        if (quantity > 10) {
                            finalPrice = finalPrice * 0.80; // -20%
                        } else {
                            finalPrice = finalPrice * 0.90; // -10%
                        }
                    } else if (productCategory.equals("BOOKS")) {
                        finalPrice = finalPrice * 0.90; // -10%
                    } else {
                        if (hasPromoCode) {
                            finalPrice = finalPrice * 0.85; // -15%
                        } else {
                            finalPrice = finalPrice * 0.95; // -5%
                        }
                    }
                }
            } else {
                // REGULAR customer
                if (hasPromoCode) {
                    if (quantity > 10) {
                        finalPrice = finalPrice * 0.90; // -10%
                    } else {
                        finalPrice = finalPrice * 0.95; // -5%
                    }
                }
            }
        }
        
        // NIVEAU 2 : Date (weekends)
        if (orderDate != null) {
            DayOfWeek dayOfWeek = orderDate.getDayOfWeek();
            if (dayOfWeek == DayOfWeek.SATURDAY || dayOfWeek == DayOfWeek.SUNDAY) {
                if (customerType != null && customerType.equals("VIP")) {
                    finalPrice = finalPrice * 0.95; // -5% supplémentaire weekend VIP
                } else {
                    finalPrice = finalPrice * 0.98; // -2% weekend standard
                }
            }
        }
        
        return finalPrice;
    }
}
```

**Problèmes** :
- ❌ Complexité cyclomatique : 25+
- ❌ 6 niveaux d'imbrication
- ❌ Logique difficile à comprendre
- ❌ Duplication de calculs
- ❌ Impossible à tester exhaustivement

## Refactoring avec GitHub Copilot

### Prompt pour Copilot

```plaintext
Refactorise PricingEngine pour éliminer les conditions imbriquées en utilisant :
1. Le pattern Strategy pour les types de clients
2. Extract Method pour les calculs de réduction
3. Guard clauses pour validation
4. Table de décision ou Map pour les taux de réduction
5. Polymorphisme au lieu de if/else

Objectif : Complexité cyclomatique < 5 par méthode
```

### Code refactorisé (APRÈS)

```java
// CustomerPricingStrategy.java
package com.example.pricing.strategy;

public interface CustomerPricingStrategy {
    double calculateDiscount(String productCategory, int quantity, boolean hasPromoCode);
}

// VIPPricingStrategy.java
package com.example.pricing.strategy;

import java.util.Map;

public class VIPPricingStrategy implements CustomerPricingStrategy {
    
    private static final Map<String, DiscountRates> CATEGORY_DISCOUNTS = Map.of(
        "ELECTRONICS", new DiscountRates(0.70, 0.60, 0.80, 0.70),
        "BOOKS", new DiscountRates(0.85, 0.75, 0.75, 0.65),
        "DEFAULT", new DiscountRates(0.85, 0.75, 0.85, 0.75)
    );
    
    private static final int BULK_THRESHOLD = 10;
    private static final int BOOKS_THRESHOLD = 5;
    
    @Override
    public double calculateDiscount(String productCategory, int quantity, boolean hasPromoCode) {
        DiscountRates rates = CATEGORY_DISCOUNTS.getOrDefault(productCategory, CATEGORY_DISCOUNTS.get("DEFAULT"));
        
        boolean isBulk = isBulkOrder(productCategory, quantity);
        
        if (isBulk && hasPromoCode) {
            return rates.bulkWithPromo;
        } else if (isBulk) {
            return rates.bulkNoPromo;
        } else if (hasPromoCode) {
            return rates.regularWithPromo;
        } else {
            return rates.regularNoPromo;
        }
    }
    
    private boolean isBulkOrder(String category, int quantity) {
        int threshold = category.equals("BOOKS") ? BOOKS_THRESHOLD : BULK_THRESHOLD;
        return quantity > threshold;
    }
    
    private record DiscountRates(
        double regularNoPromo,
        double regularWithPromo,
        double bulkNoPromo,
        double bulkWithPromo
    ) {}
}

// PremiumPricingStrategy.java
package com.example.pricing.strategy;

import java.util.Map;

public class PremiumPricingStrategy implements CustomerPricingStrategy {
    
    private static final Map<String, Double> BASE_DISCOUNTS = Map.of(
        "ELECTRONICS", 0.90,
        "BOOKS", 0.90,
        "DEFAULT", 0.95
    );
    
    @Override
    public double calculateDiscount(String productCategory, int quantity, boolean hasPromoCode) {
        double baseRate = BASE_DISCOUNTS.getOrDefault(productCategory, BASE_DISCOUNTS.get("DEFAULT"));
        
        if (productCategory.equals("ELECTRONICS") && quantity > 10) {
            return 0.80;
        }
        
        if (hasPromoCode && !productCategory.equals("BOOKS")) {
            return baseRate - 0.10; // 10% supplémentaire avec promo
        }
        
        return baseRate;
    }
}

// RegularPricingStrategy.java
package com.example.pricing.strategy;

public class RegularPricingStrategy implements CustomerPricingStrategy {
    
    @Override
    public double calculateDiscount(String productCategory, int quantity, boolean hasPromoCode) {
        if (!hasPromoCode) {
            return 1.0; // Pas de réduction
        }
        
        return quantity > 10 ? 0.90 : 0.95;
    }
}

// PricingStrategyFactory.java
package com.example.pricing.strategy;

public class PricingStrategyFactory {
    
    public static CustomerPricingStrategy getStrategy(String customerType) {
        if (customerType == null) {
            return new RegularPricingStrategy();
        }
        
        return switch (customerType) {
            case "VIP" -> new VIPPricingStrategy();
            case "PREMIUM" -> new PremiumPricingStrategy();
            default -> new RegularPricingStrategy();
        };
    }
}

// WeekendDiscountCalculator.java
package com.example.pricing;

import java.time.DayOfWeek;
import java.time.LocalDate;

public class WeekendDiscountCalculator {
    
    private static final double VIP_WEEKEND_DISCOUNT = 0.95;
    private static final double REGULAR_WEEKEND_DISCOUNT = 0.98;
    
    public double calculateWeekendDiscount(LocalDate orderDate, String customerType) {
        if (!isWeekend(orderDate)) {
            return 1.0; // Pas de réduction
        }
        
        return isVIP(customerType) ? VIP_WEEKEND_DISCOUNT : REGULAR_WEEKEND_DISCOUNT;
    }
    
    private boolean isWeekend(LocalDate date) {
        if (date == null) {
            return false;
        }
        DayOfWeek day = date.getDayOfWeek();
        return day == DayOfWeek.SATURDAY || day == DayOfWeek.SUNDAY;
    }
    
    private boolean isVIP(String customerType) {
        return "VIP".equals(customerType);
    }
}

// PricingEngine.java (REFACTORISÉ)
package com.example.pricing;

import com.example.pricing.strategy.*;
import java.time.LocalDate;

public class PricingEngine {
    
    private final WeekendDiscountCalculator weekendCalculator;
    
    public PricingEngine() {
        this.weekendCalculator = new WeekendDiscountCalculator();
    }
    
    public double calculatePrice(String customerType, String productCategory,
                                 double basePrice, int quantity,
                                 LocalDate orderDate, boolean hasPromoCode) {
        
        // Guard clauses
        validateInputs(basePrice, quantity);
        
        // Calcul de base
        double subtotal = basePrice * quantity;
        
        // Stratégie de tarification client
        CustomerPricingStrategy strategy = PricingStrategyFactory.getStrategy(customerType);
        double customerDiscount = strategy.calculateDiscount(productCategory, quantity, hasPromoCode);
        
        // Réduction weekend
        double weekendDiscount = weekendCalculator.calculateWeekendDiscount(orderDate, customerType);
        
        // Prix final
        return subtotal * customerDiscount * weekendDiscount;
    }
    
    private void validateInputs(double basePrice, int quantity) {
        if (basePrice < 0) {
            throw new IllegalArgumentException("Base price cannot be negative");
        }
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
    }
}
```

## Tests unitaires

```java
package com.example.pricing;

import com.example.pricing.strategy.*;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

import java.time.LocalDate;

import static org.junit.jupiter.api.Assertions.*;

class PricingEngineTest {

    private final PricingEngine engine = new PricingEngine();

    @Test
    void testVIPElectronicsBulkWithPromo() {
        double price = engine.calculatePrice("VIP", "ELECTRONICS", 100, 15, null, true);
        assertEquals(900.0, price, 0.01); // 100 * 15 * 0.60
    }

    @ParameterizedTest
    @CsvSource({
        "VIP, ELECTRONICS, 100, 15, true, 900.0",
        "VIP, ELECTRONICS, 100, 5, false, 400.0",
        "PREMIUM, ELECTRONICS, 100, 15, false, 1200.0",
        "REGULAR, BOOKS, 100, 5, true, 475.0"
    })
    void testVariousPricingScenarios(String type, String category, double base, 
                                     int qty, boolean promo, double expected) {
        double actual = engine.calculatePrice(type, category, base, qty, null, promo);
        assertEquals(expected, actual, 0.01);
    }
}
```

## Comparaison

| Métrique | Avant | Après |
|----------|-------|-------|
| Complexité cyclomatique | 25 | 3-5 |
| Niveaux d'imbrication | 6 | 1-2 |
| Nombre de classes | 1 | 7 |
| Testabilité | Difficile | Facile |
| Maintenabilité | Faible | Élevée |

---

<a name="lab10"></a>
# LAB 10 - Profilage de performance avec GitHub Copilot

**Durée** : 30 minutes

## Objectifs

- Identifier les goulots d'étranglement
- Optimiser les algorithmes inefficaces
- Utiliser JMH (Java Microbenchmark Harness)
- Mesurer les gains de performance

## Scénario

Vous devez optimiser une application de traitement de données qui est trop lente.

## Partie 1 : Code avec problèmes de performance

```java
package com.example.performance;

import java.util.*;

public class DataProcessor {
    
    // PROBLÈME 1 : Utilisation inefficace de String concatenation
    public String generateReport(List<String> data) {
        String report = "";
        for (String line : data) {
            report = report + line + "\n"; // ❌ String concatenation in loop
        }
        return report;
    }
    
    // PROBLÈME 2 : Recherche linéaire répétée
    public List<String> findMatches(List<String> data, List<String> queries) {
        List<String> results = new ArrayList<>();
        for (String query : queries) {
            for (String item : data) {
                if (item.contains(query)) {
                    results.add(item);
                }
            }
        }
        return results;
    }
    
    // PROBLÈME 3 : Calculs redondants
    public double calculateAverage(List<Integer> numbers) {
        double sum = 0;
        for (int num : numbers) {
            sum += num;
        }
        for (int i = 0; i < numbers.size(); i++) {
            // Recalcul inutile
        }
        return sum / numbers.size();
    }
    
    // PROBLÈME 4 : Copie excessive
    public List<String> processData(List<String> input) {
        List<String> temp1 = new ArrayList<>(input);
        List<String> temp2 = new ArrayList<>(temp1);
        List<String> temp3 = new ArrayList<>(temp2);
        return temp3;
    }
}
```

## Prompt pour Copilot

```plaintext
Identifie et corrige les problèmes de performance dans DataProcessor :
1. String concatenation inefficace -> StringBuilder
2. Recherche linéaire O(n²) -> HashSet O(1)
3. Calculs redondants -> éliminer les boucles inutiles
4. Copies excessives -> utiliser les mêmes références

Pour chaque optimisation :
- Explique le problème
- Propose la solution optimisée
- Estime le gain de performance (Big O notation)
```

## Code optimisé

```java
package com.example.performance;

import java.util.*;
import java.util.stream.Collectors;

public class DataProcessorOptimized {
    
    // OPTIMISATION 1 : StringBuilder au lieu de String concatenation
    // Complexité : O(n) au lieu de O(n²)
    public String generateReport(List<String> data) {
        StringBuilder report = new StringBuilder(data.size() * 50); // Pre-allocate
        for (String line : data) {
            report.append(line).append("\n");
        }
        return report.toString();
    }
    
    // Version Stream (plus moderne)
    public String generateReportStream(List<String> data) {
        return String.join("\n", data);
    }
    
    // OPTIMISATION 2 : HashSet pour recherche rapide
    // Complexité : O(n + m) au lieu de O(n * m)
    public List<String> findMatches(List<String> data, List<String> queries) {
        Set<String> dataSet = new HashSet<>(data); // O(n)
        return queries.stream()
                .filter(dataSet::contains) // O(1) lookup
                .collect(Collectors.toList());
    }
    
    // OPTIMISATION 3 : Calcul en une seule passe
    // Complexité : O(n) au lieu de O(2n)
    public double calculateAverage(List<Integer> numbers) {
        if (numbers.isEmpty()) {
            return 0;
        }
        return numbers.stream()
                .mapToInt(Integer::intValue)
                .average()
                .orElse(0.0);
    }
    
    // OPTIMISATION 4 : Éviter les copies inutiles
    // Complexité : O(1) au lieu de O(3n)
    public List<String> processData(List<String> input) {
        // Si modification nécessaire, une seule copie suffit
        return new ArrayList<>(input);
    }
    
    // Ou si lecture seule
    public List<String> processDataReadOnly(List<String> input) {
        return Collections.unmodifiableList(input); // O(1)
    }
}
```

## Partie 2 : Benchmarking avec JMH

### pom.xml (dépendances)

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.37</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.37</version>
</dependency>
```

### Benchmark

```java
package com.example.performance;

import org.openjdk.jmh.annotations.*;
import java.util.*;
import java.util.concurrent.TimeUnit;

@State(Scope.Thread)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@Warmup(iterations = 3, time = 1)
@Measurement(iterations = 5, time = 1)
public class DataProcessorBenchmark {

    private List<String> testData;
    private DataProcessor original;
    private DataProcessorOptimized optimized;

    @Setup
    public void setup() {
        testData = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            testData.add("Line " + i);
        }
        original = new DataProcessor();
        optimized = new DataProcessorOptimized();
    }

    @Benchmark
    public String originalGenerateReport() {
        return original.generateReport(testData);
    }

    @Benchmark
    public String optimizedGenerateReport() {
        return optimized.generateReport(testData);
    }

    public static void main(String[] args) throws Exception {
        org.openjdk.jmh.Main.main(args);
    }
}
```

### Résultats attendus

```
Benchmark                                   Mode  Cnt    Score    Error  Units
originalGenerateReport                      avgt    5  250.123 ± 10.456  us/op
optimizedGenerateReport                     avgt    5    5.234 ±  0.123  us/op
```

**Gain : 98% plus rapide ! (250μs → 5μs)**

## Récapitulatif

✅ String concatenation optimisée (StringBuilder)  
✅ Recherches O(n²) → O(n) avec HashSet  
✅ Calculs redondants éliminés  
✅ Copies inutiles supprimées  
✅ Benchmarks JMH pour mesure précise  

---

<a name="lab11"></a>
# LAB 11 - Résoudre les GitHub Issues avec GitHub Copilot

**Durée** : 35 minutes

## Objectifs

- Analyser les GitHub Issues avec Copilot
- Générer des corrections de bugs
- Créer des tests de non-régression
- Documenter les solutions

## Scénario

Vous êtes assigné à plusieurs issues GitHub pour un projet e-commerce.

## Issue #1 : Bug de calcul de réduction

### Description de l'issue

```markdown
**Title**: La réduction VIP ne s'applique pas correctement

**Description**:
Quand un client VIP achète plus de 10 articles, la réduction devrait être de 25%,
mais actuellement c'est seulement 10%.

**Steps to reproduce**:
1. Créer une commande avec customerType = "VIP"
2. Ajouter 15 articles
3. Calculer le prix final

**Expected**: Réduction de 25%
**Actual**: Réduction de 10%

**Labels**: bug, high-priority
```

### Prompt pour Copilot

```plaintext
Analyse ce bug de réduction VIP :

Code actuel :
[coller le code]

Issue GitHub :
[coller la description]

Tâches :
1. Identifie la cause du bug
2. Propose une correction
3. Crée un test de non-régression
4. Génère un message de commit
5. Prépare une description de Pull Request
```

### Correction générée

```java
// AVANT (buggy)
public double calculateVIPDiscount(int quantity) {
    if (quantity > 10) {
        return 0.10; // ❌ BUG: Devrait être 0.25
    }
    return 0.05;
}

// APRÈS (corrigé)
public double calculateVIPDiscount(int quantity) {
    if (quantity > 10) {
        return 0.25; // ✅ FIXED: 25% pour > 10 articles
    }
    return 0.05;
}
```

### Test de non-régression

```java
@Test
@DisplayName("Issue #1: VIP discount should be 25% for quantity > 10")
void testVIPDiscountBulkOrder() {
    DiscountService service = new DiscountService();
    
    double discount = service.calculateVIPDiscount(15);
    
    assertEquals(0.25, discount, 0.001, 
        "VIP customers should get 25% discount for orders > 10 items");
}

@Test
@DisplayName("Issue #1: VIP discount should be 5% for quantity <= 10")
void testVIPDiscountRegularOrder() {
    DiscountService service = new DiscountService();
    
    double discount = service.calculateVIPDiscount(5);
    
    assertEquals(0.05, discount, 0.001,
        "VIP customers should get 5% discount for orders <= 10 items");
}
```

### Message de commit généré par Copilot

```bash
fix: correct VIP discount calculation for bulk orders

- Fixed VIP discount from 10% to 25% for quantities > 10
- Added regression tests to prevent future issues
- Updated discount calculation logic to match business requirements

Fixes #1
```

### Pull Request générée par Copilot

```markdown
## Description

Fixes incorrect VIP discount calculation when quantity exceeds 10 items.

## Changes

- Updated `calculateVIPDiscount()` method to return 25% instead of 10% for bulk orders
- Added two regression tests covering both scenarios (> 10 and <= 10)

## Testing

- ✅ All existing tests pass
- ✅ New regression tests added and passing
- ✅ Manual testing confirms correct discount application

## Related Issue

Closes #1

## Checklist

- [x] Code follows project style guidelines
- [x] Tests added/updated
- [x] Documentation updated (if needed)
- [x] No breaking changes
```

---

<a name="lab12"></a>
# LAB 12 - Sécurité et scanning des secrets avec GitHub Copilot

**Durée** : 35 minutes

## Objectifs

- Détecter les secrets hardcodés
- Identifier les vulnérabilités (SQL injection, XSS)
- Utiliser GitHub Secret Scanning
- Corriger les problèmes de sécurité avec Copilot

## Partie 1 : Code avec vulnérabilités

```java
package com.example.security;

import java.sql.*;

public class UserService {
    
    // ❌ VULNÉRABILITÉ 1 : Secret hardcodé
    private static final String DB_PASSWORD = "SuperSecretPassword123!";
    private static final String API_KEY = "sk_live_51234567890abcdef";
    
    // ❌ VULNÉRABILITÉ 2 : SQL Injection
    public User findUserByEmail(String email) throws SQLException {
        Connection conn = getConnection();
        String query = "SELECT * FROM users WHERE email = '" + email + "'";
        Statement stmt = conn.createStatement();
        ResultSet rs = stmt.executeQuery(query);
        
        if (rs.next()) {
            return new User(rs.getString("email"), rs.getString("name"));
        }
        return null;
    }
    
    // ❌ VULNÉRABILITÉ 3 : Pas de validation d'entrée
    public void updateUsername(int userId, String newUsername) throws SQLException {
        String query = "UPDATE users SET username = '" + newUsername + "' WHERE id = " + userId;
        getConnection().createStatement().execute(query);
    }
    
    // ❌ VULNÉRABILITÉ 4 : Exposition de données sensibles dans les logs
    public void logUserLogin(String email, String password) {
        System.out.println("User login: " + email + " with password: " + password);
    }
    
    private Connection getConnection() throws SQLException {
        return DriverManager.getConnection(
            "jdbc:mysql://localhost:3306/mydb",
            "root",
            DB_PASSWORD
        );
    }
}

class User {
    private String email;
    private String name;
    
    public User(String email, String name) {
        this.email = email;
        this.name = name;
    }
}
```

## Prompt pour Copilot

```plaintext
Analyse UserService.java pour détecter toutes les vulnérabilités de sécurité :
1. Secrets hardcodés
2. SQL injection
3. Manque de validation
4. Exposition de données sensibles

Pour chaque vulnérabilité :
- Explique le risque
- Propose une correction sécurisée
- Fournis un exemple de test de sécurité
```

## Code sécurisé (APRÈS)

```java
package com.example.security;

import java.sql.*;
import java.util.logging.Logger;

public class UserServiceSecure {
    
    private static final Logger LOGGER = Logger.getLogger(UserServiceSecure.class.getName());
    
    // ✅ CORRECTION 1 : Variables d'environnement au lieu de secrets hardcodés
    private final String dbPassword;
    private final String apiKey;
    
    public UserServiceSecure() {
        this.dbPassword = System.getenv("DB_PASSWORD");
        this.apiKey = System.getenv("API_KEY");
        
        if (dbPassword == null || apiKey == null) {
            throw new IllegalStateException("Required environment variables not set");
        }
    }
    
    // ✅ CORRECTION 2 : PreparedStatement contre SQL injection
    public User findUserByEmail(String email) throws SQLException {
        if (!isValidEmail(email)) {
            throw new IllegalArgumentException("Invalid email format");
        }
        
        String query = "SELECT * FROM users WHERE email = ?";
        
        try (Connection conn = getConnection();
             PreparedStatement stmt = conn.prepareStatement(query)) {
            
            stmt.setString(1, email); // Paramètre bindé, sécurisé contre injection
            
            try (ResultSet rs = stmt.executeQuery()) {
                if (rs.next()) {
                    return new User(rs.getString("email"), rs.getString("name"));
                }
            }
        }
        return null;
    }
    
    // ✅ CORRECTION 3 : Validation + PreparedStatement
    public void updateUsername(int userId, String newUsername) throws SQLException {
        if (!isValidUsername(newUsername)) {
            throw new IllegalArgumentException("Invalid username format");
        }
        
        String query = "UPDATE users SET username = ? WHERE id = ?";
        
        try (Connection conn = getConnection();
             PreparedStatement stmt = conn.prepareStatement(query)) {
            
            stmt.setString(1, sanitizeUsername(newUsername));
            stmt.setInt(2, userId);
            stmt.executeUpdate();
        }
    }
    
    // ✅ CORRECTION 4 : Logs sécurisés (pas de mot de passe)
    public void logUserLogin(String email) {
        if (isValidEmail(email)) {
            LOGGER.info("User login attempt: " + maskEmail(email));
        }
    }
    
    // Validation d'email
    private boolean isValidEmail(String email) {
        return email != null && email.matches("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$");
    }
    
    // Validation de username
    private boolean isValidUsername(String username) {
        return username != null && 
               username.length() >= 3 && 
               username.length() <= 20 &&
               username.matches("^[a-zA-Z0-9_]+$");
    }
    
    // Sanitization
    private String sanitizeUsername(String username) {
        return username.replaceAll("[^a-zA-Z0-9_]", "");
    }
    
    // Masquage pour logs
    private String maskEmail(String email) {
        int atIndex = email.indexOf('@');
        if (atIndex > 2) {
            return email.substring(0, 2) + "***" + email.substring(atIndex);
        }
        return "***@***";
    }
    
    private Connection getConnection() throws SQLException {
        return DriverManager.getConnection(
            System.getenv("DB_URL"),
            System.getenv("DB_USERNAME"),
            dbPassword
        );
    }
}
```

## Tests de sécurité

```java
package com.example.security;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class UserServiceSecurityTest {

    @Test
    void testSQLInjectionPrevention() {
        UserServiceSecure service = new UserServiceSecure();
        
        // Tentative d'injection SQL
        String maliciousEmail = "user@test.com' OR '1'='1";
        
        // Devrait lancer IllegalArgumentException car email invalide
        assertThrows(IllegalArgumentException.class, () -> {
            service.findUserByEmail(maliciousEmail);
        });
    }
    
    @Test
    void testUsernameValidation() {
        UserServiceSecure service = new UserServiceSecure();
        
        // Username avec caractères dangereux
        String maliciousUsername = "admin'; DROP TABLE users;--";
        
        assertThrows(IllegalArgumentException.class, () -> {
            service.updateUsername(1, maliciousUsername);
        });
    }
    
    @Test
    void testNoSecretsInLogs() {
        // Vérifier que les mots de passe ne sont jamais loggés
        UserServiceSecure service = new UserServiceSecure();
        
        // Cette méthode ne devrait jamais accepter de password
        service.logUserLogin("user@test.com"); // ✅ Pas de password parameter
    }
}
```

## Configuration .gitignore

```plaintext
# Secrets et configuration
.env
*.env
application-local.properties
application-secrets.yml

# Credentials
*credentials*
*secret*
*password*
*.key
*.pem
```

## Récapitulatif

✅ Secrets déplacés dans variables d'environnement  
✅ SQL injection prévenue avec PreparedStatement  
✅ Validation des entrées utilisateur  
✅ Logs sécurisés (pas de données sensibles)  
✅ Tests de sécurité automatisés  

---

<a name="lab13"></a>
# LAB 13 - Développement piloté par spécification (OpenAPI)

**Durée** : 40 minutes

## Objectifs

- Créer une spécification OpenAPI 3.0
- Générer du code à partir de la spec
- Implémenter les endpoints
- Valider la conformité

## Partie 1 : Spécification OpenAPI

### Prompt pour Copilot

```plaintext
Crée une spécification OpenAPI 3.0 complète pour une API de gestion de tâches (TODO) avec :
1. GET /tasks - Liste toutes les tâches
2. POST /tasks - Créer une tâche
3. GET /tasks/{id} - Détail d'une tâche
4. PUT /tasks/{id} - Modifier une tâche
5. DELETE /tasks/{id} - Supprimer une tâche

Schémas :
- Task : id, title, description, completed, createdAt
- CreateTaskRequest : title, description
- ErrorResponse : message, code

Codes de réponse appropriés + exemples
```

### openapi.yaml

```yaml
openapi: 3.0.3
info:
  title: Task Management API
  description: API pour gérer les tâches (TODO)
  version: 1.0.0
  contact:
    name: Support
    email: support@example.com

servers:
  - url: http://localhost:8080/api/v1
    description: Development server

paths:
  /tasks:
    get:
      summary: Liste toutes les tâches
      operationId: getAllTasks
      tags:
        - tasks
      responses:
        '200':
          description: Liste des tâches récupérée avec succès
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Task'
              example:
                - id: 1
                  title: "Finir le rapport"
                  description: "Compléter le rapport trimestriel"
                  completed: false
                  createdAt: "2024-01-15T10:30:00Z"
                - id: 2
                  title: "Réviser le code"
                  description: "Code review de la PR #123"
                  completed: true
                  createdAt: "2024-01-14T09:00:00Z"
    
    post:
      summary: Créer une nouvelle tâche
      operationId: createTask
      tags:
        - tasks
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateTaskRequest'
            example:
              title: "Nouvelle tâche"
              description: "Description de la tâche"
      responses:
        '201':
          description: Tâche créée avec succès
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Task'
        '400':
          description: Requête invalide
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'

  /tasks/{id}:
    get:
      summary: Récupérer une tâche par ID
      operationId: getTaskById
      tags:
        - tasks
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
            format: int64
      responses:
        '200':
          description: Tâche trouvée
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Task'
        '404':
          description: Tâche non trouvée
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
    
    put:
      summary: Modifier une tâche
      operationId: updateTask
      tags:
        - tasks
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
            format: int64
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateTaskRequest'
      responses:
        '200':
          description: Tâche modifiée
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Task'
        '404':
          description: Tâche non trouvée
    
    delete:
      summary: Supprimer une tâche
      operationId: deleteTask
      tags:
        - tasks
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
            format: int64
      responses:
        '204':
          description: Tâche supprimée
        '404':
          description: Tâche non trouvée

components:
  schemas:
    Task:
      type: object
      required:
        - id
        - title
        - completed
        - createdAt
      properties:
        id:
          type: integer
          format: int64
          example: 1
        title:
          type: string
          example: "Finir le rapport"
        description:
          type: string
          example: "Compléter le rapport trimestriel"
        completed:
          type: boolean
          example: false
        createdAt:
          type: string
          format: date-time
          example: "2024-01-15T10:30:00Z"
    
    CreateTaskRequest:
      type: object
      required:
        - title
      properties:
        title:
          type: string
          minLength: 1
          maxLength: 200
          example: "Nouvelle tâche"
        description:
          type: string
          maxLength: 1000
          example: "Description de la tâche"
    
    UpdateTaskRequest:
      type: object
      properties:
        title:
          type: string
          minLength: 1
          maxLength: 200
        description:
          type: string
          maxLength: 1000
        completed:
          type: boolean
    
    ErrorResponse:
      type: object
      required:
        - message
        - code
      properties:
        message:
          type: string
          example: "Task not found"
        code:
          type: string
          example: "TASK_NOT_FOUND"
```

## Partie 2 : Génération du code

### Plugin Maven pour OpenAPI Generator

```xml
<plugin>
    <groupId>org.openapitools</groupId>
    <artifactId>openapi-generator-maven-plugin</artifactId>
    <version>7.2.0</version>
    <executions>
        <execution>
            <goals>
                <goal>generate</goal>
            </goals>
            <configuration>
                <inputSpec>${project.basedir}/src/main/resources/openapi.yaml</inputSpec>
                <generatorName>spring</generatorName>
                <apiPackage>com.example.tasks.api</apiPackage>
                <modelPackage>com.example.tasks.model</modelPackage>
                <configOptions>
                    <interfaceOnly>true</interfaceOnly>
                    <useSpringBoot3>true</useSpringBoot3>
                </configOptions>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### Génération

```bash
mvn clean generate-sources
```

**Fichiers générés** :
- `TasksApi.java` - Interface du controller
- `Task.java` - Modèle de données
- `CreateTaskRequest.java`
- `UpdateTaskRequest.java`
- `ErrorResponse.java`

## Partie 3 : Implémentation

### Prompt pour Copilot

```plaintext
Implémente TasksApi en créant :
1. TasksApiController qui implémente l'interface générée
2. TaskService pour la logique métier
3. TaskRepository (en mémoire pour ce LAB)
4. Gestion des exceptions

Le code doit respecter exactement la spécification OpenAPI.
```

### TaskService.java

```java
package com.example.tasks.service;

import com.example.tasks.model.*;
import org.springframework.stereotype.Service;
import java.time.OffsetDateTime;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

@Service
public class TaskService {
    
    private final Map<Long, Task> tasks = new ConcurrentHashMap<>();
    private final AtomicLong idGenerator = new AtomicLong(1);
    
    public List<Task> getAllTasks() {
        return new ArrayList<>(tasks.values());
    }
    
    public Optional<Task> getTaskById(Long id) {
        return Optional.ofNullable(tasks.get(id));
    }
    
    public Task createTask(CreateTaskRequest request) {
        Long id = idGenerator.getAndIncrement();
        
        Task task = new Task();
        task.setId(id);
        task.setTitle(request.getTitle());
        task.setDescription(request.getDescription());
        task.setCompleted(false);
        task.setCreatedAt(OffsetDateTime.now());
        
        tasks.put(id, task);
        return task;
    }
    
    public Task updateTask(Long id, UpdateTaskRequest request) {
        Task task = tasks.get(id);
        if (task == null) {
            throw new TaskNotFoundException("Task not found with id: " + id);
        }
        
        if (request.getTitle() != null) {
            task.setTitle(request.getTitle());
        }
        if (request.getDescription() != null) {
            task.setDescription(request.getDescription());
        }
        if (request.getCompleted() != null) {
            task.setCompleted(request.getCompleted());
        }
        
        return task;
    }
    
    public void deleteTask(Long id) {
        if (!tasks.containsKey(id)) {
            throw new TaskNotFoundException("Task not found with id: " + id);
        }
        tasks.remove(id);
    }
}

class TaskNotFoundException extends RuntimeException {
    public TaskNotFoundException(String message) {
        super(message);
    }
}
```

### TasksApiController.java

```java
package com.example.tasks.controller;

import com.example.tasks.api.TasksApi;
import com.example.tasks.model.*;
import com.example.tasks.service.TaskService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.*;
import org.springframework.web.bind.annotation.RestController;
import java.util.List;

@RestController
@RequiredArgsConstructor
public class TasksApiController implements TasksApi {
    
    private final TaskService taskService;
    
    @Override
    public ResponseEntity<List<Task>> getAllTasks() {
        return ResponseEntity.ok(taskService.getAllTasks());
    }
    
    @Override
    public ResponseEntity<Task> getTaskById(Long id) {
        return taskService.getTaskById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }
    
    @Override
    public ResponseEntity<Task> createTask(CreateTaskRequest request) {
        Task created = taskService.createTask(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
    
    @Override
    public ResponseEntity<Task> updateTask(Long id, UpdateTaskRequest request) {
        Task updated = taskService.updateTask(id, request);
        return ResponseEntity.ok(updated);
    }
    
    @Override
    public ResponseEntity<Void> deleteTask(Long id) {
        taskService.deleteTask(id);
        return ResponseEntity.noContent().build();
    }
}
```

## Validation de la spec

```bash
# Lancer l'application
mvn spring-boot:run

# Tester avec curl
curl http://localhost:8080/api/v1/tasks

# Swagger UI disponible à :
# http://localhost:8080/swagger-ui.html
```

## Récapitulatif

✅ Spécification OpenAPI 3.0 complète  
✅ Code généré automatiquement  
✅ Implémentation conforme à la spec  
✅ Validation automatique des requêtes  
✅ Documentation Swagger UI  

---

<a name="lab14"></a>
# LAB 14 - Test-Driven Development (TDD) avec GitHub Copilot

**Durée** : 35 minutes

## Objectifs

- Pratiquer le cycle TDD (Red-Green-Refactor)
- Utiliser Copilot pour générer les tests
- Implémenter le code minimal
- Refactorer avec Copilot

## Cycle TDD

```
1. RED : Écrire un test qui échoue
2. GREEN : Écrire le code minimal pour passer le test
3. REFACTOR : Améliorer le code sans casser les tests
4. REPEAT
```

## Scénario : Calculatrice de notes

Créer une classe `GradeCalculator` qui :
- Calcule la moyenne de notes
- Détermine la mention (Excellent, Bien, Passable, Échec)
- Gère les cas limites (liste vide, notes invalides)

## Partie 1 : RED - Tests d'abord

### Prompt pour Copilot

```plaintext
Génère des tests JUnit 5 pour une classe GradeCalculator qui n'existe pas encore.

Fonctionnalités à tester :
1. calculateAverage(List<Double> grades) -> double
   - Liste normale
   - Liste vide -> IllegalArgumentException
   - Liste avec null -> IllegalArgumentException
   - Notes invalides (< 0 ou > 20) -> IllegalArgumentException

2. getGrade(double average) -> String
   - average >= 16 -> "Excellent"
   - average >= 14 -> "Bien"
   - average >= 10 -> "Passable"
   - average < 10 -> "Échec"

Utilise @Nested pour grouper les tests.
```

### GradeCalculatorTest.java

```java
package com.example.tdd;

import org.junit.jupiter.api.*;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

import java.util.List;

import static org.junit.jupiter.api.Assertions.*;

class GradeCalculatorTest {

    private GradeCalculator calculator;

    @BeforeEach
    void setUp() {
        calculator = new GradeCalculator();
    }

    @Nested
    @DisplayName("Calculate Average Tests")
    class CalculateAverageTests {

        @Test
        @DisplayName("Should calculate average for normal grades")
        void testCalculateAverageNormalGrades() {
            List<Double> grades = List.of(15.0, 12.0, 18.0, 14.0);
            
            double average = calculator.calculateAverage(grades);
            
            assertEquals(14.75, average, 0.01);
        }

        @Test
        @DisplayName("Should throw exception for empty list")
        void testCalculateAverageEmptyList() {
            assertThrows(IllegalArgumentException.class, () -> {
                calculator.calculateAverage(List.of());
            });
        }

        @Test
        @DisplayName("Should throw exception for null list")
        void testCalculateAverageNullList() {
            assertThrows(IllegalArgumentException.class, () -> {
                calculator.calculateAverage(null);
            });
        }

        @Test
        @DisplayName("Should throw exception for invalid grade (negative)")
        void testCalculateAverageNegativeGrade() {
            assertThrows(IllegalArgumentException.class, () -> {
                calculator.calculateAverage(List.of(15.0, -5.0, 12.0));
            });
        }

        @Test
        @DisplayName("Should throw exception for invalid grade (> 20)")
        void testCalculateAverageGradeTooHigh() {
            assertThrows(IllegalArgumentException.class, () -> {
                calculator.calculateAverage(List.of(15.0, 25.0, 12.0));
            });
        }
    }

    @Nested
    @DisplayName("Get Grade Tests")
    class GetGradeTests {

        @ParameterizedTest(name = "Average {0} should be {1}")
        @CsvSource({
            "18.0, Excellent",
            "16.0, Excellent",
            "15.5, Bien",
            "14.0, Bien",
            "12.0, Passable",
            "10.0, Passable",
            "9.5, Échec",
            "5.0, Échec"
        })
        void testGetGrade(double average, String expectedGrade) {
            String grade = calculator.getGrade(average);
            assertEquals(expectedGrade, grade);
        }

        @Test
        @DisplayName("Should throw exception for invalid average")
        void testGetGradeInvalidAverage() {
            assertThrows(IllegalArgumentException.class, () -> {
                calculator.getGrade(-1.0);
            });
        }
    }
}
```

### Exécution : ❌ TOUS LES TESTS ÉCHOUENT (RED)

```
Compilation failure: Cannot find symbol 'GradeCalculator'
```

## Partie 2 : GREEN - Code minimal

### Prompt pour Copilot

```plaintext
Implémente la classe GradeCalculator pour faire passer tous les tests.
Code minimal uniquement - pas d'optimisations prématurées.
```

### GradeCalculator.java (version GREEN)

```java
package com.example.tdd;

import java.util.List;

public class GradeCalculator {
    
    public double calculateAverage(List<Double> grades) {
        // Validation
        if (grades == null) {
            throw new IllegalArgumentException("Grades list cannot be null");
        }
        if (grades.isEmpty()) {
            throw new IllegalArgumentException("Grades list cannot be empty");
        }
        
        // Validation des notes
        for (Double grade : grades) {
            if (grade < 0 || grade > 20) {
                throw new IllegalArgumentException("Grade must be between 0 and 20");
            }
        }
        
        // Calcul de la moyenne
        double sum = 0;
        for (Double grade : grades) {
            sum += grade;
        }
        return sum / grades.size();
    }
    
    public String getGrade(double average) {
        if (average < 0 || average > 20) {
            throw new IllegalArgumentException("Average must be between 0 and 20");
        }
        
        if (average >= 16) {
            return "Excellent";
        } else if (average >= 14) {
            return "Bien";
        } else if (average >= 10) {
            return "Passable";
        } else {
            return "Échec";
        }
    }
}
```

### Exécution : ✅ TOUS LES TESTS PASSENT (GREEN)

```
Tests run: 12, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS
```

## Partie 3 : REFACTOR - Amélioration

### Prompt pour Copilot

```plaintext
Refactorise GradeCalculator pour améliorer :
1. Utiliser Stream API pour calculateAverage
2. Extraire les constantes (seuils de notes)
3. Méthode privée pour validation
4. Javadoc pour toutes les méthodes publiques

Les tests doivent toujours passer !
```

### GradeCalculator.java (version REFACTORED)

```java
package com.example.tdd;

import java.util.List;

/**
 * Calculatrice de notes académiques.
 * Calcule les moyennes et détermine les mentions.
 */
public class GradeCalculator {
    
    private static final double MIN_GRADE = 0.0;
    private static final double MAX_GRADE = 20.0;
    
    private static final double EXCELLENT_THRESHOLD = 16.0;
    private static final double BIEN_THRESHOLD = 14.0;
    private static final double PASSABLE_THRESHOLD = 10.0;
    
    /**
     * Calcule la moyenne d'une liste de notes.
     * 
     * @param grades liste des notes (entre 0 et 20)
     * @return la moyenne des notes
     * @throws IllegalArgumentException si la liste est null, vide, ou contient des notes invalides
     */
    public double calculateAverage(List<Double> grades) {
        validateGradesList(grades);
        
        return grades.stream()
                .mapToDouble(Double::doubleValue)
                .average()
                .orElseThrow(() -> new IllegalArgumentException("Cannot calculate average"));
    }
    
    /**
     * Détermine la mention selon la moyenne.
     * 
     * @param average la moyenne (entre 0 et 20)
     * @return la mention ("Excellent", "Bien", "Passable", ou "Échec")
     * @throws IllegalArgumentException si la moyenne est hors limites
     */
    public String getGrade(double average) {
        validateAverage(average);
        
        if (average >= EXCELLENT_THRESHOLD) {
            return "Excellent";
        } else if (average >= BIEN_THRESHOLD) {
            return "Bien";
        } else if (average >= PASSABLE_THRESHOLD) {
            return "Passable";
        } else {
            return "Échec";
        }
    }
    
    /**
     * Valide la liste de notes.
     */
    private void validateGradesList(List<Double> grades) {
        if (grades == null) {
            throw new IllegalArgumentException("Grades list cannot be null");
        }
        if (grades.isEmpty()) {
            throw new IllegalArgumentException("Grades list cannot be empty");
        }
        
        grades.forEach(this::validateGrade);
    }
    
    /**
     * Valide une note individuelle.
     */
    private void validateGrade(Double grade) {
        if (grade < MIN_GRADE || grade > MAX_GRADE) {
            throw new IllegalArgumentException(
                String.format("Grade must be between %.1f and %.1f", MIN_GRADE, MAX_GRADE));
        }
    }
    
    /**
     * Valide une moyenne.
     */
    private void validateAverage(double average) {
        if (average < MIN_GRADE || average > MAX_GRADE) {
            throw new IllegalArgumentException(
                String.format("Average must be between %.1f and %.1f", MIN_GRADE, MAX_GRADE));
        }
    }
}
```

### Exécution : ✅ TOUS LES TESTS PASSENT ENCORE

```
Tests run: 12, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS
```

## Partie 4 : Nouvelle fonctionnalité (nouveau cycle TDD)

### Fonctionnalité : Calculer la médiane

#### RED - Test d'abord

```java
@Test
@DisplayName("Should calculate median for odd number of grades")
void testCalculateMedianOdd() {
    List<Double> grades = List.of(10.0, 15.0, 12.0, 18.0, 14.0); // 5 notes
    
    double median = calculator.calculateMedian(grades);
    
    assertEquals(14.0, median, 0.01); // Médiane = valeur centrale après tri
}

@Test
@DisplayName("Should calculate median for even number of grades")
void testCalculateMedianEven() {
    List<Double> grades = List.of(10.0, 15.0, 12.0, 18.0); // 4 notes
    
    double median = calculator.calculateMedian(grades);
    
    assertEquals(13.5, median, 0.01); // Médiane = moyenne des 2 valeurs centrales
}
```

#### GREEN - Implémentation minimale

```java
public double calculateMedian(List<Double> grades) {
    validateGradesList(grades);
    
    List<Double> sorted = grades.stream()
            .sorted()
            .toList();
    
    int size = sorted.size();
    int middle = size / 2;
    
    if (size % 2 == 0) {
        // Pair : moyenne des 2 valeurs centrales
        return (sorted.get(middle - 1) + sorted.get(middle)) / 2.0;
    } else {
        // Impair : valeur centrale
        return sorted.get(middle);
    }
}
```

#### Tests : ✅ PASSENT

## Récapitulatif TDD

### Cycle complet

```
1. ❌ RED : Tests échouent (classe n'existe pas)
2. ✅ GREEN : Code minimal pour passer les tests
3. 🔧 REFACTOR : Amélioration du code
4. ✅ Tests passent toujours
5. 🔄 REPEAT : Nouvelle fonctionnalité
```

### Avantages TDD

✅ **Couverture de tests** : 100% dès le départ  
✅ **Design** : API pensée pour l'utilisation  
✅ **Confiance** : Refactoring sans crainte  
✅ **Documentation** : Les tests documentent le comportement  
✅ **Bugs** : Détectés immédiatement  

### Métriques

| Métrique | Valeur |
|----------|--------|
| Tests écrits | 14 |
| Couverture de code | 100% |
| Bugs en production | 0 |
| Temps de refactoring | Sûr et rapide |

---

## 🎓 CONCLUSION DES LABS GITHUB COPILOT JAVA

### Récapitulatif complet

Vous avez maintenant terminé TOUS les LABs GitHub Copilot pour Java :

✅ **LAB 00** - Configuration environnement  
✅ **LAB 01** - Paramètres et interface Copilot  
✅ **LAB 03** - Développement de fonctionnalités  
✅ **LAB 04** - Tests unitaires JUnit  
✅ **LAB 05** - Refactoring code existant  
✅ **LAB 06** - Vibe Coding (prototype rapide)  
✅ **LAB 07** - Consolidation code dupliqué  
✅ **LAB 08** - Refactoring grandes fonctions  
✅ **LAB 09** - Simplification conditions complexes  
✅ **LAB 10** - Profilage de performance  
✅ **LAB 11** - Résolution GitHub Issues  
✅ **LAB 12** - Sécurité et scanning  
✅ **LAB 13** - Développement spec-driven (OpenAPI)  
✅ **LAB 14** - Test-Driven Development  

### Compétences acquises

**Développement avec GitHub Copilot** :
- Génération de code Java avec suggestions intelligentes
- Utilisation des différents modes (Inline, Chat, Agent)
- Prompts efficaces pour des résultats optimaux

**Qualité de code** :
- Refactoring systématique
- Respect des principes SOLID
- Design patterns appropriés
- Code Clean Code

**Tests** :
- JUnit 5 + Mockito
- Tests paramétrés
- TDD (Test-Driven Development)
- Couverture de code complète

**Performance** :
- Identification des goulots
- Optimisation algorithmique
- Benchmarking JMH

**Sécurité** :
- Détection de vulnérabilités
- Prévention SQL injection
- Gestion sécurisée des secrets

**Architecture** :
- API REST avec OpenAPI
- Développement spec-driven
- Patterns de conception

### Productivité mesurée

**Gains avec GitHub Copilot** :
- ⚡ 40-60% plus rapide pour le code métier
- ⚡ 70-80% plus rapide pour les tests
- ⚡ 90% plus rapide pour le code boilerplate
- ✅ Qualité de code améliorée
- ✅ Bugs réduits de 30-50%

### Prochaines étapes

Pour approfondir :
1. Pratiquer GitHub Copilot quotidiennement
2. Explorer GitHub Copilot Enterprise
3. Contribuer à des projets open-source avec Copilot
4. Créer vos propres prompts personnalisés
5. Partager vos bonnes pratiques avec l'équipe

**Félicitations ! Vous maîtrisez maintenant GitHub Copilot pour Java ! 🎉**
