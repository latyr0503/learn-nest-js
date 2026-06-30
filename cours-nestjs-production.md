# 🚀 Cours NestJS — Guide de Niveau Production
### Pour Talent4Startups | Rédigé par un Ingénieur Backend Senior

---

> **À qui s'adresse ce cours ?**
> Développeurs juniors ou en montée de compétences, ayant des bases en TypeScript et Node.js.
> À la fin de ce cours, vous serez capables de construire une API RESTful robuste, validée, et sécurisée avec NestJS.

---

## 📋 Table des Matières

1. [Qu'est-ce que NestJS ?](#1-quest-ce-que-nestjs)
2. [Installation & Configuration du projet](#2-installation--configuration-du-projet)
3. [Architecture : Modules, Contrôleurs, Services](#3-architecture--modules-contrôleurs-services)
4. [Validation des données avec les DTOs](#4-validation-des-données-avec-les-dtos)
5. [Gestion propre des erreurs](#5-gestion-propre-des-erreurs)
6. [Middleware dans NestJS](#6-middleware-dans-nestjs)
7. [Configuration sécurisée avec @nestjs/config](#7-configuration-sécurisée-avec-nestjsconfig)
8. [Intégration TypeORM + PostgreSQL](#8-intégration-typeorm--postgresql)
9. [Exercice Pratique : API Bibliothèque CRUD complète](#9-exercice-pratique--api-bibliothèque-crud-complète)

---

## 1. Qu'est-ce que NestJS ?

NestJS est un **framework Node.js progressif** pour créer des applications serveur efficaces, fiables et évolutives. Il est construit avec TypeScript et s'inspire fortement d'Angular (modules, décorateurs, injection de dépendances).

| Propriété | Valeur |
|---|---|
| Langage | TypeScript (natif) |
| Base HTTP | Express (par défaut) ou Fastify |
| Paradigme | OOP + Programmation fonctionnelle |
| Site officiel | [nestjs.com](https://nestjs.com) |

### Pourquoi NestJS en production ?

- ✅ **Architecture modulaire** — le code est organisé, testé, réutilisable
- ✅ **Injection de dépendances** — découplage fort des composants
- ✅ **TypeScript first** — typage fort, détection d'erreurs à la compilation
- ✅ **Écosystème riche** — guards, pipes, interceptors, microservices, WebSockets
- ✅ **Testabilité** — conçu pour les tests unitaires et e2e

---

## 2. Installation & Configuration du projet

### Prérequis

- Node.js v18+ et npm installés
- Connaissances de base en TypeScript

### Étape 1 — Installer la CLI NestJS

```bash
npm install -g @nestjs/cli
```

### Étape 2 — Créer un nouveau projet

```bash
nest new api-bibliotheque
cd api-bibliotheque
```

La CLI vous propose de choisir un gestionnaire de paquets (npm, yarn ou pnpm). Choisissez **npm**.

### Étape 3 — Structure du projet générée

```
src/
├── app.controller.ts       # Contrôleur racine
├── app.module.ts           # Module racine
├── app.service.ts          # Service racine
└── main.ts                 # Point d'entrée de l'application
```

### Étape 4 — Lancer le serveur de développement

```bash
npm run start:dev
```

Visitez `http://localhost:3000` — vous verrez `Hello World!`.

> [!TIP]
> Utilisez `npm run start:dev` (avec le flag `--watch`) en développement. L'application se rechargera automatiquement à chaque modification.

---

## 3. Architecture : Modules, Contrôleurs, Services

NestJS repose sur trois briques fondamentales que vous devez maîtriser.

```
Requête HTTP
     │
     ▼
[Contrôleur]  ← Gère les routes, délègue au service
     │
     ▼
  [Service]   ← Contient la logique métier
     │
     ▼
[Repository / Base de données]
```

### 3.1 Le Module

Le module est le **conteneur** qui regroupe un ensemble de fonctionnalités liées. Chaque feature doit avoir son propre module.

```bash
# Générer le module livres
nest generate module livres
```

```typescript
// src/livres/livres.module.ts
import { Module } from '@nestjs/common';
import { LivresController } from './livres.controller';
import { LivresService } from './livres.service';

@Module({
  controllers: [LivresController], // Contrôleurs déclarés dans ce module
  providers: [LivresService],      // Services injectables dans ce module
  exports: [LivresService],        // Services accessibles par d'autres modules
})
export class LivresModule {}
```

> [!IMPORTANT]
> **Règle d'or** : exportez uniquement le **service**, jamais le module lui-même dans le tableau `exports`. C'est une erreur très courante.

### 3.2 Le Contrôleur

Le contrôleur reçoit les requêtes HTTP et **délègue** la logique au service. Il ne contient aucune logique métier.

```bash
nest generate controller livres
```

```typescript
// src/livres/livres.controller.ts
import { Controller, Get, Post, Put, Delete, Body, Param } from '@nestjs/common';
import { LivresService } from './livres.service';

@Controller('livres') // Préfixe de route : /livres
export class LivresController {
  // Injection de dépendance via le constructeur (pattern recommandé)
  constructor(private readonly livresService: LivresService) {}

  @Get()
  findAll() {
    return this.livresService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.livresService.findOne(+id); // +id convertit string → number
  }
}
```

### 3.3 Le Service

Le service contient toute la **logique métier**. Il est injecté dans le contrôleur.

```bash
nest generate service livres
```

```typescript
// src/livres/livres.service.ts
import { Injectable } from '@nestjs/common';

@Injectable() // Marque la classe comme injectable (nécessaire pour le DI)
export class LivresService {
  findAll() {
    return [];
  }

  findOne(id: number) {
    return null;
  }
}
```

---

## 4. Validation des données avec les DTOs

Un **DTO (Data Transfer Object)** est une classe qui définit la forme des données attendues dans une requête. C'est ici qu'on valide les entrées utilisateur.

> [!WARNING]
> Ne jamais utiliser le type `any` pour les données entrantes. Sans validation, votre API accepte n'importe quelle donnée, ce qui ouvre la porte aux bugs et aux failles de sécurité.

### Installation des dépendances

```bash
npm install class-validator class-transformer
```

### Activation globale du ValidationPipe

C'est la configuration la plus importante. À faire **une seule fois** dans `main.ts` :

```typescript
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Active la validation globale sur toutes les routes
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,       // Supprime les champs non déclarés dans le DTO
      forbidNonWhitelisted: true, // Rejette la requête si des champs inconnus sont présents
      transform: true,       // Transforme automatiquement les types (ex: string → number)
    }),
  );

  await app.listen(3000);
}
bootstrap();
```

### Créer un DTO de création

```typescript
// src/livres/dto/create-livre.dto.ts
import { IsString, IsInt, IsISBN, Min, MaxLength } from 'class-validator';

export class CreateLivreDto {
  @IsString({ message: 'Le titre doit être une chaîne de caractères.' })
  @MaxLength(200, { message: 'Le titre ne peut pas dépasser 200 caractères.' })
  titre: string;

  @IsString({ message: "L'auteur doit être une chaîne de caractères." })
  auteur: string;

  @IsInt({ message: "L'année de publication doit être un entier." })
  @Min(1000, { message: "L'année de publication semble invalide." })
  anneePublication: number;

  @IsISBN(undefined, { message: "L'ISBN fourni est invalide." })
  isbn: string;
}
```

### Créer un DTO de mise à jour

NestJS fournit un utilitaire `PartialType` qui rend tous les champs optionnels — parfait pour le `PATCH`/`PUT`.

```bash
npm install @nestjs/mapped-types
```

```typescript
// src/livres/dto/update-livre.dto.ts
import { PartialType } from '@nestjs/mapped-types';
import { CreateLivreDto } from './create-livre.dto';

// Tous les champs de CreateLivreDto deviennent optionnels
export class UpdateLivreDto extends PartialType(CreateLivreDto) {}
```

### Utiliser les DTOs dans le contrôleur

```typescript
// src/livres/livres.controller.ts (mise à jour)
import { Controller, Get, Post, Put, Delete, Body, Param, ParseIntPipe } from '@nestjs/common';
import { LivresService } from './livres.service';
import { CreateLivreDto } from './dto/create-livre.dto';
import { UpdateLivreDto } from './dto/update-livre.dto';

@Controller('livres')
export class LivresController {
  constructor(private readonly livresService: LivresService) {}

  @Get()
  findAll() {
    return this.livresService.findAll();
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    // ParseIntPipe valide et convertit l'id en number automatiquement
    return this.livresService.findOne(id);
  }

  @Post()
  create(@Body() createLivreDto: CreateLivreDto) {
    // NestJS valide automatiquement createLivreDto grâce au ValidationPipe global
    return this.livresService.create(createLivreDto);
  }

  @Put(':id')
  update(
    @Param('id', ParseIntPipe) id: number,
    @Body() updateLivreDto: UpdateLivreDto,
  ) {
    return this.livresService.update(id, updateLivreDto);
  }

  @Delete(':id')
  remove(@Param('id', ParseIntPipe) id: number) {
    return this.livresService.remove(id);
  }
}
```

---

## 5. Gestion propre des erreurs

NestJS fournit des **exceptions HTTP natives** qui retournent automatiquement le bon code de statut HTTP. Plus besoin de `res.status(404).json(...)` à la main.

### Les exceptions les plus courantes

| Exception | Code HTTP | Cas d'usage |
|---|---|---|
| `NotFoundException` | 404 | Ressource introuvable |
| `BadRequestException` | 400 | Données invalides |
| `UnauthorizedException` | 401 | Non authentifié |
| `ForbiddenException` | 403 | Non autorisé |
| `ConflictException` | 409 | Conflit (ex: email déjà utilisé) |
| `InternalServerErrorException` | 500 | Erreur serveur inattendue |

### Intégration dans le service

```typescript
// src/livres/livres.service.ts (version production)
import { Injectable, NotFoundException } from '@nestjs/common';
import { CreateLivreDto } from './dto/create-livre.dto';
import { UpdateLivreDto } from './dto/update-livre.dto';

// Interface TypeScript pour typer nos données (sera remplacée par une entité TypeORM plus tard)
interface Livre {
  id: number;
  titre: string;
  auteur: string;
  anneePublication: number;
  isbn: string;
}

@Injectable()
export class LivresService {
  // Simulation d'une base de données en mémoire
  private readonly livres: Livre[] = [];
  private nextId = 1;

  findAll(): Livre[] {
    return this.livres;
  }

  findOne(id: number): Livre {
    const livre = this.livres.find((l) => l.id === id);

    // Si le livre n'existe pas, on lance une exception — NestJS retourne automatiquement un 404
    if (!livre) {
      throw new NotFoundException(`Le livre avec l'ID ${id} est introuvable.`);
    }

    return livre;
  }

  create(createLivreDto: CreateLivreDto): Livre {
    const nouveauLivre: Livre = {
      id: this.nextId++,
      ...createLivreDto,
    };

    this.livres.push(nouveauLivre);
    return nouveauLivre;
  }

  update(id: number, updateLivreDto: UpdateLivreDto): Livre {
    // On réutilise findOne qui lance déjà une NotFoundException si nécessaire
    const livre = this.findOne(id);
    const index = this.livres.indexOf(livre);

    const livreModifie: Livre = { ...livre, ...updateLivreDto };
    this.livres[index] = livreModifie;

    return livreModifie;
  }

  remove(id: number): { message: string } {
    const livre = this.findOne(id);
    const index = this.livres.indexOf(livre);
    this.livres.splice(index, 1);

    return { message: `Le livre "${livre.titre}" a été supprimé avec succès.` };
  }
}
```

> [!TIP]
> En réutilisant `this.findOne(id)` dans `update()` et `remove()`, vous évitez de dupliquer la logique de vérification. C'est du Clean Code.

---

## 6. Middleware dans NestJS

Un middleware est une fonction exécutée **avant** le gestionnaire de route. Il peut lire/modifier la requête et la réponse, ou bloquer le cycle requête-réponse.

**Ordre d'exécution NestJS :**
```
Requête → Middleware → Guards → Interceptors → Pipes → Handler → Interceptors → Réponse
```

### Créer un middleware de logging

```typescript
// src/common/middleware/logger.middleware.ts
import { Injectable, NestMiddleware, Logger } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  private readonly logger = new Logger('HTTP');

  use(req: Request, res: Response, next: NextFunction): void {
    const { method, originalUrl } = req;
    const startTime = Date.now();

    // On s'abonne à l'événement 'finish' pour logger après la réponse
    res.on('finish', () => {
      const { statusCode } = res;
      const duration = Date.now() - startTime;
      this.logger.log(`${method} ${originalUrl} ${statusCode} - ${duration}ms`);
    });

    next(); // Indispensable : passe la requête au composant suivant
  }
}
```

### Appliquer le middleware via le module

```typescript
// src/app.module.ts
import { Module, NestModule, MiddlewareConsumer, RequestMethod } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { LivresModule } from './livres/livres.module';

@Module({
  imports: [LivresModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes({ path: '*', method: RequestMethod.ALL }); // Appliqué à toutes les routes
  }
}
```

> [!IMPORTANT]
> N'utilisez **jamais** `app.use(new LoggerMiddleware().use)` dans `main.ts` : le contexte `this` sera perdu et votre middleware plantera. La bonne approche est d'utiliser `configure()` dans le module.

---

## 7. Configuration sécurisée avec @nestjs/config

> [!CAUTION]
> **Ne jamais écrire vos identifiants de base de données, clés secrètes ou mots de passe en dur dans le code.** Utilisez impérativement des variables d'environnement.

### Installation

```bash
npm install @nestjs/config
```

### Créer le fichier .env

```bash
# .env (à la racine du projet)
# IMPORTANT : Ajoutez ce fichier à votre .gitignore !

DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USER=postgres
DATABASE_PASSWORD=votre_mot_de_passe_secret
DATABASE_NAME=bibliotheque_db
```

### Ajouter .env au .gitignore

```bash
# .gitignore
.env
.env.local
.env.production
```

### Configurer le ConfigModule globalement

```typescript
// src/app.module.ts
import { Module, NestModule, MiddlewareConsumer, RequestMethod } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { LivresModule } from './livres/livres.module';

@Module({
  imports: [
    // isGlobal: true → ConfigModule accessible partout sans l'importer dans chaque module
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: '.env',
    }),
    LivresModule,
  ],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes({ path: '*', method: RequestMethod.ALL });
  }
}
```

---

## 8. Intégration TypeORM + PostgreSQL

### Installation

```bash
npm install @nestjs/typeorm typeorm pg
```

### Configurer TypeORM de manière sécurisée

On utilise `forRootAsync` pour charger la configuration **après** que les variables d'environnement soient disponibles via `ConfigService`.

```typescript
// src/app.module.ts (version finale)
import { Module, NestModule, MiddlewareConsumer, RequestMethod } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { LivresModule } from './livres/livres.module';

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),

    // forRootAsync : charge la config DB de manière asynchrone après ConfigModule
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get<string>('DATABASE_HOST'),
        port: configService.get<number>('DATABASE_PORT'),
        username: configService.get<string>('DATABASE_USER'),
        password: configService.get<string>('DATABASE_PASSWORD'),
        database: configService.get<string>('DATABASE_NAME'),
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: true, // ⚠️ Mettre à false en production ! Utiliser les migrations.
      }),
    }),

    LivresModule,
  ],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes({ path: '*', method: RequestMethod.ALL });
  }
}
```

### Créer une entité TypeORM

```typescript
// src/livres/entities/livre.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn, UpdateDateColumn } from 'typeorm';

