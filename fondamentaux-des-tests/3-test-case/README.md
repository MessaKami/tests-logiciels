# Test Case 📝

## Qu'est-ce qu'un Test Case ?

Un Test Case est un scénario de test spécifique qui vérifie un comportement particulier du système. C'est comme une recette de cuisine qui décrit précisément les étapes à suivre et le résultat attendu.

## Structure d'un Test Case

### 1. Le Pattern AAA (Arrange-Act-Assert)

```java
@Test
void testCalculRemise() {
    // Arrange (Préparer)
    Panier panier = new Panier();
    panier.ajouterArticle(new Article("Livre", 100.0));
    
    // Act (Agir)
    double remise = panier.calculerRemise();
    
    // Assert (Vérifier)
    assertEquals(10.0, remise, "La remise devrait être de 10%");
}
```

### 2. Tests Paramétrés

```java
@ParameterizedTest
@CsvSource({
    "100.0, 10.0",
    "200.0, 20.0",
    "50.0, 5.0"
})
void testCalculRemiseParametre(double montant, double remiseAttendue) {
    Panier panier = new Panier();
    panier.ajouterArticle(new Article("Produit", montant));
    
    double remise = panier.calculerRemise();
    
    assertEquals(remiseAttendue, remise);
}
```

## Exemples de Test Cases

### 1. Test Simple
```java
@Test
@DisplayName("L'addition de deux nombres positifs")
void testAdditionPositifs() {
    Calculatrice calc = new Calculatrice();
    int resultat = calc.additionner(2, 3);
    assertEquals(5, resultat);
}
```

### 2. Test avec Exception Attendue
```java
@Test
@DisplayName("Division par zéro doit lever une exception")
void testDivisionParZero() {
    Calculatrice calc = new Calculatrice();
    
    Exception exception = assertThrows(
        ArithmeticException.class,
        () -> calc.diviser(10, 0)
    );
    
    assertEquals("Division par zéro impossible", exception.getMessage());
}
```

### 3. Test avec Timeout
```java
@Test
@Timeout(value = 100, unit = TimeUnit.MILLISECONDS)
void testPerformance() {
    Service service = new Service();
    List<Donnee> resultat = service.traiterDonnees();
    assertNotNull(resultat);
}
```

### 4. Test avec Conditions
```java
@Test
void testValidationAge() {
    Utilisateur utilisateur = new Utilisateur("Jean", 17);
    
    assertAll("Validation de l'âge",
        () -> assertTrue(utilisateur.estMineur()),
        () -> assertFalse(utilisateur.peutAcheterAlcool()),
        () -> assertThrows(AgeInvalidException.class,
            () -> utilisateur.creerCompteBancaire())
    );
}
```

## Exemple Complet : Système de Réservation

```java
class SystemeReservationTest {
    private SystemeReservation systeme;
    private Hotel hotel;
    private Client client;

    @BeforeEach
    void setUp() {
        hotel = new Hotel("Hôtel Test");
        systeme = new SystemeReservation(hotel);
        client = new Client("Jean Dupont", "jean@email.com");
    }

    @Test
    @DisplayName("Réservation réussie d'une chambre disponible")
    void testReservationReussie() {
        // Arrange
        LocalDate dateArrivee = LocalDate.now().plusDays(1);
        LocalDate dateDepart = LocalDate.now().plusDays(3);
        Chambre chambre = hotel.ajouterChambre(new Chambre(101, "Double", 100.0));

        // Act
        Reservation reservation = systeme.reserver(client, chambre, dateArrivee, dateDepart);

        // Assert
        assertAll("Vérification de la réservation",
            () -> assertNotNull(reservation, "La réservation ne devrait pas être null"),
            () -> assertEquals(client, reservation.getClient()),
            () -> assertEquals(chambre, reservation.getChambre()),
            () -> assertEquals(dateArrivee, reservation.getDateArrivee()),
            () -> assertEquals(dateDepart, reservation.getDateDepart()),
            () -> assertEquals(200.0, reservation.getPrixTotal()),
            () -> assertTrue(chambre.estReservee(dateArrivee))
        );
    }

    @Test
    @DisplayName("Réservation impossible sur une chambre déjà réservée")
    void testReservationChambreDejaReservee() {
        // Arrange
        LocalDate dateArrivee = LocalDate.now().plusDays(1);
        LocalDate dateDepart = LocalDate.now().plusDays(3);
        Chambre chambre = hotel.ajouterChambre(new Chambre(101, "Double", 100.0));
        systeme.reserver(client, chambre, dateArrivee, dateDepart);

        // Act & Assert
        Client autreClient = new Client("Marie Martin", "marie@email.com");
        ReservationException exception = assertThrows(
            ReservationException.class,
            () -> systeme.reserver(autreClient, chambre, dateArrivee, dateDepart)
        );

        assertEquals("Chambre non disponible pour ces dates", exception.getMessage());
    }

    @Test
    @DisplayName("Test des dates de réservation invalides")
    void testDatesReservationInvalides() {
        // Arrange
        Chambre chambre = hotel.ajouterChambre(new Chambre(101, "Double", 100.0));
        LocalDate dateArrivee = LocalDate.now().minusDays(1);
        LocalDate dateDepart = LocalDate.now().plusDays(3);

        // Act & Assert
        assertThrows(
            DateInvalideException.class,
            () -> systeme.reserver(client, chambre, dateArrivee, dateDepart),
            "Une réservation dans le passé devrait être impossible"
        );
    }
}
```

## Bonnes Pratiques 👍

1. **Nommage Explicite**
```java
// ✅ Bon : nom descriptif
@Test
void devraitLeverExceptionQuandEmailInvalide() {
    // Test ici
}

// ❌ Mauvais : nom peu clair
@Test
void testEmail() {
    // Test ici
}
```

2. **Une Seule Assertion par Concept**
```java
@Test
void testCreationUtilisateur() {
    Utilisateur user = service.creerUtilisateur("test@email.com");
    
    assertAll("Validation création utilisateur",
        () -> assertNotNull(user.getId()),
        () -> assertEquals("test@email.com", user.getEmail()),
        () -> assertTrue(user.estActif())
    );
}
```

3. **Tests Indépendants**
```java
@Test
void testIndependant() {
    // ✅ Bon : données créées dans le test
    List<String> liste = new ArrayList<>();
    liste.add("test");
    assertEquals(1, liste.size());
}

// ❌ Mauvais : dépendance à l'état global
private static List<String> listeGlobale;
@Test
void testDependant() {
    listeGlobale.add("test");
    // Le test dépend de l'état de listeGlobale
}
```

## Points Clés à Retenir 🎯

1. Suivez le pattern AAA (Arrange-Act-Assert)
2. Un test case doit tester une seule chose
3. Utilisez des noms descriptifs
4. Gardez les tests indépendants
5. Incluez des cas positifs et négatifs
6. Testez les cas limites 