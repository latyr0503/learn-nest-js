# 🌱 NestJS pour Débutants — De Zéro à ta Première API
### Aucune expérience backend requise

---

> **Ce cours est pour toi si :**
> - Tu sais ce qu'est une variable, une fonction, une boucle
> - Tu n'as jamais créé de serveur ou d'API
> - Les mots "backend", "HTTP", "REST" te semblent flous
>
> 🎯 **Objectif :** À la fin, tu auras créé ta première API qui fonctionne vraiment.

---

## 🗺️ Plan du cours

1. [C'est quoi un backend ?](#1-cest-quoi-un-backend-)
2. [C'est quoi une API ?](#2-cest-quoi-une-api-)
3. [C'est quoi NestJS ?](#3-cest-quoi-nestjs-)
4. [Installer les outils](#4-installer-les-outils)
5. [Créer ton premier projet](#5-créer-ton-premier-projet)
6. [Comprendre la structure du projet](#6-comprendre-la-structure-du-projet)
7. [Ton premier contrôleur](#7-ton-premier-contrôleur)
8. [Ton premier service](#8-ton-premier-service)
9. [Exercice : API de Livres](#9-exercice--api-de-livres)

---

## 1. C'est quoi un backend ?

Imagine une application comme un restaurant 🍽️

| Partie | Rôle | Équivalent web |
|---|---|---|
| La **salle** (ce que voit le client) | Interface visuelle | **Frontend** (React, Vue...) |
| La **cuisine** (ce que fait le cuisinier) | Traitement des données | **Backend** (NestJS, Express...) |
| Le **réfrigérateur** (stockage) | Données persistantes | **Base de données** (PostgreSQL...) |

> Le **backend** = le serveur qui reçoit des demandes, les traite, et renvoie une réponse.

Quand tu cliques sur "Connexion" sur un site :
1. Le **frontend** envoie tes identifiants au backend
2. Le **backend** vérifie dans la base de données
3. Il répond "OK connecté" ou "Mauvais mot de passe"

---

## 2. C'est quoi une API ?

**API** = *Application Programming Interface* = une interface de communication.

C'est comme un **menu de restaurant** : il liste tout ce que tu peux demander, et comment le demander.

### Le protocole HTTP

Quand ton navigateur parle à un serveur, il utilise HTTP. Il y a 4 actions principales :

| Action HTTP | Signification | Analogie restaurant |
|---|---|---|
| `GET` | Récupérer des données | "Montre-moi le menu" |
| `POST` | Créer quelque chose | "Je veux commander un plat" |
| `PUT` | Modifier quelque chose | "Change mon plat pour un autre" |
| `DELETE` | Supprimer quelque chose | "Annule ma commande" |

### Les codes de réponse

Le serveur répond toujours avec un **code** :

| Code | Signification |
|---|---|
| `200` | Succès |
| `201` | Créé avec succès |
| `400` | Ta demande est incorrecte |
| `404` | Introuvable |
| `500` | Erreur du serveur |

### Le format JSON

Les données s'échangent en **JSON** (JavaScript Object Notation) :

```json
{
  "id": 1,
  "titre": "Clean Code",
  "auteur": "Robert Martin"
}
```

C'est juste du texte structuré que tout le monde peut lire.

---

## 3. C'est quoi NestJS ?

NestJS est un **framework backend** écrit en TypeScript.

Un **framework** c'est comme un kit de construction LEGO : les pièces sont déjà faites, tu n'as qu'à les assembler.

Sans NestJS, créer un serveur c'est comme construire une maison avec des matériaux bruts.
Avec NestJS, c'est comme avoir des murs préfabriqués — plus rapide, plus solide, plus organisé.

**NestJS organise ton code en 3 pièces :**

```
┌─────────────────────────────────────┐
│            MODULE                   │  <- La boîte qui contient tout
│                                     │
│  ┌──────────────┐  ┌─────────────┐  │
│  │ CONTRÔLEUR   │  │  SERVICE    │  │
│  │              │  │             │  │
│  │ Reçoit les   │->│ Fait le     │  │
│  │ requêtes     │  │ vrai travail│  │
│  └──────────────┘  └─────────────┘  │
└─────────────────────────────────────┘
```

---

## 4. Installer les outils

### Ce dont tu as besoin

**1. Node.js** — Le moteur qui fait tourner JavaScript côté serveur.

Télécharge-le sur [nodejs.org](https://nodejs.org) (choisis la version **LTS**).

Vérifie l'installation :
```bash
node --version
# Doit afficher : v18.x.x ou supérieur

npm --version
# Doit afficher : 9.x.x ou supérieur
```

**2. La CLI NestJS** — L'outil en ligne de commande de NestJS.

```bash
npm install -g @nestjs/cli
```

Vérifie l'installation :
```bash
nest --version
# Doit afficher : 10.x.x
```

> CLI = Command Line Interface = outil qu'on utilise dans le terminal.
> La CLI NestJS te permet de générer du code automatiquement.

---

## 5. Créer ton premier projet

```bash
nest new mon-premier-projet
```

La CLI va te poser une question :
```
? Which package manager would you like to use?
  npm
  yarn
  pnpm
```

Choisis **npm** avec les touches fléchées, puis Entrée.

Attends que l'installation se termine (1-2 minutes), puis :

```bash
cd mon-premier-projet
npm run start:dev
```

Tu devrais voir dans le terminal :
```
[Nest] LOG [NestApplication] Nest application successfully started
```

Ouvre ton navigateur et va sur : **http://localhost:3000**

Tu vois `Hello World!` ? Félicitations ! Ton premier serveur tourne !

> `localhost` = "mon propre ordinateur"
> `3000` = le numéro de port (comme un numéro d'appartement)

---

## 6. Comprendre la structure du projet

Ouvre le dossier dans ton éditeur (VS Code recommandé). Tu vois :

```
mon-premier-projet/
├── src/                    <- Tout ton code est ici
│   ├── app.controller.ts   <- Le contrôleur principal
│   ├── app.module.ts       <- Le module principal
│   ├── app.service.ts      <- Le service principal
│   └── main.ts             <- Le point de départ de l'app
├── package.json            <- La liste des outils installés
└── tsconfig.json           <- La config TypeScript
```

### Le fichier main.ts — Le point de départ

```typescript
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

// async = cette fonction peut attendre (utile pour les opérations lentes)
async function bootstrap() {
  // Crée l'application en utilisant AppModule comme point d'entrée
  const app = await NestFactory.create(AppModule);

  // Démarre l'écoute sur le port 3000
  await app.listen(3000);
}

// Lance la fonction
bootstrap();
```

### Le fichier app.module.ts — Le chef d'orchestre

```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

// @Module() est un "décorateur" : il ajoute des informations à la classe
@Module({
  controllers: [AppController], // Liste des contrôleurs
  providers: [AppService],      // Liste des services
})
export class AppModule {}
```

> Un **décorateur** (le `@` devant) c'est comme une étiquette qu'on colle sur une boîte pour dire ce qu'elle contient.

---

## 7. Ton premier contrôleur

### À quoi ça sert ?

Le contrôleur est comme un **réceptionniste d'hôtel** :
- Il reçoit les demandes des clients (requêtes HTTP)
- Il les oriente vers la bonne personne (le service)
- Il transmet la réponse au client

### Lire le contrôleur existant

```typescript
// src/app.controller.ts
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

// @Controller() dit : "Cette classe gère les routes qui commencent par ''"
// (chaîne vide = la racine, c'est-à-dire http://localhost:3000/)
@Controller()
export class AppController {
  // NestJS crée le service et l'injecte ici automatiquement
  constructor(private readonly appService: AppService) {}

  // @Get() dit : "Cette méthode répond aux requêtes GET sur '/'"
  @Get()
  getHello(): string {
    // On délègue au service et on retourne sa réponse
    return this.appService.getHello();
  }
}
```

### Ajouter ta propre route

Modifie `app.controller.ts` :

```typescript
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }

  // Nouvelle route : répond à GET http://localhost:3000/bonjour
  @Get('bonjour')
  direBonjour(): string {
    return 'Bonjour depuis mon serveur NestJS !';
  }

  // Nouvelle route : répond à GET http://localhost:3000/info
  @Get('info')
  getInfo() {
    // On peut retourner un objet, NestJS le convertit automatiquement en JSON
    return {
      nom: 'Mon API',
      version: '1.0.0',
      message: 'Bienvenue !',
    };
  }
}
```

Sauvegarde, et visite :
- `http://localhost:3000/bonjour` → tu vois le message
- `http://localhost:3000/info` → tu vois le JSON

---

## 8. Ton premier service

### À quoi ça sert ?

Le service est comme le **cuisinier en cuisine** :
- Il fait le vrai travail (logique métier)
- Le contrôleur (réceptionniste) lui transmet les demandes

### Modifier le service existant

```typescript
// src/app.service.ts
import { Injectable } from '@nestjs/common';

// @Injectable() dit : "Cette classe peut être injectée dans d'autres classes"
@Injectable()
export class AppService {
  // Une donnée stockée dans le service
  private readonly salutations = ['Bonjour', 'Hello', 'Salut', 'Ciao', 'Hola'];

  getHello(): string {
    // Choisit une salutation au hasard
    const index = Math.floor(Math.random() * this.salutations.length);
    return this.salutations[index] + ' depuis NestJS !';
  }
}
```

Rafraîchis `http://localhost:3000` plusieurs fois — le message change !

---

## 9. Exercice : API de Livres

Maintenant, construisons quelque chose de concret : une API pour gérer une liste de livres.

### Étape 1 — Générer les fichiers

```bash
# Dans ton terminal
nest generate module livres
nest generate controller livres
nest generate service livres
```

NestJS crée automatiquement les fichiers et met à jour `app.module.ts`.

Tu verras maintenant :
```
src/
└── livres/
    ├── livres.controller.ts
    ├── livres.module.ts
    └── livres.service.ts
```

### Étape 2 — Créer le service

```typescript
// src/livres/livres.service.ts
import { Injectable } from '@nestjs/common';

// On définit la forme d'un livre avec une interface TypeScript
interface Livre {
  id: number;
  titre: string;
  auteur: string;
}

@Injectable()
export class LivresService {
  // Notre "base de données" temporaire : un simple tableau
  private livres: Livre[] = [
    { id: 1, titre: 'Le Petit Prince', auteur: 'Antoine de Saint-Exupéry' },
    { id: 2, titre: "L'Alchimiste", auteur: 'Paulo Coelho' },
  ];

  // Numéro du prochain livre
  private prochainId = 3;

  // Retourne tous les livres
  findAll(): Livre[] {
    return this.livres;
  }

  // Retourne UN livre par son identifiant
  findOne(id: number): Livre | undefined {
    // .find() parcourt le tableau et retourne le premier élément qui correspond
    return this.livres.find((livre) => livre.id === id);
  }

  // Crée un nouveau livre
  create(titre: string, auteur: string): Livre {
    const nouveauLivre: Livre = {
      id: this.prochainId++, // Attribue l'ID, puis l'incrémente
      titre,
      auteur,
    };
    this.livres.push(nouveauLivre);
    return nouveauLivre;
  }

  // Supprime un livre
  remove(id: number): boolean {
    const index = this.livres.findIndex((livre) => livre.id === id);
    if (index === -1) {
      return false; // Livre non trouvé
    }
    this.livres.splice(index, 1);
    return true; // Suppression réussie
  }
}
```

### Étape 3 — Créer le contrôleur

```typescript
// src/livres/livres.controller.ts
import { Controller, Get, Post, Delete, Param, Body } from '@nestjs/common';
import { LivresService } from './livres.service';

// Toutes les routes commencent par /livres
@Controller('livres')
export class LivresController {
  constructor(private readonly livresService: LivresService) {}

  // GET http://localhost:3000/livres
  @Get()
  findAll() {
    return this.livresService.findAll();
  }

  // GET http://localhost:3000/livres/1
  // :id est un paramètre dynamique
  @Get(':id')
  findOne(@Param('id') id: string) {
    // +id convertit la chaîne "1" en nombre 1
    const livre = this.livresService.findOne(+id);
    if (!livre) {
      return { erreur: 'Livre introuvable' };
    }
    return livre;
  }

  // POST http://localhost:3000/livres
  // @Body() extrait les données envoyées dans le corps de la requête
  @Post()
  create(@Body() body: { titre: string; auteur: string }) {
    return this.livresService.create(body.titre, body.auteur);
  }

  // DELETE http://localhost:3000/livres/1
  @Delete(':id')
  remove(@Param('id') id: string) {
    const succes = this.livresService.remove(+id);
    if (!succes) {
      return { erreur: 'Livre introuvable, impossible de supprimer' };
    }
    return { message: 'Livre supprimé avec succès' };
  }
}
```

### Étape 4 — Tester ton API

**GET tous les livres** (dans ton navigateur) :

Visite : `http://localhost:3000/livres`

```json
[
  { "id": 1, "titre": "Le Petit Prince", "auteur": "Antoine de Saint-Exupéry" },
  { "id": 2, "titre": "L'Alchimiste", "auteur": "Paulo Coelho" }
]
```

**GET un livre par ID** :

Visite : `http://localhost:3000/livres/1`

**POST — Créer un livre** (avec curl ou Thunder Client) :

```bash
curl -X POST http://localhost:3000/livres \
  -H "Content-Type: application/json" \
  -d '{"titre": "Harry Potter", "auteur": "J.K. Rowling"}'
```

**DELETE — Supprimer un livre** :

```bash
curl -X DELETE http://localhost:3000/livres/1
```

---

## Récapitulatif — Ce que tu as appris

| Concept | Ce que c'est | Exemple |
|---|---|---|
| **Backend** | Le serveur qui traite les demandes | NestJS sur ton ordinateur |
| **API** | Le contrat entre frontend et backend | Les routes `/livres` |
| **HTTP** | Le langage de communication | GET, POST, DELETE |
| **JSON** | Le format des données échangées | `{"titre": "..."}` |
| **Module** | Le conteneur d'une feature | `LivresModule` |
| **Contrôleur** | Le réceptionniste des requêtes | `LivresController` |
| **Service** | Le coeur de la logique métier | `LivresService` |
| **Décorateur** | L'étiquette qui configure une classe | `@Get()`, `@Injectable()` |

---

## Et maintenant ?

Tu as créé ta première API ! Prochaines étapes :

1. **Valider les données** — Et si quelqu'un envoie un titre vide ? Apprends les DTOs.
2. **Sauvegarder en base de données** — Ton tableau en mémoire se vide au redémarrage. Prochaine étape : PostgreSQL + TypeORM.
3. **Gérer les erreurs proprement** — Utiliser `NotFoundException` au lieu d'un objet `{ erreur: ... }`.

> Quand tu te sens à l'aise avec ce cours, passe au **cours NestJS Production** pour apprendre les bonnes pratiques professionnelles.

---

*Cours débutant NestJS*
