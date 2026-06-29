# SAH Cloud — Site Web

## Présentation du projet

Site institutionnel et commercial de **SAH Cloud** (marque de SAH Analytics), présentant l'offre d'infrastructure cloud hyper-convergée à destination des entreprises en Côte d'Ivoire et en Afrique de l'Ouest.

Le site est entièrement statique (HTML/CSS/JS), sans framework back-end ni build tool.
La traduction FR/EN est gérée côté client via un objet `T` embarqué dans chaque page.

---

## Structure des dossiers

```
sahcloud/
├── index.html                  # Page d'accueil
├── 404.html                    # Page d'erreur personnalisée
├── README.md                   # Ce fichier
│
├── about/
│   └── index.html              # À propos — mission & valeurs
├── contact/
│   └── index.html              # Formulaire de contact
├── features/
│   └── index.html              # Fonctionnalités & SLA
├── managed/
│   └── index.html              # Services Gérés (infogérance cloud)
├── pricing/
│   └── index.html              # Configurateur de prix interactif
├── questionnaire/
│   └── index.html              # Questionnaire Prospect (outil interne commercial)
├── services/
│   └── index.html              # Catalogue de services cloud
├── solutions/
│   └── index.html              # Solutions cloud (tabs : serveur, stockage, sécu, pourquoi)
│
└── img/
    └── logo.png                # Logo SAH Cloud (388×162 px, ~49 KB)
```

---

## Dépendances externes

Toutes les dépendances sont chargées depuis des CDN — aucun fichier npm, aucun bundler.

| Bibliothèque | Version | URL CDN | Utilisé sur |
|---|---|---|---|
| **Tailwind CSS** | latest (CDN) | `https://cdn.tailwindcss.com` | Toutes les pages sauf questionnaire |
| **Google Fonts — Inter** | — | `https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&display=swap` | Toutes les pages |
| **Bootstrap Icons** | 1.11.3 | `https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css` | questionnaire/index.html uniquement |
| **SheetJS (xlsx)** | 0.18.5 | `https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js` | questionnaire/index.html uniquement |

---

## Intégration dans le domaine SAH Analytics

### Sous-domaine recommandé

```
cloud.sahanalytics.com
```

### Configuration Apache (Virtual Host)

```apache
<VirtualHost *:80>
    ServerName cloud.sahanalytics.com
    DocumentRoot /var/www/sahcloud
    DirectoryIndex index.html

    # Page 404 personnalisée
    ErrorDocument 404 /404.html

    <Directory /var/www/sahcloud>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    # Redirection HTTPS
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</VirtualHost>

<VirtualHost *:443>
    ServerName cloud.sahanalytics.com
    DocumentRoot /var/www/sahcloud
    DirectoryIndex index.html

    ErrorDocument 404 /404.html

    SSLEngine on
    SSLCertificateFile      /etc/letsencrypt/live/cloud.sahanalytics.com/fullchain.pem
    SSLCertificateKeyFile   /etc/letsencrypt/live/cloud.sahanalytics.com/privkey.pem

    <Directory /var/www/sahcloud>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    # Cache des assets statiques
    <FilesMatch "\.(png|jpg|jpeg|gif|ico|css|js|woff2?)$">
        Header set Cache-Control "max-age=2592000, public"
    </FilesMatch>
</VirtualHost>
```

### Configuration Nginx

```nginx
server {
    listen 80;
    server_name cloud.sahanalytics.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name cloud.sahanalytics.com;

    root /var/www/sahcloud;
    index index.html;

    ssl_certificate     /etc/letsencrypt/live/cloud.sahanalytics.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/cloud.sahanalytics.com/privkey.pem;

    # Page 404 personnalisée
    error_page 404 /404.html;
    location = /404.html { internal; }

    location / {
        try_files $uri $uri/ $uri.html =404;
    }

    # Cache assets
    location ~* \.(png|jpg|jpeg|gif|ico|css|js|woff2?)$ {
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    # Sécurité
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header Referrer-Policy "strict-origin-when-cross-origin";
}
```

