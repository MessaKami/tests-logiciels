# Test Runner 🏃

## Qu'est-ce qu'un Test Runner ?

Un Test Runner est le moteur qui exécute les tests et rapporte les résultats. C'est comme un arbitre dans un match sportif : il organise le déroulement, applique les règles et annonce les résultats.

## Configuration avec JUnit 5

### 1. Configuration Maven
```xml
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-engine</artifactId>
        <version>5.9.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.0.0</version>
        </plugin>
    </plugins>
</build>
```

## Fonctionnalités du Test Runner

### 1. Découverte des Tests
```java
@DisplayName("Suite de Tests Calculatrice")
public class CalculatriceTest {
    // Le Test Runner trouve automatiquement ces tests
    @Test
    void addition() { /* ... */ }

    @Test
    void soustraction() { /* ... */ }
    
    @TestFactory  // Tests dynamiques
    Stream<DynamicTest> testsMultiplication() {
        return Stream.of(
            dynamicTest("2 x 2 = 4", () -> assertEquals(4, calculer(2, 2))),
            dynamicTest("3 x 3 = 9", () -> assertEquals(9, calculer(3, 3)))
        );
    }
}
```

### 2. Ordre d'Exécution
```java
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class ProcessusCommandeTest {
    @Test
    @Order(1)
    void verifierPanier() { /* ... */ }
    
    @Test
    @Order(2)
    void validerPaiement() { /* ... */ }
    
    @Test
    @Order(3)
    void confirmerCommande() { /* ... */ }
}
```

### 3. Filtrage des Tests
```java
@Tag("integration")
class TestsIntegration {
    @Test
    @Tag("lent")
    void testConnexionBDD() { /* ... */ }
    
    @Test
    @Tag("rapide")
    void testCache() { /* ... */ }
}

// Dans pom.xml
<configuration>
    <groups>rapide</groups>
    <excludedGroups>lent</excludedGroups>
</configuration>
```

## Exemple Complet : Configuration et Utilisation

```java
@ExtendWith(MockitoExtension.class)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class SystemeVenteTest {
    
    @Nested
    @DisplayName("Tests Unitaires")
    @Tag("unitaire")
    class TestsUnitaires {
        @Test
        void calculPrix() { /* ... */ }
    }
    
    @Nested
    @DisplayName("Tests d'Intégration")
    @Tag("integration")
    class TestsIntegration {
        @Test
        void processusComplet() { /* ... */ }
    }
    
    @TestFactory
    @DisplayName("Tests Paramétrés")
    Stream<DynamicTest> testsParametres() {
        return Stream.of(
            dynamicTest("Scénario 1", () -> { /* ... */ }),
            dynamicTest("Scénario 2", () -> { /* ... */ })
        );
    }
}
```

## Configuration Avancée

### 1. Parallel Test Execution
```java
// junit-platform.properties
junit.jupiter.execution.parallel.enabled=true
junit.jupiter.execution.parallel.mode.default=concurrent

@Execution(ExecutionMode.CONCURRENT)
class TestsParalleles {
    @Test
    void test1() { /* ... */ }
    
    @Test
    void test2() { /* ... */ }
}
```

### 2. Custom Test Engine
```java
public class MonTestEngine implements TestEngine {
    @Override
    public String getId() {
        return "mon-test-engine";
    }
    
    @Override
    public TestDescriptor discover(EngineDiscoveryRequest request, UniqueId uniqueId) {
        // Logique de découverte des tests
    }
    
    @Override
    public void execute(ExecutionRequest request) {
        // Logique d'exécution des tests
    }
}
```

## Rapports et Résultats

### 1. Configuration des Rapports Maven
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <reportFormat>detailed</reportFormat>
        <includes>
            <include>**/*Test.java</include>
        </includes>
    </configuration>
</plugin>
```

### 2. Listeners Personnalisés
```java
public class MonTestListener implements TestExecutionListener {
    @Override
    public void testSuccessful(TestIdentifier testIdentifier) {
        System.out.println("Test réussi : " + testIdentifier.getDisplayName());
    }
    
    @Override
    public void testFailed(TestIdentifier testIdentifier, Throwable throwable) {
        System.err.println("Test échoué : " + testIdentifier.getDisplayName());
        throwable.printStackTrace();
    }
}
```

## Bonnes Pratiques 👍

1. **Organisation des Tests**
```java
// ✅ Bon : Tests bien organisés
@Tag("integration")
@DisplayName("Tests d'intégration du panier")
class PanierIntegrationTest { /* ... */ }

// ❌ Mauvais : Tests mal organisés
class TestTout { /* ... */ }
```

2. **Gestion des Ressources**
```java
// ✅ Bon : Utilisation des extensions
@ExtendWith(TemporaryFolderExtension.class)
class FichierTest {
    @Test
    void testAvecFichierTemp(@TempDir Path tempDir) { /* ... */ }
}
```

3. **Configuration Claire**
```java
// ✅ Bon : Configuration explicite
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class TestsOrdonnes { /* ... */ }
```

## Points Clés à Retenir 🎯

1. Le Test Runner est responsable de l'exécution des tests
2. Il gère la découverte, l'ordre et le filtrage des tests
3. La configuration peut être faite via annotations ou fichiers de configuration
4. Les rapports permettent d'analyser les résultats
5. L'exécution parallèle peut améliorer les performances
6. Les extensions permettent de personnaliser le comportement 