# Tests Unitaires 🎯

## Définition et Objectifs

Les tests unitaires sont la base de la pyramide des tests. Ils vérifient le bon fonctionnement des plus petites unités de code testables de manière isolée.

### Objectifs
- Vérifier le comportement d'une unité de code isolée
- Détecter les régressions rapidement
- Documenter le comportement attendu du code
- Faciliter la refactorisation

## Principes FIRST

### Fast (Rapide) ⚡
```java
@Test
void devraitEtreRapide() {
    // ✅ Bon : Test rapide sans dépendances externes
    Calculator calc = new Calculator();
    assertEquals(4, calc.add(2, 2));

    // ❌ Mauvais : Test lent avec dépendance externe
    DatabaseConnection db = new DatabaseConnection();
    db.connect(); // Lent !
    assertEquals(4, db.queryResult("2 + 2"));
}
```

### Isolated/Independent (Isolé/Indépendant) 🔒
```java
class UserServiceTest {
    private UserService service;
    private UserRepository mockRepo;

    @BeforeEach
    void setUp() {
        // ✅ Bon : Utilisation de mocks pour l'isolation
        mockRepo = mock(UserRepository.class);
        service = new UserService(mockRepo);
    }

    @Test
    void devraitCreerUtilisateur() {
        // Test isolé avec des données spécifiques
        User user = new User("test@example.com");
        when(mockRepo.save(any(User.class))).thenReturn(user);
        
        User result = service.createUser("test@example.com");
        assertEquals("test@example.com", result.getEmail());
    }
}
```

### Repeatable (Répétable) 🔄
```java
@Test
void devraitEtreRepetable() {
    // ✅ Bon : Résultat constant
    DateProvider mockDate = mock(DateProvider.class);
    when(mockDate.now()).thenReturn(
        LocalDateTime.of(2024, 1, 1, 12, 0)
    );

    // ❌ Mauvais : Résultat variable
    LocalDateTime now = LocalDateTime.now(); // Dépend du moment d'exécution
}
```

### Self-validating (Auto-validant) ✅
```java
@Test
void devraitEtreAutoValidant() {
    // ✅ Bon : Validation automatique
    Calculator calc = new Calculator();
    assertEquals(4, calc.add(2, 2), "2 + 2 devrait être égal à 4");

    // ❌ Mauvais : Nécessite une validation manuelle
    System.out.println("Résultat : " + calc.add(2, 2));
}
```

### Timely (Au bon moment) ⏰
```java
// ✅ Bon : Tests écrits avec le code
public class NewFeature {
    @Test
    void devraitFonctionnerCommePrevu() {
        // Test écrit pendant le développement
    }
}

// ❌ Mauvais : Tests écrits après coup
public class LegacyCode {
    // Code écrit il y a longtemps
    // Tests ajoutés bien plus tard
}
```

## Cas d'Usage

### 1. Test d'une Classe Métier
```java
public class PrixService {
    public double calculerPrixTTC(double prixHT, double tauxTVA) {
        if (prixHT < 0) {
            throw new IllegalArgumentException("Prix HT ne peut pas être négatif");
        }
        return prixHT * (1 + tauxTVA);
    }
}

class PrixServiceTest {
    private PrixService service;

    @BeforeEach
    void setUp() {
        service = new PrixService();
    }

    @Test
    void devraitCalculerPrixTTC() {
        assertEquals(120.0, service.calculerPrixTTC(100.0, 0.20));
    }

    @Test
    void devraitRejeterPrixNegatif() {
        assertThrows(IllegalArgumentException.class,
            () -> service.calculerPrixTTC(-100.0, 0.20));
    }
}
```

### 2. Test d'une Méthode de Validation
```java
public class ValidateurEmail {
    public boolean estValide(String email) {
        if (email == null || email.trim().isEmpty()) {
            return false;
        }
        return email.matches("^[A-Za-z0-9+_.-]+@(.+)$");
    }
}

class ValidateurEmailTest {
    private ValidateurEmail validateur;

    @BeforeEach
    void setUp() {
        validateur = new ValidateurEmail();
    }

    @Test
    void devraitValiderEmailCorrect() {
        assertTrue(validateur.estValide("test@example.com"));
    }

    @Test
    void devraitRejeterEmailInvalide() {
        assertAll(
            () -> assertFalse(validateur.estValide(null)),
            () -> assertFalse(validateur.estValide("")),
            () -> assertFalse(validateur.estValide("test")),
            () -> assertFalse(validateur.estValide("test@"))
        );
    }
}
```

### 3. Test avec Mocks
```java
public class CommandeService {
    private final StockService stockService;
    private final PaiementService paiementService;

    public CommandeService(StockService stockService, PaiementService paiementService) {
        this.stockService = stockService;
        this.paiementService = paiementService;
    }

    public boolean passerCommande(String produitId, int quantite) {
        if (!stockService.verifierStock(produitId, quantite)) {
            return false;
        }
        return paiementService.effectuerPaiement();
    }
}

class CommandeServiceTest {
    @Mock private StockService stockService;
    @Mock private PaiementService paiementService;
    private CommandeService service;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
        service = new CommandeService(stockService, paiementService);
    }

    @Test
    void devraitPasserCommandeAvecSucces() {
        // Arrange
        when(stockService.verifierStock("PROD1", 1)).thenReturn(true);
        when(paiementService.effectuerPaiement()).thenReturn(true);

        // Act
        boolean resultat = service.passerCommande("PROD1", 1);

        // Assert
        assertTrue(resultat);
        verify(stockService).verifierStock("PROD1", 1);
        verify(paiementService).effectuerPaiement();
    }

    @Test
    void devraitEchouerSiStockInsuffisant() {
        when(stockService.verifierStock("PROD1", 1)).thenReturn(false);

        assertFalse(service.passerCommande("PROD1", 1));
        verify(paiementService, never()).effectuerPaiement();
    }
}
```

## Bonnes Pratiques 👍

1. **Nommage Explicite**
```java
// ✅ Bon : nom descriptif
@Test
void devraitRetournerFalseQuandEmailInvalide() { }

// ❌ Mauvais : nom peu clair
@Test
void testEmail() { }
```

2. **Une Seule Assertion par Concept**
```java
// ✅ Bon : assertions groupées logiquement
@Test
void devraitValiderUtilisateur() {
    assertAll("Validation utilisateur",
        () -> assertTrue(validator.validateEmail("test@example.com")),
        () -> assertTrue(validator.validateAge(18))
    );
}
```

3. **Isolation des Tests**
```java
// ✅ Bon : chaque test est indépendant
@Test
void devraitAjouterArticle() {
    Panier panier = new Panier();
    panier.ajouter(new Article("A1"));
    assertEquals(1, panier.getNombreArticles());
}
```

## Points Clés à Retenir 🎯

1. Les tests unitaires sont la base de la qualité du code
2. Ils doivent suivre les principes FIRST
3. Chaque test doit être indépendant et répétable
4. Utilisez des mocks pour isoler les dépendances
5. Écrivez les tests en même temps que le code
6. Maintenez vos tests aussi soigneusement que votre code de production 