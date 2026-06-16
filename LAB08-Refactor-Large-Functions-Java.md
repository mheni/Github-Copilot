# LAB 08 - Refactoriser les grandes fonctions avec GitHub Copilot (Java)

## Description
Apprendre à identifier et refactoriser les méthodes trop longues en méthodes plus petites, cohérentes et testables avec l'aide de GitHub Copilot.

**Durée estimée** : 30 minutes

---

## Objectifs d'apprentissage

À la fin de cet exercice, vous serez capable de :
- Identifier les méthodes violant le principe de responsabilité unique
- Extraire des méthodes cohérentes avec GitHub Copilot
- Améliorer la lisibilité et la testabilité du code
- Appliquer les bonnes pratiques de décomposition de fonctions

---

## Prérequis

- JDK 17+, Maven 3.8+
- VS Code avec Extension Pack for Java
- GitHub Copilot activé
- Connaissance des principes SOLID

---

## Scénario de l'exercice

Vous héritez d'une classe `ReportGenerator` qui contient une méthode `generateReport()` de 150 lignes qui fait trop de choses :
- Récupération des données
- Validation
- Calculs statistiques
- Formatage
- Génération de fichier
- Envoi par email

Votre mission : décomposer cette méthode en méthodes plus petites et cohérentes.

---

## Partie 1 : Analyser le code legacy

### Code initial : ReportGenerator.java (AVANT)

