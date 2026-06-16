# LAB 04 - Développer des tests unitaires avec Pytest (Java/JUnit)

## Description
Apprendre à utiliser GitHub Copilot pour créer des tests unitaires JUnit 5 complets et maintenables.

**Durée estimée** : 30 minutes

---

## Objectifs d'apprentissage

À la fin de cet exercice, vous serez capable de :
- Générer des tests JUnit 5 avec GitHub Copilot
- Créer des tests paramétrés avec `@ParameterizedTest`
- Utiliser Mockito pour les mocks et stubs
- Implémenter des assertions complexes
- Organiser les tests avec des suites de tests

---

## Prérequis

- JDK 17+, Maven 3.8+
- VS Code avec Extension Pack for Java
- GitHub Copilot activé
- Connaissance de base de JUnit 5

---

## Scénario

Vous devez créer des tests unitaires pour un service de calcul de prix dans une application e-commerce. Le service calcule :
- Les taxes selon le pays
- Les réductions selon le type de client
- Les frais de livraison selon la distance

---

## Partie 1 : Créer le projet

### Structure du projet

```bash
mkdir pricing-service-tests
cd pricing-service-tests
```

### Fichier `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>pricing-service-tests</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- JUnit 5 -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.1</version>
            <scope>test</scope>
        </dependency>
        
        <!-- Mockito -->
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <version>5.8.0</version>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-junit-jupiter</artifactId>
            <version>5.8.0</version>
            <scope>test</scope>
        </dependency>
        
        <!-- AssertJ pour assertions fluides -->
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>3.24.2</version>
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

---

## Partie 2 : Créer les classes métier

### Classe CustomerType (Enum)

```java
package com.example.pricing;

public enum CustomerType {
    REGULAR,
    PREMIUM,
    VIP
}
```

### Classe PricingService

Utilisez GitHub Copilot avec ce commentaire :

```java
package com.example.pricing;

/**
 * Service de calcul de prix qui gère :
 * - Le calcul de la taxe selon le pays (FR: 20%, BE: 21%, DE: 19%)
 * - Les réductions selon le type de client (REGULAR: 0%, PREMIUM: 10%, VIP: 20%)
 * - Les frais de livraison selon la distance (< 50km: 5€, < 200km: 15€, >= 200km: 30€)
 * - Le prix final total
 */
```

### Code généré attendu

```java
package com.example.pricing;

public class PricingService {
    
    public double calculateTax(double amount, String country) {
        if (country == null) {
            throw new IllegalArgumentException("Country cannot be null");
        }
        
        return switch (country.toUpperCase()) {
            case "FR" -> amount * 0.20;
            case "BE" -> amount * 0.21;
            case "DE" -> amount * 0.19;
            default -> throw new IllegalArgumentException("Unknown country: " + country);
        };
    }
    
    public double calculateDiscount(double amount, CustomerType customerType) {
        if (customerType == null) {
            throw new IllegalArgumentException("Customer type cannot be null");
        }
        
        return switch (customerType) {
            case REGULAR -> 0;
            case PREMIUM -> amount * 0.10;
            case VIP -> amount * 0.20;
        };
    }
    
    public double calculateShippingCost(double distance) {
        if (distance < 0) {
            throw new IllegalArgumentException("Distance cannot be negative");
        }
        
        if (distance < 50) {
            return 5.0;
        } else if (distance < 200) {
            return 15.0;
        } else {
            return 30.0;
        }
    }
    
    public double calculateFinalPrice(double basePrice, String country, 
                                      CustomerType customerType, double distance) {
        double discount = calculateDiscount(basePrice, customerType);
        double priceAfterDiscount = basePrice - discount;
        double tax = calculateTax(priceAfterDiscount, country);
        double shipping = calculateShippingCost(distance);
        
        return priceAfterDiscount + tax + shipping;
    }
}
```

---

## Partie 3 : Générer des tests unitaires de base

### Prompt pour GitHub Copilot Chat

