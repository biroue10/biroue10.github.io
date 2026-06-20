---
layout: post
title: "WC-04 : Coupons et promotions WooCommerce — Réduction fixe, pourcentage, livraison gratuite"
date: 2026-06-21
categories: [wordpress, woocommerce, ecommerce]
tags: [woocommerce, coupons, promotions, wp-cli]
---

## Objectif

Créer et tester les trois types de coupons WooCommerce : réduction fixe, pourcentage, et livraison gratuite.

## Environnement

- **WooCommerce** : 10.8.1
- **Boutique** : `http://192.168.11.103/boutique/`

---

## Coupons créés

### 1. Réduction fixe — BIROUE20

| Champ | Valeur |
|-------|--------|
| Code | `BIROUE20` |
| Type | Fixed cart discount |
| Montant | 20 MAD |
| Expiration | 2026-12-31 |

Réduit le total du panier de 20 MAD, peu importe les produits.

---

### 2. Pourcentage — BIROUE10

| Champ | Valeur |
|-------|--------|
| Code | `BIROUE10` |
| Type | Percentage discount |
| Montant | 10% |
| Expiration | 2026-12-31 |

Réduit le total du panier de 10%.

---

### 3. Livraison gratuite — FREESHIP

| Champ | Valeur |
|-------|--------|
| Code | `FREESHIP` |
| Type | Fixed cart discount |
| Montant | 0 |
| Allow free shipping | ✅ (onglet Shipping) |

Active la livraison gratuite lors de l'application du coupon.

---

## Test

1. `http://192.168.11.103/boutique/` → Ajouter un produit au panier
2. `http://192.168.11.103/cart/` → Appliquer le coupon `BIROUE20`
3. Résultat : le prix a diminué de 20 MAD

---

## Vérification WP-CLI

```bash
sudo /usr/local/bin/wp post list --post_type=shop_coupon \
  --fields=ID,post_title,post_status \
  --allow-root --path=/var/www/monsite
```

```
+----+------------+-------------+
| ID | post_title | post_status |
+----+------------+-------------+
| 24 | FREESHIP   | publish     |
| 23 | BIROUE10   | publish     |
| 22 | BIROUE20   | publish     |
+----+------------+-------------+
```

---

## Concepts clés appris

| Concept | Explication |
|---------|-------------|
| Fixed cart discount | Réduit le total du panier d'un montant fixe |
| Percentage discount | Réduit le total d'un pourcentage |
| Free shipping coupon | Active la livraison gratuite via l'onglet Shipping du coupon |
| `shop_coupon` | Type de post WordPress utilisé par WooCommerce pour les coupons |

## Résultat

3 coupons créés et fonctionnels. Le coupon `BIROUE20` testé en conditions réelles — réduction appliquée correctement au panier.

**Prochaine étape** : WC-05 — Rapports et analytiques WooCommerce.
