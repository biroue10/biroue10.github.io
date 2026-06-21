---
layout: post
title: "WC-05 : Troubleshooting WooCommerce — Checkout, Emails, Stock"
date: 2026-06-21
categories: [wordpress, woocommerce, troubleshooting]
tags: [woocommerce, troubleshooting, smtp, javascript, stock, happiness-engineer]
---

## Objectif

Diagnostiquer et résoudre trois problèmes WooCommerce courants rencontrés par les Happiness Engineers chez Automattic.

## Environnement

- **WooCommerce** : 10.8.1
- **Boutique** : `http://192.168.11.103/boutique/`

---

## Scénario 1 — Checkout bloqué (bouton "Place order" inactif)

### Symptôme

Le client clique sur "Place order" mais rien ne se passe.

### Diagnostic

Ouvrir la console du navigateur (F12 → Console) sur la page checkout et chercher des erreurs JavaScript en rouge.

Erreur typique :
```
Uncaught TypeError: $ is not a function
```

### Cause

`$` est le raccourci de jQuery. Cette erreur signifie que **jQuery n'est pas chargé** ou qu'un plugin charge une autre bibliothèque JS qui entre en conflit avec jQuery.

### Solution

Désactiver les plugins un par un (mode debug) jusqu'à ce que l'erreur disparaisse — le dernier plugin désactivé est la cause du conflit.

---

## Scénario 2 — Email de confirmation absent

### Symptôme

Le client dit ne pas avoir reçu l'email de confirmation après sa commande.

### Diagnostic

**Étape 1 — Vérifier la configuration des emails :**

**WooCommerce → Settings → Emails** — vérifier que "Processing order" est activé pour le client.

**Étape 2 — Tester l'envoi :**

En bas de la page Emails → **"Send a test email"** → si l'erreur `Couldn't send the test email` apparaît, le problème est au niveau de l'envoi SMTP.

**Étape 3 — Vérifier les logs :**

**WooCommerce → Status → Logs** → fichier `wc_logger` → chercher des erreurs liées à l'envoi d'email.

### Cause

WordPress utilise `wp_mail()` qui dépend de la fonction PHP `mail()` (sendmail). En hébergement lab ou mutualisé sans sendmail configuré, les emails partent dans le vide.

### Solution

Installer un plugin SMTP :
- **WP Mail SMTP** (le plus courant)
- Configurer avec Gmail, SendGrid, ou Mailgun
- Le plugin remplace `wp_mail()` par une connexion SMTP authentifiée

### Autres causes possibles

| Cause | Vérification |
|-------|-------------|
| Email dans les spams | Demander au client de vérifier |
| Adresse email incorrecte | WooCommerce → Orders → vérifier l'email de la commande |
| Plugin de cache | Vider le cache |

---

## Scénario 3 — Stock non décrémenté

### Symptôme

Commande payée et complétée, mais le stock du produit ne change pas.

### Diagnostic

**Products → [Produit] → Inventory** → vérifier que **"Manage stock"** est coché.

### Vérification sur notre boutique

Stock initial du T-Shirt Biroue Lab : **25 unités**

Après la commande test passée en WC-03 :

```bash
sudo /usr/local/bin/wp post meta get 13 _stock \
  --allow-root --path=/var/www/monsite
# Résultat : 24
```

WooCommerce a correctement décrémenté le stock de 25 → 24.

### Cause du problème (si ça ne fonctionne pas)

**"Manage stock"** désactivé sur le produit → WooCommerce ne suit pas les quantités et ne décrémente jamais.

### Solution

Products → [Produit] → Inventory → cocher **"Manage stock"** → entrer la quantité actuelle → Save.

---

## Résumé

| Scénario | Cause | Diagnostic | Solution |
|----------|-------|------------|----------|
| Checkout bloqué | Conflit JavaScript | F12 → Console → erreur `$` | Désactiver plugins un par un |
| Email non reçu | Pas de SMTP configuré | Send test email → erreur | Installer WP Mail SMTP |
| Stock non décrémenté | "Manage stock" désactivé | Produit → Inventory | Activer "Manage stock" |

## Concepts clés appris

| Concept | Explication |
|---------|-------------|
| `$ is not a function` | jQuery non chargé ou conflit avec autre bibliothèque JS |
| `wp_mail()` | Fonction WordPress d'envoi d'email — dépend de sendmail par défaut |
| WP Mail SMTP | Plugin qui remplace `wp_mail()` par SMTP authentifié |
| Manage stock | Option produit WooCommerce — doit être activée pour décrémenter le stock |
| WC Status Logs | WooCommerce → Status → Logs — journal des événements système |

## Résultat

3 scénarios de troubleshooting diagnostiqués. Ces cas représentent les tickets les plus fréquents reçus par les Happiness Engineers WooCommerce.

**Prochaine étape** : HE-WC-01 — Scénarios paiements Stripe/PayPal.
