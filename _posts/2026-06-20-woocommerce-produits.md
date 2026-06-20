---
layout: post
title: "WC-02 : Créer des produits WooCommerce — Journal complet avec troubleshooting"
date: 2026-06-20
categories: [wordpress, woocommerce, ecommerce]
tags: [woocommerce, produits, variations, téléchargeable, wp-cli, troubleshooting, multisite, nginx]
---

## Objectif

Créer les trois types de produits principaux WooCommerce (simple, variable, téléchargeable) et les rendre visibles sur la boutique publique.

## Environnement

- **OS** : Red Hat Enterprise Linux 10
- **WordPress** : Multisite (4 sous-sites)
- **WooCommerce** : 10.8.1
- **Thème actif** : Storefront 4.6.2
- **Chemin WordPress** : `/var/www/monsite`

---

## Produits créés

### 1. Produit simple — T-Shirt Biroue Lab

| Champ | Valeur |
|-------|--------|
| ID | 13 |
| Type | Simple product |
| Prix | 150 MAD |
| Catégorie | Vêtements |
| Stock | 25 unités (Manage stock activé) |

### 2. Produit variable — Hoodie Biroue Lab

| Champ | Valeur |
|-------|--------|
| ID | 14 |
| Type | Variable product |
| Attribut | Taille : S / M / L / XL |
| Prix par variation | 200 MAD |
| Catégorie | Vêtements |

Création des variations : **Attributes → Add → "Used for variations" → Save → Variations → Create from all attributes → Go**

WooCommerce génère 4 variations automatiquement (S, M, L, XL).

### 3. Produit téléchargeable — Guide Linux pour Débutants

| Champ | Valeur |
|-------|--------|
| ID | 19 |
| Type | Simple + Downloadable |
| Prix | 50 MAD |
| Catégorie | Ebooks |
| Fichier | guide-linux.pdf |
| Limite downloads | 3 |
| Expiration | 30 jours |

**Vérification WP-CLI :**
```bash
sudo /usr/local/bin/wp post list --post_type=product \
  --fields=ID,post_title,post_status \
  --allow-root --path=/var/www/monsite
```

```
+----+----------------------------+-------------+
| ID | post_title                 | post_status |
+----+----------------------------+-------------+
| 19 | Guide Linux pour Débutants | publish     |
| 14 | Hoodie Biroue Lab          | publish     |
| 13 | T-Shirt Biroue Lab         | publish     |
+----+----------------------------+-------------+
```

---

## Problèmes rencontrés — Journal complet

### Problème 1 : Produits invisibles sur `/shop/`

**Symptôme :** `http://192.168.11.103/shop/` affichait "Hello World" et des articles de blog au lieu des produits WooCommerce.

**Diagnostic 1 — Vérification de la page shop :**
```bash
sudo /usr/local/bin/wp option get woocommerce_shop_page_id \
  --allow-root --path=/var/www/monsite
# Résultat : 7
```

La page shop existe (ID 7).

**Diagnostic 2 — Flush des règles de réécriture :**
```bash
sudo /usr/local/bin/wp rewrite flush --allow-root --path=/var/www/monsite
# Résultat : Success: Rewrite rules flushed.
```

Pas de changement.

---

### Problème 2 : Nginx — server_name ne couvre pas l'IP

**Cause identifiée :** Le fichier Nginx `/etc/nginx/conf.d/monsite.conf` avait `server_name monsite.local;` sans inclure l'adresse IP.

```nginx
# Avant :
server_name monsite.local;

# Après correction :
server_name monsite.local 192.168.11.103;
```

```bash
sudo nginx -t && sudo systemctl reload nginx
```

Résultat : le bon vhost est maintenant servi via l'IP, mais les produits restent invisibles.

---

### Problème 3 : Visibilité catalogue des produits vide

**Diagnostic :**
```bash
sudo /usr/local/bin/wp post meta get 13 _visibility \
  --allow-root --path=/var/www/monsite
# Résultat : (vide)
```

**Correction :**
```bash
sudo /usr/local/bin/wp post meta update 13 _visibility visible \
  --allow-root --path=/var/www/monsite
sudo /usr/local/bin/wp post meta update 14 _visibility visible \
  --allow-root --path=/var/www/monsite
sudo /usr/local/bin/wp post meta update 19 _visibility visible \
  --allow-root --path=/var/www/monsite
sudo /usr/local/bin/wp cache flush --allow-root --path=/var/www/monsite
```

Pas de changement visible.

---

### Problème 4 : Mauvais thème — Twenty Twenty-Five (FSE)

**Diagnostic via curl :**
```bash
curl -sL http://192.168.11.103/ | grep -i "storefront\|twentytwentyfive\|theme"
```

Le thème actif était **Twenty Twenty-Five** (Full Site Editing), incompatible avec les templates WooCommerce.

**Correction — Installation de Storefront :**
```bash
sudo /usr/local/bin/wp theme install storefront --activate \
  --allow-root --path=/var/www/monsite
```

**Vérification :**
```bash
sudo /usr/local/bin/wp option get template --allow-root --path=/var/www/monsite
# storefront
sudo /usr/local/bin/wp option get stylesheet --allow-root --path=/var/www/monsite
# storefront
```

