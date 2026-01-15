# Guide de Séparation Backend / Frontend

## Option 1 : Deux branches dans le même dépôt

### Étape 1 : Créer la branche backend
```bash
# Assurez-vous d'être sur main avec les derniers changements
git checkout main
git pull

# Créer une branche backend
git checkout -b backend

# Supprimer le dossier frontend
git rm -r Projet_FullStack_FrontEnd
git commit -m "Backend: Suppression du dossier frontend"

# Optionnel: créer un README spécifique backend
echo "# PokéDesc Backend API" > README.md
git add README.md
git commit -m "Backend: Mise à jour README"

# Pousser la branche
git push -u origin backend
```

### Étape 2 : Créer la branche frontend
```bash
# Revenir sur main
git checkout main

# Créer une branche frontend
git checkout -b frontend

# Supprimer les dossiers backend
git rm -r PokéDesc.API
git rm -r PokéDesc.Business
git rm -r PokéDesc.Domain
git rm PokéDesc.sln

# Optionnel: créer un README spécifique frontend
echo "# PokéDesc Frontend" > README.md
git add README.md
git commit -m "Frontend: Mise à jour README"

# Pousser la branche
git push -u origin frontend
```

### Résultat
- Branch `main` : Code complet
- Branch `backend` : Seulement API + Business + Domain
- Branch `frontend` : Seulement Projet_FullStack_FrontEnd

---

## Option 2 : Deux dépôts GitHub séparés (RECOMMANDÉ)

### Avantages
- Gestion indépendante des versions
- CI/CD séparé
- Déploiement indépendant
- Historique Git plus propre

### Étape 1 : Préparer le backend

```bash
# Créer un nouveau dossier pour le backend
cd ..
mkdir PokeDesc-Backend
cd PokeDesc-Backend

# Initialiser un nouveau dépôt
git init

# Copier les dossiers backend depuis le projet original
cp -r ../PokeDesc-deploy/PokéDesc.API .
cp -r ../PokeDesc-deploy/PokéDesc.Business .
cp -r ../PokeDesc-deploy/PokéDesc.Domain .
cp ../PokeDesc-deploy/PokéDesc.sln .

# Créer un .gitignore
cat > .gitignore << 'EOF'
bin/
obj/
.vs/
*.user
*.suo
appsettings.Development.json
.vscode/
EOF

# Premier commit
git add .
git commit -m "Initial commit: Backend API"

# Créer le dépôt sur GitHub et le lier
# (Créez d'abord le dépôt "PokeDesc-Backend" sur GitHub)
git remote add origin https://github.com/VOTRE_USERNAME/PokeDesc-Backend.git
git branch -M main
git push -u origin main
```

### Étape 2 : Préparer le frontend

```bash
# Créer un nouveau dossier pour le frontend
cd ..
mkdir PokeDesc-Frontend
cd PokeDesc-Frontend

# Initialiser un nouveau dépôt
git init

# Copier le dossier frontend depuis le projet original
cp -r ../PokeDesc-deploy/Projet_FullStack_FrontEnd/* .

# Créer un .gitignore
cat > .gitignore << 'EOF'
bin/
obj/
.vs/
*.user
*.suo
appsettings.Development.json
.vscode/
wwwroot/dist/
EOF

# Premier commit
git add .
git commit -m "Initial commit: Frontend Blazor"

# Créer le dépôt sur GitHub et le lier
# (Créez d'abord le dépôt "PokeDesc-Frontend" sur GitHub)
git remote add origin https://github.com/VOTRE_USERNAME/PokeDesc-Frontend.git
git branch -M main
git push -u origin main
```

---

## Recommandation

**Option 2 (deux dépôts)** est recommandée pour :
- ✅ Meilleure séparation des responsabilités
- ✅ Déploiement indépendant (backend sur Azure App Service, frontend sur Azure Static Web Apps)
- ✅ Équipes différentes peuvent travailler indépendamment
- ✅ Gestion des versions séparée
- ✅ CI/CD plus simple

**Option 1 (deux branches)** est acceptable si :
- 👤 Vous êtes seul développeur
- 📦 Vous voulez garder l'historique Git unifié
- 🔄 Les deux parties évoluent toujours ensemble

---

## Prochaines étapes après la séparation

1. **Backend** : 
   - Configurer CORS pour accepter les requêtes du frontend
   - Déployer sur Azure App Service
   
2. **Frontend** :
   - Mettre à jour l'URL de l'API dans la configuration
   - Déployer sur Azure Static Web Apps

3. **Configuration** :
   - Utiliser Azure Key Vault pour les secrets
   - Configurer les variables d'environnement pour chaque environnement
