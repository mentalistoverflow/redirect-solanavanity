# 🔗 Netlify Redirect - Lien Fixe Sécurisé

## 🎯 Problème résolu

L'URL Cloudflare Tunnel change à chaque redémarrage (exemple : `https://abc-def-ghi.trycloudflare.com`), rendant impossible le partage d'un lien permanent.

## 💡 Solution : Lien fixe Netlify

Ce dossier contient une page de redirection qui sera déployée sur Netlify avec un **lien fixe** (exemple : `https://vanity-solana.netlify.app`).

Ce lien redirige automatiquement vers l'URL Cloudflare actuelle, peu importe combien de fois elle change.

## 🏗️ Architecture sécurisée

```
Utilisateur
    ↓
📍 Netlify (lien fixe)
    ↓
📄 GitHub (lit current-url.json)
    ↑
🔄 Serveur (met à jour le JSON via GitHub)
    ↓
🔒 Cloudflare Tunnel (masque l'IP)
```

**Point crucial :** Le serveur ne contacte **JAMAIS** Netlify directement. GitHub agit comme proxy sécurisé.

### Pourquoi c'est sécurisé ?

✅ **IP du serveur masquée** - Cloudflare Tunnel masque l'IP réelle  
✅ **Serveur → GitHub uniquement** - Le serveur ne contacte que l'API GitHub  
✅ **Netlify → GitHub** - Netlify fetch le JSON depuis GitHub (pas depuis le serveur)  
✅ **Aucun log compromettant** - L'IP du serveur n'apparaît nulle part sauf dans les logs GitHub (acceptable)  

## 📋 Installation rapide (5 étapes)

### Étape 1 : Créer le repo GitHub

1. Créez un repo GitHub **public** nommé `netlify-redirect`
2. Copiez tous les fichiers de ce dossier dans le repo
3. Commit et push :

```bash
cd netlify-redirect
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/VOTRE-USERNAME/netlify-redirect.git
git push -u origin main
```

### Étape 2 : Déployer sur Netlify

