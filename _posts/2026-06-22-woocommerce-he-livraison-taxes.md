---
layout: post
title: "HE-WC-04 : Livraison & Taxes WooCommerce — Journal Happiness Engineer"
date: 2026-06-22
categories: [wordpress, woocommerce, happiness-engineer]
tags: [woocommerce, livraison, taxes, shipping-zones, free-shipping, happiness-engineer]
---

## Objectif

Pratiquer les scénarios de support livraison et taxes les plus fréquents rencontrés par un Happiness Engineer chez Automattic.

---

## Scénario 21 — Aucune méthode de livraison au checkout

### Ticket client type

> "Quand j'arrive au checkout, il n'y a aucune option de livraison disponible."

### Diagnostic

Première question au client : **dans quel pays souhaitez-vous être livré ?**

Si la zone de livraison du client n'est pas configurée dans WooCommerce, aucune méthode ne s'affiche au checkout.

**WooCommerce → Settings → Shipping → Shipping zones** — vérifier si une zone couvre le pays du client.

### Situation initiale

| Zone | Méthode | Couverture |
|------|---------|------------|
| Morocco | Free shipping | Maroc uniquement |
| ~~Rest of the world~~ | ~~Aucune~~ | ~~Non configuré~~ |

Tout client hors Maroc voyait un checkout sans méthode de livraison.

### Solution — Créer une zone "Reste du monde"

**WooCommerce → Settings → Shipping → Add shipping zone :**

- **Zone name** : Rest of the world
- **Zone regions** : Everywhere else
- **Shipping method** : Flat rate — 50 MAD

### Résultat

| Zone | Méthode | Couverture |
|------|---------|------------|
| Morocco | Free shipping | Maroc |
| Rest of the world | Flat rate 50 MAD | Tous les autres pays |

> **Règle HE** : toujours créer une zone "Everywhere else" comme fallback — évite les checkouts vides pour les clients internationaux.

---

## Scénario 22 — Livraison gratuite ne se déclenche pas

### Ticket client type

> "J'ai un coupon de livraison gratuite mais ça ne s'applique pas au checkout."

### Cause

Le coupon `FREESHIP` était bien configuré avec **"Allow free shipping"** coché, mais la méthode Free Shipping dans la zone était configurée sur **"No requirement"** — ce qui signifie que tout le monde avait la livraison gratuite sans coupon, rendant le coupon inutile.

### Diagnostic

**WooCommerce → Settings → Shipping → [Zone] → Free Shipping → Edit → "Free Shipping Requires"**

| Valeur | Comportement |
|--------|-------------|
| No requirement | Livraison gratuite pour tout le monde |
| A valid free shipping coupon | Requiert le coupon FREESHIP |
| A minimum order amount | Requiert un montant minimum |

### Solution

Changer **"Free Shipping Requires"** de `No requirement` → `A valid free shipping coupon`.

---

## Scénario 24 — TVA calculée incorrectement

### Ticket client type

> "La TVA s'affiche sur ma facture mais il ne devrait pas y avoir de TVA."

### Diagnostic

**WooCommerce → Settings → General → Enable taxes** — vérifier si les taxes sont activées.

### Solution

Si les taxes ne doivent pas s'appliquer : décocher **"Enable taxes"** → Save changes.

Pour une configuration TVA avancée (EU VAT, exonérations) : **WooCommerce → Settings → Tax** → configurer les taux par pays.

---

## Concepts clés appris

| Concept | Explication |
|---------|-------------|
| Shipping zone | Zone géographique avec ses méthodes de livraison associées |
| Everywhere else | Zone fallback — couvre tous les pays non listés explicitement |
| Free Shipping Requires | Condition pour déclencher la livraison gratuite (coupon, montant, etc.) |
| Enable taxes | Option globale pour activer/désactiver le calcul de TVA |

## Résultat

3 scénarios livraison et taxes maîtrisés. La zone "Rest of the world" est maintenant configurée comme fallback.

**Prochaine étape** : HE-WC-05 — Checkout & Panier (5 scénarios).
