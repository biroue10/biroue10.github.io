---
layout: post
title: "HE-WC-02 : Gestion des commandes WooCommerce — Journal Happiness Engineer"
date: 2026-06-21
categories: [wordpress, woocommerce, happiness-engineer]
tags: [woocommerce, commandes, remboursement, stripe, pending, happiness-engineer]
---

## Objectif

Pratiquer les scénarios de gestion de commandes les plus fréquents rencontrés par un Happiness Engineer chez Automattic.

---

## Scénario 10 — Commande bloquée en "Pending"

### Ticket client type

> "J'ai passé une commande il y a 3 heures, elle est toujours en 'Pending'."

### Deux causes possibles

| Cause | Explication |
|-------|-------------|
| Client a abandonné le checkout | WooCommerce crée la commande en "Pending" dès le début du checkout, avant le paiement |
| Webhook manqué | Client a payé, Stripe a débité, mais WooCommerce n'a jamais reçu la confirmation |

### Comment distinguer les deux cas

Aller dans **Stripe Dashboard → Payments** et chercher un paiement pour ce client à cette heure.

- **Paiement trouvé** → webhook manqué → mettre la commande en "Processing" manuellement
- **Aucun paiement** → client a abandonné → demander de repasser la commande

### Vérification MySQL (HPOS)

```bash
mysql -u monsite_user -p'WordPress2025!' -e "
  SELECT id, status, date_created_gmt, total_amount
  FROM monsite_db.wp_wc_orders
  WHERE status = 'wc-pending'
  ORDER BY date_created_gmt DESC LIMIT 10;"
```

---

## Scénario 11 — Client dit avoir commandé mais aucune commande trouvée

### Causes possibles

- Commande en statut "Failed" (paiement refusé)
- Commande sous un autre email
- Sur un multisite : mauvais sous-site consulté

### Diagnostic MySQL

```bash
mysql -u monsite_user -p'WordPress2025!' -e "
  SELECT id, status, billing_email, total_amount
  FROM monsite_db.wp_wc_orders
  WHERE billing_email = 'client@example.com';"
```

---

## Scénario 14 — Remboursement impossible via WooCommerce

### Ticket client type

> "Je veux me faire rembourser ma commande de il y a 6 mois."

### Cause

Stripe limite les remboursements via API à **120 jours**. Au-delà, WooCommerce ne peut plus déclencher le remboursement automatiquement.

### Solution

Rembourser directement depuis **Stripe Dashboard → Payments → [paiement] → Refund**.

WooCommerce est bypassé — Stripe traite le remboursement directement.

---

## Scénario 15 — Client veut annuler une commande déjà expédiée

### Ticket client type

> "Je veux annuler ma commande mais elle est déjà marquée Completed."

### Procédure

1. WooCommerce → Orders → [commande] → **Refund** en bas de page
2. Entrer le montant à rembourser
3. **Ne pas cocher "Restock items"** — le produit est déjà expédié, il ne revient pas
4. Confirmer → statut passe automatiquement en **Refunded**

---

## Concepts clés appris

| Concept | Explication |
|---------|-------------|
| Commande "Pending" | Créée dès le début du checkout, avant le paiement |
| Webhook manqué | Stripe a débité mais WooCommerce n'a pas reçu la confirmation |
| Limite remboursement Stripe | 120 jours maximum via API |
| Refund sans restock | Ne pas cocher "Restock items" si produit déjà expédié |
| `wc-pending` | Préfixe WooCommerce pour le statut "Pending" en base HPOS |

## Résultat

4 scénarios de gestion de commandes maîtrisés. Ces cas couvrent la majorité des tickets commandes reçus par les HE WooCommerce.

**Prochaine étape** : HE-WC-03 — Produits & Catalogue (5 scénarios).
