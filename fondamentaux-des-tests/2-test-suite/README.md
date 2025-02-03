# Test Suite 📚

## Qu'est-ce qu'une Test Suite ?

Une Test Suite est un regroupement logique de tests qui permet d'organiser et d'exécuter ensemble des tests liés. C'est comme un chapitre dans un livre de recettes, où chaque recette est un test individuel.

## Organisation des Test Suites

### 1. Par Fonctionnalité
```java
@DisplayName("Tests du Panier d'Achat")
class PanierTest {
    @Nested
    @DisplayName("Gestion des Articles")
    class GestionArticles {
        @Test void ajoutArticle() { /* ... */ }
        @Test void suppressionArticle() { /* ... */ }
        @Test void modificationQuantite() { /* ... */ }
    }

    @Nested
    @DisplayName("Calcul des Prix")
    class CalculPrix {
        @Test void calculSousTotal() { /* ... */ }
        @Test void calculTVA() { /* ... */ }
        @Test void calculTotal() { /* ... */ }
    }
}
```

### 2. Par Scénario
```java
@DisplayName("Processus de Commande")
class ProcessusCommandeTest {
    @Nested
    @DisplayName("Parcours Client Normal")
    class ParcoursNormal {
        @Test void ajoutPanier() { /* ... */ }
        @Test void validationPanier() { /* ... */ }
        @Test void paiement() { /* ... */ }
    }

    @Nested
    @DisplayName("Gestion des Erreurs")
    class GestionErreurs {
        @Test void panierVide() { /* ... */ }
        @Test void paiementRefuse() { /* ... */ }
        @Test void stockInsuffisant() { /* ... */ }
    }
}
```

## Exemple Complet d'une Test Suite

```java
@DisplayName("Système de Gestion Utilisateurs")
class GestionUtilisateursTest {
    private UtilisateurService service;
    private UtilisateurRepository repository;

    @BeforeEach
    void setUp() {
        repository = new UtilisateurRepository();
        service = new UtilisateurService(repository);
    }

    @Nested
    @DisplayName("Inscription Utilisateur")
    class Inscription {
        @Test
        void inscriptionReussie() {
            UtilisateurDTO dto = new UtilisateurDTO("test@email.com", "password123");
            Utilisateur utilisateur = service.inscrire(dto);
            assertNotNull(utilisateur.getId());
        }

        @Test
        void emailDejaUtilise() {
            UtilisateurDTO dto = new UtilisateurDTO("existant@email.com", "password123");
            assertThrows(EmailDejaUtiliseException.class, () -> service.inscrire(dto));
        }

        @Test
        void motDePasseTropCourt() {
            UtilisateurDTO dto = new UtilisateurDTO("test@email.com", "123");
            assertThrows(ValidationException.class, () -> service.inscrire(dto));
        }
    }

    @Nested
    @DisplayName("Authentification")
    class Authentification {
        @Test
        void connexionReussie() {
            assertTrue(service.authentifier("user@email.com", "password123"));
        }

        @Test
        void mauvaisMotDePasse() {
            assertFalse(service.authentifier("user@email.com", "wrong"));
        }

        @Test
        void utilisateurInexistant() {
            assertFalse(service.authentifier("inconnu@email.com", "password123"));
        }
    }

    @Nested
    @DisplayName("Gestion du Profil")
    class GestionProfil {
        private Utilisateur utilisateur;

        @BeforeEach
        void preparerUtilisateur() {
            utilisateur = service.inscrire(new UtilisateurDTO("profil@test.com", "password123"));
        }

        @Test
        void modificationProfil() {
            utilisateur.setNom("Nouveau Nom");
            Utilisateur modifie = service.mettreAJour(utilisateur);
            assertEquals("Nouveau Nom", modifie.getNom());
        }

        @Test
        void suppressionCompte() {
            service.supprimer(utilisateur.getId());
            assertFalse(service.existe(utilisateur.getId()));
        }
    }
}
```

## Bonnes Pratiques 👍

1. **Organisation Logique**
```java
@DisplayName("API REST Utilisateurs")
class UtilisateursAPITest {
    @Nested
    class GET { /* Tests des endpoints GET */ }
    @Nested
    class POST { /* Tests des endpoints POST */ }
    @Nested
    class PUT { /* Tests des endpoints PUT */ }
    @Nested
    class DELETE { /* Tests des endpoints DELETE */ }
}
```

2. **Tests Indépendants**
```java
@Nested
class ValidationEmail {
    @Test void emailValide() { /* ... */ }
    @Test void emailInvalide() { /* ... */ }
    @Test void emailVide() { /* ... */ }
}
```

3. **Configuration Partagée**
```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class BaseDeDonneesTest {
    @BeforeAll
    void initialiserBDD() { /* ... */ }

    @Nested
    class TestsLecture { /* ... */ }

    @Nested
    class TestsEcriture { /* ... */ }

    @AfterAll
    void nettoyerBDD() { /* ... */ }
}
```

## Points Clés à Retenir 🎯

1. Les Test Suites permettent une organisation claire des tests
2. Utilisez `@Nested` pour créer des groupes logiques
3. Chaque suite peut avoir sa propre configuration
4. Les suites peuvent partager des ressources communes
5. Une bonne organisation facilite la maintenance
6. Les suites permettent d'exécuter des sous-ensembles de tests 