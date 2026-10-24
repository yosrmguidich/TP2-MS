# TP2 REST : Service web « Calculatrice » avec JAX-RS et Tomcat

| | |
|---|---|
| **Établissement** | Faculté des Sciences de Sfax, Département d'informatique et des Communications |
| **Matière** | Architecture MicroServices |
| **Filière** | MPSRCC1 |
| **Année universitaire** | 2026-2027 |
| **Enseignantes** | Corinne Amel ZAYANI, Amal ABID |
| **Réalisé par** | *(Nom Prénom)* |

---

## Table des matières

1. [Introduction et objectifs](#1-introduction-et-objectifs)
2. [Environnement de travail](#2-environnement-de-travail)
3. [Partie A : Conception du service](#3-partie-a--conception-du-service)
4. [Partie B : Création du projet Maven](#4-partie-b--création-du-projet-maven)
5. [Test préliminaire : Hello World](#5-test-préliminaire--hello-world)
6. [Partie C : Premier service REST, la somme](#6-partie-c--premier-service-rest--la-somme)
7. [Partie D : Soustraction et produit](#7-partie-d--soustraction-et-produit)
8. [Récapitulatif des tests](#8-récapitulatif-des-tests)
9. [Conclusion](#9-conclusion)
10. [Annexe : structure du dépôt](#10-annexe--structure-du-dépôt)

---

## 1. Introduction et objectifs

Dans ce TP, nous développons un **service web REST** qui expose une calculatrice avec **JAX-RS** (implémentation Jersey). Il est déployé dans le serveur **Apache Tomcat 10.1**, puis testé avec un navigateur, **curl** et **Postman**.

Ce compte rendu couvre le test Hello World, puis les trois premiers calculs : **somme**, **soustraction** et **produit**.

### Objectifs pédagogiques et où ils sont traités

| Objectif du TP | Traité dans |
|---|---|
| Concevoir une ressource REST : choisir ses URI et les verbes HTTP adaptés | [Partie A](#3-partie-a--conception-du-service) |
| Créer un projet Maven de type application web (`.war`) | [Partie B](#4-partie-b--création-du-projet-maven) |
| Écrire un service REST avec les annotations JAX-RS | [Hello World](#5-test-préliminaire--hello-world), [Parties C et D](#6-partie-c--premier-service-rest--la-somme) |
| Déployer l'application dans Tomcat | [C.4 et C.5](#c4-construire-le-fichier-war) |
| Échanger des données (texte brut ici, JSON dans la suite du TP) | [C.3](#c3-la-ressource-rest--classe-calculrest) |
| Tester un service avec curl et Postman et interpréter les codes de statut HTTP | [C.7](#c7-tester-le-service) et [D](#7-partie-d--soustraction-et-produit) |

---

## 2. Environnement de travail

| Outil | Version utilisée |
|---|---|
| JDK | 17 ou supérieur |
| Maven | 3.9+ (intégré à IntelliJ IDEA) |
| Apache Tomcat | **10.1.61** |
| IDE | IntelliJ IDEA |
| Clients de test | Navigateur, curl, Postman |

> **Pourquoi Tomcat 10.1 ?** Le TP utilise les paquetages `jakarta.*` (Jakarta EE 10). Tomcat 9 et les versions antérieures utilisent les anciens paquetages `javax.*`, et l'application n'y démarrerait pas.

---

## 3. Partie A : Conception du service

Avant de coder, on choisit les ressources, les URI et les verbes HTTP.

### Famille 1 : calculs directs (rien n'est enregistré sur le serveur)

| Verbe | URI | Rôle |
|---|---|---|
| GET | `/calcul/somme?a=2&b=3` | Retourne a + b |
| GET | `/calcul/soustraction?a=2&b=3` | Retourne a − b |
| GET | `/calcul/produit?a=2&b=3` | Retourne a × b |
| GET | `/calcul/division?a=6&b=3` | Retourne a ÷ b |

### Famille 2 : historique des opérations (ressource `operations`)

| Verbe | URI | Rôle |
|---|---|---|
| POST | `/operations` | Crée une opération (calcul effectué et enregistré) |
| GET | `/operations` | Retourne toutes les opérations |
| GET | `/operations/{id}` | Retourne l'opération numéro id |
| PUT | `/operations/{id}` | Modifie l'opération numéro id |
| DELETE | `/operations/{id}` | Supprime l'opération numéro id |

### Réponses aux questions de conception

**1. Pourquoi les calculs directs utilisent-ils le verbe GET et non POST ?**
Un calcul direct est une simple **lecture** : le client demande un résultat et le serveur ne crée ni ne modifie rien. GET est le verbe prévu pour cela. Il est **sûr** (aucun effet de bord) et **idempotent** (répéter la requête donne le même résultat). Il permet aussi de tester l'URL directement dans un navigateur. POST sert à créer une ressource.

**2. Comment s'appelle la partie située après le `?` dans `/calcul/somme?a=2&b=3` ?**
C'est la **chaîne de requête** (*query string*). Elle contient des **paramètres de requête** (*query parameters*) sous la forme `nom=valeur`, séparés par `&`. Ici, `a=2` et `b=3`.

**3. Quelle différence y a-t-il entre `/operations` et `/operations/3` ?**
`/operations` désigne la **collection** de toutes les opérations. `/operations/3` désigne **une ressource précise** de cette collection, celle dont l'identifiant est 3.

**4. Quel code de statut HTTP pour `/operations/99` si cette opération n'existe pas ?**
Le code **404 (Not Found)**, qui signale que la ressource demandée n'existe pas.

---

## 4. Partie B : Création du projet Maven

### B.1 Création du projet dans IntelliJ IDEA

![Fenêtre New Project, générateur Jakarta EE](images/1.png)
*Figure 1 : fenêtre New Project (générateur Jakarta EE)*
Sur l'écran suivant : version **Jakarta EE 10**, dépendances du modèle conservées, puis **Create**.

### B.2 Structure du projet

Le paquetage `com.exemple.rest` est créé dans `src/main/java`.

![Structure du projet dans IntelliJ](images/2.png)
*Figure 2 : structure du projet générée par le modèle (HelloApplication et HelloResource)*

### B.3 Configuration du `pom.xml`

Le contenu du `pom.xml` est remplacé par la configuration du TP. Elle contient les éléments suivants.

```xml
<packaging>war</packaging>  <!-- Application web -->

<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <jersey.version>3.1.9</jersey.version>
</properties>
```

| Dépendance | Rôle |
|---|---|
| `jakarta.ws.rs-api` 3.1.0 | API JAX-RS : annotations `@Path`, `@GET`, etc. |
| `jersey-container-servlet` | Intégration de Jersey avec le conteneur de servlets (Tomcat) |
| `jersey-hk2` | Moteur d'injection, indispensable au démarrage de Jersey |
| `jersey-media-json-binding` | Conversion automatique objet Java ↔ JSON |

Points importants :

- `<packaging>war</packaging>` : Maven produit une **application web** (`.war`) et non un `.jar`.
- `<finalName>RestWebApp</finalName>` : fixe le nom du fichier `RestWebApp.war`. Ce nom apparaît dans l'URL du service.
- Aucun `web.xml` n'est nécessaire : l'annotation `@ApplicationPath` le remplace.

### B.4 Vérification

Après avoir rechargé le projet Maven (**Reload All Maven Projects**), les dépendances sont téléchargées et le projet compile sans erreur.

![Rechargement Maven](images/03-maven-reload.png)
*Figure 3 : projet Maven rechargé avec succès*

---

## 5. Test préliminaire : Hello World

Avant d'écrire notre propre service, nous vérifions que la chaîne **IntelliJ → Maven → Tomcat → Jersey** fonctionne, avec l'exemple généré par le modèle.

### 5.1 Les classes générées

**`HelloApplication`** : active JAX-RS sous le préfixe `/api`.

```java
package com.exemple.rest;

import jakarta.ws.rs.ApplicationPath;
import jakarta.ws.rs.core.Application;

@ApplicationPath("/api")
public class HelloApplication extends Application {
}
```

**`HelloResource`** : ressource d'exemple.

```java
package com.exemple.rest;

import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.Produces;

@Path("/hello-world")
public class HelloResource {
    @GET
    @Produces("text/plain")
    public String hello() {
        return "Hello, World!";
    }
}
```

### 5.2 Configuration de Tomcat dans IntelliJ

Le modèle crée une configuration d'exécution nommée **Tomcat 10.1.61** (`Run > Edit Configurations...`).

- **Onglet Server** : serveur utilisé (Tomcat 10.1.61), navigateur lancé après le démarrage et URL ouverte (`http://localhost:8080/RestWebApp_war_exploded/`).
- **Onglet Deployment** : artefact déployé au démarrage, `RestWebApp:war exploded`. C'est l'application sous forme de **dossier non compressé**, pratique en développement.

![Configuration Tomcat, onglet Server](images/04-tomcat-server.png)
*Figure 4 : configuration Tomcat, onglet Server*

![Configuration Tomcat, onglet Deployment](images/05-tomcat-deployment.png)
*Figure 5 : configuration Tomcat, onglet Deployment*

> **Contexte de l'application.** Avec l'artefact *war exploded*, le contexte par défaut est `/RestWebApp_war_exploded`. Avec un déploiement manuel du fichier `RestWebApp.war` dans `TOMCAT_HOME/webapps`, le contexte est `/RestWebApp`, c'est-à-dire le `finalName`. Les URL du sujet utilisent `/RestWebApp`. Avec le lancement depuis IntelliJ, il faut donc utiliser `/RestWebApp_war_exploded`, ou changer le champ *Application context* dans l'onglet Deployment.

### 5.3 Lancement et test

On clique sur **Run** : Tomcat démarre et déploie l'application. Le test se fait à l'URL :

```
http://localhost:8080/RestWebApp_war_exploded/api/hello-world
```

| Partie de l'URL | Origine |
|---|---|
| `localhost:8080` | Machine locale, port par défaut de Tomcat |
| `/RestWebApp_war_exploded` | Contexte de l'application |
| `/api` | `@ApplicationPath("/api")` |
| `/hello-world` | `@Path("/hello-world")` |

**Test avec le navigateur**

![Hello World dans le navigateur](images/06-hello-navigateur.png)
*Figure 6 : la page affiche « Hello, World! »*

**Test avec curl**

```bash
curl -i "http://localhost:8080/RestWebApp_war_exploded/api/hello-world"
```

![Hello World avec curl](images/07-hello-curl.png)
*Figure 7 : réponse `HTTP/1.1 200`, `Content-Type: text/plain`, corps `Hello, World!`*

**Résultat : le serveur est opérationnel, Jersey démarre et l'annotation JAX-RS est bien prise en compte.**

### 5.4 Avant de passer à la calculatrice

Le modèle génère `HelloApplication`, et le TP demande de créer `RestApplication` (étape C.2). **Il ne doit exister qu'une seule classe `@ApplicationPath` dans le projet.** Sinon Jersey signale un conflit à l'exécution. Nous avons donc supprimé `HelloApplication` après le test Hello World (`HelloResource` peut être conservée ou supprimée).

---

## 6. Partie C : Premier service REST, la somme

### C.1 La logique métier : classe `Calculatrice`

Cette classe ne connaît **ni HTTP ni REST** : c'est une classe Java ordinaire. Elle sépare la logique métier de l'exposition sur le Web.

```java
package com.exemple.rest;

public class Calculatrice {

    public int somme(int a, int b) {
        return a + b;
    }

    public int soustraction(int a, int b) {
        return a - b;
    }

    public int produit(int a, int b) {
        return a * b;
    }

    public double division(int a, int b) {
        if (b == 0) {
            throw new ArithmeticException("Division par 0 impossible");
        }
        return (double) a / b;
    }
}
```

### C.2 Activation de JAX-RS : classe `RestApplication`

```java
package com.exemple.rest;

import jakarta.ws.rs.ApplicationPath;
import jakarta.ws.rs.core.Application;

@ApplicationPath("/api")
public class RestApplication extends Application {
    // L'annotation suffit pour activer JAX-RS
}
```

`@ApplicationPath("/api")` signifie que **toutes les URI du service commencent par `/api`**.

### C.3 La ressource REST : classe `CalculRest`

```java
package com.exemple.rest;

import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.Produces;
import jakarta.ws.rs.QueryParam;
import jakarta.ws.rs.core.MediaType;
import jakarta.ws.rs.core.Response;

@Path("/calcul")
public class CalculRest {

    private final Calculatrice calculService = new Calculatrice();

    @GET
    @Path("/somme")
    @Produces(MediaType.TEXT_PLAIN)
    public Response somme(@QueryParam("a") int a, @QueryParam("b") int b) {
        int result = calculService.somme(a, b);
        return Response.ok(result).build();
    }
}
```

| Annotation | Rôle |
|---|---|
| `@Path("/calcul")` (classe) | Toutes les méthodes de la classe répondent sous `/calcul` |
| `@GET` | La méthode répond aux requêtes de verbe GET |
| `@Path("/somme")` (méthode) | Complète l'URI : `/calcul/somme` |
| `@Produces(MediaType.TEXT_PLAIN)` | La réponse est du texte brut (`text/plain`) |
| `@QueryParam("a")` | Lit le paramètre `a` de l'URI (`?a=2`) et le convertit en entier |

`Response.ok(result).build()` construit une réponse de statut **200 (OK)** dont le corps contient le résultat.

### C.4 Construire le fichier `.war`

Deux méthodes sont possibles.

**Avec Maven, en terminal :**

```bash
mvn clean package
```

Résultat attendu : `BUILD SUCCESS` et le fichier `target/RestWebApp.war`.

**Avec IntelliJ :** menu `Build > Build Project`, ou `Build > Build Artifacts...`.

![Menu Build d'IntelliJ](images/08-menu-build.png)
*Figure 8 : menu Build d'IntelliJ IDEA*

![Fichier RestWebApp.war généré](images/09-war-genere.png)
*Figure 9 : le fichier `target/RestWebApp.war` est produit*

### C.5 Déploiement dans Tomcat

Le déploiement se fait via la configuration **Tomcat 10.1.61** décrite au [§5.2](#52-configuration-de-tomcat-dans-intellij). L'artefact `RestWebApp:war exploded` est déployé au démarrage. Après chaque modification du code, il faut **reconstruire puis redéployer** (ou utiliser *Update resources* / *Redeploy*).

### C.6 Comprendre l'URL du service

```
http://localhost:8080/RestWebApp/api/calcul/somme?a=2&b=3
```

| Partie de l'URL | Origine |
|---|---|
| `localhost:8080` | Machine locale et port par défaut de Tomcat |
| `/RestWebApp` | Nom du fichier `.war` (`<finalName>` du `pom.xml`) |
| `/api` | `@ApplicationPath("/api")` |
| `/calcul` | `@Path("/calcul")` de la classe |
| `/somme` | `@Path("/somme")` de la méthode |
| `?a=2&b=3` | Paramètres de requête lus par `@QueryParam` |

> Dans la suite, l'URL de base utilisée est `http://localhost:8080/RestWebApp_war_exploded/api` (lancement depuis IntelliJ). Elle est notée `BASE` dans les tableaux.

### C.7 Tester le service

**a) Avec le navigateur**

URL : `BASE/calcul/somme?a=2&b=3`. Résultat attendu : la page affiche `5`.

![Test somme navigateur](images/10-somme-navigateur.png)
*Figure 10 : somme dans le navigateur, résultat 5*

**b) Avec curl**

Les guillemets autour de l'URL sont **obligatoires** à cause du caractère `&`. L'option `-i` affiche le statut et les en-têtes.

```bash
curl -i "http://localhost:8080/RestWebApp_war_exploded/api/calcul/somme?a=2&b=3"
```

Résultat attendu : `HTTP/1.1 200`, `Content-Type: text/plain`, corps `5`.

![Test somme curl](images/11-somme-curl.png)
*Figure 11 : somme avec `curl -i`*

**c) Avec Postman**

1. Nouvelle requête (bouton **New** ou **+**)
2. Verbe **GET** et saisie de l'URL
3. Clic sur **Send**
4. Observation du corps (`5`), du statut (`200 OK`) et de l'onglet **Headers**

![Test somme Postman](images/12-somme-postman.png)
*Figure 12 : somme avec Postman, 200 OK et corps 5*

### Question C.1 : que se passe-t-il avec POST à la place de GET ?

En remplaçant GET par POST dans Postman, on obtient le statut **405 Method Not Allowed**.

**Explication :** l'URI `/calcul/somme` existe, mais la méthode Java n'est annotée qu'avec `@GET`. Aucune méthode de la ressource ne répond au verbe POST pour cette URI. Jersey répond donc 405 (« méthode non autorisée pour cette ressource »), et non 404, car la ressource existe bien. La réponse contient aussi un en-tête `Allow` qui liste les verbes acceptés.

![POST sur somme, 405](images/13-somme-post-405.png)
*Figure 13 : requête POST sur `/calcul/somme`, statut 405*

---

## 7. Partie D : Soustraction et produit

### D.1 Ajout des méthodes dans `CalculRest`

En s'inspirant de la méthode `somme`, nous ajoutons deux méthodes à la classe `CalculRest`.

```java
    @GET
    @Path("/soustraction")
    @Produces(MediaType.TEXT_PLAIN)
    public Response soustraction(@QueryParam("a") int a, @QueryParam("b") int b) {
        int result = calculService.soustraction(a, b);
        return Response.ok(result).build();
    }

    @GET
    @Path("/produit")
    @Produces(MediaType.TEXT_PLAIN)
    public Response produit(@QueryParam("a") int a, @QueryParam("b") int b) {
        int result = calculService.produit(a, b);
        return Response.ok(result).build();
    }
```

Après modification : **reconstruction et redéploiement**, puis tests.

### Tests de la soustraction et du produit

| Requête | Résultat attendu |
|---|---|
| `GET BASE/calcul/soustraction?a=10&b=4` | `6` |
| `GET BASE/calcul/produit?a=6&b=7` | `42` |

```bash
curl -i "http://localhost:8080/RestWebApp_war_exploded/api/calcul/soustraction?a=10&b=4"
curl -i "http://localhost:8080/RestWebApp_war_exploded/api/calcul/produit?a=6&b=7"
```

![Test soustraction](images/14-soustraction.png)
*Figure 14 : soustraction `a=10&b=4`, statut 200, résultat 6*

![Test produit](images/15-produit.png)
*Figure 15 : produit `a=6&b=7`, statut 200, résultat 42*

### D.3 Observations : paramètres absents ou invalides

Deux requêtes ont été envoyées pour observer le comportement de JAX-RS.

| N° | Requête | Statut | Corps |
|---|---|---|---|
| 1 | `GET BASE/calcul/somme?a=2` (b absent) | **200** | `2` |
| 2 | `GET BASE/calcul/somme?a=abc&b=3` (a non entier) | **404** | page d'erreur Tomcat (corps vide ou HTML) |

*(À confirmer et compléter avec vos propres captures.)*

![Paramètre b absent](images/16-param-absent.png)
*Figure 16 : somme avec `b` absent*

![Paramètre a invalide](images/17-param-invalide.png)
*Figure 17 : somme avec `a=abc`*

**Question D.1 : quelle valeur JAX-RS donne-t-il à un paramètre entier absent ?**
Pour un type primitif (`int`), JAX-RS affecte la **valeur par défaut Java, `0`**. Ici, `somme?a=2` calcule `2 + 0 = 2`. On peut imposer une autre valeur avec `@DefaultValue`, ou utiliser `Integer` pour recevoir `null`.

**Question D.2 : quel statut quand le paramètre n'est pas un entier ? La méthode `somme` a-t-elle été exécutée ?**
Le statut obtenu est **404 (Not Found)**. Pour un `@QueryParam`, JAX-RS ne parvient pas à convertir `"abc"` en `int` et refuse la requête avant d'appeler la méthode. **La méthode `somme` n'est donc pas exécutée.**

### D.4 Paramètres dans le chemin : `@PathParam`

Les valeurs peuvent aussi être placées dans le chemin, par exemple `/calcul/produit/6/7`. On les lit avec `@PathParam` (import `jakarta.ws.rs.PathParam`).

```java
    @GET
    @Path("/produit/{a}/{b}")
    @Produces(MediaType.TEXT_PLAIN)
    public Response produitChemin(@PathParam("a") int a, @PathParam("b") int b) {
        int result = calculService.produit(a, b);
        return Response.ok(result).build();
    }
```

Test : `GET BASE/calcul/produit/6/7` doit retourner `42`.

![Produit avec PathParam](images/18-produit-pathparam.png)
*Figure 18 : produit avec `@PathParam`, résultat 42*

**Question D.3 : quelle différence entre `@QueryParam` et `@PathParam` ?**

| | `@QueryParam` | `@PathParam` |
|---|---|---|
| Emplacement | Après le `?` : `?a=6&b=7` | Dans le chemin : `/produit/6/7` |
| Déclaration | `@QueryParam("a")` | Variable `{a}` dans `@Path`, puis `@PathParam("a")` |
| Usage typique | Paramètres **optionnels**, filtres, tri, pagination | Valeurs qui **identifient** la ressource |
| Absence | Valeur par défaut (0 pour `int`) | Non applicable : le chemin ne correspond plus (404) |

Pour lire l'identifiant dans `/operations/{id}`, on utilisera **`@PathParam("id")`**, car l'identifiant fait partie du chemin et désigne la ressource.

---

## 8. Récapitulatif des tests

| Fonction | Requête | Statut | Résultat |
|---|---|---|---|
| Hello World | `GET BASE/hello-world` | 200 | `Hello, World!` |
| Somme | `GET BASE/calcul/somme?a=2&b=3` | 200 | `5` |
| Somme (POST) | `POST BASE/calcul/somme?a=2&b=3` | 405 | Méthode non autorisée |
| Soustraction | `GET BASE/calcul/soustraction?a=10&b=4` | 200 | `6` |
| Produit | `GET BASE/calcul/produit?a=6&b=7` | 200 | `42` |
| Produit (chemin) | `GET BASE/calcul/produit/6/7` | 200 | `42` |
| Paramètre absent | `GET BASE/calcul/somme?a=2` | 200 | `2` |
| Paramètre invalide | `GET BASE/calcul/somme?a=abc&b=3` | 404 | Méthode non exécutée |

---

## 9. Conclusion

Ce TP nous a permis de :

- **concevoir** une API REST : une ressource par concept, des URI claires, et le verbe HTTP adapté à chaque action (GET pour lire, POST pour créer) ;
- **créer** un projet Maven de type `war` avec les dépendances Jersey / JAX-RS (Jakarta EE 10) ;
- **écrire** un service avec les annotations `@ApplicationPath`, `@Path`, `@GET`, `@Produces`, `@QueryParam` et `@PathParam`, en séparant la logique métier (`Calculatrice`) de l'exposition REST (`CalculRest`) ;
- **déployer** l'application dans **Tomcat 10.1** et vérifier la chaîne complète avec l'exemple Hello World ;
- **tester** le service avec le navigateur, curl et Postman, et **interpréter les codes de statut** : 200 (succès), 404 (ressource ou paramètre non trouvé), 405 (verbe non autorisé).

Les parties suivantes du TP (division avec gestion d'erreur 400, échange JSON avec `Resultat`, ressource `operations` avec POST / GET / PUT / DELETE) feront l'objet d'un complément.

---

## 10. Annexe : structure du dépôt

```
TP2-MS/
├── README.md
├── images/                 (captures d'écran citées dans ce compte rendu)
├── pom.xml
└── src/
    └── main/
        └── java/
            └── com/exemple/rest/
                ├── Calculatrice.java       (logique métier)
                ├── RestApplication.java    (activation de JAX-RS)
                └── CalculRest.java         (ressource /calcul)
```
