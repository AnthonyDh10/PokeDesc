# Guide de Déploiement Azure - PokéDesc

## 📋 Vue d'ensemble

Vous allez créer :
1. **Azure App Service** pour le backend (API)
2. **Azure Static Web App** pour le frontend (Blazor)
3. **Azure Cosmos DB** pour la base de données (optionnel)

---

## 🔧 Partie 1 : Backend API (Déjà configuré)

### ✅ Votre backend est déjà déployé !

Vous avez déjà une Azure Web App : **pokedesc-app**
- URL : https://pokedesc-app.azurewebsites.net
- Le workflow GitHub Actions est configuré sur la branche `backend`
- À chaque push sur `backend`, le déploiement se fait automatiquement

### Configuration requise

1. **Variables d'environnement** à ajouter dans Azure Portal :
   - Allez sur https://portal.azure.com
   - Cherchez "pokedesc-app" dans la barre de recherche
   - Cliquez sur votre App Service
   - Dans le menu de gauche : **Configuration** → **Application settings**
   - Ajoutez vos variables (ex: ConnectionString pour la DB, JWT secret, etc.)

---

## 🌐 Partie 2 : Frontend (Azure Static Web Apps)

### Étape 1 : Créer une Static Web App

1. **Connectez-vous au portail Azure** : https://portal.azure.com

2. **Créer la ressource** :
   - Cliquez sur **+ Create a resource**
   - Cherchez **"Static Web App"**
   - Cliquez sur **Create**

3. **Configuration de base** :
   ```
   Subscription: Votre abonnement
   Resource Group: Choisissez le même que votre backend (ou créez-en un nouveau)
   Name: pokedesc-frontend
   Plan type: Free
   Region: West Europe (ou votre région préférée)
   ```

4. **Détails du déploiement** :
   ```
   Source: GitHub
   GitHub Account: AnthonyDh10
   Organization: AnthonyDh10
   Repository: PokeDesc
   Branch: frontend
   ```

5. **Détails de build** :
   ```
   Build Presets: Blazor
   App location: /Projet_FullStack_FrontEnd
   Api location: (laissez vide)
   Output location: wwwroot
   ```

6. Cliquez sur **Review + create** puis **Create**

### Étape 2 : Récupérer le token de déploiement

Une fois la Static Web App créée :

1. Allez dans votre ressource **pokedesc-frontend**
2. Dans le menu de gauche : **Overview**
3. Cliquez sur **Manage deployment token**
4. **Copiez le token** (vous en aurez besoin pour GitHub)

### Étape 3 : Configurer le secret GitHub

1. Allez sur GitHub : https://github.com/AnthonyDh10/PokeDesc
2. **Settings** → **Secrets and variables** → **Actions**
3. Cliquez sur **New repository secret**
4. Créez un secret :
   ```
   Name: AZURE_STATIC_WEB_APPS_API_TOKEN
   Value: [Collez le token copié à l'étape 2]
   ```
5. Cliquez sur **Add secret**

### Étape 4 : Configurer l'URL de l'API dans le frontend

L'URL de votre Static Web App sera quelque chose comme :
`https://pokedesc-frontend.azurestaticapps.net`

Vous devez maintenant configurer l'URL de l'API backend dans le frontend.

---

## 🔗 Partie 3 : Connecter Backend et Frontend

### Sur la branche frontend

1. **Modifier appsettings.Production.json** :
   ```json
   {
     "ApiBaseUrl": "https://pokedesc-app.azurewebsites.net"
   }
   ```

2. **Commit et push** :
   ```bash
   git checkout frontend
   git add Projet_FullStack_FrontEnd/appsettings.Production.json
   git commit -m "Configure production API URL"
   git push origin frontend
   ```

### Sur la branche backend - Configurer CORS

Le backend doit autoriser les requêtes depuis le frontend.

1. **Sur le portail Azure** :
   - Allez sur votre App Service **pokedesc-app**
   - Menu de gauche : **CORS**
   - Ajoutez l'origine :
     ```
     https://pokedesc-frontend.azurestaticapps.net
     ```
   - Cochez **Enable Access-Control-Allow-Credentials** si nécessaire
   - Cliquez sur **Save**

2. **OU configurez dans le code** (Program.cs) :
   ```csharp
   builder.Services.AddCors(options =>
   {
       options.AddPolicy("AllowFrontend", policy =>
       {
           policy.WithOrigins("https://pokedesc-frontend.azurestaticapps.net")
                 .AllowAnyHeader()
                 .AllowAnyMethod()
                 .AllowCredentials();
       });
   });

   // Après app.UseRouting();
   app.UseCors("AllowFrontend");
   ```

