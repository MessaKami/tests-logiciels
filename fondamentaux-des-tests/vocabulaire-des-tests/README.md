# Vocabulaire des Tests avec JUnit 5

## Test Fixture

Un "Test Fixture" représente l'environnement et les conditions préalables nécessaires pour exécuter un test. En Java avec JUnit 5, nous utilisons les annotations `@BeforeEach` et `@BeforeAll`.

### Exemple Concret
```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculatriceTest {
    private Calculatrice calculatrice;
    private List<Integer> valeursTest;

    @BeforeEach  // Test Fixture
    void setUp() {
        calculatrice = new Calculatrice();
        valeursTest = Arrays.asList(1, 2, 3);
    }

    @Test
    void testAddition() {
        assertEquals(5, calculatrice.additionner(2, 3));
    }
}
```

## Test Suite

Une "Test Suite" est un ensemble de tests regroupés logiquement. En JUnit 5, nous utilisons `@Nested` pour créer des suites de tests imbriquées.

### Exemple Concret
```java
@DisplayName("Tests de la Calculatrice")
class CalculatriceTest {
    
    @Nested
    @DisplayName("Tests des opérations de base")
    class OperationsDeBase {
        
        @Test
        void testAddition() {
            // Test d'addition
        }
        
        @Test
        void testSoustraction() {
            // Test de soustraction
        }
    }
    
    @Nested
    @DisplayName("Tests des opérations avancées")
    class OperationsAvancees {
        
        @Test
        void testMultiplication() {
            // Test de multiplication
        }
        
        @Test
        void testDivision() {
            // Test de division
        }
    }
}
```

## Test Case

Un "Test Case" est un scénario de test individuel. En JUnit 5, chaque méthode annotée avec `@Test` représente un cas de test.

### Exemple Concret
```java
@Test
@DisplayName("Création d'un utilisateur avec des données valides")
void testCreationUtilisateur() {
    // Arrange
    UtilisateurDTO donnees = new UtilisateurDTO("Dupont", "dupont@example.com");
    
    // Act
    Utilisateur utilisateur = service.creerUtilisateur(donnees);
    
    // Assert
    assertAll(
        () -> assertEquals("Dupont", utilisateur.getNom()),
        () -> assertEquals("dupont@example.com", utilisateur.getEmail())
    );
}
```

## System Under Test (SUT)

Le SUT est le composant que nous testons. Voici un exemple de classe à tester :

### Exemple Concret
```java
public class Calculatrice {  // Ceci est notre SUT
    private List<String> historique = new ArrayList<>();
    
    public double additionner(double a, double b) {
        double resultat = a + b;
        historique.add(String.format("%.2f + %.2f = %.2f", a, b, resultat));
        return resultat;
    }
    
    public List<String> getHistorique() {
        return new ArrayList<>(historique);
    }
}
```

## Assertions

JUnit 5 fournit une riche API d'assertions dans la classe `Assertions`.

### Exemple Concret
```java
@Test
void testDifferentesAssertions() {
    // Assertions basiques
    assertEquals(4, calculatrice.additionner(2, 2));
    assertTrue(email.estValide("test@example.com"));
    assertFalse(email.estValide("invalid-email"));
    
    // Assertions multiples
    assertAll("Validation utilisateur",
        () -> assertNotNull(utilisateur),
        () -> assertEquals("Dupont", utilisateur.getNom()),
        () -> assertTrue(utilisateur.estActif())
    );
    
    // Assertion d'exception
    assertThrows(IllegalArgumentException.class, 
        () -> calculatrice.diviser(10, 0));
}
```

## Mocks, Stubs, Spies avec Mockito

Mockito s'intègre parfaitement avec JUnit 5 pour la création de mocks.

### Exemple Concret
```java
@ExtendWith(MockitoExtension.class)
class ServiceUtilisateurTest {
    
    @Mock
    private RepositoryUtilisateur repository;
    
    @InjectMocks
    private ServiceUtilisateur service;
    
    @Test
    void testRecuperationUtilisateur() {
        // Arrangement du mock
        Utilisateur utilisateur = new Utilisateur(1L, "Dupont");
        when(repository.findById(1L)).thenReturn(Optional.of(utilisateur));
        
        // Action
        Utilisateur resultat = service.getUtilisateur(1L);
        
        // Assertions
        assertNotNull(resultat);
        assertEquals("Dupont", resultat.getNom());
        verify(repository).findById(1L);
    }
    
    @Test
    void testUtilisateurInexistant() {
        // Stub - retour de données fixes
        when(repository.findById(999L)).thenReturn(Optional.empty());
        
        // Action & Assert
        assertThrows(UtilisateurNotFoundException.class,
            () -> service.getUtilisateur(999L));
    }
}
```

## Points Clés à Retenir

- Les fixtures (`@BeforeEach`, `@BeforeAll`) préparent l'environnement de test
- Les suites de tests (`@Nested`) organisent les tests logiquement
- Les cas de test (`@Test`) vérifient des comportements spécifiques
- Les assertions vérifient les résultats attendus
- Mockito permet de simuler des dépendances pour des tests isolés
- JUnit 5 offre des fonctionnalités avancées comme les tests paramétrés et les extensions 