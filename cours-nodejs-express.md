# 🚀 API REST avec Node.js, Express & TypeScript
### Guide de Niveau Production — Architecture en Couches

---

> **À qui s'adresse ce cours ?**
> Développeurs ayant des bases en JavaScript/TypeScript, souhaitant construire des API REST robustes sans framework opinionné comme NestJS.
>
> **Stack technique :** Node.js · Express · TypeScript · Zod · TypeORM · PostgreSQL

---

## 📋 Table des Matières

1. [Pourquoi Node.js + Express + TypeScript ?](#1-pourquoi-nodejs--express--typescript-)
2. [Initialisation du projet](#2-initialisation-du-projet)
3. [Architecture en couches](#3-architecture-en-couches)
4. [CRUD en mémoire — Gestion de Livres](#4-crud-en-mémoire--gestion-de-livres)
5. [Validation des données avec Zod](#5-validation-des-données-avec-zod)
6. [Gestion globale des erreurs](#6-gestion-globale-des-erreurs)
7. [Persistance PostgreSQL avec TypeORM](#7-persistance-postgresql-avec-typeorm)
8. [Récapitulatif et structure finale](#8-récapitulatif-et-structure-finale)

---

## 1. Pourquoi Node.js + Express + TypeScript ?

### Node.js

Node.js est un **environnement d'exécution JavaScript côté serveur**. Il utilise un modèle non-bloquant basé sur les événements, ce qui le rend très efficace pour les API I/O-intensives.

### Express

Express est le framework HTTP le plus populaire de l'écosystème Node.js. Il est **minimaliste et non-opinionné** : il ne t'impose aucune structure. C'est à toi de définir ton architecture — ce qui est un avantage en production car tu gardes le contrôle total.

### TypeScript

TypeScript ajoute le **typage statique** à JavaScript. En production, cela signifie :
- Les erreurs sont détectées **à la compilation**, pas à l'exécution
- Le code est **auto-documenté** grâce aux types
- La refactorisation est **sécurisée**

### Comparaison rapide

| Critère | Express + TS | NestJS |
|---|---|---|
| Structure | Libre (tu décides) | Imposée (modules, DI) |
| Courbe d'apprentissage | Douce | Moyenne |
| Flexibilité | Maximale | Limitée par le framework |
| Cas d'usage | APIs simples à moyennes | Applications enterprise |

---

## 2. Initialisation du projet

### Étape 1 — Créer le projet

```bash
mkdir api-bibliotheque
cd api-bibliotheque
npm init -y
```

### Étape 2 — Installer les dépendances

```bash
# Dépendances de production
npm install express dotenv

# Dépendances de développement (TypeScript, types, rechargement à chaud)
npm install -D typescript ts-node-dev @types/node @types/express

# Validation
npm install zod
```

### Étape 3 — Configurer TypeScript

```bash
npx tsc --init
```

Remplace le contenu de `tsconfig.json` par :

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Étape 4 — Configurer les scripts dans package.json

```json
{
  "scripts": {
    "dev": "ts-node-dev --respawn --transpile-only src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js"
  }
}
```

> `--respawn` relance automatiquement le serveur à chaque modification. C'est l'équivalent de `npm run start:dev` dans NestJS.

### Étape 5 — Le point d'entrée

```typescript
// src/index.ts
import express from 'express';
import { livresRouter } from './routes/livres.routes';
import { errorHandler } from './middlewares/error.middleware';

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware global : parse le JSON dans le corps des requêtes
app.use(express.json());

// Routes
app.use('/livres', livresRouter);

// Middleware de gestion globale des erreurs (DOIT être en dernier)
app.use(errorHandler);

app.listen(PORT, () => {
  console.log(`Serveur démarré sur http://localhost:${PORT}`);
});

export default app;
```

---

## 3. Architecture en couches

Sans framework opinionné, **tu es responsable de l'organisation**. L'architecture en couches est le standard de l'industrie.

```
Requête HTTP
     │
     ▼
┌─────────────┐
│   ROUTES    │  src/routes/       ← Définit les URLs et méthodes HTTP
└──────┬──────┘
       │
       ▼
┌─────────────┐
│CONTRÔLEURS  │  src/controllers/  ← Parse req, appelle le service, renvoie res
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  SERVICES   │  src/services/     ← Logique métier pure (pas d'Express ici !)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ BASE DE     │  src/entities/     ← Entités TypeORM / Repository
│ DONNÉES     │
└─────────────┘
```

### Règle d'or

> Le service **ne connaît pas Express**. Il ne manipule jamais `req` ni `res`. Il reçoit des données, fait son travail, et retourne un résultat. Cela le rend **testable indépendamment**.

### Structure des dossiers

```
src/
├── controllers/
│   └── livres.controller.ts
├── services/
│   └── livres.service.ts
├── routes/
│   └── livres.routes.ts
├── middlewares/
│   ├── error.middleware.ts
│   └── validate.middleware.ts
├── schemas/
│   └── livre.schema.ts          ← Schémas Zod (équivalent des DTOs)
├── types/
│   └── livre.types.ts           ← Interfaces TypeScript
└── index.ts
```

---

## 4. CRUD en mémoire — Gestion de Livres

### Les types TypeScript

```typescript
// src/types/livre.types.ts

// Interface qui définit la forme d'un livre en base de données
export interface Livre {
  id: number;
  titre: string;
  auteur: string;
  anneePublication: number;
  isbn: string;
}

// Type pour la création (sans id, il est généré automatiquement)
export type CreateLivrePayload = Omit<Livre, 'id'>;

// Type pour la mise à jour (tous les champs sont optionnels)
export type UpdateLivrePayload = Partial<CreateLivrePayload>;
```

### Le service

```typescript
// src/services/livres.service.ts
import { Livre, CreateLivrePayload, UpdateLivrePayload } from '../types/livre.types';

// Erreur métier personnalisée
export class NotFoundError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'NotFoundError';
  }
}

export class ConflictError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'ConflictError';
  }
}

export class LivresService {
  // Base de données en mémoire (sera remplacée par TypeORM)
  private livres: Livre[] = [
    {
      id: 1,
      titre: 'Clean Code',
      auteur: 'Robert C. Martin',
      anneePublication: 2008,
      isbn: '9780132350884',
    },
  ];

  private nextId = 2;

  findAll(): Livre[] {
    return this.livres;
  }

  findById(id: number): Livre {
    const livre = this.livres.find((l) => l.id === id);
    if (!livre) {
      throw new NotFoundError(`Livre avec l'ID ${id} introuvable.`);
    }
    return livre;
  }

  create(payload: CreateLivrePayload): Livre {
    // Vérification de l'unicité de l'ISBN
    const existant = this.livres.find((l) => l.isbn === payload.isbn);
    if (existant) {
      throw new ConflictError(`Un livre avec l'ISBN ${payload.isbn} existe déjà.`);
    }

    const nouveauLivre: Livre = { id: this.nextId++, ...payload };
    this.livres.push(nouveauLivre);
    return nouveauLivre;
  }

  update(id: number, payload: UpdateLivrePayload): Livre {
    const livre = this.findById(id); // Lance NotFoundError si absent
    const index = this.livres.indexOf(livre);
    this.livres[index] = { ...livre, ...payload };
    return this.livres[index];
  }

  delete(id: number): void {
    const livre = this.findById(id);
    this.livres = this.livres.filter((l) => l.id !== livre.id);
  }
}
```

### Le contrôleur

```typescript
// src/controllers/livres.controller.ts
import { Request, Response, NextFunction } from 'express';
import { LivresService } from '../services/livres.service';

// Instance du service (en production avec un vrai DI container, ce serait injecté)
const livresService = new LivresService();

// GET /livres
export const getAll = (req: Request, res: Response, next: NextFunction): void => {
  try {
    const livres = livresService.findAll();
    res.status(200).json(livres);
  } catch (error) {
    next(error); // Passe l'erreur au middleware global
  }
};

// GET /livres/:id
export const getById = (req: Request, res: Response, next: NextFunction): void => {
  try {
    const id = parseInt(req.params.id, 10);
    const livre = livresService.findById(id);
    res.status(200).json(livre);
  } catch (error) {
    next(error);
  }
};

// POST /livres
export const create = (req: Request, res: Response, next: NextFunction): void => {
  try {
    // req.body est déjà validé par le middleware Zod (voir section 5)
    const livre = livresService.create(req.body);
    res.status(201).json(livre);
  } catch (error) {
    next(error);
  }
};

// PUT /livres/:id
export const update = (req: Request, res: Response, next: NextFunction): void => {
  try {
    const id = parseInt(req.params.id, 10);
    const livre = livresService.update(id, req.body);
    res.status(200).json(livre);
  } catch (error) {
    next(error);
  }
};

// DELETE /livres/:id
export const remove = (req: Request, res: Response, next: NextFunction): void => {
  try {
    const id = parseInt(req.params.id, 10);
    livresService.delete(id);
    res.status(200).json({ message: 'Livre supprimé avec succès.' });
  } catch (error) {
    next(error);
  }
};
```

### Les routes

```typescript
// src/routes/livres.routes.ts
import { Router } from 'express';
import { getAll, getById, create, update, remove } from '../controllers/livres.controller';
import { validate } from '../middlewares/validate.middleware';
import { createLivreSchema, updateLivreSchema } from '../schemas/livre.schema';

export const livresRouter = Router();

livresRouter.get('/', getAll);
livresRouter.get('/:id', getById);

// Le middleware validate() s'exécute AVANT le contrôleur
livresRouter.post('/', validate(createLivreSchema), create);
livresRouter.put('/:id', validate(updateLivreSchema), update);
livresRouter.delete('/:id', remove);
```

---

## 5. Validation des données avec Zod

Zod est l'équivalent de `class-validator` dans NestJS. Il permet de définir un schéma de validation et de le faire respecter.

### Les schémas Zod

```typescript
// src/schemas/livre.schema.ts
import { z } from 'zod';

// Schéma de création — tous les champs sont obligatoires
export const createLivreSchema = z.object({
  titre: z
    .string({ required_error: 'Le titre est obligatoire.' })
    .min(1, 'Le titre ne peut pas être vide.')
    .max(200, 'Le titre ne peut pas dépasser 200 caractères.'),

  auteur: z
    .string({ required_error: "L'auteur est obligatoire." })
    .min(2, "Le nom de l'auteur est trop court."),

  anneePublication: z
    .number({ required_error: "L'année de publication est obligatoire." })
    .int("L'année doit être un entier.")
    .min(1000, "L'année semble invalide."),

  isbn: z
    .string({ required_error: "L'ISBN est obligatoire." })
    .regex(/^(?:ISBN(?:-1[03])?:? )?(?=[0-9X]{10}$|(?=(?:[0-9]+[- ]){3})[- 0-9X]{13}$|97[89][0-9]{10}$|(?=(?:[0-9]+[- ]){4})[- 0-9]{17}$)(?:97[89][- ]?)?[0-9]{1,5}[- ]?[0-9]+[- ]?[0-9]+[- ]?[0-9X]$/, {
      message: "L'ISBN fourni est invalide.",
    }),
});

// Schéma de mise à jour — tous les champs deviennent optionnels
export const updateLivreSchema = createLivreSchema.partial();

// Types inférés depuis les schémas (DRY : une seule source de vérité)
export type CreateLivreDto = z.infer<typeof createLivreSchema>;
export type UpdateLivreDto = z.infer<typeof updateLivreSchema>;
```

### Le middleware de validation

```typescript
// src/middlewares/validate.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { ZodSchema, ZodError } from 'zod';

// Factory : retourne un middleware configuré avec le schéma fourni
export const validate = (schema: ZodSchema) => {
  return (req: Request, res: Response, next: NextFunction): void => {
    const result = schema.safeParse(req.body);

    if (!result.success) {
      // Formate les erreurs Zod pour une réponse lisible
      const errors = result.error.errors.map((err) => ({
        champ: err.path.join('.'),
        message: err.message,
      }));

      res.status(400).json({
        statusCode: 400,
        error: 'Validation échouée',
        details: errors,
      });
      return;
    }

    // Remplace req.body par les données validées et transformées par Zod
    req.body = result.data;
    next();
  };
};
```

**Exemple de réponse d'erreur de validation :**

```json
{
  "statusCode": 400,
  "error": "Validation échouée",
  "details": [
    { "champ": "titre", "message": "Le titre est obligatoire." },
    { "champ": "anneePublication", "message": "L'année doit être un entier." }
  ]
}
```

---

## 6. Gestion globale des erreurs

Sans gestionnaire d'erreurs global, une exception non capturée fait **planter le processus Node.js**. Ce middleware centralise la gestion des erreurs.

```typescript
// src/middlewares/error.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { NotFoundError, ConflictError } from '../services/livres.service';

// Interface pour typer les erreurs HTTP
interface AppError extends Error {
  statusCode?: number;
}

// Un middleware d'erreur Express a TOUJOURS 4 paramètres (err, req, res, next)
export const errorHandler = (
  err: AppError,
  req: Request,
  res: Response,
  next: NextFunction,
): void => {
  // Log l'erreur côté serveur (en production, utilise un logger comme Winston)
  console.error(`[ERROR] ${err.name}: ${err.message}`);

  // Erreurs métier connues — on leur attribue le bon code HTTP
  if (err instanceof NotFoundError) {
    res.status(404).json({ statusCode: 404, error: err.message });
    return;
  }

  if (err instanceof ConflictError) {
    res.status(409).json({ statusCode: 409, error: err.message });
    return;
  }

  // Erreur avec un statusCode explicite (ex: erreur personnalisée)
  if (err.statusCode) {
    res.status(err.statusCode).json({ statusCode: err.statusCode, error: err.message });
    return;
  }

  // Erreur inconnue — 500 par défaut, ne jamais exposer les détails en production
  res.status(500).json({
    statusCode: 500,
    error: 'Une erreur interne est survenue.',
  });
};
```

> **Pourquoi `next(error)` dans les contrôleurs ?**
> En passant l'erreur à `next()`, Express sait qu'il doit **sauter tous les middlewares normaux** et aller directement au middleware d'erreur (celui avec 4 paramètres).

---

## 7. Persistance PostgreSQL avec TypeORM

### Installation

```bash
npm install typeorm reflect-metadata pg
npm install -D @types/pg
```

### Configurer les variables d'environnement

```bash
# .env (ajoutez ce fichier à .gitignore !)
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USER=postgres
DATABASE_PASSWORD=votre_mot_de_passe
DATABASE_NAME=bibliotheque_db
NODE_ENV=development
```

```bash
# .gitignore
.env
node_modules/
dist/
```

### La connexion à la base de données

```typescript
// src/config/database.ts
import 'reflect-metadata';
import { DataSource } from 'typeorm';
import { Livre } from '../entities/livre.entity';
import * as dotenv from 'dotenv';

// Charge les variables d'environnement depuis .env
dotenv.config();

export const AppDataSource = new DataSource({
  type: 'postgres',
  host: process.env.DATABASE_HOST,
  port: parseInt(process.env.DATABASE_PORT || '5432', 10),
  username: process.env.DATABASE_USER,
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE_NAME,
  entities: [Livre],
  synchronize: process.env.NODE_ENV === 'development', // false en production !
  logging: process.env.NODE_ENV === 'development',
});
```

> En production, `synchronize: false` est **obligatoire**. Utilisez les **migrations TypeORM** pour modifier le schéma de la base de données.

### L'entité TypeORM

```typescript
// src/entities/livre.entity.ts
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
} from 'typeorm';

@Entity('livres')
export class Livre {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ length: 200 })
  titre: string;

  @Column()
  auteur: string;

  @Column({ name: 'annee_publication' })
  anneePublication: number;

  @Column({ unique: true })
  isbn: string;

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;
}
```

### Refactoring du service avec TypeORM

```typescript
// src/services/livres.service.ts (version base de données)
import { Repository } from 'typeorm';
import { AppDataSource } from '../config/database';
import { Livre } from '../entities/livre.entity';
import { CreateLivreDto, UpdateLivreDto } from '../schemas/livre.schema';