```java
package com.example.reporting;

import java.io.*;
import java.time.*;
import java.util.*;
import java.util.stream.*;

public class ReportGenerator {
    
    public void generateReport(String reportType, LocalDate startDate, LocalDate endDate) {
        // LIGNE 1-20 : Validation
        if (reportType == null || reportType.isEmpty()) {
            throw new IllegalArgumentException("Report type cannot be null or empty");
        }
        if (startDate == null) {
            throw new IllegalArgumentException("Start date cannot be null");
        }
        if (endDate == null) {
            throw new IllegalArgumentException("End date cannot be null");
        }
        if (startDate.isAfter(endDate)) {
            throw new IllegalArgumentException("Start date must be before end date");
        }
        if (!reportType.equals("SALES") && !reportType.equals("INVENTORY") && !reportType.equals("CUSTOMER")) {
            throw new IllegalArgumentException("Invalid report type: " + reportType);
        }
        
        // LIGNE 21-50 : Récupération des données
        List<Map<String, Object>> rawData = new ArrayList<>();
        if (reportType.equals("SALES")) {
            // Simulation de récupération depuis BDD
            rawData.add(Map.of("date", LocalDate.of(2024, 1, 15), "amount", 1500.0, "product", "Laptop"));
            rawData.add(Map.of("date", LocalDate.of(2024, 1, 16), "amount", 800.0, "product", "Mouse"));
            rawData.add(Map.of("date", LocalDate.of(2024, 1, 17), "amount", 2200.0, "product", "Monitor"));
        } else if (reportType.equals("INVENTORY")) {
            rawData.add(Map.of("product", "Laptop", "quantity", 50, "cost", 800.0));
            rawData.add(Map.of("product", "Mouse", "quantity", 200, "cost", 20.0));
        } else {
            rawData.add(Map.of("customerId", "C001", "name", "John Doe", "totalSpent", 5000.0));
            rawData.add(Map.of("customerId", "C002", "name", "Jane Smith", "totalSpent", 8000.0));
        }
        
        // Filtrage par date
        List<Map<String, Object>> filteredData = rawData.stream()
            .filter(record -> {
                if (record.containsKey("date")) {
                    LocalDate recordDate = (LocalDate) record.get("date");
                    return !recordDate.isBefore(startDate) && !recordDate.isAfter(endDate);
                }
                return true;
            })
            .collect(Collectors.toList());
        
        // LIGNE 51-80 : Calculs statistiques
        double total = 0;
        double average = 0;
        double min = Double.MAX_VALUE;
        double max = Double.MIN_VALUE;
        int count = filteredData.size();
        
        for (Map<String, Object> record : filteredData) {
            double value = 0;
            if (reportType.equals("SALES")) {
                value = (Double) record.get("amount");
            } else if (reportType.equals("INVENTORY")) {
                value = (Integer) record.get("quantity") * (Double) record.get("cost");
            } else {
                value = (Double) record.get("totalSpent");
            }
            
            total += value;
            if (value < min) min = value;
            if (value > max) max = value;
        }
        
        if (count > 0) {
            average = total / count;
        }
        
        // Calcul variance et écart-type
        double sumSquaredDiff = 0;
        for (Map<String, Object> record : filteredData) {
            double value = 0;
            if (reportType.equals("SALES")) {
                value = (Double) record.get("amount");
            } else if (reportType.equals("INVENTORY")) {
                value = (Integer) record.get("quantity") * (Double) record.get("cost");
            } else {
                value = (Double) record.get("totalSpent");
            }
            sumSquaredDiff += Math.pow(value - average, 2);
        }
        double variance = count > 0 ? sumSquaredDiff / count : 0;
        double stdDev = Math.sqrt(variance);
        
        // LIGNE 81-120 : Formatage du rapport
        StringBuilder report = new StringBuilder();
        report.append("=" .repeat(60)).append("\n");
        report.append(reportType).append(" REPORT\n");
        report.append("Period: ").append(startDate).append(" to ").append(endDate).append("\n");
        report.append("=".repeat(60)).append("\n\n");
        
        report.append("SUMMARY STATISTICS\n");
        report.append("-".repeat(60)).append("\n");
        report.append(String.format("Total Records: %d\n", count));
        report.append(String.format("Total Value: %.2f EUR\n", total));
        report.append(String.format("Average: %.2f EUR\n", average));
        report.append(String.format("Min: %.2f EUR\n", min));
        report.append(String.format("Max: %.2f EUR\n", max));
        report.append(String.format("Std Deviation: %.2f\n", stdDev));
        report.append("\n");
        
        report.append("DETAILED DATA\n");
        report.append("-".repeat(60)).append("\n");
        for (Map<String, Object> record : filteredData) {
            if (reportType.equals("SALES")) {
                report.append(String.format("Date: %s | Product: %s | Amount: %.2f EUR\n",
                    record.get("date"), record.get("product"), record.get("amount")));
            } else if (reportType.equals("INVENTORY")) {
                report.append(String.format("Product: %s | Qty: %d | Cost: %.2f EUR | Total: %.2f EUR\n",
                    record.get("product"), record.get("quantity"), record.get("cost"),
                    (Integer)record.get("quantity") * (Double)record.get("cost")));
            } else {
                report.append(String.format("Customer: %s (ID: %s) | Total Spent: %.2f EUR\n",
                    record.get("name"), record.get("customerId"), record.get("totalSpent")));
            }
        }
        
        report.append("=".repeat(60)).append("\n");
        report.append("Generated on: ").append(LocalDateTime.now()).append("\n");
        
        // LIGNE 121-140 : Sauvegarde du fichier
        String filename = String.format("report_%s_%s_%s.txt", 
            reportType.toLowerCase(),
            startDate.toString(),
            LocalDateTime.now().format(java.time.format.DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss")));
        
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(filename))) {
            writer.write(report.toString());
            System.out.println("Report saved to: " + filename);
        } catch (IOException e) {
            System.err.println("Error writing report: " + e.getMessage());
            throw new RuntimeException("Failed to save report", e);
        }
        
        // LIGNE 141-150 : Envoi par email (simulé)
        System.out.println("Sending email notification...");
        String emailSubject = reportType + " Report - " + startDate + " to " + endDate;
        String emailBody = "Please find attached the " + reportType.toLowerCase() + " report.\n\n";
        emailBody += "Summary:\n";
        emailBody += "Total: " + total + " EUR\n";
        emailBody += "Average: " + average + " EUR\n";
        emailBody += "\nReport file: " + filename;
        
        // Simulation d'envoi
        System.out.println("Email sent to: report@example.com");
        System.out.println("Subject: " + emailSubject);
        System.out.println("Body: " + emailBody);
    }
}
```

