# Test Fixture 🔧

## Qu'est-ce qu'un Test Fixture ?

Un Test Fixture représente l'ensemble des préconditions et de l'environnement nécessaires pour exécuter un test. C'est comme préparer un laboratoire avant une expérience scientifique.

## Annotations Importantes

### @BeforeEach
Exécuté avant chaque test :
```java
@BeforeEach
void setUp() {
    utilisateur = new Utilisateur("test@email.com");
    panier = new Panier(utilisateur);
    produit = new Produit("Livre", 29.99);
}
```

### @BeforeAll
Exécuté une seule fois avant tous les tests :
```java
@BeforeAll
static void initialiserBaseDeDonnees() {
    database = new Database();
    database.connect("jdbc:mysql://localhost:3306/test");
}
```

### @AfterEach
Nettoyage après chaque test :
```java
@AfterEach
void tearDown() {
    panier.vider();
    utilisateur.deconnecter();
}
```

### @AfterAll
Nettoyage final après tous les tests :
```java
@AfterAll
static void fermerConnexion() {
    database.disconnect();
}
```

## Exemple Complet

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class GestionPanierTest {
    private Utilisateur utilisateur;
    private Panier panier;
    private BaseDeDonnees bdd;
    private List<Produit> produitsTest;

    @BeforeAll
    void initialiserEnvironnement() {
        bdd = new BaseDeDonnees();
        bdd.connect();
        produitsTest = Arrays.asList(
            new Produit("Livre", 29.99),
            new Produit("Cahier", 4.99),
            new Produit("Stylo", 1.99)
        );
        bdd.ajouterProduits(produitsTest);
    }

    @BeforeEach
    void preparerTest() {
        utilisateur = new Utilisateur("test@email.com");
        panier = new Panier(utilisateur);
        // Réinitialiser l'état du panier avant chaque test
        panier.vider();
    }

    @Test
    void testAjoutProduit() {
        // Le test utilise l'environnement préparé
        panier.ajouter(produitsTest.get(0));
        assertEquals(1, panier.getNombreArticles());
    }

    @AfterEach
    void nettoyerTest() {
        panier.vider();
        utilisateur.deconnecter();
    }

    @AfterAll
    void nettoyerEnvironnement() {
        bdd.supprimerProduits(produitsTest);
        bdd.disconnect();
    }
}
```

## Bonnes Pratiques 👍

1. **Isolation** : Chaque test doit partir d'un état connu et propre
```java
@BeforeEach
void setUp() {
    // Toujours partir d'un panier vide
    panier = new Panier();
    // Réinitialiser les compteurs
    Panier.reinitialiserCompteurs();
}
```

2. **Données de Test Explicites** : Préférer des données claires et significatives
```java
@BeforeEach
void setUp() {
    produitTest = new Produit(
        "Livre Java pour les Débutants",
        29.99,
        "ISBN-123-456"
    );
}
```

3. **Gestion des Ressources** : Toujours nettoyer les ressources
```java
@AfterEach
void tearDown() {
    if (connexion != null) {
        connexion.close();
    }
    fichierTemporaire.delete();
}
```

4. **Fixtures Minimales** : Ne préparer que ce qui est nécessaire
```java
class TestProduit {
    // ✅ Bon : fixture minimale
    @BeforeEach
    void setUp() {
        produit = new Produit("Test", 10.0);
    }

    // ❌ Mauvais : trop de préparation inutile
    @BeforeEach
    void setUpTropComplexe() {
        produit = new Produit("Test", 10.0);
        utilisateur = new Utilisateur();  // Non nécessaire
        panier = new Panier();           // Non nécessaire
        bdd = new Database();            // Non nécessaire
    }
}
```

## Points Clés à Retenir 🎯

1. Un test fixture garantit un environnement cohérent pour les tests
2. Utilisez `@BeforeEach` pour l'initialisation par test
3. Utilisez `@BeforeAll` pour l'initialisation unique
4. Nettoyez toujours avec `@AfterEach` et `@AfterAll`
5. Gardez les fixtures aussi simples que possible
6. Assurez-vous que chaque test part d'un état connu 