1. Créez un compte gratuit sur [netlify.com](https://netlify.com)
2. Cliquez sur **"New site from Git"**
3. Connectez votre compte GitHub
4. Sélectionnez le repo `netlify-redirect`
5. Configuration de build :
   - **Base directory:** (laisser vide)
   - **Build command:** (laisser vide)
   - **Publish directory:** `.`
6. Cliquez sur **"Deploy site"**
7. Notez votre URL Netlify (exemple : `https://random-name-123.netlify.app`)
8. (Optionnel) Personnalisez l'URL dans **Site settings → Domain management**

### Étape 3 : Créer le Personal Access Token GitHub

1. Allez sur [github.com/settings/tokens](https://github.com/settings/tokens)
2. Cliquez sur **"Generate new token (classic)"**
3. Nom du token : `Netlify Redirect Updater`
4. Scope requis : **`repo`** (Full control of private repositories)
5. Cliquez sur **"Generate token"**
6. **IMPORTANT :** Copiez le token immédiatement (il ne sera plus visible)

### Étape 4 : Configurer le serveur

Éditez le fichier `.env` du serveur et ajoutez :

```env
# GitHub/Netlify Redirect
GITHUB_UPDATE_ENABLED=true
GITHUB_REPO_OWNER=votre-username
GITHUB_REPO_NAME=netlify-redirect
GITHUB_ACCESS_TOKEN=ghp_votre_token_ici
NETLIFY_FIXED_URL=https://votre-site.netlify.app
```

### Étape 5 : Tester

1. Redémarrez le serveur : `npm start`
2. Le serveur va automatiquement mettre à jour GitHub
3. Netlify se redéploie automatiquement (1-2 minutes)
4. Visitez votre lien Netlify
5. Vous devez être redirigé vers l'URL Cloudflare actuelle

## 🚀 Utilisation quotidienne

### Lien à partager

**À partager :** Votre lien Netlify fixe (exemple : `https://vanity-solana.netlify.app`)  
**NE PAS partager :** L'URL Cloudflare (elle change à chaque redémarrage)

### Mises à jour automatiques

Le système se met à jour automatiquement :

1. Le serveur redémarre avec une nouvelle URL Cloudflare
2. Le script `update-env-url.ps1` détecte le changement
3. Le fichier `current-url.json` est mis à jour sur GitHub
4. Netlify détecte le commit et redéploie automatiquement
5. Le lien Netlify redirige vers la nouvelle URL

**Durée totale :** ~1-2 minutes

### QR Codes

Le lien Netlify est parfait pour les QR codes car il ne change jamais !

Générez un QR code avec votre lien Netlify et partagez-le sans crainte.

## 🔒 Sécurité garantie

### IP du serveur

**Question :** Est-ce que l'IP du serveur est exposée ?  
**Réponse :** Non, jamais. Voici pourquoi :

1. **Cloudflare Tunnel** masque l'IP du serveur
2. Le serveur contacte uniquement **GitHub** (API publique)
3. Netlify fetch depuis **GitHub** (pas depuis le serveur)
4. Les utilisateurs visitent **Netlify** puis sont redirigés vers **Cloudflare**

**Seule l'URL Cloudflare publique est visible.**

### Logs

- **Logs Netlify :** Ne voient que l'IP de GitHub (lors du déploiement)
- **Logs GitHub :** Voient l'IP du serveur (acceptable, GitHub est sécurisé)
- **Logs Cloudflare :** Ne voient que les IPs des visiteurs

### Token GitHub

- Stocké dans `.env` (jamais commité grâce à `.gitignore`)
- Scope minimal requis (`repo`)
- Peut être révoqué à tout moment
- Utilisé uniquement pour mettre à jour le JSON

## 🐛 Troubleshooting

### Erreur 404 sur le JSON

**Symptôme :** Page de redirection affiche "Impossible de récupérer l'URL"

**Causes possibles :**
- Le fichier `current-url.json` n'existe pas sur GitHub
- Netlify n'a pas encore déployé

**Solution :**
1. Vérifiez que le fichier existe sur GitHub
2. Allez sur Netlify → Site → Deploys
3. Vérifiez que le déploiement est terminé
4. Déclenchez un redéploiement manuel si nécessaire

### Redirection ne fonctionne pas

**Symptôme :** La page se charge mais ne redirige pas

**Causes possibles :**
- Le JSON contient une URL invalide
- JavaScript désactivé dans le navigateur

**Solution :**
1. Vérifiez le contenu de `current-url.json` sur GitHub
2. L'URL doit être valide et accessible
3. Testez l'URL directement dans le navigateur

### Token GitHub invalide

**Symptôme :** Erreur "Bad credentials" dans les logs du serveur

**Solution :**
1. Vérifiez que le token est correctement copié dans `.env`
2. Vérifiez que le scope `repo` est activé
3. Régénérez un nouveau token si nécessaire

### Mise à jour ne se déclenche pas

**Symptôme :** L'URL Cloudflare change mais Netlify n'est pas mis à jour

**Causes possibles :**
- `GITHUB_UPDATE_ENABLED=false` dans `.env`
- Token GitHub manquant ou invalide
- Erreur dans le script

**Solution :**
1. Vérifiez les logs du serveur au démarrage
2. Cherchez `✅ GitHub mis à jour` dans les logs
3. Vérifiez le dernier commit sur GitHub
4. Testez manuellement avec l'endpoint admin : `POST /api/admin/update-redirect`

## 📞 Besoin d'aide ?

Consultez le guide complet : `docs/NETLIFY_REDIRECT_GUIDE.md`

## 🎉 Récapitulatif

✅ Lien fixe qui ne change jamais  
✅ Redirection automatique vers Cloudflare  
✅ IP du serveur complètement masquée  
✅ Mises à jour automatiques via GitHub  
✅ Compatible avec QR codes  
✅ Gratuit (Netlify + GitHub)  
✅ HTTPS partout  

**Partagez votre lien Netlify en toute confiance !**
