# Mocks, Stubs et Spies 🕵️

## Introduction

Les Mocks, Stubs et Spies sont des types de "test doubles" qui permettent de simuler le comportement des dépendances dans nos tests. C'est comme utiliser des cascadeurs dans un film : ils remplacent les acteurs pour les scènes dangereuses.

## Configuration Mockito

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <version>5.3.1</version>
    <scope>test</scope>
</dependency>
```

## 1. Mocks 🎭

Un Mock est un objet simulé qui vérifie les interactions comportementales. Il permet de vérifier si certaines méthodes ont été appelées avec les bons paramètres.

### Création et Utilisation Basique
```java
@ExtendWith(MockitoExtension.class)
class ServicePaiementTest {
    @Mock
    private BanqueService banqueService;
    
    @InjectMocks
    private ServicePaiement servicePaiement;
    
    @Test
    void testPaiement() {
        // Arrange
        when(banqueService.verifierSolde("compte123", 100.0))
            .thenReturn(true);
        
        // Act
        boolean resultat = servicePaiement.effectuerPaiement("compte123", 100.0);
        
        // Assert
        assertTrue(resultat);
        verify(banqueService).verifierSolde("compte123", 100.0);
        verify(banqueService).debiter("compte123", 100.0);
    }
}
```

### Comportements Avancés
```java
@Test
void testComportementsAvances() {
    // Réponses multiples
    when(banqueService.verifierSolde(anyString(), anyDouble()))
        .thenReturn(true, false, true);
    
    // Exception
    when(banqueService.debiter("compteInvalide", anyDouble()))
        .thenThrow(new CompteInvalideException());
    
    // Réponse personnalisée
    when(banqueService.calculerFrais(anyDouble()))
        .thenAnswer(invocation -> {
            double montant = invocation.getArgument(0);
            return montant * 0.02;
        });
}
```

## 2. Stubs 📝

Un Stub fournit des réponses prédéfinies aux appels. Il est plus simple qu'un mock car il ne vérifie pas les interactions.

### Exemple de Stub
```java
class ServiceMeteoTest {
    @Test
    void testPrevisionMeteo() {
        // Création d'un stub
        MeteoAPI stubAPI = new MeteoAPI() {
            @Override
            public Temperature getTemperature(String ville) {
                return new Temperature(20.0);
            }
            
            @Override
            public Humidite getHumidite(String ville) {
                return new Humidite(65);
            }
        };
        
        ServiceMeteo service = new ServiceMeteo(stubAPI);
        PrevisionMeteo prevision = service.obtenirPrevision("Paris");
        
        assertEquals(20.0, prevision.getTemperature());
        assertEquals(65, prevision.getHumidite());
    }
}
```

## 3. Spies 🔍

Un Spy est un wrapper autour d'un objet réel qui permet de suivre les interactions tout en gardant le comportement réel.

### Utilisation des Spies
```java
@Test
void testSpy() {
    // Création d'un spy sur une vraie liste
    List<String> listeSpy = spy(new ArrayList<>());
    
    // Utilisation normale
    listeSpy.add("un");
    listeSpy.add("deux");
    
    // Vérification des interactions
    verify(listeSpy, times(2)).add(anyString());
    assertEquals(2, listeSpy.size());
    
    // Stubbing partiel
    doReturn(100).when(listeSpy).size();
    assertEquals(100, listeSpy.size());
}
```

## Exemple Complet : Service de Commande

```java
@ExtendWith(MockitoExtension.class)
class ServiceCommandeTest {
    @Mock
    private RepositoryProduit repositoryProduit;
    
    @Mock
    private ServicePaiement servicePaiement;
    
    @Spy
    private NotificationService notificationService;
    
    @InjectMocks
    private ServiceCommande serviceCommande;
    
    @Test
    @DisplayName("Test complet du processus de commande")
    void testProcessusCommande() {
        // Arrange
        Produit produit = new Produit("1", "Livre", 29.99);
        Client client = new Client("C1", "client@example.com");
        Commande commande = new Commande(client, Arrays.asList(produit));
        
        // Configuration des mocks
        when(repositoryProduit.findById("1"))
            .thenReturn(Optional.of(produit));
        when(servicePaiement.processPayment(anyDouble()))
            .thenReturn(true);
        doNothing().when(notificationService)
            .envoyerConfirmation(any());
        
        // Act
        ResultatCommande resultat = serviceCommande.passerCommande(commande);
        
        // Assert
        assertAll("Vérification du processus de commande",
            () -> assertEquals(ResultatCommande.SUCCES, resultat),
            () -> verify(repositoryProduit).findById("1"),
            () -> verify(servicePaiement).processPayment(29.99),
            () -> verify(notificationService).envoyerConfirmation(any()),
            () -> verify(repositoryProduit, never()).delete(any())
        );
    }
    
    @Test
    @DisplayName("Test d'échec de paiement")
    void testEchecPaiement() {
        // Arrange
        when(servicePaiement.processPayment(anyDouble()))
            .thenReturn(false);
        
        // Act & Assert
        assertAll(
            () -> assertEquals(
                ResultatCommande.ECHEC_PAIEMENT,
                serviceCommande.passerCommande(new Commande())
            ),
            () -> verify(notificationService, never())
                .envoyerConfirmation(any())
        );
    }
}
```

## Bonnes Pratiques 👍

1. **Choix du Bon Type de Double**
```java
// ✅ Bon : Mock pour vérifier les interactions
@Mock
private ServicePaiement servicePaiement;

// ✅ Bon : Stub pour données simples
private PrixService stubPrixService = montant -> montant * 1.2;

// ✅ Bon : Spy pour comportement partiel
@Spy
private Logger logger;
```

2. **Vérifications Précises**
```java
// ✅ Bon : vérification précise
verify(service, times(1)).methode("valeur");

// ❌ Mauvais : vérification trop générale
verify(service).methode(any());
```

3. **Stubbing Clair**
```java
// ✅ Bon : stubbing explicite
when(service.getData(eq("id123")))
    .thenReturn(expectedData);

// ❌ Mauvais : stubbing trop permissif
when(service.getData(any()))
    .thenReturn(expectedData);
```

## Points Clés à Retenir 🎯

1. **Mocks** : Pour vérifier les interactions comportementales
2. **Stubs** : Pour fournir des données de test
3. **Spies** : Pour suivre l'utilisation d'objets réels
4. Utilisez `@Mock` pour les mocks Mockito
5. Utilisez `@InjectMocks` pour l'injection automatique
6. Vérifiez les interactions avec `verify()`
7. Configurez les comportements avec `when()`
8. Évitez de mocker les types de valeur
9. Préférez les stubs pour les données simples
10. Utilisez les spies avec précaution 