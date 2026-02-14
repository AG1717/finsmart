# ⚡ Configuration MongoDB Atlas - 5 minutes

## Étape 1 : Créer le compte (1 min)

1. La page https://www.mongodb.com/cloud/atlas/register vient de s'ouvrir
2. **Inscrivez-vous** avec votre email ou Google
3. Cliquez sur **"Create an account"**

---

## Étape 2 : Créer le cluster GRATUIT (2 min)

Après connexion :

1. Cliquez sur **"Build a Database"** (ou "Create")
2. Choisissez **"M0 FREE"** (la version gratuite)
   - ✅ 512 MB de stockage
   - ✅ Parfait pour démarrer
3. **Région** : Choisissez la plus proche de vous :
   - Europe : **Frankfurt (eu-central-1)**
   - Amérique : **US East (us-east-1)**
4. **Nom du cluster** : Laissez "Cluster0" (ou changez si vous voulez)
5. Cliquez sur **"Create Cluster"**
6. ⏱️ **Attendez 1-3 minutes** pendant la création

---

## Étape 3 : Créer un utilisateur de base de données (30 sec)

Une fenêtre "Security Quickstart" s'ouvre :

1. **Username** : Tapez `finsmart`
2. **Password** : Cliquez sur **"Autogenerate Secure Password"**
3. **⚠️ IMPORTANT** : Cliquez sur **"Copy"** pour copier le mot de passe !
4. **Collez ce mot de passe quelque part** (Notepad) - VOUS EN AUREZ BESOIN !
5. Cliquez sur **"Create User"**

---

## Étape 4 : Autoriser les connexions (30 sec)

Toujours dans "Security Quickstart" :

1. **Network Access** :
2. Cliquez sur **"Add My Current IP Address"**
3. **OU** pour permettre toutes les connexions Vercel :
   - Cliquez sur **"Allow Access from Anywhere"**
   - IP : `0.0.0.0/0`
4. Cliquez sur **"Finish and Close"**

---

## Étape 5 : Obtenir la connection string (1 min)

1. Sur la page principale, cliquez sur **"Connect"** (à côté de Cluster0)
2. Choisissez **"Connect your application"**
3. **Driver** : Node.js
4. **Version** : 5.5 or later
5. **Copiez** la connection string (ressemble à) :

```
mongodb+srv://finsmart:<password>@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0
```

6. **⚠️ IMPORTANT** : Remplacez `<password>` par le mot de passe que vous avez copié à l'étape 3 !

**Exemple :**
```
Si votre password est : abc123XYZ
Connection string devient :
mongodb+srv://finsmart:abc123XYZ@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0
```

---

## Étape 6 : Ajouter la connection string dans Vercel (1 min)

### Option A : Via le Dashboard Vercel (Recommandé)

1. Allez sur : https://vercel.com/aboubacars-projects-abbddc25/finsmart-backend/settings/environment-variables

2. Cliquez sur **"Add New"**

3. **Key** : `MONGODB_URI`
4. **Value** : Collez votre connection string (avec le vrai password !)
5. **Environments** : Cochez **Production**, **Preview**, **Development**
6. Cliquez sur **"Save"**

### Option B : Via la ligne de commande

Ouvrez un terminal et exécutez :

```bash
cd c:\Users\aboub\finsmart\finsmart-backend

# Remplacez VOTRE_CONNECTION_STRING par la vraie valeur
vercel env add MONGODB_URI

# Vercel vous demandera la valeur, collez votre connection string
# Puis choisissez : Production, Preview, Development (toutes)
```

---

## Étape 7 : Redéployer le backend (30 sec)

```bash
cd c:\Users\aboub\finsmart\finsmart-backend
vercel --prod
```

Attendez 30 secondes, puis testez :
```
https://finsmart-backend-chi.vercel.app/api/v1
```

Devrait retourner :
```json
{
  "message": "FinSmart API is running",
  "version": "1.0.0"
}
```

---

## ✅ C'est fait !

Maintenant vos applications vont fonctionner :

- ✅ **Admin** : https://finsmart-admin.vercel.app
- ✅ **Mobile** : https://finsmart-mobile-new.vercel.app
- ✅ **Backend** : https://finsmart-backend-chi.vercel.app

---

## 🔐 Créer un compte admin

Une fois MongoDB configuré, créez votre premier admin :

1. Allez sur : https://finsmart-admin.vercel.app
2. Cliquez sur "Sign up" (ou utilisez l'API directement)

**OU via l'API :**

```bash
curl -X POST https://finsmart-backend-chi.vercel.app/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin",
    "email": "admin@finsmart.com",
    "password": "VotreMotDePasseSecurise123!",
    "firstName": "Admin",
    "lastName": "FinSmart"
  }'
```

**Puis donnez-lui le rôle admin dans MongoDB :**
- Allez dans Atlas → Browse Collections
- Database : `finsmart` → Collection : `users`
- Trouvez votre utilisateur
- Changez `role: "user"` en `role: "admin"`

---

## 🐛 Dépannage

### ❌ "Authentication failed"
- Vérifiez que le mot de passe dans la connection string est correct
- Pas d'espaces avant/après le mot de passe

### ❌ "IP not whitelisted"
- Retournez dans Network Access
- Ajoutez `0.0.0.0/0` pour autoriser toutes les IPs

### ❌ Backend toujours en erreur
- Vérifiez que MONGODB_URI est bien ajoutée dans Vercel
- Redéployez : `vercel --prod`
- Vérifiez les logs : `vercel logs https://finsmart-backend-chi.vercel.app`

---

**✨ Une fois configuré, tout fonctionnera automatiquement ! ✨**
