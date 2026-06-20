---
layout: post
title: "WC-02 : Créer et gérer des produits WooCommerce"
date: 2026-06-20
categories: [wordpress, woocommerce, ecommerce]
tags: [woocommerce, produits, variations, téléchargeable, wp-cli]
---

## Objectif

Créer les trois types de produits principaux dans WooCommerce : simple, variable (avec attributs), et téléchargeable.

## Environnement

- **OS** : Red Hat Enterprise Linux 10
- **WordPress** : Multisite
- **WooCommerce** : 10.8.1
- **Thème** : Storefront 4.6.2

## Produits créés

### 1. Produit simple — T-Shirt Biroue Lab

| Champ | Valeur |
|-------|--------|
| Type | Simple product |
| Prix | 150 MAD |
| Catégorie | Vêtements |
| Stock | 25 unités |

Produit basique sans variation. Idéal pour des articles avec une seule configuration.

### 2. Produit variable — Hoodie Biroue Lab

| Champ | Valeur |
|-------|--------|
| Type | Variable product |
| Attribut | Taille : S, M, L, XL |
| Prix par variation | 200 MAD |
| Catégorie | Vêtements |

Utilisation des **attributs** pour générer des variations automatiques. WooCommerce crée une variation par combinaison d'attributs.

### 3. Produit téléchargeable — Guide Linux pour Débutants

| Champ | Valeur |
|-------|--------|
| Type | Simple product + Downloadable |
| Prix | 50 MAD |
| Catégorie | Ebooks |
| Limite de téléchargements | 3 |
| Expiration | 30 jours |

Le client reçoit un lien sécurisé après achat, limité en nombre et durée.

## Vérification via WP-CLI

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

## Problèmes rencontrés et solutions

### Conflit URL multisite
Le slug WooCommerce `/shop` entrait en conflit avec un sous-site multisite (`blog_id 3`) à l'URL `http://192.168.11.103/shop/`. Solution : renommer le slug de la page boutique en `/boutique/`.

```bash
sudo /usr/local/bin/wp post update 7 --post_name=boutique \
  --allow-root --path=/var/www/monsite
sudo /usr/local/bin/wp rewrite flush --allow-root --path=/var/www/monsite
```

### Mode Coming Soon WooCommerce
La boutique était inaccessible à cause du mode "Coming Soon" activé par WooCommerce par défaut.

```bash
sudo /usr/local/bin/wp option update woocommerce_coming_soon no \
  --allow-root --path=/var/www/monsite
```

## Concepts clés appris

| Concept | Explication |
|---------|-------------|
| Produit simple | Un seul SKU, un seul prix |
| Produit variable | Plusieurs variations via attributs (taille, couleur...) |
| Produit téléchargeable | Fichier numérique avec accès limité post-achat |
| Attributs WooCommerce | Propriétés qui génèrent des variations |
| Slug de page | L'identifiant URL d'une page WordPress |

## Résultat

La boutique est accessible sur `http://192.168.11.103/boutique/` avec 3 produits publiés et visibles.

**Prochaine étape** : WC-03 — Passer une commande de test et gérer le cycle de vie d'une commande.
