# Viking Transport

Application web complète développée en **PHP / Oracle (PDO)** pour **Viking Transport**, une entreprise fictive de transport régional par autocar en Normandie (19 lignes régulières, 77 communes desservies). Elle propose un site vitrine, la consultation des lignes et horaires, un calculateur d'itinéraires multi-correspondances, la réservation en ligne, un espace client avec programme de fidélité et un back-office administrateur.

> Projet réalisé en équipe dans le cadre du **BUT Informatique** à l'**IUT Grand Ouest Normandie (Caen — site d'Ifs, Semestre 2, Groupe 1)**.

---

## Fonctionnalités clés

### Côté voyageurs & espace client
- **Site vitrine & identité visuelle** (`public/index.php`) : présentation de l'entreprise, du réseau normand et du programme de fidélité.
- **Consultation des lignes & horaires** (`frontend/lignes.php`, `frontend/horaire.php`) : affichage des lignes regroupées par numéro et liste ordonnée des communes desservies.
- **Calculateur d'itinéraires multi-correspondances** (`includes/itineraire.php`) : recherche de trajets par algorithme **BFS** (*Breadth-First Search*, jusqu'à 4 segments), combinaison automatique des horaires de correspondance et transfert direct vers le formulaire de réservation (clients connectés).
- **Système de réservation à 3 modes** (`frontend/reserver.php`) :
  - **Ligne complète** (`ligne`) : accessible sans compte.
  - **Trajet partiel** (`partielle`) : entre deux arrêts d'une même ligne, accessible sans compte.
  - **Multi-lignes** (`multi`) : enchaînement manuel ou pré-rempli de plusieurs étapes avec correspondances (réservé aux clients connectés).
  - Calcul automatique du tarif par tranches kilométriques (`VIK_TARIF`), écran récapitulatif avant validation et attribution de points de fidélité.
- **Compte client & programme de fidélité** (`frontend/inscription.php`, `connexion.php`, `compte.php`, `modifier_compte.php`, `profile.php`, `reservations.php`) : gestion des coordonnées, suivi des points « drakkars » (en cours / cumulés) et historique des réservations avec détail des étapes.
- **Pages légales** : Conditions Générales d'Utilisation (`frontend/CGU.php`) et Politique de Confidentialité (`frontend/PDC.php`).

