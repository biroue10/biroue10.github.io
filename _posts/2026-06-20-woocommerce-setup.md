---
layout: post
title: "WC-01 : Installation et configuration de WooCommerce — Journal complet"
date: 2026-06-20
categories: [wordpress, woocommerce, ecommerce]
tags: [woocommerce, stripe, wordpress, ecommerce, wp-cli, troubleshooting]
---

## Objectif

Installer et configurer WooCommerce sur un serveur WordPress auto-hébergé (RHEL 10), configurer la devise, les modes de livraison, et les passerelles de paiement (hors ligne + Stripe).

## Environnement

- **OS** : Red Hat Enterprise Linux 10
- **Web Server** : Nginx
- **PHP** : PHP-FPM
- **Base de données** : MariaDB
- **WordPress** : Multisite, installé via WP-CLI
- **Chemin** : `/var/www/monsite`

---

## Étape 1 — Installation de WooCommerce

```bash
sudo /usr/local/bin/wp plugin install woocommerce --activate \
  --allow-root --path=/var/www/monsite
```

### Problème rencontré : `wp` introuvable avec sudo

Au départ, `sudo wp` échouait car `/usr/local/bin` n'était pas dans le `secure_path` de sudo.

**Erreur :**
```
sudo: wp: command not found
```

**Solution :** Utiliser le chemin absolu et ajouter `/usr/local/bin` à `secure_path` via `visudo`.

```bash
sudo visudo
# Modifier la ligne secure_path :
Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin
```

---

## Étape 2 — Setup Wizard WooCommerce

Via l'interface admin WordPress → **WooCommerce → Setup Wizard** :

| Paramètre | Valeur |
|-----------|--------|
| Localisation | Maroc |
| Devise | MAD (Dirham marocain) |
| Unité de poids | kg |
| Unité de dimension | cm |

### Configuration de la livraison

- Mode de géolocalisation : **Géolocaliser (recommandé)**
- Zone de livraison : **Ship to all countries**
- Mode : Ship to billing address by default

---

## Étape 3 — Paiements hors ligne

Dans **WooCommerce → Settings → Payments** :

- **Virement bancaire (BACS)** : activé
- **Paiement à la livraison (COD)** : activé

---

## Étape 4 — Configuration Stripe

### Installation du plugin Stripe

```bash
sudo /usr/local/bin/wp plugin install woocommerce-gateway-stripe \
  --activate --allow-root --path=/var/www/monsite
```

### Connexion via OAuth (Stripe Connect)

Au lieu d'entrer manuellement les clés API, Stripe for WooCommerce utilise OAuth.
Lors du setup, l'account Stripe a été lié directement.

**Résultat dans WooCommerce → Payments → Stripe :**

```
Account status
Test Mode
WooCommerce Inc.
acct_1TjT2S8tRhMGRdiA

Payment  : Disabled
Payout   : Disabled
Webhook  : Enabled
Sync     : Enabled
```

**Mode Test activé** — toutes les transactions sont simulées.

### Avertissements normaux en environnement lab

| Avertissement | Cause | Action requise en prod |
|---------------|-------|------------------------|
| No SSL certificate | HTTP local, pas HTTPS | Certificat SSL valide |
| Apple Pay domain registration failed | Requiert domaine HTTPS public | Configurer en prod |
| Geolocation not configured | MaxMind requiert licence payante | Acheter licence MaxMind |

Ces avertissements sont **attendus en lab local** et n'empêchent pas le fonctionnement.

---

## Désactiver le mode "Coming Soon"

WooCommerce active le mode "Coming Soon" par défaut. La boutique était inaccessible.

**Diagnostic :**
```bash
sudo /usr/local/bin/wp option list --search="*coming*" \
  --allow-root --path=/var/www/monsite
```

**Résultat :**
```
+-------------------------+--------------+
| option_name             | option_value |
+-------------------------+--------------+
| woocommerce_coming_soon | yes          |
+-------------------------+--------------+
```

**Correction :**
```bash
sudo /usr/local/bin/wp option update woocommerce_coming_soon no \
  --allow-root --path=/var/www/monsite
```

---

## Résultat final WC-01

WooCommerce opérationnel avec :
- Devise MAD configurée
- Livraison mondiale activée
- Paiement hors ligne (virement + COD) actif
- Stripe en mode test connecté (acct_1TjT2S8tRhMGRdiA)

**Plugins actifs liés à WooCommerce :**
```
woocommerce               active  10.8.1
woocommerce-gateway-stripe active  10.8.2
```