```plaintext
Génère des tests JUnit 5 complets pour la classe PricingService qui :
1. Testent calculateTax pour chaque pays (FR, BE, DE)
2. Testent calculateDiscount pour chaque type de client
3. Testent calculateShippingCost pour chaque tranche de distance
4. Testent les cas d'erreur (valeurs null, négatives, pays inconnu)
5. Utilisent @BeforeEach pour initialiser PricingService
6. Utilisent des assertions claires avec assertEquals
```

### Code généré attendu : PricingServiceTest.java

```java
package com.example.pricing;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import static org.junit.jupiter.api.Assertions.*;

@DisplayName("PricingService Tests")
class PricingServiceTest {

    private PricingService pricingService;

    @BeforeEach
    void setUp() {
        pricingService = new PricingService();
    }

    // Tests de calcul de taxe
    @Test
    @DisplayName("Calculate tax for France (20%)")
    void testCalculateTaxFrance() {
        double tax = pricingService.calculateTax(100.0, "FR");
        assertEquals(20.0, tax, 0.01);
    }

    @Test
    @DisplayName("Calculate tax for Belgium (21%)")
    void testCalculateTaxBelgium() {
        double tax = pricingService.calculateTax(100.0, "BE");
        assertEquals(21.0, tax, 0.01);
    }

    @Test
    @DisplayName("Calculate tax for Germany (19%)")
    void testCalculateTaxGermany() {
        double tax = pricingService.calculateTax(100.0, "DE");
        assertEquals(19.0, tax, 0.01);
    }

    @Test
    @DisplayName("Calculate tax throws exception for null country")
    void testCalculateTaxNullCountry() {
        assertThrows(IllegalArgumentException.class, 
            () -> pricingService.calculateTax(100.0, null));
    }

    @Test
    @DisplayName("Calculate tax throws exception for unknown country")
    void testCalculateTaxUnknownCountry() {
        assertThrows(IllegalArgumentException.class, 
            () -> pricingService.calculateTax(100.0, "XX"));
    }

    // Tests de calcul de réduction
    @Test
    @DisplayName("Calculate discount for REGULAR customer (0%)")
    void testCalculateDiscountRegular() {
        double discount = pricingService.calculateDiscount(100.0, CustomerType.REGULAR);
        assertEquals(0.0, discount, 0.01);
    }

    @Test
    @DisplayName("Calculate discount for PREMIUM customer (10%)")
    void testCalculateDiscountPremium() {
        double discount = pricingService.calculateDiscount(100.0, CustomerType.PREMIUM);
        assertEquals(10.0, discount, 0.01);
    }

    @Test
    @DisplayName("Calculate discount for VIP customer (20%)")
    void testCalculateDiscountVIP() {
        double discount = pricingService.calculateDiscount(100.0, CustomerType.VIP);
        assertEquals(20.0, discount, 0.01);
    }

    @Test
    @DisplayName("Calculate discount throws exception for null customer type")
    void testCalculateDiscountNullType() {
        assertThrows(IllegalArgumentException.class, 
            () -> pricingService.calculateDiscount(100.0, null));
    }

    // Tests de calcul de frais de livraison
    @Test
    @DisplayName("Calculate shipping cost for distance < 50km")
    void testCalculateShippingCostNearby() {
        double cost = pricingService.calculateShippingCost(30.0);
        assertEquals(5.0, cost, 0.01);
    }

    @Test
    @DisplayName("Calculate shipping cost for distance between 50km and 200km")
    void testCalculateShippingCostMedium() {
        double cost = pricingService.calculateShippingCost(100.0);
        assertEquals(15.0, cost, 0.01);
    }

    @Test
    @DisplayName("Calculate shipping cost for distance >= 200km")
    void testCalculateShippingCostFar() {
        double cost = pricingService.calculateShippingCost(250.0);
        assertEquals(30.0, cost, 0.01);
    }

    @Test
    @DisplayName("Calculate shipping cost throws exception for negative distance")
    void testCalculateShippingCostNegativeDistance() {
        assertThrows(IllegalArgumentException.class, 
            () -> pricingService.calculateShippingCost(-10.0));
    }

    // Tests de calcul de prix final
    @Test
    @DisplayName("Calculate final price for complete scenario")
    void testCalculateFinalPrice() {
        // Base price: 100€, France (20% tax), PREMIUM (10% discount), 100km distance (15€ shipping)
        // Expected: (100 - 10) + (90 * 0.20) + 15 = 90 + 18 + 15 = 123€
        double finalPrice = pricingService.calculateFinalPrice(100.0, "FR", CustomerType.PREMIUM, 100.0);
        assertEquals(123.0, finalPrice, 0.01);
    }
}
```

