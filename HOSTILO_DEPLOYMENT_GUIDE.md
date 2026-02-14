# Guide de Deploiement FinSmart sur Hostilo

## Etape 1: Preparer les fichiers Backend

### 1.1 Creer le fichier ZIP du backend

Sur votre ordinateur, ouvrez un terminal dans le dossier `finsmart-backend` et executez:

```bash
# Windows (PowerShell)
cd c:\Users\aboub\finsmart\finsmart-backend
Compress-Archive -Path * -DestinationPath ..\finsmart-backend.zip
```

Ou simplement:
- Selectionnez TOUS les fichiers dans `finsmart-backend` (sauf node_modules)
- Clic droit > Envoyer vers > Dossier compresse

### 1.2 Fichiers a inclure dans le ZIP:
```
finsmart-backend/
├── src/
├── package.json
├── package-lock.json
├── .env.production (renommer en .env apres upload)
├── render.yaml (optionnel)
└── README.md
```

**IMPORTANT:** Ne PAS inclure:
- `node_modules/` (sera installe sur le serveur)
- `.env` (contient les secrets locaux)

---

## Etape 2: Configurer l'Application Node.js sur Hostilo

### 2.1 Corriger l'App URI

Dans votre cPanel Hostilo, l'App URI actuel est incorrect:
- **Actuel:** `sbsylla.com/api.finsmart.com` ❌
- **Correct:** Devrait etre un sous-domaine ou un chemin

**Option A - Sous-domaine (Recommande):**
1. Allez dans "Subdomains" dans cPanel
2. Creez un sous-domaine: `api.finsmart.hostsilo.com`
3. Dans Node.js, configurez l'App URI vers ce sous-domaine

**Option B - Chemin sur domaine principal:**
- App URI: `finsmart.hostsilo.com/api`

### 2.2 Configuration Node.js correcte

```
Application Root: /home/sbsy38115902/finsmart-backend
Application URL: https://api.finsmart.hostsilo.com (ou votre choix)
Application Startup File: src/server.js
Node.js Version: 18.x ou superieur
```

---

## Etape 3: Uploader les fichiers

### 3.1 Via File Manager cPanel

1. Connectez-vous a cPanel Hostilo
2. Ouvrez "File Manager"
3. Naviguez vers `/home/sbsy38115902/finsmart-backend`
4. Cliquez sur "Upload"
5. Uploadez le fichier `finsmart-backend.zip`
6. Selectionnez le fichier ZIP > clic droit > "Extract"
7. Supprimez le fichier ZIP apres extraction

### 3.2 Verifier la structure

Apres extraction, la structure doit etre:
```
/home/sbsy38115902/finsmart-backend/
├── src/
│   ├── server.js
│   ├── app.js
│   ├── config/
│   ├── controllers/
│   ├── middlewares/
│   ├── models/
│   ├── routes/
│   └── utils/
├── package.json
├── package-lock.json
└── .env
```

---

## Etape 4: Configurer les Variables d'Environnement

### 4.1 Creer/Modifier le fichier .env

Dans File Manager, creez un fichier `.env` dans `/home/sbsy38115902/finsmart-backend/` avec:

```env
# Server Configuration
NODE_ENV=production
PORT=3000

# MongoDB Atlas - REMPLACEZ PAR VOS VRAIES VALEURS
MONGODB_URI=mongodb+srv://VOTRE_USER:VOTRE_PASSWORD@cluster0.xxxxx.mongodb.net/finsmart?retryWrites=true&w=majority

# JWT Secrets - REMPLACEZ PAR VOS SECRETS UNIQUES
JWT_SECRET=votre-secret-jwt-tres-long-et-securise-minimum-32-caracteres
JWT_REFRESH_SECRET=votre-secret-refresh-tres-long-et-securise-minimum-32-caracteres

JWT_EXPIRES_IN=1h
JWT_REFRESH_EXPIRES_IN=7d

# CORS - Domaines autorises
ALLOWED_ORIGINS=https://finsmart.hostsilo.com,https://app.finsmart.hostsilo.com

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100

# Logging
LOG_LEVEL=info
```

