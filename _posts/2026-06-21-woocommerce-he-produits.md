---
layout: post
title: "HE-WC-03 : Produits & Catalogue WooCommerce — Journal Happiness Engineer"
date: 2026-06-21
categories: [wordpress, woocommerce, happiness-engineer]
tags: [woocommerce, produits, catalogue, thumbnails, visibilité, happiness-engineer]
---

## Objectif

Pratiquer les scénarios de support produits et catalogue les plus fréquents rencontrés par un Happiness Engineer chez Automattic.

---

## Scénario — Produit disparu de la boutique

### Ticket client type

> "Mon produit n'apparaît plus dans la boutique, il était visible avant."

### Causes principales

| Cause | Vérification | Solution |
|-------|-------------|----------|
| `_visibility` vide ou `hidden` | `wp post meta get [ID] _visibility` | `wp post meta update [ID] _visibility visible` |
| Stock à 0 + "Hide out of stock" activé | Products → Inventory | Désactiver "Hide out of stock" ou réapprovisionner |
| Produit en statut "Draft" | `wp post list --post_type=product` | Publier le produit |

### Commandes de diagnostic

```bash
# Vérifier la visibilité d'un produit
sudo /usr/local/bin/wp post meta get 13 _visibility \
  --allow-root --path=/var/www/monsite

# Corriger si vide
sudo /usr/local/bin/wp post meta update 13 _visibility visible \
  --allow-root --path=/var/www/monsite

# Vérifier le statut de publication
sudo /usr/local/bin/wp post list --post_type=product \
  --fields=ID,post_title,post_status \
  --allow-root --path=/var/www/monsite
```

---

## Scénario — Images produits floues ou cassées

### Ticket client type

> "Les images de mes produits sont floues depuis que j'ai changé de thème."

### Cause

WooCommerce génère des miniatures (thumbnails) à des dimensions précises définies par le thème. Après un changement de thème, les anciennes miniatures ont les mauvaises dimensions.

### Solution

**WooCommerce → Status → Tools → Regenerate shop thumbnails**

```
Résultat : "Thumbnail regeneration has been scheduled to run in the background."
```

WooCommerce régénère toutes les miniatures via Action Scheduler (wp-cron) en arrière-plan.

> **Règle HE** : à chaque changement de thème signalé par un client avec des images mal affichées → Regenerate shop thumbnails en premier.

---

## Scénario — Prix promotionnel non affiché

### Ticket client type

> "J'ai mis un prix promo sur mon produit mais il ne s'affiche pas sur la boutique."

### Cause

Le champ **Sale price** a des options de dates optionnelles (**Sale price dates from / to**). Si la période est expirée ou pas encore commencée, WooCommerce n'affiche pas le prix promotionnel — même si le montant est rempli.

### Solution

Products → [produit] → General → **Sale price dates** → vider les dates ou corriger la période pour qu'elle inclue la date actuelle.

---

## Concepts clés appris

| Concept | Explication |
|---------|-------------|
| `_visibility` | Meta produit qui contrôle l'affichage dans le catalogue |
| Regenerate thumbnails | Recalcule les miniatures après changement de thème |
| Action Scheduler | Système de tâches en arrière-plan de WooCommerce (basé sur wp-cron) |
| Sale price dates | Dates de validité du prix promo — cause fréquente de promo non affichée |

## Résultat

3 scénarios produits et catalogue maîtrisés.

**Prochaine étape** : HE-WC-04 — Livraison & Taxes (5 scénarios).