---

## Partie 4 : Tests paramétrés avec @ParameterizedTest

### Prompt pour Copilot

```plaintext
Crée des tests paramétrés JUnit 5 avec @ParameterizedTest pour :
1. Tester plusieurs pays avec @CsvSource
2. Tester plusieurs types de clients avec @EnumSource
3. Tester plusieurs distances avec @ValueSource
```

### Code attendu : PricingServiceParameterizedTest.java

```java
package com.example.pricing;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;
import static org.junit.jupiter.api.Assertions.*;

class PricingServiceParameterizedTest {

    private PricingService pricingService;

    @BeforeEach
    void setUp() {
        pricingService = new PricingService();
    }

    @ParameterizedTest(name = "Tax for {0}: {1}% on 100€ = {2}€")
    @CsvSource({
        "FR, 20, 20.0",
        "BE, 21, 21.0",
        "DE, 19, 19.0"
    })
    void testCalculateTaxParameterized(String country, int taxRate, double expectedTax) {
        double actualTax = pricingService.calculateTax(100.0, country);
        assertEquals(expectedTax, actualTax, 0.01, 
            () -> "Tax for " + country + " should be " + taxRate + "%");
    }

    @ParameterizedTest(name = "Discount for {0}")
    @EnumSource(CustomerType.class)
    void testCalculateDiscountForAllTypes(CustomerType type) {
        double discount = pricingService.calculateDiscount(100.0, type);
        
        double expected = switch (type) {
            case REGULAR -> 0.0;
            case PREMIUM -> 10.0;
            case VIP -> 20.0;
        };
        
        assertEquals(expected, discount, 0.01);
    }

    @ParameterizedTest(name = "Shipping cost for {0}km")
    @ValueSource(doubles = {10.0, 49.9, 50.0, 150.0, 199.9, 200.0, 500.0})
    void testCalculateShippingCostVariousDistances(double distance) {
        double cost = pricingService.calculateShippingCost(distance);
        
        double expected;
        if (distance < 50) {
            expected = 5.0;
        } else if (distance < 200) {
            expected = 15.0;
        } else {
            expected = 30.0;
        }
        
        assertEquals(expected, cost, 0.01);
    }

    @ParameterizedTest(name = "Scenario {index}: Price={0}, Country={1}, Type={2}, Distance={3}")
    @CsvSource({
        "100.0, FR, REGULAR, 30.0, 125.0",     // 100 + 20 (tax) + 5 (shipping)
        "100.0, FR, PREMIUM, 100.0, 123.0",    // 90 + 18 (tax) + 15 (shipping)
        "100.0, BE, VIP, 250.0, 126.8",        // 80 + 16.8 (tax) + 30 (shipping)
        "200.0, DE, PREMIUM, 50.0, 233.2"      // 180 + 34.2 (tax) + 15 (shipping)
    })
    void testCalculateFinalPriceScenarios(double basePrice, String country, 
                                          CustomerType type, double distance, 
                                          double expectedPrice) {
        double actualPrice = pricingService.calculateFinalPrice(basePrice, country, type, distance);
        assertEquals(expectedPrice, actualPrice, 0.1, 
            () -> String.format("Final price incorrect for scenario: %.2f, %s, %s, %.2f", 
                basePrice, country, type, distance));
    }
}
```