Malgré cela, le navigateur affichait encore l'ancien thème → cache navigateur. Hard refresh (Ctrl+Shift+R) testé sans succès.

---

### Problème 5 : Slug de la page shop incorrect

**Diagnostic :**
```bash
sudo /usr/local/bin/wp post list --post_type=page \
  --fields=ID,post_title,post_name,post_status \
  --allow-root --path=/var/www/monsite
```

```
+----+---------+----------+-------------+
| ID | title   | post_name| post_status |
+----+---------+----------+-------------+
| 7  | Shop    | shop5    | publish     |
...
```

La page shop avait le slug `shop5` au lieu de `shop`.

**Correction :**
```bash
sudo /usr/local/bin/wp post update 7 --post_name=shop \
  --allow-root --path=/var/www/monsite
sudo /usr/local/bin/wp rewrite flush --allow-root --path=/var/www/monsite
```

Toujours pas de produits visibles.

---

### Problème 6 (RACINE) : Conflit URL avec sous-site WordPress Multisite

**Analyse curl :**
```bash
curl -sL http://192.168.11.103/shop/ | grep "body class\|theme"
```

```html
<body class="home blog wp-embed-responsive wp-theme-twentytwentyfive">
```

```
/*# sourceURL=http://192.168.11.103/shop/wp-content/themes/twentytwentyfive/style.min.css */
```

Malgré Storefront actif en base de données, le navigateur chargeait **twentytwentyfive**. Les fichiers CSS pointaient vers `/shop/wp-content/...` — ce qui révèle qu'il existe une **installation WordPress distincte dans le répertoire `/shop/`**.

**Confirmation — liste des sites du réseau multisite :**
```bash
sudo /usr/local/bin/wp site list --allow-root --path=/var/www/monsite
```

```
+---------+-------------------------------+
| blog_id | url                           |
+---------+-------------------------------+
| 1       | http://192.168.11.103/        |
| 2       | http://192.168.11.103/myblog/ |
| 3       | http://192.168.11.103/shop/   |  ← conflit !
| 4       | http://192.168.11.103/dev/    |
+---------+-------------------------------+
```

**Cause racine :** Le sous-site multisite `blog_id 3` occupe l'URL `/shop/` et intercepte toutes les requêtes avant que WooCommerce ne puisse servir la page boutique du site principal.

**Solution — renommer le slug de la page boutique :**
```bash
sudo /usr/local/bin/wp post update 7 --post_name=boutique \
  --allow-root --path=/var/www/monsite
sudo /usr/local/bin/wp rewrite flush --allow-root --path=/var/www/monsite
```

---

### Problème 7 : Mode "Coming Soon" WooCommerce

En accédant à `http://192.168.11.103/boutique/`, la page affichait :

> **Pardon our dust! We're working on something amazing — check back soon!**

**Diagnostic :**
```bash
sudo /usr/local/bin/wp option list --search="*coming*" \
  --allow-root --path=/var/www/monsite
```

```
+-------------------------+--------------+
| option_name             | option_value |
+-------------------------+--------------+
| woocommerce_coming_soon | yes          |
+-------------------------+--------------+
```

WooCommerce active le mode Coming Soon automatiquement après installation.

**Correction :**
```bash
sudo /usr/local/bin/wp option update woocommerce_coming_soon no \
  --allow-root --path=/var/www/monsite
```

**Résultat :** La boutique est maintenant accessible à `http://192.168.11.103/boutique/` avec les 3 produits visibles.

---

## Résumé des problèmes et solutions

| # | Problème | Cause | Solution |
|---|----------|-------|----------|
| 1 | Produits invisibles | Règles de réécriture obsolètes | `wp rewrite flush` |
| 2 | Nginx répond mal à l'IP | `server_name` sans IP | Ajouter l'IP dans `server_name` |
| 3 | Visibilité produits vide | Meta `_visibility` non défini | `wp post meta update _visibility visible` |
| 4 | Mauvais thème (FSE) | Twenty Twenty-Five incompatible WooCommerce | Installer et activer Storefront |
| 5 | Slug shop incorrect | WooCommerce avait créé `shop5` | Renommer en `shop` puis `boutique` |
| 6 | Conflit URL multisite | Sous-site `blog_id 3` à `/shop/` | Renommer boutique en `/boutique/` |
| 7 | Page "Coming Soon" | Option `woocommerce_coming_soon = yes` | Mettre à `no` via WP-CLI |

---

## Concepts clés appris

| Concept | Explication |
|---------|-------------|
| Produit simple | Un SKU, un prix, stock optionnel |
| Produit variable | Variations via attributs (taille, couleur...) |
| Produit téléchargeable | Accès fichier post-achat, limité en nombre/durée |
| WordPress Multisite | Réseau de sites partageant une installation WordPress |
| Conflit de slug | Un slug de page peut entrer en conflit avec un sous-site multisite |
| `woocommerce_coming_soon` | Option activée par défaut bloquant l'accès public |
| Storefront | Thème officiel WooCommerce, compatible avec tous les templates |

## Résultat final

Boutique accessible sur `http://192.168.11.103/boutique/` avec 3 produits publiés.

**Prochaine étape** : WC-03 — Passer une commande de test et gérer le cycle de vie d'une commande.