export class NotFoundError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'NotFoundError';
  }
}

export class ConflictError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'ConflictError';
  }
}

export class LivresService {
  private readonly repo: Repository<Livre>;

  constructor() {
    this.repo = AppDataSource.getRepository(Livre);
  }

  async findAll(): Promise<Livre[]> {
    return this.repo.find();
  }

  async findById(id: number): Promise<Livre> {
    const livre = await this.repo.findOneBy({ id });
    if (!livre) {
      throw new NotFoundError(`Livre avec l'ID ${id} introuvable.`);
    }
    return livre;
  }

  async create(payload: CreateLivreDto): Promise<Livre> {
    const existant = await this.repo.findOneBy({ isbn: payload.isbn });
    if (existant) {
      throw new ConflictError(`Un livre avec l'ISBN ${payload.isbn} existe déjà.`);
    }
    const livre = this.repo.create(payload);
    return this.repo.save(livre);
  }

  async update(id: number, payload: UpdateLivreDto): Promise<Livre> {
    const livre = await this.findById(id);
    const livreModifie = this.repo.merge(livre, payload);
    return this.repo.save(livreModifie);
  }

  async delete(id: number): Promise<void> {
    const livre = await this.findById(id);
    await this.repo.remove(livre);
  }
}
```

### Mettre à jour le point d'entrée

```typescript
// src/index.ts (version finale avec connexion DB)
import 'reflect-metadata';
import * as dotenv from 'dotenv';
dotenv.config(); // Doit être appelé en premier

