---
layout: post
title: "HE-WC-01 : Scénarios paiements Stripe — Journal Happiness Engineer"
date: 2026-06-21
categories: [wordpress, woocommerce, happiness-engineer]
tags: [woocommerce, stripe, webhooks, decline-code, refund, happiness-engineer]
---

## Objectif

Pratiquer les scénarios de support paiement les plus fréquents rencontrés par un Happiness Engineer chez Automattic.

## Environnement

- **WooCommerce** : 10.8.1 avec Stripe en mode test
- **Boutique** : `http://192.168.11.103/boutique/`

---

## Scénario 1 — Carte refusée alors que le client dit qu'elle est valide

### Ticket client type

> "J'essaie de payer mais ma carte est refusée, pourtant elle fonctionne partout ailleurs."

### Diagnostic

Aller sur **dashboard.stripe.com → Payments** → chercher la transaction refusée → lire le champ `decline_code`.

### Codes d'erreur fréquents

| `decline_code` | Signification | Réponse au client |
|----------------|---------------|-------------------|
| `insufficient_funds` | Fonds insuffisants | Vérifiez le solde de votre carte |
| `card_not_supported` | Carte non acceptée | Essayez une Visa ou Mastercard |
| `authentication_required` | 3D Secure requis | Vérifiez votre téléphone pour le code SMS |
| `do_not_honor` | Banque bloque génériquement | Contactez votre banque |

### Règle HE

Toujours communiquer le `decline_code` exact au client. Ne jamais inventer une explication.

---

## Scénario 2 — Client débité mais aucune commande dans WooCommerce

### Ticket client type

> "J'ai été débité mais je n'ai reçu aucune confirmation de commande."

### Cause

Le mécanisme qui relie Stripe à WooCommerce s'appelle les **webhooks** :

```
Client → Stripe → paiement confirmé
                → Stripe envoie HTTP POST → URL WooCommerce
                                          → WooCommerce crée la commande
```

Si le webhook échoue (SSL invalide, firewall, serveur inaccessible) → Stripe a débité le client mais WooCommerce ne le sait pas → aucune commande créée.

### Diagnostic

**Étape 1 — Vérifier les webhooks Stripe :**
- Dashboard Stripe → Developers → Webhooks → voir les événements en erreur

**Étape 2 — Vérifier les logs WooCommerce :**
- WooCommerce → Status → Logs → `woocommerce-gateway-stripe`

**Étape 3 — Vérifier les paramètres Stripe :**
```bash
sudo /usr/local/bin/wp option get woocommerce_stripe_settings \
  --allow-root --path=/var/www/monsite
```

### Solution

Réenregistrer le webhook : WooCommerce → Settings → Payments → Stripe → Configure.

Si le paiement est réel et la commande absente : créer manuellement la commande dans WooCommerce et noter le PaymentIntent ID Stripe.

---

## Scénario 3 — Double facturation

### Ticket client type

> "J'ai été débité deux fois pour une seule commande."

### Diagnostic

WooCommerce → Orders → vérifier combien de commandes existent pour ce client.

- **2 commandes** → le client a cliqué deux fois sur "Place order" ou bug JS frontend
- **1 commande** → le doublon vient de Stripe directement (rare)

### Solution

Pour la commande en double :

1. WooCommerce → Orders → [commande en double] → **Refund** → rembourser le montant total
2. WooCommerce envoie automatiquement le remboursement à Stripe
3. Changer le statut de la commande en **Cancelled**

### Prévention

Le plugin Stripe for WooCommerce utilise des **idempotency keys** automatiquement — chaque tentative de paiement a un identifiant unique, ce qui empêche Stripe de débiter deux fois la même tentative.

---

## Concepts clés appris

| Concept | Explication |
|---------|-------------|
| `decline_code` | Code Stripe expliquant pourquoi une carte est refusée |
| Webhooks | Requêtes HTTP POST envoyées par Stripe à WooCommerce pour confirmer un paiement |
| Idempotency key | Clé unique par tentative de paiement — évite les doubles débits côté Stripe |
| Refund WooCommerce | WooCommerce → Order → Refund → remboursement automatique via Stripe |

## Résultat

3 scénarios Stripe maîtrisés. Ces cas représentent la majorité des tickets paiement reçus par les HE WooCommerce.

**Prochaine étape** : HE-WC-02 — Gestion des commandes (6 scénarios).
