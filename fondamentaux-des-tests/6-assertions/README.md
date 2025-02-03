# Assertions ✅

## Qu'est-ce qu'une Assertion ?

Une assertion est une déclaration qui vérifie si une condition est vraie. C'est comme un contrôle qualité dans une usine : on vérifie que le produit correspond aux spécifications attendues.

## Types d'Assertions dans JUnit 5

### 1. Assertions Basiques
```java
@Test
void testAssertionsBasiques() {
    // Égalité
    assertEquals(4, 2 + 2, "2 + 2 devrait être égal à 4");
    assertNotEquals(5, 2 + 2);
    
    // Booléens
    assertTrue(estPair(2));
    assertFalse(estPair(3));
    
    // Nullité
    assertNull(objetVide);
    assertNotNull(objetRempli);
}
```

### 2. Assertions sur les Collections
```java
@Test
void testAssertionsCollections() {
    List<String> fruits = Arrays.asList("pomme", "banane", "orange");
    
    // Taille
    assertEquals(3, fruits.size());
    
    // Contenu
    assertTrue(fruits.contains("pomme"));
    assertFalse(fruits.isEmpty());
    
    // Collections complètes
    assertIterableEquals(
        Arrays.asList("pomme", "banane", "orange"),
        fruits
    );
}
```

### 3. Assertions sur les Exceptions
```java
@Test
void testAssertionsExceptions() {
    // Exception simple
    Exception exception = assertThrows(
        IllegalArgumentException.class,
        () -> diviserParZero()
    );
    assertEquals("Division par zéro impossible", exception.getMessage());
    
    // Vérifier qu'aucune exception n'est levée
    assertDoesNotThrow(() -> calculSimple());
    
    // Exception avec timeout
    assertTimeout(
        Duration.ofMillis(100),
        () -> operationLongue()
    );
}
```

### 4. Assertions Groupées
```java
@Test
void testAssertionsGroupees() {
    Utilisateur utilisateur = new Utilisateur("Jean", "Dupont", 30);
    
    assertAll("Vérification utilisateur",
        () -> assertEquals("Jean", utilisateur.getPrenom()),
        () -> assertEquals("Dupont", utilisateur.getNom()),
        () -> assertTrue(utilisateur.getAge() >= 18),
        () -> assertNotNull(utilisateur.getDateCreation())
    );
}
```

## Assertions Avancées

### 1. Assertions Personnalisées
```java
public class AssertionsPersonnalisees {
    public static void assertValeurPositive(double valeur) {
        assertTrue(valeur > 0, () -> "La valeur " + valeur + " devrait être positive");
    }
    
    public static void assertEmailValide(String email) {
        assertTrue(
            email.matches("^[A-Za-z0-9+_.-]+@(.+)$"),
            () -> "L'email " + email + " n'est pas valide"
        );
    }
    
    @Test
    void testAssertionsPersonnalisees() {
        assertValeurPositive(42.0);
        assertEmailValide("test@example.com");
    }
}
```

### 2. Assertions avec Messages Dynamiques
```java
@Test
void testAssertionsMessagesDynamiques() {
    Produit produit = new Produit("Livre", -10.0);
    
    assertEquals(
        0.0,
        produit.getPrix(),
        () -> String.format(
            "Le prix du produit %s ne peut pas être négatif (valeur : %.2f)",
            produit.getNom(),
            produit.getPrix()
        )
    );
}
```

### 3. Assertions sur les Objets
```java
@Test
void testAssertionsObjets() {
    Adresse adresse1 = new Adresse("123", "Rue Example", "75000", "Paris");
    Adresse adresse2 = new Adresse("123", "Rue Example", "75000", "Paris");
    Adresse adresse3 = null;
    
    // Égalité des objets
    assertEquals(adresse1, adresse2);
    
    // Même instance
    assertSame(adresse1, adresse1);
    assertNotSame(adresse1, adresse2);
    
    // Nullité
    assertNull(adresse3);
}
```

## Exemple Complet : Test d'une Classe Commande

```java
@DisplayName("Tests de la classe Commande")
class CommandeTest {
    private Commande commande;
    private List<Article> articles;
    
    @BeforeEach
    void setUp() {
        articles = Arrays.asList(
            new Article("Livre", 29.99),
            new Article("Cahier", 4.99)
        );
        commande = new Commande("CMD001", articles);
    }
    
    @Test
    @DisplayName("Vérification complète d'une commande")
    void testCommandeComplete() {
        assertAll("Vérification de la commande",
            // Vérification des propriétés basiques
            () -> assertNotNull(commande.getNumero()),
            () -> assertEquals("CMD001", commande.getNumero()),
            () -> assertFalse(commande.getArticles().isEmpty()),
            
            // Vérification du contenu
            () -> assertEquals(2, commande.getArticles().size()),
            () -> assertTrue(commande.getArticles().stream()
                    .anyMatch(a -> a.getNom().equals("Livre"))),
            
            // Vérification des calculs
            () -> assertEquals(34.98, commande.getTotal(), 0.01),
            () -> assertTrue(commande.getTotal() > 0),
            
            // Vérification des dates
            () -> assertNotNull(commande.getDateCreation()),
            () -> assertTrue(commande.getDateCreation()
                    .isBefore(LocalDateTime.now()))
        );
    }
    
    @Test
    @DisplayName("Vérification des exceptions")
    void testExceptionsCommande() {
        assertAll("Vérification des exceptions",
            // Test article null
            () -> assertThrows(
                IllegalArgumentException.class,
                () -> commande.ajouterArticle(null)
            ),
            
            // Test prix négatif
            () -> assertThrows(
                IllegalArgumentException.class,
                () -> commande.ajouterArticle(
                    new Article("Test", -10.0)
                )
            ),
            
            // Test numéro commande invalide
            () -> assertThrows(
                IllegalArgumentException.class,
                () -> new Commande("", articles)
            )
        );
    }
}
```

## Bonnes Pratiques 👍

1. **Messages d'Erreur Descriptifs**
```java
// ✅ Bon : message explicite
assertEquals(
    expected,
    actual,
    "Le calcul de la TVA pour " + montant + "€ est incorrect"
);

// ❌ Mauvais : message peu clair
assertEquals(expected, actual, "Erreur de calcul");
```

2. **Assertions Précises**
```java
// ✅ Bon : assertion spécifique
assertNull(resultat);

// ❌ Mauvais : assertion trop générale
assertTrue(resultat == null);
```

3. **Groupement Logique**
```java
// ✅ Bon : assertions groupées logiquement
assertAll("Validation utilisateur",
    () -> assertNotNull(user.getId()),
    () -> assertNotNull(user.getNom()),
    () -> assertTrue(user.getAge() > 0)
);
```

## Points Clés à Retenir 🎯

1. Utilisez des messages d'erreur descriptifs
2. Groupez les assertions logiquement avec `assertAll`
3. Préférez les assertions spécifiques aux génériques
4. Testez les cas limites et les exceptions
5. Utilisez des assertions personnalisées pour la réutilisabilité
6. Vérifiez toujours les valeurs nulles quand nécessaire 