# 🚀 Guide de Déploiement FinSmart

## Vue d'ensemble
- **Backend** → Render (Node.js API)
- **Admin** → Vercel (React Dashboard)
- **Mobile Web** → Vercel (React Native Web)

---

## 📦 1. Déployer le Backend sur Render

### Étape 1.1 : Créer un compte Render
1. Allez sur [render.com](https://render.com)
2. Cliquez **Sign Up** et connectez-vous avec GitHub

### Étape 1.2 : Créer le Web Service
1. Cliquez **New +** → **Web Service**
2. Sélectionnez le repo **finsmart-backend**
3. Render détecte automatiquement les paramètres depuis `render.yaml`
4. Cliquez **Create Web Service**

### Étape 1.3 : Configurer MongoDB
1. Dans l'onglet **Environment**, ajoutez :
   - `MONGODB_URI` = Votre URL MongoDB Atlas
   
   **Pour obtenir MongoDB Atlas (GRATUIT) :**
   - Allez sur [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas)
   - Créez un compte gratuit
   - Créez un cluster M0 (gratuit)
   - Créez un utilisateur de base de données
   - Whitelist 0.0.0.0/0 (autoriser toutes les IPs)
   - Copiez la connection string
   - Format: `mongodb+srv://username:password@cluster.mongodb.net/finsmart`

### Étape 1.4 : Attendre le déploiement
- Render va installer les dépendances et démarrer l'app
- URL finale : `https://finsmart-backend.onrender.com`
- **IMPORTANT : Notez cette URL !**

⚠️ **Note** : Le plan gratuit s'endort après 15 minutes d'inactivité. Le premier appel peut prendre 30-60 secondes.

---

## 🎨 2. Déployer l'Admin sur Vercel

### Étape 2.1 : Créer un compte Vercel
1. Allez sur [vercel.com](https://vercel.com)
2. Cliquez **Sign Up** et connectez-vous avec GitHub

### Étape 2.2 : Importer le projet
1. Cliquez **Add New** → **Project**
2. Sélectionnez **finsmart-admin**
3. Configuration :
   - **Framework Preset** : Autre
   - **Build Command** : (laisser vide)
   - **Output Directory** : . (point)
4. Cliquez **Deploy**

### Étape 2.3 : Configurer l'API Backend
1. Après déploiement, allez dans **Settings** → **Environment Variables**
2. Ajoutez :
   - `API_URL` = `https://finsmart-backend.onrender.com/api/v1`
3. Redéployez depuis **Deployments** → **Redeploy**

### Résultat
- URL finale : `https://finsmart-admin.vercel.app`

---

## 📱 3. Déployer le Mobile Web sur Vercel

### Étape 3.1 : Importer le projet
1. Dans Vercel, cliquez **Add New** → **Project**
2. Sélectionnez **finsmart-mobile**
3. Configuration :
   - **Framework Preset** : Autre
   - **Build Command** : `npm run export`
   - **Output Directory** : `dist`
   - **Install Command** : `npm install`

### Étape 3.2 : Configurer les variables d'environnement
1. Allez dans **Settings** → **Environment Variables**
2. Ajoutez :
   - `EXPO_PUBLIC_API_URL` = `https://finsmart-backend.onrender.com/api/v1`
3. Redéployez

### Résultat
- URL finale : `https://finsmart-mobile.vercel.app`

---

## ✅ 4. Vérification Finale

### Tester le Backend
```bash
curl https://finsmart-backend.onrender.com/api/v1/health
```

### Tester l'Admin
Ouvrez : `https://finsmart-admin.vercel.app`

### Tester le Mobile Web
Ouvrez : `https://finsmart-mobile.vercel.app`

---

## 🔧 5. Mise à jour du code

Chaque fois que vous poussez du code sur GitHub, Vercel et Render redéploient automatiquement !

```bash
# Backend
cd finsmart-backend
git add .
git commit -m "Update backend"
git push origin main

# Admin
cd finsmart-admin
git add .
git commit -m "Update admin"
git push origin master

# Mobile
cd finsmart-mobile-new
git add .
git commit -m "Update mobile"
git push origin main
```

---

## 📊 Récapitulatif des URLs

| Service | URL |
|---------|-----|
| Backend API | `https://finsmart-backend.onrender.com/api/v1` |
| Admin Dashboard | `https://finsmart-admin.vercel.app` |
| Mobile Web App | `https://finsmart-mobile.vercel.app` |
| MongoDB | `mongodb+srv://...` |

---

## ⚠️ Points Importants

1. **Cold Start Render** : Première requête peut prendre 30-60s
2. **Variables d'environnement** : À configurer dans chaque plateforme
3. **CORS** : Déjà configuré pour accepter toutes les origines
4. **HTTPS** : Automatique sur Render et Vercel
5. **Déploiement automatique** : À chaque push sur GitHub

---

## 🆘 Dépannage

### Le backend ne démarre pas
- Vérifiez les logs sur Render
- Vérifiez que MONGODB_URI est correcte
- Vérifiez que tous les secrets sont configurés

### L'admin ne se connecte pas au backend
- Vérifiez l'URL de l'API dans les variables d'environnement
- Vérifiez les logs de la console du navigateur
- Testez l'API directement avec curl

### Le mobile web ne marche pas
- Vérifiez que `EXPO_PUBLIC_API_URL` est configurée
- Vérifiez les logs de build sur Vercel
- Testez l'export local avec `npm run export`

---

## 🎉 Félicitations !

Votre application FinSmart est maintenant déployée et accessible de partout ! 🌍
