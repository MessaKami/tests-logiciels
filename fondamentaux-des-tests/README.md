# Les Fondamentaux des Tests

## Introduction

Les tests sont une partie essentielle du développement logiciel. Ils nous permettent de vérifier que notre code fonctionne comme prévu et de détecter les bugs avant qu'ils n'atteignent la production.

## Vocabulaire des Tests

### 1. Test Fixture 🔧

Un "Test Fixture" représente l'environnement de test. C'est comme préparer votre cuisine avant de commencer à cuisiner :
- Vous sortez tous les ingrédients
- Vous préparez vos ustensiles
- Vous préchauffez le four

En Java avec JUnit :
```java
@BeforeEach
void setUp() {
    // Préparation de l'environnement
    calculatrice = new Calculatrice();
    utilisateur = new Utilisateur("test");
}
```

### 2. Test Suite 📚

Une Test Suite est un regroupement logique de tests, comme les chapitres d'un livre. Par exemple :
- Tests des opérations mathématiques
  - Tests d'addition
  - Tests de soustraction
- Tests de validation des données
  - Tests des emails
  - Tests des mots de passe

```java
@Nested
class OperationsMathematiques {
    @Test void testAddition() { /* ... */ }
    @Test void testSoustraction() { /* ... */ }
}
```

### 3. Test Case 📝

Un Test Case est un scénario de test spécifique. Il suit généralement le pattern "Arrange-Act-Assert" :
1. **Arrange** : Préparer les données
2. **Act** : Exécuter l'action à tester
3. **Assert** : Vérifier le résultat

```java
@Test
void testAdditionDeDeuxNombres() {
    // Arrange
    int a = 2, b = 3;
    
    // Act
    int resultat = calculatrice.additionner(a, b);
    
    // Assert
    assertEquals(5, resultat);
}
```

### 4. System Under Test (SUT) 🎯

Le SUT est le composant que vous testez. C'est comme le plat que vous êtes en train de préparer :
- Si vous testez une classe `Calculatrice`, c'est votre SUT
- Tout le reste (données de test, mocks) sont des éléments de support

```java
public class Calculatrice {  // Ceci est le SUT
    public int additionner(int a, int b) {
        return a + b;
    }
}
```

### 5. Test Runner 🏃

Le Test Runner est le chef d'orchestre qui :
- Découvre les tests
- Les exécute dans un ordre défini
- Collecte les résultats
- Génère des rapports

Dans JUnit, c'est intégré et automatique. Il suffit de lancer :
```bash
mvn test
```

### 6. Assertions ✅

Les assertions sont vos vérifications. Comme un chef qui goûte son plat pour vérifier :
- Le goût
- La texture
- La température

```java
@Test
void testValidationEmail() {
    // Différents types d'assertions
    assertTrue(email.estValide());        // Vérifie si c'est vrai
    assertEquals(5, resultat);            // Vérifie l'égalité
    assertNotNull(utilisateur);           // Vérifie si non null
    assertThrows(Exception.class, () -> { // Vérifie si une exception est lancée
        diviserParZero();
    });
}
```

### 7. Mocks, Stubs, Spies 🕵️

Ces éléments permettent d'isoler le code testé en simulant des dépendances :

#### Mock
Un remplaçant complet d'un objet :
```java
@Mock
private BaseDeDonnees bdd;
when(bdd.recupererUtilisateur(1)).thenReturn(new Utilisateur("test"));
```

#### Stub
Une implémentation simplifiée qui retourne des valeurs fixes :
```java
// Stub d'une API météo
class MeteoServiceStub implements MeteoService {
    public Temperature getTemperature() {
        return new Temperature(20); // Retourne toujours 20°C
    }
}
```

#### Spy
Un observateur qui surveille comment un objet est utilisé :
```java
List<String> spy = spy(new ArrayList<>());
spy.add("test");
verify(spy).add("test"); // Vérifie que add() a été appelé avec "test"
```

## Points Clés à Retenir 🗝️

1. Les tests doivent être :
   - Indépendants (pas d'influence entre les tests)
   - Répétables (même résultat à chaque exécution)
   - Simples à comprendre
   - Maintenables

2. La pyramide de tests recommande :
   - Beaucoup de tests unitaires (base)
   - Moins de tests d'intégration (milieu)
   - Peu de tests end-to-end (sommet)

3. Un bon test suit le principe "FIRST" :
   - **F**ast (rapide à exécuter)
   - **I**ndependent (indépendant des autres tests)
   - **R**epeatable (répétable)
   - **S**elf-validating (auto-validant)
   - **T**imely (écrit au bon moment) 