### Instructions de déploiement

1. Copier le dossier `sahcloud/` dans `/var/www/sahcloud/`
2. Ajuster les permissions : `chown -R www-data:www-data /var/www/sahcloud`
3. Activer le virtual host et recharger Apache/Nginx
4. Pointer l'enregistrement DNS `cloud.sahanalytics.com` → IP du serveur
5. Générer le certificat SSL : `certbot --nginx -d cloud.sahanalytics.com`

---

## URLs à mettre à jour avant mise en production

Aucun `href` ou `src` absolu de développement n'a été trouvé dans le code.
Tous les liens internes sont **relatifs** (`./`, `../`, noms de fichiers directs).

**Liens internes vérifiés — tous résolus :**

| Page source | Cible | Statut |
|---|---|---|
| Toutes | `img/logo.png` ou `../img/logo.png` | ✅ Fichier présent |
| Toutes | `index.html`, `about/index.html`, `contact/index.html`, `features/index.html`, `managed/index.html`, `pricing/index.html`, `services/index.html`, `solutions/index.html`, `questionnaire/index.html` | ✅ Tous présents |
| `contact/index.html` | `#` (lien Politique de confidentialité) | ⚠️ Placeholder — à remplacer par la vraie URL |

**CDN à valider en production :**
- Tailwind CDN (`cdn.tailwindcss.com`) : version non épinglée — pour la production, envisager une version fixe ou un build local.
- Google Fonts : vérifier la disponibilité depuis la Côte d'Ivoire si latence observée.

---

## Fonctionnalités

### Pages et leur rôle

| URL | Rôle |
|---|---|
| `/` | Accueil — hero, stats, barre de confiance, navigation rapide |
| `/services/` | Catalogue de services cloud (VPS, IA, Analytics, Stockage, Sécurité, BDD) |
| `/solutions/` | Solutions techniques en tabs (Serveur, Stockage, Sécurité, Pourquoi SAH Cloud) |
| `/managed/` | Services Gérés — infogérance, supervision 24/7, DRM |
| `/features/` | Fonctionnalités & SLA (haute dispo, scalabilité, sécurité, monitoring) |
| `/pricing/` | Configurateur de prix interactif (VMs, stockage, backup, réseau, NFV, sécurité) |
| `/about/` | À propos — mission, valeurs, statistiques |
| `/contact/` | Formulaire de contact (nom, email, service concerné, message) |
| `/questionnaire/` | **Outil interne** — questionnaire prospect en 7 sections, export Excel `.xlsx` |
| `/404.html` | Page d'erreur 404 personnalisée avec liens de retour |

### Système i18n FR/EN

Chaque page embarque son propre objet de traduction `T = { fr: {...}, en: {...} }`.
Le basculement de langue est déclenché par le bouton `.lang-btn` et persisté via `localStorage['sah-lang']`.

Tous les éléments traduisibles portent l'attribut `data-i18n="clé"`.
Les pages contact et questionnaire utilisent également `data-i18n-placeholder` / `data-i18n-ph` pour les champs de formulaire.

**Mega menu « Cloud & Infrastructure »** : 3 colonnes (Produits / Solutions / Services), présent sur les 9 pages avec i18n complet FR/EN.

### Export Excel du questionnaire

La page `questionnaire/index.html` génère un fichier `.xlsx` en local, sans aucun appel serveur, grâce à SheetJS v0.18.5.

Les 7 sections collectées :
1. Informations Entreprise
2. Contact Principal
3. Infrastructure Actuelle
4. Produits d'Intérêt (12 produits SAH Cloud)
5. Budget Estimé
6. Calendrier de Déploiement
7. Notes & Contexte

---

## Contact & support

**SAH Analytics**
- Email : info@sahanalytics.com
- Téléphone : +225 25 22 01 97 40
- Site web : https://sahanalytics.com