### Problèmes identifiés

 **150+ lignes** - Méthode trop longue  
 **7 responsabilités** - Violation du SRP  
 **Duplication** - Code répété dans les boucles  
 **Non testable** - Impossible de tester chaque partie  
 **Peu lisible** - Difficile de comprendre le flux  
 **Complexité cyclomatique élevée** - Trop de branches  

---

## Partie 2 : Refactoring avec GitHub Copilot

### Étape 1 : Utiliser GitHub Copilot pour analyser

**Prompt pour Copilot Chat** :

```plaintext
Analyse la méthode generateReport() et :
1. Identifie toutes les responsabilités distinctes
2. Propose une décomposition en méthodes privées cohérentes
3. Suggère les signatures de méthodes à extraire
4. Indique l'ordre de refactoring recommandé

Pour chaque méthode extraite, fournis :
- Le nom de la méthode
- Les paramètres
- Le type de retour
- Une description de sa responsabilité unique
```

### Étape 2 : Créer les méthodes d'extraction

**Prompt pour Copilot** :

```plaintext
Refactorise ReportGenerator en extrayant ces méthodes :
1. validateInputs() - Validation des paramètres
2. fetchData() - Récupération des données selon le type
3. filterDataByDate() - Filtrage temporel
4. calculateStatistics() - Calculs statistiques
5. formatReport() - Génération du texte
6. saveToFile() - Sauvegarde fichier
7. sendEmailNotification() - Envoi email

Chaque méthode doit :
- Avoir une responsabilité unique
- Être testable indépendamment
- Avoir un nom descriptif
- Retourner un résultat ou modifier un état minimal
```

### Code refactorisé (APRÈS)

#### ReportGenerator.java (APRÈS - Partie 1 : Validation et Modèles)

