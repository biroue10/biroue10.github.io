---
layout: post
title: "WC-03 : Passer une commande test et gérer le cycle de vie — Journal complet"
date: 2026-06-20
categories: [wordpress, woocommerce, ecommerce]
tags: [woocommerce, commandes, stripe, hpos, wp-cli, troubleshooting]
---

## Objectif

Simuler un achat complet avec Stripe en mode test, gérer le cycle de vie d'une commande (Processing → Completed), et vérifier les données en base.

## Environnement

- **WooCommerce** : 10.8.1 avec HPOS activé
- **Paiement** : Stripe en mode test
- **Boutique** : `http://192.168.11.103/boutique/`

---

## Étape 1 — Ajouter un produit au panier

Sur `http://192.168.11.103/boutique/` → **Add to cart** sur le T-Shirt Biroue Lab (150 MAD).

Puis → **View cart** → `http://192.168.11.103/cart/`

---

## Étape 2 — Checkout

URL : `http://192.168.11.103/checkout/`

**Données de test utilisées :**

| Champ | Valeur |
|-------|--------|
| First name | Test |
| Last name | Client |
| Address | 123 Rue Mohammed V |
| City | Casablanca |
| Country | Morocco |
| Phone | 0600000000 |
| Email | test@example.com |

**Carte Stripe de test :**

| Champ | Valeur |
|-------|--------|
| Card number | `4242 4242 4242 4242` |
| Expiry | `12/29` |
| CVC | `123` |

> La carte `4242 4242 4242 4242` est la carte Stripe universelle pour simuler un paiement accepté en mode test.

Résultat : page **"Order received"** — commande créée avec succès.

---

## Étape 3 — Gestion dans l'admin

**WooCommerce → Orders** → commande en statut `Processing`.

Cycle de vie d'une commande WooCommerce :

```
Pending → Processing → Completed
                    ↘ Cancelled / Refunded / Failed
```

| Statut | Signification |
|--------|---------------|
| Pending | Commande créée, paiement non confirmé |
| Processing | Paiement confirmé, en cours de traitement |
| Completed | Commande expédiée/livrée |
| Cancelled | Annulée |
| Refunded | Remboursée |
| Failed | Paiement échoué |

**Action :** Changement manuel de `Processing` → `Completed` via le menu déroulant → **Update**.

---

## Étape 4 — Vérification en base de données

### Problème : WP-CLI ne reconnaît pas la commande WooCommerce

```bash
sudo /usr/local/bin/wp wc order list --user=1 --allow-root --path=/var/www/monsite
# Error: 'order' is not a registered subcommand of 'wc'

sudo /usr/local/bin/wp post list --post_type=shop_order \
  --fields=ID,post_status,post_date --allow-root --path=/var/www/monsite
# Résultat vide
```

**Cause :** WooCommerce 8+ utilise **HPOS (High-Performance Order Storage)** — les commandes sont stockées dans des tables dédiées (`wp_wc_orders`) et non plus dans `wp_posts`. WP-CLI n'a pas accès à ces tables via les commandes standard.

### Vérification directe MySQL

**Récupérer les credentials depuis wp-config.php :**
```bash
sudo grep -E "DB_NAME|DB_USER|DB_PASSWORD" /var/www/monsite/wp-config.php
```

```
DB_NAME     : monsite_db
DB_USER     : monsite_user
DB_PASSWORD : WordPress2025!
```

**Lister les tables de commandes :**
```bash
mysql -u monsite_user -p'WordPress2025!' \
  -e "SHOW TABLES LIKE '%order%';" monsite_db
```

```
+--------------------------------+
| Tables_in_monsite_db (%order%) |
+--------------------------------+
| wp_wc_order_addresses          |
| wp_wc_order_coupon_lookup      |
| wp_wc_order_operational_data   |
| wp_wc_order_product_lookup     |
| wp_wc_order_stats              |
| wp_wc_order_tax_lookup         |
| wp_wc_orders                   |
| wp_wc_orders_meta              |
| wp_woocommerce_order_itemmeta  |
| wp_woocommerce_order_items     |
+--------------------------------+
```

**Requête sur la table principale :**
```bash
mysql -u monsite_user -p'WordPress2025!' \
  -e "SELECT id, status, date_created_gmt, total_amount FROM monsite_db.wp_wc_orders;"
```

```
+----+--------------+---------------------+--------------+
| id | status       | date_created_gmt    | total_amount |
+----+--------------+---------------------+--------------+
| 21 | wc-completed | 2026-06-20 21:07:51 | 150.00000000 |
+----+--------------+---------------------+--------------+
```

---

## Concepts clés appris

| Concept | Explication |
|---------|-------------|
| Carte test Stripe | `4242 4242 4242 4242` simule un paiement accepté |
| Cycle de vie commande | Pending → Processing → Completed |
| HPOS | High-Performance Order Storage : tables MySQL dédiées depuis WooCommerce 8+ |
| `wp_wc_orders` | Table principale des commandes avec HPOS |
| `wc-completed` | Préfixe `wc-` ajouté par WooCommerce aux statuts en base |

## Résultat

- Commande ID 21 créée et complétée
- Statut `wc-completed` confirmé en base
- Cycle de vie complet maîtrisé

**Prochaine étape** : WC-04 — Coupons et promotions.