### Côté administrateur
- **Tableau de bord statistiques** (`frontend/statistique.php`) : KPIs globaux (clients, réservations, chiffre d'affaires total, lignes), évolution mensuelle du CA sur les 12 derniers mois, Top 10 des trajets les plus empruntés et Top 10 des clients les plus fidèles.
- **Modification des horaires en cascade** (`frontend/modifierLignesHoraire.php`) : décalage d'un horaire d'arrêt propagé automatiquement en base (`VIK_NOEUD`) à l'ensemble des arrêts suivants de la ligne.
- **Gestion des comptes clients & rétention des données** (`frontend/listeclient.php`, `frontend/supprimer_client.php`, `frontend/pseudocron.php`) :
  - Passage automatique en statut `inactif` après **30 jours** sans connexion et vérification de cohérence des points.
  - Suppression automatique des comptes inactifs depuis plus de **2 ans** (732 jours) avec action de sauvegarde manuelle par l'administrateur pour les comptes en sursis.

### Automatisation
- **Bot de notifications Discord** (`.github/workflows/Viking_Bot.yml`) : workflow GitHub Actions envoyant un résumé détaillé (auteur, branche, fichiers ajoutés/modifiés/supprimés) sur Discord à chaque push.

---

## Stack technique

- **Backend** : PHP 8+ avec extension **PDO** (`PDO_OCI` pour Oracle ou `PDO_MYSQL`)
- **Base de données** : Oracle Database (support MySQL configurable via la constante `MOD_BDD`)
- **Frontend** : HTML5, CSS3, JavaScript, [Bootstrap 5.3](https://getbootstrap.com/) & [Bootstrap Icons 1.11](https://icons.getbootstrap.com/)
- **Serveur web** : Apache HTTP Server (`public/.htaccess`)
- **CI / Collaboration** : Git, GitHub Actions (Webhook Discord)

---

## Prérequis

- **PHP 8.0+** avec l'extension **PDO_OCI** (Oracle) — ou **PDO_MYSQL** selon la base cible
- Une instance **Oracle Database** disposant du schéma `VIK_*`
- Un serveur web local compatible PHP (Apache / XAMPP / WAMP / Laragon ou serveur PHP intégré)

---

## Installation et configuration

### 1. Cloner le dépôt

```bash
git clone https://github.com/Clemzzz/Viking-Transport.git
cd Viking-Transport
```

### 2. Configurer la connexion à la base de données

Renseigner vos identifiants de connexion dans `config/param_connexion.php` (variables `$dbOracle`, `$db_usernameOracle`, `$db_passwordOracle`, et éventuellement les équivalents MySQL).

> **Sécurité :** ne commitez jamais d'identifiants de production dans le dépôt.

### 3. Tables Oracle utilisées

L'application s'appuie sur le schéma relationnel suivant :

| Table | Rôle |
| :--- | :--- |
| `VIK_CLIENT` | Comptes utilisateurs, rôles (`admin`, `client`, `inactif`), dates de connexion, points fidélité |
| `VIK_TYPE_CLIENT` | Catégories de clients |
| `VIK_COMMUNE` | Communes normandes desservies (code INSEE, nom) |
| `VIK_LIGNE` | Lignes régulières d'autocar |
| `VIK_NOEUD` | Arrêts, heures de passage et durées/distances entre arrêts consécutifs |
| `VIK_TARIF` | Tranches kilométriques et tarification |
| `VIK_RESERVATION` | Réservations validées (date, prix total, points gagnés) |
| `VIK_ETAPE` | Étapes individuelles composant chaque réservation |

### 4. Lancer l'application en local

Placez le projet dans le répertoire racine de votre serveur Apache (ex. `htdocs/` sous XAMPP) ou démarrez le serveur intégré de PHP depuis la racine du projet :

```bash
php -S localhost:8000
```

Accédez ensuite à `http://localhost:8000/public/index.php` (ou configurez le `DocumentRoot` d'Apache selon votre environnement).

### 5. Activer les notifications Discord (optionnel)

Pour activer le workflow `.github/workflows/Viking_Bot.yml`, ajoutez le secret `DISCORD_WEBHOOK_URL` dans **Settings → Secrets and variables → Actions** sur GitHub.

---

## Architecture du projet

```text
Viking-Transport/
├── .github/
│   └── workflows/
│       └── Viking_Bot.yml             # Notification Discord automatique à chaque push
├── config/
│   ├── db.php                         # Script de test de connexion PDO (Oracle / MySQL)
│   ├── param_connexion.php            # Paramètres de connexion à la base de données
│   ├── pdo_agile.php                  # Fonctions utilitaires PDO (requêtes préparées, lecture, MAJ)
│   └── util.php                       # Fonctions utilitaires d'affichage / débogage
├── includes/
│   ├── header.php                     # Barre de navigation dynamique (visiteur / client / admin)
│   ├── footer.php                     # Pied de page et liens légaux
│   └── itineraire.php                 # Moteur de recherche d'itinéraires multi-correspondances (BFS)
├── frontend/
│   ├── connexion.php                  # Authentification des utilisateurs
│   ├── inscription.php                # Création de compte client
│   ├── compte.php                     # Tableau de bord du compte & points fidélité
│   ├── modifier_compte.php            # Modification des informations personnelles
│   ├── profile.php                    # Affichage du profil utilisateur
│   ├── lignes.php                     # Liste des lignes, arrêts et calculateur d'itinéraires
│   ├── horaire.php                    # Consultation détaillée des horaires d'une ligne
│   ├── reserver.php                   # Tunnel de réservation (ligne, partielle, multi-lignes)
│   ├── reservations.php               # Historique des réservations du client
│   ├── statistique.php                # Tableau de bord statistiques (Admin)
│   ├── modifierLignesHoraire.php      # Édition des horaires avec décalage en cascade (Admin)
│   ├── listeclient.php                # Gestion des clients et comptes en sursis (Admin)
│   ├── supprimer_client.php           # Suppression d'un compte client (Admin)
│   ├── pseudocron.php                 # Tâche de maintenance (inactivité 30j & cohérence points)
│   ├── CGU.php                        # Conditions Générales d'Utilisation
│   ├── PDC.php                        # Politique de Confidentialité
│   └── testsql.php                    # Script de vérification SQL
├── public/
│   ├── index.php                      # Page d'accueil vitrine
│   ├── .htaccess                      # Configuration Apache
│   ├── img/                           # Logos, icônes et carte du réseau
│   ├── gif/                           # Médias d'animation
│   └── js/
│       └── template.js                # Scripts JavaScript côté client
├── LICENSE
└── README.md
```

---

## Équipe (Groupe 1)

- **BAZIN** Clément ([@Clemzzz](https://github.com/Clemzzz))
- **CHAMPEIL** Robin
- **COLOGON** Etienne
- **COSSÉ** Damien
- **GIGUERRE** Aurélien
- **MARÉCHAL** Nils
- **POIRIER** Nolan
- **SCHERRER** Ricky

---

## Licence

Voir le fichier [LICENSE](./LICENSE) pour plus d'informations.