@Entity('livres') // Nom de la table en base de données
export class Livre {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ length: 200 })
  titre: string;

  @Column()
  auteur: string;

  @Column({ name: 'annee_publication' })
  anneePublication: number;

  @Column({ unique: true }) // L'ISBN est unique pour chaque livre
  isbn: string;

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;
}
```

### Connecter l'entité au module

```typescript
// src/livres/livres.module.ts (version finale)
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { LivresController } from './livres.controller';
import { LivresService } from './livres.service';
import { Livre } from './entities/livre.entity';

@Module({
  imports: [
    TypeOrmModule.forFeature([Livre]), // Enregistre le repository Livre pour ce module
  ],
  controllers: [LivresController],
  providers: [LivresService],
  exports: [LivresService],
})
export class LivresModule {}
```

### Service avec repository TypeORM

```typescript
// src/livres/livres.service.ts (version TypeORM finale)
import { Injectable, NotFoundException, ConflictException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Livre } from './entities/livre.entity';
import { CreateLivreDto } from './dto/create-livre.dto';
import { UpdateLivreDto } from './dto/update-livre.dto';

@Injectable()
export class LivresService {
  constructor(
    // Injecte le repository TypeORM de l'entité Livre
    @InjectRepository(Livre)
    private readonly livresRepository: Repository<Livre>,
  ) {}