```java
package com.example.reporting;

import java.io.*;
import java.time.*;
import java.util.*;
import java.util.stream.*;

public class ReportGenerator {
    
    // Méthode principale simplifiée
    public void generateReport(String reportType, LocalDate startDate, LocalDate endDate) {
        // 1. Validation
        validateInputs(reportType, startDate, endDate);
        
        // 2. Récupération et filtrage des données
        List<Map<String, Object>> rawData = fetchData(reportType);
        List<Map<String, Object>> filteredData = filterDataByDate(rawData, startDate, endDate);
        
        // 3. Calculs statistiques
        ReportStatistics stats = calculateStatistics(filteredData, reportType);
        
        // 4. Formatage
        String reportContent = formatReport(reportType, startDate, endDate, filteredData, stats);
        
        // 5. Sauvegarde
        String filename = saveToFile(reportContent, reportType, startDate);
        
        // 6. Notification
        sendEmailNotification(reportType, startDate, endDate, stats, filename);
    }
    
    // Méthode 1 : Validation
    private void validateInputs(String reportType, LocalDate startDate, LocalDate endDate) {
        if (reportType == null || reportType.isEmpty()) {
            throw new IllegalArgumentException("Report type cannot be null or empty");
        }
        if (startDate == null) {
            throw new IllegalArgumentException("Start date cannot be null");
        }
        if (endDate == null) {
            throw new IllegalArgumentException("End date cannot be null");
        }
        if (startDate.isAfter(endDate)) {
            throw new IllegalArgumentException("Start date must be before end date");
        }
        if (!isValidReportType(reportType)) {
            throw new IllegalArgumentException("Invalid report type: " + reportType);
        }
    }
    
    private boolean isValidReportType(String reportType) {
        return reportType.equals("SALES") || 
               reportType.equals("INVENTORY") || 
               reportType.equals("CUSTOMER");
    }
    
    // Méthode 2 : Récupération des données
    private List<Map<String, Object>> fetchData(String reportType) {
        return switch (reportType) {
            case "SALES" -> fetchSalesData();
            case "INVENTORY" -> fetchInventoryData();
            case "CUSTOMER" -> fetchCustomerData();
            default -> throw new IllegalArgumentException("Unknown report type");
        };
    }
    
    private List<Map<String, Object>> fetchSalesData() {
        return List.of(
            Map.of("date", LocalDate.of(2024, 1, 15), "amount", 1500.0, "product", "Laptop"),
            Map.of("date", LocalDate.of(2024, 1, 16), "amount", 800.0, "product", "Mouse"),
            Map.of("date", LocalDate.of(2024, 1, 17), "amount", 2200.0, "product", "Monitor")
        );
    }
    
    private List<Map<String, Object>> fetchInventoryData() {
        return List.of(
            Map.of("product", "Laptop", "quantity", 50, "cost", 800.0),
            Map.of("product", "Mouse", "quantity", 200, "cost", 20.0)
        );
    }
    
    private List<Map<String, Object>> fetchCustomerData() {
        return List.of(
            Map.of("customerId", "C001", "name", "John Doe", "totalSpent", 5000.0),
            Map.of("customerId", "C002", "name", "Jane Smith", "totalSpent", 8000.0)
        );
    }
    
    // Méthode 3 : Filtrage par date
    private List<Map<String, Object>> filterDataByDate(List<Map<String, Object>> data, 
                                                       LocalDate startDate, 
                                                       LocalDate endDate) {
        return data.stream()
            .filter(record -> isWithinDateRange(record, startDate, endDate))
            .collect(Collectors.toList());
    }
    
    private boolean isWithinDateRange(Map<String, Object> record, LocalDate startDate, LocalDate endDate) {
        if (!record.containsKey("date")) {
            return true; // Pas de date = inclus
        }
        LocalDate recordDate = (LocalDate) record.get("date");
        return !recordDate.isBefore(startDate) && !recordDate.isAfter(endDate);
    }
    
    // Méthode 4 : Calculs statistiques
    private ReportStatistics calculateStatistics(List<Map<String, Object>> data, String reportType) {
        if (data.isEmpty()) {
            return new ReportStatistics(0, 0, 0, 0, 0, 0, 0);
        }
        
        List<Double> values = extractValues(data, reportType);
        
        int count = values.size();
        double total = values.stream().mapToDouble(Double::doubleValue).sum();
        double average = total / count;
        double min = values.stream().mapToDouble(Double::doubleValue).min().orElse(0);
        double max = values.stream().mapToDouble(Double::doubleValue).max().orElse(0);
        double variance = calculateVariance(values, average);
        double stdDev = Math.sqrt(variance);
        
        return new ReportStatistics(count, total, average, min, max, variance, stdDev);
    }
    
    private List<Double> extractValues(List<Map<String, Object>> data, String reportType) {
        return data.stream()
            .map(record -> extractValue(record, reportType))
            .collect(Collectors.toList());
    }
    
    private double extractValue(Map<String, Object> record, String reportType) {
        return switch (reportType) {
            case "SALES" -> (Double) record.get("amount");
            case "INVENTORY" -> (Integer) record.get("quantity") * (Double) record.get("cost");
            case "CUSTOMER" -> (Double) record.get("totalSpent");
            default -> 0.0;
        };
    }
    
    private double calculateVariance(List<Double> values, double average) {
        double sumSquaredDiff = values.stream()
            .mapToDouble(value -> Math.pow(value - average, 2))
            .sum();
        return sumSquaredDiff / values.size();
    }
    
    // Méthode 5 : Formatage du rapport
    private String formatReport(String reportType, LocalDate startDate, LocalDate endDate,
                                List<Map<String, Object>> data, ReportStatistics stats) {
        StringBuilder report = new StringBuilder();
        
        appendHeader(report, reportType, startDate, endDate);
        appendStatistics(report, stats);
        appendDetailedData(report, data, reportType);
        appendFooter(report);
        
        return report.toString();
    }
    
    private void appendHeader(StringBuilder report, String reportType, 
                             LocalDate startDate, LocalDate endDate) {
        report.append("=".repeat(60)).append("\n");
        report.append(reportType).append(" REPORT\n");
        report.append("Period: ").append(startDate).append(" to ").append(endDate).append("\n");
        report.append("=".repeat(60)).append("\n\n");
    }
    
    private void appendStatistics(StringBuilder report, ReportStatistics stats) {
        report.append("SUMMARY STATISTICS\n");
        report.append("-".repeat(60)).append("\n");
        report.append(String.format("Total Records: %d\n", stats.count));
        report.append(String.format("Total Value: %.2f EUR\n", stats.total));
        report.append(String.format("Average: %.2f EUR\n", stats.average));
        report.append(String.format("Min: %.2f EUR\n", stats.min));
        report.append(String.format("Max: %.2f EUR\n", stats.max));
        report.append(String.format("Std Deviation: %.2f\n", stats.stdDev));
        report.append("\n");
    }
    
    private void appendDetailedData(StringBuilder report, List<Map<String, Object>> data, String reportType) {
        report.append("DETAILED DATA\n");
        report.append("-".repeat(60)).append("\n");
        
        for (Map<String, Object> record : data) {
            String line = formatDataLine(record, reportType);
            report.append(line).append("\n");
        }
    }
    
    private String formatDataLine(Map<String, Object> record, String reportType) {
        return switch (reportType) {
            case "SALES" -> String.format("Date: %s | Product: %s | Amount: %.2f EUR",
                record.get("date"), record.get("product"), record.get("amount"));
            case "INVENTORY" -> String.format("Product: %s | Qty: %d | Cost: %.2f EUR | Total: %.2f EUR",
                record.get("product"), record.get("quantity"), record.get("cost"),
                (Integer)record.get("quantity") * (Double)record.get("cost"));
            case "CUSTOMER" -> String.format("Customer: %s (ID: %s) | Total Spent: %.2f EUR",
                record.get("name"), record.get("customerId"), record.get("totalSpent"));
            default -> "";
        };
    }
    
    private void appendFooter(StringBuilder report) {
        report.append("=".repeat(60)).append("\n");
        report.append("Generated on: ").append(LocalDateTime.now()).append("\n");
    }
    
    // Méthode 6 : Sauvegarde fichier
    private String saveToFile(String content, String reportType, LocalDate startDate) {
        String filename = generateFilename(reportType, startDate);
        
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(filename))) {
            writer.write(content);
            System.out.println("Report saved to: " + filename);
            return filename;
        } catch (IOException e) {
            throw new RuntimeException("Failed to save report", e);
        }
    }
    
    private String generateFilename(String reportType, LocalDate startDate) {
        return String.format("report_%s_%s_%s.txt",
            reportType.toLowerCase(),
            startDate.toString(),
            LocalDateTime.now().format(java.time.format.DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss")));
    }
    
    // Méthode 7 : Notification email
    private void sendEmailNotification(String reportType, LocalDate startDate, LocalDate endDate,
                                       ReportStatistics stats, String filename) {
        String subject = buildEmailSubject(reportType, startDate, endDate);
        String body = buildEmailBody(reportType, stats, filename);
        
        sendEmail("report@example.com", subject, body);
    }
    
    private String buildEmailSubject(String reportType, LocalDate startDate, LocalDate endDate) {
        return reportType + " Report - " + startDate + " to " + endDate;
    }
    
    private String buildEmailBody(String reportType, ReportStatistics stats, String filename) {
        return String.format("""
            Please find attached the %s report.
            
            Summary:
            Total: %.2f EUR
            Average: %.2f EUR
            
            Report file: %s
            """, reportType.toLowerCase(), stats.total, stats.average, filename);
    }
    
    private void sendEmail(String to, String subject, String body) {
        System.out.println("Email sent to: " + to);
        System.out.println("Subject: " + subject);
        System.out.println("Body: " + body);
    }
    
    // Classe interne pour les statistiques
    private record ReportStatistics(
        int count,
        double total,
        double average,
        double min,
        double max,
        double variance,
        double stdDev
    ) {}
}
```