import express from 'express';
import { AppDataSource } from './config/database';
import { livresRouter } from './routes/livres.routes';
import { errorHandler } from './middlewares/error.middleware';

const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());
app.use('/livres', livresRouter);
app.use(errorHandler);

// On démarre le serveur seulement après avoir connecté la base de données
AppDataSource.initialize()
  .then(() => {
    console.log('Connexion à PostgreSQL établie.');
    app.listen(PORT, () => {
      console.log(`Serveur démarré sur http://localhost:${PORT}`);
    });
  })
  .catch((err) => {
    console.error('Erreur de connexion à la base de données :', err);
    process.exit(1); // Arrête le processus si la DB est inaccessible
  });
```

---

## 8. Récapitulatif et structure finale

### Structure complète du projet

```
api-bibliotheque/
├── src/
│   ├── config/
│   │   └── database.ts           ← Connexion TypeORM
│   ├── controllers/
│   │   └── livres.controller.ts  ← Parse req/res, délègue au service
│   ├── entities/
│   │   └── livre.entity.ts       ← Entité TypeORM / schéma de table
│   ├── middlewares/
│   │   ├── error.middleware.ts   ← Gestionnaire global d'erreurs
│   │   └── validate.middleware.ts ← Validation Zod
│   ├── routes/
│   │   └── livres.routes.ts      ← Définition des endpoints
│   ├── schemas/
│   │   └── livre.schema.ts       ← Schémas Zod + types inférés
│   ├── services/
│   │   └── livres.service.ts     ← Logique métier pure
│   ├── types/
│   │   └── livre.types.ts        ← Interfaces partagées
│   └── index.ts                  ← Point d'entrée
├── .env                          ← Variables d'environnement (non versionné)
├── .gitignore
├── package.json
└── tsconfig.json
```

### Tableau des routes de l'API

| Méthode | Route | Description | Validation |
|---|---|---|---|
| `GET` | `/livres` | Tous les livres | — |
| `GET` | `/livres/:id` | Un livre par ID | — |
| `POST` | `/livres` | Créer un livre | `createLivreSchema` |
| `PUT` | `/livres/:id` | Modifier un livre | `updateLivreSchema` |
| `DELETE` | `/livres/:id` | Supprimer un livre | — |

### Tester avec curl

```bash
# Créer un livre
curl -X POST http://localhost:3000/livres \
  -H "Content-Type: application/json" \
  -d '{"titre":"Clean Code","auteur":"Robert Martin","anneePublication":2008,"isbn":"9780132350884"}'