  async findAll(): Promise<Livre[]> {
    return this.livresRepository.find();
  }

  async findOne(id: number): Promise<Livre> {
    const livre = await this.livresRepository.findOneBy({ id });

    if (!livre) {
      throw new NotFoundException(`Le livre avec l'ID ${id} est introuvable.`);
    }

    return livre;
  }

  async create(createLivreDto: CreateLivreDto): Promise<Livre> {
    // Vérifier si un livre avec le même ISBN existe déjà
    const livreExistant = await this.livresRepository.findOneBy({
      isbn: createLivreDto.isbn,
    });

    if (livreExistant) {
      throw new ConflictException(
        `Un livre avec l'ISBN ${createLivreDto.isbn} existe déjà.`,
      );
    }

    const nouveauLivre = this.livresRepository.create(createLivreDto);
    return this.livresRepository.save(nouveauLivre);
  }

  async update(id: number, updateLivreDto: UpdateLivreDto): Promise<Livre> {
    // findOne lance déjà une NotFoundException si le livre est introuvable
    const livre = await this.findOne(id);

    // merge() fusionne les propriétés du DTO dans l'entité existante
    const livreModifie = this.livresRepository.merge(livre, updateLivreDto);
    return this.livresRepository.save(livreModifie);
  }

  async remove(id: number): Promise<{ message: string }> {
    const livre = await this.findOne(id);
    await this.livresRepository.remove(livre);

    return { message: `Le livre "${livre.titre}" a été supprimé avec succès.` };
  }
}
```

---

## 9. Exercice Pratique : API Bibliothèque CRUD complète

Vous avez maintenant tous les outils. Voici la synthèse de ce que vous venez de construire.

### Structure finale du projet

```
src/
├── common/
│   └── middleware/
│       └── logger.middleware.ts
├── livres/
│   ├── dto/
│   │   ├── create-livre.dto.ts
│   │   └── update-livre.dto.ts
│   ├── entities/
│   │   └── livre.entity.ts
│   ├── livres.controller.ts
│   ├── livres.module.ts
│   └── livres.service.ts
├── app.module.ts
└── main.ts
```

### Récapitulatif des routes de l'API

| Méthode | Route | Description | Body |
|---|---|---|---|
| `GET` | `/livres` | Récupérer tous les livres | — |
| `GET` | `/livres/:id` | Récupérer un livre par ID | — |
| `POST` | `/livres` | Créer un nouveau livre | `CreateLivreDto` |
| `PUT` | `/livres/:id` | Mettre à jour un livre | `UpdateLivreDto` |
| `DELETE` | `/livres/:id` | Supprimer un livre | — |

### Tester avec curl

```bash
# Créer un livre
curl -X POST http://localhost:3000/livres \
  -H "Content-Type: application/json" \
  -d '{
    "titre": "Clean Code",
    "auteur": "Robert C. Martin",
    "anneePublication": 2008,
    "isbn": "9780132350884"
  }'