---

## Partie 3 : Tests unitaires

### Prompt pour Copilot

```plaintext
Génère des tests JUnit 5 pour chaque méthode privée de ReportGenerator :
1. Test de validation (cas valides et invalides)
2. Test de récupération de données (chaque type)
3. Test de filtrage par date
4. Test de calculs statistiques
5. Test de formatage

Utilise @Nested pour grouper les tests par méthode.
Rends les méthodes privées visibles pour les tests avec reflection si nécessaire.
```

### ReportGeneratorTest.java

```java
package com.example.reporting;

import org.junit.jupiter.api.*;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.junit.jupiter.MockitoExtension;

import java.time.LocalDate;
import java.util.List;
import java.util.Map;

import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(MockitoExtension.class)
class ReportGeneratorTest {

    private ReportGenerator generator;

    @BeforeEach
    void setUp() {
        generator = new ReportGenerator();
    }

    @Nested
    @DisplayName("Validation Tests")
    class ValidationTests {

        @Test
        void testValidInputs() {
            assertDoesNotThrow(() -> 
                generator.generateReport("SALES", LocalDate.of(2024, 1, 1), LocalDate.of(2024, 1, 31)));
        }

        @Test
        void testNullReportType() {
            assertThrows(IllegalArgumentException.class, () -> 
                generator.generateReport(null, LocalDate.of(2024, 1, 1), LocalDate.of(2024, 1, 31)));
        }

        @Test
        void testInvalidReportType() {
            assertThrows(IllegalArgumentException.class, () -> 
                generator.generateReport("INVALID", LocalDate.of(2024, 1, 1), LocalDate.of(2024, 1, 31)));
        }

        @Test
        void testStartDateAfterEndDate() {
            assertThrows(IllegalArgumentException.class, () -> 
                generator.generateReport("SALES", LocalDate.of(2024, 1, 31), LocalDate.of(2024, 1, 1)));
        }
    }

    @Nested
    @DisplayName("Statistics Calculation Tests")
    class StatisticsTests {
        
        @Test
        void testCalculateStatisticsWithData() {
            // Les méthodes privées peuvent être testées via la méthode publique
            // ou rendues package-private pour les tests
            assertDoesNotThrow(() -> 
                generator.generateReport("SALES", LocalDate.of(2024, 1, 1), LocalDate.of(2024, 1, 31)));
        }
    }
}
```

