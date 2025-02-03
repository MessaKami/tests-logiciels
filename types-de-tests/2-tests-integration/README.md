# Tests d'Intégration 🔄

## Définition et Objectifs

Les tests d'intégration vérifient que différents composants ou services d'une application fonctionnent correctement ensemble. Ils se situent au milieu de la pyramide des tests.

### Objectifs
- Vérifier les interactions entre composants
- Tester les flux de données complets
- Valider l'intégration avec les systèmes externes
- Détecter les problèmes d'interface

## Différence avec les Tests Unitaires

### 1. Portée
```java
// Test Unitaire : teste une seule classe
class CalculatriceTest {
    @Test
    void testAddition() {
        Calculatrice calc = new Calculatrice();
        assertEquals(4, calc.additionner(2, 2));
    }
}

// Test d'Intégration : teste plusieurs composants
class CommandeIntegrationTest {
    @Autowired private CommandeService commandeService;
    @Autowired private StockService stockService;
    @Autowired private PaiementService paiementService;

    @Test
    void testProcessusCommande() {
        // Test du flux complet
        Commande commande = commandeService.creerCommande("PROD1", 2);
        assertTrue(stockService.verifierEtReserver(commande));
        assertTrue(paiementService.processPayment(commande));
        assertEquals(StatutCommande.COMPLETE, commande.getStatut());
    }
}
```

### 2. Configuration
```java
// Configuration pour Tests d'Intégration
@SpringBootTest
class IntegrationTestConfig {
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .addScript("schema.sql")
            .addScript("test-data.sql")
            .build();
    }
}
```

### 3. Performance
```java
// Test d'Intégration avec Timeout
@Test
@Timeout(value = 5, unit = TimeUnit.SECONDS)
void testIntegrationPerformance() {
    // Test plus lent car implique plusieurs composants
    commandeService.traiterCommande(commande);
}
```

## Cas d'Usage

### 1. Test d'Intégration avec Base de Données
```java
@SpringBootTest
class UtilisateurRepositoryIntegrationTest {
    @Autowired
    private UtilisateurRepository repository;

    @Test
    void devraitSauvegarderEtRecupererUtilisateur() {
        // Arrange
        Utilisateur utilisateur = new Utilisateur("test@example.com");
        
        // Act
        Utilisateur sauvegarde = repository.save(utilisateur);
        Utilisateur recupere = repository.findById(sauvegarde.getId()).orElse(null);
        
        // Assert
        assertNotNull(recupere);
        assertEquals("test@example.com", recupere.getEmail());
    }
}
```

### 2. Test d'Intégration API REST
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class CommandeControllerIntegrationTest {
    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void devraitCreerCommande() {
        // Arrange
        CommandeDTO commande = new CommandeDTO("PROD1", 2);
        
        // Act
        ResponseEntity<CommandeResponse> response = restTemplate.postForEntity(
            "/api/commandes",
            commande,
            CommandeResponse.class
        );
        
        // Assert
        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody());
        assertNotNull(response.getBody().getId());
    }
}
```

### 3. Test d'Intégration avec File d'Attente
```java
@SpringBootTest
class MessageServiceIntegrationTest {
    @Autowired private MessageProducer producer;
    @Autowired private MessageConsumer consumer;
    @Autowired private MessageRepository repository;

    @Test
    void devraitTraiterMessage() throws InterruptedException {
        // Arrange
        String message = "Test Message";
        
        // Act
        producer.send(message);
        
        // Assert - attend le traitement
        await()
            .atMost(5, TimeUnit.SECONDS)
            .until(() -> repository.findByContenu(message).isPresent());
        
        Message traite = repository.findByContenu(message).get();
        assertEquals(StatutMessage.TRAITE, traite.getStatut());
    }
}
```

## Bonnes Pratiques 👍

1. **Isolation de l'Environnement**
```java
@SpringBootTest
class IntegrationTest {
    @BeforeEach
    void setUp() {
        // ✅ Bon : Nettoie l'environnement avant chaque test
        databaseCleaner.clean();
        cacheManager.clearAll();
    }
}
```

2. **Gestion des Données de Test**
```java
// ✅ Bon : Utilisation de données de test dédiées
@Sql({
    "classpath:schema.sql",
    "classpath:test-data.sql"
})
@Test
void testIntegration() {
    // Test avec données connues
}
```

3. **Timeouts et Retry**
```java
// ✅ Bon : Gestion des opérations asynchrones
@Test
void testAsynchrone() {
    await()
        .atMost(10, TimeUnit.SECONDS)
        .pollInterval(1, TimeUnit.SECONDS)
        .until(() -> serviceStatus.isReady());
}
```

## Points Clés à Retenir 🎯

1. Les tests d'intégration vérifient les interactions entre composants
2. Ils sont plus lents mais plus complets que les tests unitaires
3. Nécessitent une configuration d'environnement spécifique
4. Utilisent souvent des bases de données de test
5. Doivent gérer l'asynchronisme et les timeouts
6. Sont essentiels pour valider les interfaces entre services 