---

## Partie 5 : Tests avec Mockito

### Créer un service externe

```java
package com.example.pricing;

public interface TaxRateProvider {
    double getTaxRate(String country);
}
```

### Modifier PricingService pour utiliser le provider

```java
package com.example.pricing;

public class PricingService {
    private final TaxRateProvider taxRateProvider;
    
    public PricingService(TaxRateProvider taxRateProvider) {
        this.taxRateProvider = taxRateProvider;
    }
    
    public double calculateTax(double amount, String country) {
        double taxRate = taxRateProvider.getTaxRate(country);
        return amount * taxRate;
    }
    
    // ... autres méthodes
}
```

### Tests avec Mockito

```java
package com.example.pricing;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(MockitoExtension.class)
class PricingServiceWithMockTest {

    @Mock
    private TaxRateProvider taxRateProvider;

    private PricingService pricingService;

    @BeforeEach
    void setUp() {
        pricingService = new PricingService(taxRateProvider);
    }

    @Test
    void testCalculateTaxWithMock() {
        // Arrange
        when(taxRateProvider.getTaxRate("FR")).thenReturn(0.20);
        
        // Act
        double tax = pricingService.calculateTax(100.0, "FR");
        
        // Assert
        assertEquals(20.0, tax, 0.01);
        verify(taxRateProvider, times(1)).getTaxRate("FR");
    }

    @Test
    void testCalculateTaxMultipleCountries() {
        // Arrange
        when(taxRateProvider.getTaxRate("FR")).thenReturn(0.20);
        when(taxRateProvider.getTaxRate("BE")).thenReturn(0.21);
        when(taxRateProvider.getTaxRate("DE")).thenReturn(0.19);
        
        // Act & Assert
        assertEquals(20.0, pricingService.calculateTax(100.0, "FR"), 0.01);
        assertEquals(21.0, pricingService.calculateTax(100.0, "BE"), 0.01);
        assertEquals(19.0, pricingService.calculateTax(100.0, "DE"), 0.01);
        
        verify(taxRateProvider).getTaxRate("FR");
        verify(taxRateProvider).getTaxRate("BE");
        verify(taxRateProvider).getTaxRate("DE");
        verifyNoMoreInteractions(taxRateProvider);
    }
}
```

---

## Partie 6 : Exécuter tous les tests

```bash
mvn clean test
```

### Résultat attendu

```plaintext
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.example.pricing.PricingServiceTest
[INFO] Tests run: 14, Failures: 0, Errors: 0, Skipped: 0
[INFO] Running com.example.pricing.PricingServiceParameterizedTest
[INFO] Tests run: 15, Failures: 0, Errors: 0, Skipped: 0
[INFO] Running com.example.pricing.PricingServiceWithMockTest
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 31, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] BUILD SUCCESS
```

---

## Récapitulatif

### Ce que vous avez appris

✅ Générer des tests JUnit 5 avec GitHub Copilot  
✅ Créer des tests paramétrés avec @ParameterizedTest  
✅ Utiliser Mockito pour les mocks  
✅ Organiser les tests de manière structurée  
✅ Écrire des assertions claires et expressives  

### Bonnes pratiques

1. **@DisplayName** : Descriptions claires des tests
2. **@BeforeEach** : Initialisation commune
3. **Tests paramétrés** : Éviter la duplication
4. **Mocks** : Isoler le code testé
5. **Assertions** : Messages d'erreur informatifs

---

## Exercices supplémentaires

1. Ajouter des tests pour la gestion des arrondis
2. Créer une suite de tests avec @Suite
3. Utiliser @Nested pour grouper les tests
4. Ajouter des tests de performance avec @Timeout
5. Implémenter des tests de couverture de code

Utilisez GitHub Copilot pour chacun de ces exercices !