# Récupérer tous les livres
curl http://localhost:3000/livres

# Récupérer un livre par ID
curl http://localhost:3000/livres/1

# Mettre à jour un livre
curl -X PUT http://localhost:3000/livres/1 \
  -H "Content-Type: application/json" \
  -d '{"auteur": "Uncle Bob"}'

# Supprimer un livre
curl -X DELETE http://localhost:3000/livres/1

# Tester la validation (doit retourner une erreur 400)
curl -X POST http://localhost:3000/livres \
  -H "Content-Type: application/json" \
  -d '{"titre": 123, "champInconnu": "valeur"}'
```

### Ce que vous venez de maîtriser ✅

- [x] Architecture NestJS (Module → Contrôleur → Service)
- [x] Typage strict TypeScript — zéro `any`
- [x] Validation des données avec DTOs et `class-validator`
- [x] Gestion propre des erreurs avec les exceptions NestJS
- [x] Middleware de logging propre
- [x] Configuration sécurisée via variables d'environnement
- [x] Intégration TypeORM + PostgreSQL (niveau production)

---

## 🎯 Pour aller plus loin

| Sujet | Documentation |
|---|---|
| Guards (authentification JWT) | [docs.nestjs.com/guards](https://docs.nestjs.com/guards) |
| Interceptors (transformation de réponse) | [docs.nestjs.com/interceptors](https://docs.nestjs.com/interceptors) |
| Migrations TypeORM | [typeorm.io/migrations](https://typeorm.io/migrations) |
| Tests unitaires et e2e | [docs.nestjs.com/fundamentals/testing](https://docs.nestjs.com/fundamentals/testing) |
| Swagger / OpenAPI | [docs.nestjs.com/openapi](https://docs.nestjs.com/openapi/introduction) |

---

*Cours rédigé pour Talent4Startups — Niveau Production NestJS*