### 4.2 IMPORTANT: MongoDB Atlas

Vous devez avoir une base de donnees MongoDB Atlas:
1. Allez sur https://cloud.mongodb.com
2. Creez un compte gratuit (si pas deja fait)
3. Creez un cluster gratuit (M0)
4. Creez un utilisateur de base de donnees
5. Ajoutez votre IP (ou 0.0.0.0/0 pour permettre toutes les IPs)
6. Copiez la connection string et remplacez dans `.env`

---

## Etape 5: Installer les Dependencies

### 5.1 Via Terminal SSH ou cPanel

Dans la section Node.js de cPanel:
1. Cliquez sur le bouton "Run NPM Install" (ou similaire)

Ou via SSH:
```bash
cd /home/sbsy38115902/finsmart-backend
npm install
```

---

## Etape 6: Demarrer l'Application

### 6.1 Dans cPanel Node.js

1. Verifiez que "Application Startup File" est `src/server.js`
2. Cliquez sur "Restart" pour redemarrer l'application
3. Verifiez le statut: doit afficher "started"

### 6.2 Tester l'API

Ouvrez votre navigateur et testez:
```
https://api.finsmart.hostsilo.com/api/v1/health
```

Reponse attendue:
```json
{
  "success": true,
  "message": "FinSmart API is running",
  "data": {
    "status": "healthy",
    "timestamp": "2026-01-30T..."
  }
}
```

---

## Etape 7: Deployer le Frontend Mobile (Web)

### 7.1 Preparer le build

Le frontend est deja build dans:
```
c:\Users\aboub\finsmart\finsmart-mobile-new\dist\
```

### 7.2 Uploader le frontend

1. Dans File Manager, naviguez vers `/home/sbsy38115902/public_html/` (ou le dossier de votre domaine finsmart.hostsilo.com)
2. Uploadez TOUS les fichiers du dossier `dist/`:
   - `index.html`
   - `assets/` (dossier complet)

### 7.3 Configurer le domaine

Assurez-vous que `finsmart.hostsilo.com` pointe vers le dossier contenant `index.html`

---

## Etape 8: Mettre a jour l'URL API dans le Frontend

### 8.1 Verifier l'URL API

Le fichier `App.tsx` doit pointer vers votre backend:

```typescript
// Si votre backend est sur api.finsmart.hostsilo.com:
const API_URL = 'https://api.finsmart.hostsilo.com/api/v1';

// Ou si le backend est sur le meme domaine avec /api:
const API_URL = 'https://finsmart.hostsilo.com/api/v1';
```

### 8.2 Rebuilder si necessaire

Si vous changez l'URL:
```bash
cd c:\Users\aboub\finsmart\finsmart-mobile-new
npx expo export --platform web
```

Puis re-uploadez le dossier `dist/`

---

## Verification Finale

### Checklist

- [ ] Backend uploade dans `/home/sbsy38115902/finsmart-backend/`
- [ ] Fichier `.env` configure avec MongoDB Atlas
- [ ] `npm install` execute
- [ ] Application Node.js demarree (status: started)
- [ ] API accessible: `https://[votre-domaine]/api/v1/health`
- [ ] Frontend uploade dans `public_html`
- [ ] Frontend accessible: `https://finsmart.hostsilo.com`
- [ ] Login/Register fonctionne

### En cas de probleme

1. **Verifiez les logs** dans cPanel > Node.js > Logs
2. **Verifiez la connection MongoDB** - assurez-vous que l'IP du serveur est whitelistee
3. **Verifiez les CORS** - le domaine frontend doit etre dans ALLOWED_ORIGINS

---

## URLs Finales

- **Frontend:** https://finsmart.hostsilo.com
- **Backend API:** https://api.finsmart.hostsilo.com/api/v1 (ou le domaine que vous choisissez)
- **Health Check:** https://api.finsmart.hostsilo.com/api/v1/health

---

## Support

Si vous rencontrez des problemes:
1. Verifiez les logs Node.js dans cPanel
2. Testez l'API avec un outil comme Postman
3. Verifiez que MongoDB Atlas accepte les connections
