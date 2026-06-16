# LAB 00 - Configuration de l'environnement pour GitHub Copilot (Java)

## Description
Vérifier les prérequis et configurer les ressources avant de commencer les exercices GitHub Copilot avec Java.

---

## Prérequis de l'environnement

Votre environnement de laboratoire doit être configuré pour le développement Java avec Visual Studio Code et GitHub Copilot. L'accès à un compte GitHub avec GitHub Copilot activé est requis.

### Étape 1 : Vérifier Git

Vérifiez que Git version 2.48 ou ultérieure est installé :

```bash
git --version
```

Si nécessaire, téléchargez Git depuis : https://git-scm.com/downloads

### Étape 2 : Vérifier Java (JDK)

Vérifiez que le JDK 17 ou ultérieur est installé :

```bash
java --version
```

Si nécessaire, téléchargez le JDK depuis :
- OpenJDK : https://adoptium.net/
- Oracle JDK : https://www.oracle.com/java/technologies/downloads/

### Étape 3 : Vérifier Maven

Vérifiez que Maven 3.8 ou ultérieur est installé :

```bash
mvn --version
```

Si nécessaire, téléchargez Maven depuis : https://maven.apache.org/download.cgi

### Étape 4 : Vérifier Visual Studio Code

Vérifiez que Visual Studio Code et l'Extension Pack for Java sont installés :

1. Téléchargez VS Code : https://code.visualstudio.com/download
2. Dans VS Code, installez **Extension Pack for Java** (Microsoft)

### Étape 5 : Configurer GitHub Copilot

1. Connectez-vous à votre compte GitHub : https://github.com/login

2. Si vous n'avez pas de compte GitHub, créez-en un gratuitement

3. Ouvrez votre page de paramètres GitHub et vérifiez votre abonnement GitHub Copilot

4. Si vous avez un abonnement actif (Pro, Pro+, Business, Enterprise), utilisez-le pour la formation

5. Si vous avez un compte individuel sans abonnement, vous pouvez configurer **GitHub Copilot Free** depuis VS Code pendant les exercices

**IMPORTANT** : Le plan GitHub Copilot Free est limité à 2 000 complétions de code et 50 chats ou requêtes premium par mois.

---

## Configuration de Git

Configurez Git avec votre nom et email :

```bash
git config --global user.name "Votre Nom"
git config --global user.email "votre.email@example.com"
```

---

## Activation de GitHub Copilot dans VS Code

1. Ouvrez Visual Studio Code

2. Cliquez sur l'icône **Extensions** (barre latérale gauche)

3. Recherchez et installez :
   - **GitHub Copilot**
   - **GitHub Copilot Chat**

4. Cliquez sur l'icône **Comptes** en bas à gauche

5. Sélectionnez **Sign in to GitHub to use GitHub Copilot**

6. Suivez les instructions pour autoriser VS Code à accéder à votre compte GitHub

7. Une fois connecté, GitHub Copilot est activé

---

## Vérification de la configuration

### Test 1 : GitHub Copilot activé

1. Créez un fichier Java de test : `Test.java`

2. Tapez :
```java
public class Test {
    // Fonction pour calculer la somme de deux nombres
```

3. GitHub Copilot devrait suggérer automatiquement le code de la méthode

4. Si vous voyez des suggestions, GitHub Copilot fonctionne ! 

### Test 2 : GitHub Copilot Chat

1. Ouvrez le Chat (Ctrl+Shift+I ou icône de chat)

2. Tapez :
```plaintext
Comment créer une classe Java avec des getters et setters ?
```

3. Vous devriez recevoir une réponse de GitHub Copilot 

---

## Résumé de la configuration

 Git installé et configuré  
 JDK 17+ installé  
 Maven 3.8+ installé  
 VS Code avec Extension Pack for Java  
 GitHub Copilot activé  
 GitHub Copilot Chat fonctionnel  

Vous êtes prêt à commencer les exercices !

---

## Ressources supplémentaires

- Documentation GitHub Copilot : https://docs.github.com/copilot
- Extension Pack for Java : https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack
- Maven Getting Started : https://maven.apache.org/guides/getting-started/
