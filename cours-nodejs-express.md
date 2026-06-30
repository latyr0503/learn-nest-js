# 🚀 Développer une API REST Complète avec Node.js, Express & Sequelize
## Le Cours Complet

---

> **Objectif de ce cours :** Apprendre à créer une API REST professionnelle de A à Z en JavaScript. Au lieu d'utiliser des exemples théoriques, ce cours se base sur un cas pratique réel : la création d'un backend complet gérant des utilisateurs, des objets/produits, une base de données SQL et des uploads d'images.
>
> **Stack technique abordée :** Node.js · Express · JavaScript · Sequelize · PostgreSQL (ou MySQL) · JWT · Bcrypt · Multer

---

## 📋 Table des Matières

1. [Introduction à Node.js, Express et aux APIs REST](#chapitre-1--introduction-à-nodejs-express-et-aux-apis-rest)
2. [Initialisation et Création du Serveur](#chapitre-2--initialisation-et-création-du-serveur)
3. [L'Architecture MVC (Modèle-Vue-Contrôleur)](#chapitre-3--larchitecture-mvc)
4. [La Base de Données avec l'ORM Sequelize](#chapitre-4--la-base-de-données-avec-lorm-sequelize)
5. [Le Routage et les Opérations CRUD](#chapitre-5--le-routage-et-les-opérations-crud)
6. [Sécurité et Authentification (Bcrypt & JWT)](#chapitre-6--sécurité-et-authentification-bcrypt--jwt)
7. [Gérer l'Upload de Fichiers (Multer)](#chapitre-7--gérer-lupload-de-fichiers-multer)
8. [Finalisation : CORS et Fichiers Statiques](#chapitre-8--finalisation--cors-et-fichiers-statiques)

---

## Chapitre 1 : Introduction à Node.js, Express et aux APIs REST

### Qu'est-ce que Node.js ?
Historiquement, le JavaScript ne fonctionnait que dans les navigateurs web (Chrome, Firefox, etc.). **Node.js** est un environnement d'exécution qui permet de faire tourner du JavaScript côté serveur. Il est extrêmement performant grâce à son fonctionnement "non-bloquant" (asynchrone).

### Qu'est-ce qu'Express ?
**Express** est un framework minimaliste pour Node.js. Il fournit une série d'outils robustes pour créer des serveurs web et des APIs facilement, en gérant le routage (les URLs) et les requêtes HTTP.

### Qu'est-ce qu'une API REST ?
Une API (Application Programming Interface) permet à deux applications de communiquer entre elles. Dans notre cas, un "Frontend" (site web en React, application mobile, etc.) va envoyer des requêtes HTTP à notre "Backend" (notre API Node.js), qui va interroger la base de données et renvoyer une réponse au format JSON.

---

## Chapitre 2 : Initialisation et Création du Serveur

Pour démarrer un projet, on initialise npm (le gestionnaire de paquets de Node) et on installe Express.

```bash
npm init -y
npm install express
```

### Le point d'entrée : `server.js`

Le fichier `server.js` a une seule responsabilité : créer un serveur HTTP avec Node.js et le mettre en écoute sur un port spécifique (par exemple 3000).

```javascript
// server.js
const http = require('http');
const app = require('./app'); // On importe notre application Express

const port = process.env.PORT || 3000;
app.set('port', port);

const server = http.createServer(app);

server.listen(port, () => {
    console.log(`Le serveur écoute sur le port ${port}`);
});
```

### L'application Express : `app.js`

Le fichier `app.js` contient la logique de notre framework Express.

```javascript
// app.js
const express = require('express');

// Création de l'application Express
const app = express();

// Middleware pour parser le corps des requêtes en JSON
app.use(express.json());

// Exportation de l'application pour l'utiliser dans server.js
module.exports = app;
```
> **La notion de Middleware :** Dans Express, un middleware est une fonction qui intercepte la requête HTTP, fait quelque chose avec, et passe la main à la fonction suivante. `express.json()` est un middleware qui prend les données envoyées par l'utilisateur et les transforme en un objet JavaScript lisible dans `req.body`.

---

## Chapitre 3 : L'Architecture MVC

Pour éviter d'avoir un fichier `app.js` de 5000 lignes, nous allons organiser notre code selon l'architecture **MVC** (Modèle, Vue, Contrôleur) adaptée aux APIs.

- **Models (`models/`)** : Représentent la structure de nos données dans la base de données (Utilisateurs, Objets).
- **Controllers (`controllers/`)** : Contiennent la logique métier (Que se passe-t-il quand on crée un objet ?).
- **Routes (`routes/`)** : Définissent les URLs de l'API (`/api/stuff`) et les relient aux contrôleurs.
- **Middlewares (`middleware/`)** : Fonctions intermédiaires de vérification (sécurité, gestion de fichiers).

Voici la structure de notre projet :
```text
backend/
├── config/database.js
├── models/
├── controllers/
├── routes/
├── middleware/
├── images/
├── app.js
└── server.js
```

---

## Chapitre 4 : La Base de Données avec l'ORM Sequelize

Plutôt que d'écrire des requêtes SQL complexes (`SELECT * FROM users`), nous utilisons un **ORM** (Object-Relational Mapping) appelé **Sequelize**. Il permet de manipuler la base de données en utilisant du JavaScript.

### Connexion à la base de données
Nous créons un fichier `config/database.js` pour configurer la connexion à PostgreSQL.

```javascript
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize('nom_de_la_base', 'utilisateur', 'mot_de_passe', {
    host: 'localhost',
    dialect: 'postgres'
});

module.exports = sequelize;
```

### Création d'un Modèle (`models/Thing.js`)

Un modèle définit la structure d'une table dans notre base de données. Créons le modèle `Thing` (un objet mis en vente).

```javascript
const { DataTypes } = require('sequelize');
const sequelize = require('../config/database');
const User = require('./User'); // On importe le modèle utilisateur

const Thing = sequelize.define('Thing', {
  title: {
    type: DataTypes.STRING,
    allowNull: false // Ce champ est obligatoire
  },
  price: {
    type: DataTypes.INTEGER,
    allowNull: false
  },
  description: {
    type: DataTypes.STRING,
    allowNull: false
  },
  imageUrl: {
    type: DataTypes.STRING,
    allowNull: false
  },
  userId: {
    type: DataTypes.INTEGER,
    allowNull: false
  },
});

// Définition de la relation : Un utilisateur possède plusieurs "Things"
User.hasMany(Thing, { foreignKey: 'userId' });
Thing.belongsTo(User, { foreignKey: 'userId' });

module.exports = Thing;
```

---

## Chapitre 5 : Le Routage et les Opérations CRUD

CRUD signifie **Create, Read, Update, Delete**. Ce sont les 4 opérations de base de toute application.

### Le Contrôleur (`controllers/stuff.js`)

C'est ici qu'on écrit la logique pour manipuler nos modèles.

```javascript
const Thing = require('../models/Thing');

// Read - Récupérer tous les objets (GET)
exports.getAllThings = (req, res, next) => {
    Thing.findAll()
        .then(things => res.status(200).json(things))
        .catch(error => res.status(400).json({ error }));
};

// Read - Récupérer un objet précis par son ID (GET)
exports.getThingById = (req, res, next) => {
    Thing.findByPk({ _id: req.params.id })
        .then(thing => res.status(200).json(thing))
        .catch(error => res.status(404).json({ error }));
};
```

### Le Routeur (`routes/stuff.js`)

Le routeur associe une méthode HTTP (GET, POST...) et une URL à une fonction de notre contrôleur.

```javascript
const express = require('express');
const router = express.Router();
const stuffCtrl = require('../controllers/stuff');

router.get('/', stuffCtrl.getAllThings);
router.get('/:id', stuffCtrl.getThingById);

module.exports = router;
```

On enregistre ensuite ce routeur dans `app.js` :
```javascript
const stuffRoutes = require('./routes/stuff');
app.use('/api/stuff', stuffRoutes);
```

---

## Chapitre 6 : Sécurité et Authentification (Bcrypt & JWT)

Stocker des mots de passe en clair dans une base de données est extrêmement dangereux. Nous allons utiliser **Bcrypt** pour les hacher, et **JSON Web Token (JWT)** pour garder l'utilisateur connecté de manière sécurisée sans utiliser de sessions côté serveur.

### Inscription et Connexion (`controllers/user.js`)

```javascript
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const User = require('../models/User');

exports.signup = (req, res, next) => {
    // On hache le mot de passe (10 est le nombre de tours de salage)
    bcrypt.hash(req.body.password, 10)
        .then(hash => {
            User.create({
                email: req.body.email,
                password: hash // On enregistre le hash, pas le mot de passe clair !
            })
            .then(() => res.status(201).json({ message: 'Utilisateur créé !' }))
            .catch(error => res.status(400).json({ error }));
        });
};

exports.login = (req, res, next) => {
    User.findOne({ where: { email: req.body.email } })
        .then(user => {
            if (!user) return res.status(401).json({ error: 'Utilisateur non trouvé !' });

            // On compare le mot de passe saisi avec le hash enregistré
            bcrypt.compare(req.body.password, user.password)
                .then(valid => {
                    if (!valid) return res.status(401).json({ error: 'Mot de passe incorrect !' });
                    
                    // Si tout est bon, on crée un Token JWT valable 24h
                    res.status(200).json({
                        userId: user.id,
                        token: jwt.sign(
                            { userId: user.id },
                            'MA_CLE_SECRETE_TRES_LONGUE',
                            { expiresIn: '24h' }
                        )
                    });
                });
        });
};
```

### Le Middleware de protection (`middleware/auth.js`)

Pour protéger nos routes (ex: empêcher la création d'un objet si on n'est pas connecté), on crée un middleware qui vérifie la présence et la validité du Token.

```javascript
const jwt = require('jsonwebtoken');

module.exports = (req, res, next) => {
    try {
        // Le token arrive sous la forme "Bearer eyJhbGciOiJIUz..."
        const token = req.headers.authorization.split(' ')[1];
        const decodedToken = jwt.verify(token, 'MA_CLE_SECRETE_TRES_LONGUE');
        
        // On extrait l'ID utilisateur et on le stocke dans req.auth
        req.auth = { userId: decodedToken.userId };
        
        next(); // Autorisation accordée, on passe au contrôleur
    } catch (error) {
        res.status(401).json({ error: 'Requête non authentifiée !' });
    }
};
```

On l'ajoute ensuite dans notre routeur (`routes/stuff.js`) :
```javascript
const auth = require('../middleware/auth');
router.post('/', auth, stuffCtrl.createThing); // Le middleware auth s'exécute avant createThing
```

---

## Chapitre 7 : Gérer l'Upload de Fichiers (Multer)

Pour permettre aux utilisateurs d'envoyer des images, on utilise le package **Multer**.

### Configuration de Multer (`middleware/multer-config.js`)

```javascript
const multer = require('multer');

const MIME_TYPES = {
    'image/jpg': 'jpg',
    'image/jpeg': 'jpg',
    'image/png': 'png'
};

const storage = multer.diskStorage({
    // Indique où sauvegarder les fichiers
    destination: (req, file, callback) => {
        callback(null, 'images');
    },
    // Génère un nom de fichier unique
    filename: (req, file, callback) => {
        const name = file.originalname.split(' ').join('_');
        const extension = MIME_TYPES[file.mimetype];
        callback(null, name + Date.now() + '.' + extension);
    }
});

module.exports = multer({ storage }).single('image');
```

### Mise à jour du Contrôleur de création (`controllers/stuff.js`)

Puisque les données arrivent sous forme de `multipart/form-data`, on doit récupérer l'objet JSON (transformé en chaîne de caractères) et construire l'URL de l'image.

```javascript
exports.createThing = (req, res, next) => {
    // On parse la chaîne de caractères envoyée par le frontend
    const thingObject = JSON.parse(req.body.thing);
    
    // Construction de l'URL absolue de l'image
    const imageUrl = `${req.protocol}://${req.get('host')}/images/${req.file.filename}`;
    
    const thing = new Thing({
        ...thingObject,
        userId: req.auth.userId, // Mesure de sécurité : utiliser le userId du token, pas de la requête !
        imageUrl: imageUrl
    });
    
    thing.create()
        .then(() => res.status(201).json({ message: 'Objet créé !' }))
        .catch(error => res.status(400).json({ error }));
};
```

### Suppression d'un objet et de son image (`fs.unlink`)

Lorsqu'on supprime un objet, il faut aussi supprimer l'image physiquement du serveur en utilisant le module File System (`fs`) natif de Node.js.

```javascript
const fs = require('fs');

exports.deleteThing = (req, res, next) => {
    Thing.findByPk({ _id: req.params.id })
        .then(thing => {
            // Sécurité : Seul le propriétaire peut supprimer son objet
            if (thing.userId !== req.auth.userId) {
                return res.status(401).json({ message: 'Non autorisé' });
            }
            
            // On extrait le nom du fichier depuis l'URL
            const filename = thing.imageUrl.split('/images/')[1];
            
            // On supprime le fichier physiquement
            fs.unlink(`images/${filename}`, () => {
                // Une fois supprimé, on supprime l'entrée dans la base de données
                Thing.destroy({ _id: req.params.id })
                    .then(() => res.status(200).json({ message: 'Objet supprimé !' }))
                    .catch(error => res.status(400).json({ error }));
            });
        });
};
```

---

## Chapitre 8 : Finalisation : CORS et Fichiers Statiques

Pour que notre API soit utilisable par un navigateur web, deux dernières étapes sont nécessaires dans `app.js`.

### 1. Gérer les erreurs CORS
Les navigateurs bloquent par défaut les requêtes HTTP entre deux serveurs différents (par exemple un frontend React sur `localhost:4000` vers une API sur `localhost:3000`). Il faut explicitement autoriser ces requêtes.

```javascript
app.use((req, res, next) => {
  // Autorise toutes les origines (*)
  res.setHeader('Access-Control-Allow-Origin', '*');
  // Autorise certains headers
  res.setHeader('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content, Accept, Content-Type, Authorization');
  // Autorise certaines méthodes HTTP
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS');
  next();
});
```

### 2. Servir le dossier `images`
Par défaut, Express ne permet pas d'accéder aux dossiers de votre serveur via une URL. Il faut déclarer le dossier `images` comme un dossier "statique".

```javascript
const path = require('path');

// Dès qu'une requête arrive sur l'URL /images, on sert le dossier local "images"
app.use('/images', express.static(path.join(__dirname, 'images')));
```

---

### 🎉 Conclusion
Félicitations ! Vous venez de construire une API REST complète, structurée, sécurisée par JWT, connectée à une base de données SQL avec Sequelize, et capable de gérer des uploads de fichiers avec Multer. Cette architecture est solide et prête à être étendue pour des projets professionnels !
