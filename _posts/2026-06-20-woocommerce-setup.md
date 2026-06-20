---
layout: post
title: "WC-01 : Installation et configuration de WooCommerce"
date: 2026-06-20
categories: [wordpress, woocommerce, ecommerce]
tags: [woocommerce, stripe, wordpress, ecommerce, wp-cli]
---

## Objectif

Installer et configurer WooCommerce sur un serveur WordPress auto-hébergé (RHEL 10), en configurant la devise, les modes de livraison, et les passerelles de paiement (hors ligne + Stripe).

## Environnement

- **OS** : Red Hat Enterprise Linux 10
- **Web Server** : Nginx
- **PHP** : PHP-FPM
- **Base de données** : MariaDB
- **WordPress** : installé via WP-CLI

## Étapes réalisées

### 1. Installation de WooCommerce via WP-CLI

```bash
sudo /usr/local/bin/wp plugin install woocommerce --activate --allow-root --path=/var/www/html/wordpress
```

### 2. Configuration de base (Setup Wizard)

Via l'interface admin WordPress → WooCommerce → Setup Wizard :

- **Localisation** : Maroc
- **Devise** : MAD (Dirham marocain)
- **Unité de poids** : kg
- **Unité de dimension** : cm

### 3. Configuration de la livraison

- Mode de géolocalisation : **Géolocaliser (recommandé)**
- Zone de livraison : **Tous les pays**
- Mode : Ship to billing address by default

### 4. Passerelles de paiement

#### Paiement hors ligne
- **Virement bancaire** : activé
- **Paiement à la livraison (COD)** : activé

#### Stripe (paiement par carte)
- Plugin Stripe for WooCommerce installé
- Connexion via OAuth (Stripe Connect)
- **Mode Test activé** : toutes les transactions sont simulées
- Compte Stripe lié : `acct_1TjT2S8tRhMGRdiA`
- Webhook configuré et synchronisé

### 5. Sécurité et avertissements

Dans un environnement de lab local (sans SSL), des avertissements normaux apparaissent :
- Pas de certificat SSL → attendu en HTTP local
- Apple Pay désactivé → nécessite un domaine HTTPS public
- Géolocalisation MaxMind → requiert une licence payante

Ces éléments sont à configurer en production.

## Commandes WP-CLI utiles

```bash
# Lister les plugins actifs
sudo /usr/local/bin/wp plugin list --allow-root --path=/var/www/html/wordpress

# Vérifier le statut de WooCommerce
sudo /usr/local/bin/wp wc --allow-root --path=/var/www/html/wordpress

# Lister les produits
sudo /usr/local/bin/wp wc product list --allow-root --path=/var/www/html/wordpress
```

## Concepts clés appris

| Concept | Explication |
|---------|-------------|
| WooCommerce | Plugin e-commerce open source pour WordPress |
| Stripe Connect | OAuth permettant de lier un compte Stripe sans exposer les clés API |
| Mode Test Stripe | Simule les transactions sans argent réel |
| COD (Cash on Delivery) | Paiement à la livraison |
| MAD | Code ISO 4217 du Dirham marocain |

## Résultat

WooCommerce est opérationnel avec :
- Devise MAD configurée
- Livraison mondiale activée
- Paiement hors ligne (virement + COD) actif
- Stripe en mode test connecté et prêt

**Prochaine étape** : WC-02 — Création et gestion de produits WooCommerce.