---

## 🗄️ Partie 4 : Base de données (Azure Cosmos DB)

### Option 1 : Créer via le portail Azure

1. **Créer la ressource** :
   - **+ Create a resource** → **Azure Cosmos DB**
   - Choisissez **NoSQL** API

2. **Configuration** :
   ```
   Resource Group: Même groupe que vos autres ressources
   Account Name: pokedesc-cosmosdb
   Location: West Europe
   Capacity mode: Serverless (économique pour démarrer)
   ```

3. **Créer la base et les conteneurs** :
   - Après création, allez dans **Data Explorer**
   - Créez une database : `PokeDescDB`
   - Créez vos conteneurs :
     - `Dresseurs` (partition key: `/id`)
     - `Parties` (partition key: `/dresseurId`)
     - `PokemonCaptures` (partition key: `/partieId`)

4. **Récupérer la connection string** :
   - Menu de gauche : **Keys**
   - Copiez **PRIMARY CONNECTION STRING**

5. **Ajouter dans le backend** :
   - Allez sur **pokedesc-app** (App Service)
   - **Configuration** → **Application settings**
   - Nouveau setting :
     ```
     Name: CosmosDb__ConnectionString
     Value: [Votre connection string]
     ```
   - Cliquez sur **Save**

---

## ✅ Vérification du déploiement

### Backend
- URL : https://pokedesc-app.azurewebsites.net
- Test : https://pokedesc-app.azurewebsites.net/swagger

### Frontend
- URL : https://pokedesc-frontend.azurestaticapps.net
- Vérifiez que l'application se charge

### Logs et diagnostics

**Backend** :
- Azure Portal → pokedesc-app → **Log stream**
- **Monitoring** → **Metrics**

**Frontend** :
- Azure Portal → pokedesc-frontend → **Application Insights**

---

## 🚀 Workflow de développement

### Déployer le backend
```bash
git checkout backend
# Faites vos modifications
git add .
git commit -m "Description des changements"
git push origin backend
# Le déploiement se fait automatiquement
```

### Déployer le frontend
```bash
git checkout frontend
# Faites vos modifications
git add .
git commit -m "Description des changements"
git push origin frontend
# Le déploiement se fait automatiquement
```

### Synchroniser avec main
```bash
git checkout main
git merge backend
git merge frontend
git push origin main
```

---

## 🔒 Sécurité et bonnes pratiques

1. **Secrets** : Utilisez Azure Key Vault pour les secrets sensibles
2. **Environnements** : Créez des environnements Dev/Staging/Prod
3. **Monitoring** : Activez Application Insights sur les deux ressources
4. **Backup** : Configurez les backups pour Cosmos DB
5. **Scaling** : Configurez l'autoscaling si nécessaire

---

## 💰 Coûts estimés (Free tier)

- **App Service** : Plan gratuit disponible
- **Static Web App** : Gratuit (100 GB bandwidth/mois)
- **Cosmos DB Serverless** : Pay-per-use (~0.25€ par million d'opérations)

**Estimation mensuelle** : 0-5€ pour un projet personnel

---

## 🆘 Dépannage

### Le backend ne démarre pas
1. Vérifiez les logs : App Service → Log stream
2. Vérifiez la configuration : Application settings
3. Vérifiez les variables d'environnement

### Le frontend ne se déploie pas
1. Vérifiez que le secret GitHub est correct
2. Vérifiez les logs dans GitHub Actions
3. Vérifiez la configuration dans le workflow YML

### Erreurs CORS
1. Vérifiez la configuration CORS dans Azure
2. Vérifiez que l'URL frontend est correcte
3. Testez avec les outils de développement du navigateur

### Erreurs de connexion à la DB
1. Vérifiez la connection string
2. Vérifiez que Cosmos DB accepte les connexions
3. Vérifiez les règles de firewall

---

## 📚 Ressources utiles

- [Azure Portal](https://portal.azure.com)
- [Documentation Azure Static Web Apps](https://learn.microsoft.com/azure/static-web-apps/)
- [Documentation Azure App Service](https://learn.microsoft.com/azure/app-service/)
- [Documentation Azure Cosmos DB](https://learn.microsoft.com/azure/cosmos-db/)
- [GitHub Actions](https://github.com/AnthonyDh10/PokeDesc/actions)