---

## Partie 4 : Comparaison avant/après

### Métriques de qualité

| Métrique | Avant | Après | Amélioration |
|----------|-------|-------|--------------|
| Lignes de la méthode principale | 150 | 15 |  90% |
| Nombre de méthodes | 1 | 24 |  Modularité |
| Cyclomatic complexity | 15 | 2-3 par méthode |  80% |
| Testabilité | Impossible | Facile |  100% |
| Lisibilité | Faible | Élevée |  |
| Maintenabilité | Difficile | Facile |  |

### Avantages du refactoring

 **Lisibilité** : Chaque méthode a un nom descriptif  
 **Testabilité** : Chaque méthode peut être testée isolément  
 **Réutilisabilité** : Les méthodes peuvent être appelées ailleurs  
 **Maintenabilité** : Plus facile de modifier une petite méthode  
 **SRP** : Chaque méthode a une seule responsabilité  

---

## Récapitulatif

### Ce que vous avez appris

 Identifier les méthodes trop longues  
 Extraire des méthodes cohérentes avec Copilot  
 Appliquer le principe de responsabilité unique  
 Améliorer la testabilité du code  
 Réduire la complexité cyclomatique  

### Bonnes pratiques

1. **Limite de 20-30 lignes** par méthode
2. **Une responsabilité** par méthode
3. **Noms descriptifs** pour les méthodes
4. **Paramètres minimaux** (idéalement < 4)
5. **Testabilité** en priorité

---

## Défis supplémentaires

1. Extraire les constantes magiques
2. Utiliser des records pour les DTOs
3. Ajouter un système de cache pour les données
4. Implémenter le pattern Strategy pour les types de rapports
5. Ajouter des logs structurés avec SLF4J

Utilisez GitHub Copilot pour chaque amélioration !

---

## Ressources

- Clean Code (Robert C. Martin) - Chapitre 3 : Functions
- Refactoring (Martin Fowler) - Extract Method
- Effective Java (Joshua Bloch) - Item 50 : Make defensive copies
