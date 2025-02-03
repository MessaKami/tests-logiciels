# Tests End-to-End (E2E) 🔄

## Définition et Objectifs

Les tests End-to-End simulent le comportement d'un utilisateur réel en testant l'application de bout en bout. Ils se situent au sommet de la pyramide des tests.

### Objectifs
- Valider les parcours utilisateurs complets
- Tester l'application dans un environnement proche de la production
- Vérifier l'intégration de tous les composants
- Détecter les problèmes de flux utilisateur

## La Pyramide des Tests

### Structure
```
      /\      E2E Tests (10%)
     /  \     
    /E2E \    
   /------\   
  / Int.  \   Integration Tests (20%)
 /----------\  
/ Unitaires  \ Unit Tests (70%)
--------------
```

### Caractéristiques par Niveau
```java
// Tests Unitaires : Rapides et nombreux
@Test
void testUnitaire() {
    assertEquals(4, calculator.add(2, 2));
}

// Tests d'Intégration : Interaction entre composants
@Test
void testIntegration() {
    Order order = orderService.createOrder(items);
    assertTrue(paymentService.processPayment(order));
}

// Tests E2E : Parcours utilisateur complet
@Test
void testE2E() {
    // Simulation d'un parcours utilisateur complet
    loginPage.login("user", "password");
    catalogPage.selectProduct("laptop");
    cartPage.checkout();
    paymentPage.enterCardDetails("4111...");
    confirmationPage.verifyOrderSuccess();
}
```

## Cas d'Usage

### 1. Test E2E avec Selenium WebDriver
```java
@ExtendWith(SeleniumExtension.class)
class CommandeE2ETest {
    private WebDriver driver;
    
    @Test
    void devraitPasserCommandeComplete() {
        // 1. Connexion
        driver.get("https://example.com/login");
        driver.findElement(By.id("email")).sendKeys("user@example.com");
        driver.findElement(By.id("password")).sendKeys("password");
        driver.findElement(By.id("login-button")).click();
        
        // 2. Sélection produit
        driver.get("https://example.com/products");
        driver.findElement(By.cssSelector(".product-card")).click();
        driver.findElement(By.id("add-to-cart")).click();
        
        // 3. Panier
        driver.get("https://example.com/cart");
        driver.findElement(By.id("checkout-button")).click();
        
        // 4. Paiement
        driver.findElement(By.id("card-number")).sendKeys("4111111111111111");
        driver.findElement(By.id("expiry")).sendKeys("12/25");
        driver.findElement(By.id("cvv")).sendKeys("123");
        driver.findElement(By.id("pay-button")).click();
        
        // 5. Vérification
        WebElement confirmation = new WebDriverWait(driver, Duration.ofSeconds(10))
            .until(ExpectedConditions.presenceOfElementLocated(
                By.className("order-confirmation")
            ));
        
        assertTrue(confirmation.isDisplayed());
        assertEquals("Commande confirmée", confirmation.getText());
    }
}
```

### 2. Test E2E avec REST Assured
```java
@Test
void devraitTesterAPIComplete() {
    // 1. Création utilisateur
    String token = given()
        .contentType(ContentType.JSON)
        .body(new LoginRequest("user", "password"))
        .when()
        .post("/api/login")
        .then()
        .statusCode(200)
        .extract()
        .path("token");
    
    // 2. Création commande
    Integer orderId = given()
        .header("Authorization", "Bearer " + token)
        .contentType(ContentType.JSON)
        .body(new OrderRequest("PROD1", 1))
        .when()
        .post("/api/orders")
        .then()
        .statusCode(201)
        .extract()
        .path("orderId");
    
    // 3. Vérification statut
    given()
        .header("Authorization", "Bearer " + token)
        .when()
        .get("/api/orders/" + orderId)
        .then()
        .statusCode(200)
        .body("status", equalTo("CONFIRMED"));
}
```

### 3. Test E2E avec Cucumber
```java
@RunWith(Cucumber.class)
@CucumberOptions(features = "src/test/resources/features")
public class CommandeE2ETest {
    // Glue code
}

// commande.feature
Feature: Processus de commande
  Scenario: Commande réussie
    Given un utilisateur connecté
    When il ajoute le produit "laptop" au panier
    And il procède au paiement
    Then la commande est confirmée
    And un email de confirmation est envoyé

// Steps
public class CommandeSteps {
    @Given("un utilisateur connecté")
    public void utilisateurConnecte() {
        // Code de connexion
    }
    
    @When("il ajoute le produit {string} au panier")
    public void ajouterAuPanier(String produit) {
        // Code d'ajout au panier
    }
    
    @Then("la commande est confirmée")
    public void verifierConfirmation() {
        // Code de vérification
    }
}
```

## Bonnes Pratiques 👍

1. **Gestion des Données de Test**
```java
// ✅ Bon : Isolation des données de test
@BeforeEach
void setUp() {
    // Création d'un jeu de données isolé
    testDataBuilder
        .createUser()
        .withProducts()
        .withInventory();
}

@AfterEach
void tearDown() {
    // Nettoyage des données
    testDataCleaner.clean();
}
```

2. **Gestion des Timeouts**
```java
// ✅ Bon : Attente explicite
WebElement element = new WebDriverWait(driver, Duration.ofSeconds(10))
    .until(ExpectedConditions.elementToBeClickable(By.id("button")));

// ❌ Mauvais : Thread.sleep
Thread.sleep(10000); // Éviter
```

3. **Page Objects Pattern**
```java
// ✅ Bon : Utilisation de Page Objects
public class LoginPage {
    private final WebDriver driver;
    
    public void login(String username, String password) {
        driver.findElement(By.id("username")).sendKeys(username);
        driver.findElement(By.id("password")).sendKeys(password);
        driver.findElement(By.id("login-button")).click();
    }
}

// Utilisation
@Test
void testLogin() {
    LoginPage loginPage = new LoginPage(driver);
    loginPage.login("user", "pass");
    assertTrue(new HomePage(driver).isDisplayed());
}
```

## Points Clés à Retenir 🎯

1. Les tests E2E valident le comportement global de l'application
2. Ils sont plus lents et plus fragiles que les autres types de tests
3. Utilisez des outils adaptés (Selenium, Cypress, Playwright)
4. Gérez correctement les timeouts et les attentes
5. Isolez les données de test
6. Maintenez un nombre limité de tests E2E critiques
7. Automatisez les scénarios utilisateurs les plus importants
8. Utilisez des patterns comme Page Objects pour la maintenabilité 