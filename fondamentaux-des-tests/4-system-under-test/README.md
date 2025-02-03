# System Under Test (SUT) 🎯

## Qu'est-ce que le System Under Test ?

Le System Under Test (SUT) est le composant ou système que l'on teste. C'est l'objet principal de nos tests, isolé de ses dépendances. Imaginez que vous testez une voiture : le SUT pourrait être le moteur, tandis que la batterie et le réservoir seraient simulés.

## Identification du SUT

### 1. Exemple Simple
```java
// Ceci est notre SUT
public class CalculateurPrix {
    private final TauxTVA tauxTVA;  // Dépendance
    
    public CalculateurPrix(TauxTVA tauxTVA) {
        this.tauxTVA = tauxTVA;
    }
    
    public double calculerPrixTTC(double prixHT) {
        return prixHT * (1 + tauxTVA.getTaux());
    }
}

// Test du SUT
class CalculateurPrixTest {
    @Test
    void testCalculPrixTTC() {
        // Arrange
        TauxTVA mockTVA = mock(TauxTVA.class);
        when(mockTVA.getTaux()).thenReturn(0.20);
        CalculateurPrix sut = new CalculateurPrix(mockTVA);  // SUT clairement identifié
        
        // Act
        double prixTTC = sut.calculerPrixTTC(100.0);
        
        // Assert
        assertEquals(120.0, prixTTC);
    }
}
```

## Isolation du SUT

### 1. Utilisation des Mocks pour Isoler
```java
public class ServiceCommande {  // Notre SUT
    private final RepositoryProduit repositoryProduit;
    private final ServicePaiement servicePaiement;
    private final NotificationClient notification;
    
    public ResultatCommande passerCommande(Commande commande) {
        // Logique métier
        if (!repositoryProduit.verifierStock(commande)) {
            return ResultatCommande.STOCK_INSUFFISANT;
        }
        
        if (!servicePaiement.processPayment(commande.getMontant())) {
            return ResultatCommande.PAIEMENT_REFUSE;
        }
        
        notification.envoyerConfirmation(commande);
        return ResultatCommande.SUCCES;
    }
}

// Test avec isolation
class ServiceCommandeTest {
    @Mock private RepositoryProduit repositoryProduit;
    @Mock private ServicePaiement servicePaiement;
    @Mock private NotificationClient notification;
    
    private ServiceCommande sut;  // Notre SUT
    
    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
        sut = new ServiceCommande(repositoryProduit, servicePaiement, notification);
    }
    
    @Test
    void commandeReussie() {
        // Arrange
        Commande commande = new Commande("Produit", 100.0);
        when(repositoryProduit.verifierStock(commande)).thenReturn(true);
        when(servicePaiement.processPayment(100.0)).thenReturn(true);
        
        // Act
        ResultatCommande resultat = sut.passerCommande(commande);
        
        // Assert
        assertEquals(ResultatCommande.SUCCES, resultat);
        verify(notification).envoyerConfirmation(commande);
    }
}
```

## Exemple Complet : Gestionnaire de Panier

```java
// Notre SUT
public class GestionnairePanier {
    private final ServiceProduit serviceProduit;
    private final CalculateurRemise calculateur;
    private final List<Article> articles = new ArrayList<>();
    
    public GestionnairePanier(ServiceProduit serviceProduit, CalculateurRemise calculateur) {
        this.serviceProduit = serviceProduit;
        this.calculateur = calculateur;
    }
    
    public void ajouterArticle(String reference, int quantite) {
        Produit produit = serviceProduit.recupererProduit(reference);
        if (produit == null) {
            throw new ProduitInexistantException(reference);
        }
        
        if (!serviceProduit.verifierStock(reference, quantite)) {
            throw new StockInsuffisantException(reference);
        }
        
        articles.add(new Article(produit, quantite));
    }
    
    public double calculerTotal() {
        double sousTotal = articles.stream()
            .mapToDouble(a -> a.getProduit().getPrix() * a.getQuantite())
            .sum();
            
        return calculateur.appliquerRemise(sousTotal);
    }
}

// Tests complets du SUT
@ExtendWith(MockitoExtension.class)
class GestionnairePanierTest {
    @Mock private ServiceProduit serviceProduit;
    @Mock private CalculateurRemise calculateur;
    
    private GestionnairePanier sut;
    
    @BeforeEach
    void setUp() {
        sut = new GestionnairePanier(serviceProduit, calculateur);
    }
    
    @Nested
    @DisplayName("Tests d'ajout d'articles")
    class AjoutArticles {
        @Test
        void ajoutArticleValide() {
            // Arrange
            String reference = "REF001";
            Produit produit = new Produit(reference, "Livre", 29.99);
            when(serviceProduit.recupererProduit(reference)).thenReturn(produit);
            when(serviceProduit.verifierStock(reference, 1)).thenReturn(true);
            
            // Act
            sut.ajouterArticle(reference, 1);
            
            // Assert
            assertEquals(29.99, sut.calculerTotal());
        }
        
        @Test
        void ajoutArticleInexistant() {
            // Arrange
            String reference = "REF999";
            when(serviceProduit.recupererProduit(reference)).thenReturn(null);
            
            // Act & Assert
            assertThrows(ProduitInexistantException.class,
                () -> sut.ajouterArticle(reference, 1));
        }
    }
    
    @Nested
    @DisplayName("Tests de calcul du total")
    class CalculTotal {
        @Test
        void calculTotalAvecRemise() {
            // Arrange
            String ref1 = "REF001";
            String ref2 = "REF002";
            Produit prod1 = new Produit(ref1, "Livre", 100.0);
            Produit prod2 = new Produit(ref2, "Cahier", 50.0);
            
            when(serviceProduit.recupererProduit(ref1)).thenReturn(prod1);
            when(serviceProduit.recupererProduit(ref2)).thenReturn(prod2);
            when(serviceProduit.verifierStock(any(), anyInt())).thenReturn(true);
            when(calculateur.appliquerRemise(150.0)).thenReturn(135.0);
            
            // Act
            sut.ajouterArticle(ref1, 1);
            sut.ajouterArticle(ref2, 1);
            double total = sut.calculerTotal();
            
            // Assert
            assertEquals(135.0, total);
            verify(calculateur).appliquerRemise(150.0);
        }
    }
}
```

## Bonnes Pratiques 👍

1. **Nommage Explicite du SUT**
```java
// ✅ Bon : SUT clairement identifié
private ServiceCommande sut;

// ❌ Mauvais : nom générique
private ServiceCommande service;
```

2. **Isolation Claire des Dépendances**
```java
// ✅ Bon : dépendances mockées
@Mock private ServicePaiement servicePaiement;
@Mock private NotificationClient notification;

// ❌ Mauvais : utilisation de vraies instances
private ServicePaiement servicePaiement = new ServicePaiement();
```

3. **Tests Centrés sur le SUT**
```java
@Test
void testComportementSUT() {
    // ✅ Bon : test focalisé sur le SUT
    ResultatCommande resultat = sut.passerCommande(commande);
    assertEquals(ResultatCommande.SUCCES, resultat);
    
    // ❌ Mauvais : test des dépendances
    verify(servicePaiement).processPayment(any());
    verify(notification).envoyerConfirmation(any());
}
```

## Points Clés à Retenir 🎯

1. Le SUT doit être clairement identifié dans les tests
2. Isolez le SUT de ses dépendances
3. Concentrez-vous sur le comportement du SUT, pas ses dépendances
4. Utilisez des mocks pour simuler les dépendances
5. Testez les interactions du SUT avec ses dépendances
6. Gardez le SUT aussi simple que possible 