# Lister tous les livres
curl http://localhost:3000/livres

# Tester la validation (doit retourner 400)
curl -X POST http://localhost:3000/livres \
  -H "Content-Type: application/json" \
  -d '{"titre":""}'

# Modifier un livre
curl -X PUT http://localhost:3000/livres/1 \
  -H "Content-Type: application/json" \
  -d '{"auteur":"Uncle Bob"}'

# Supprimer un livre
curl -X DELETE http://localhost:3000/livres/1

# Tester le 404
curl http://localhost:3000/livres/999
```

### Ce que vous maîtrisez maintenant

- [x] Initialisation d'un projet Node.js + Express + TypeScript
- [x] Architecture en couches (Routes → Contrôleurs → Services)
- [x] Typage strict — zéro `any`
- [x] Validation avec Zod (équivalent DTOs NestJS)
- [x] Middleware de validation réutilisable
- [x] Gestion centralisée des erreurs HTTP
- [x] Erreurs métier personnalisées (`NotFoundError`, `ConflictError`)
- [x] Intégration TypeORM + PostgreSQL
- [x] Configuration sécurisée via `.env`

---

## Pour aller plus loin

| Sujet | Outil recommandé |
|---|---|
| Tests unitaires | Jest + ts-jest |
| Tests d'intégration | Supertest |
| Authentification JWT | jsonwebtoken + middleware custom |
| Logging production | Winston ou Pino |
| Documentation API | Swagger UI Express |
| Migrations DB | TypeORM CLI (`typeorm migration:generate`) |
| Rate limiting | express-rate-limit |

---

*Cours Node.js/Express/TypeScript — Architecture Production*
