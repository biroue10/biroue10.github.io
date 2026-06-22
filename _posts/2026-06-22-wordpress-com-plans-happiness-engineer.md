---
layout: post
title: "WPC-01 : WordPress.com Plans — Journal Happiness Engineer"
date: 2026-06-22
categories: [wordpress, wordpress-com, happiness-engineer]
tags: [wordpress-com, plans, plugins, domaines, happiness-engineer]
---

## Objectif

Maîtriser les plans WordPress.com et savoir répondre aux tickets clients liés aux limitations de plan.

---

## Les plans WordPress.com

| Plan | Fonctionnalités clés |
|------|---------------------|
| **Free** | Sous-domaine `.wordpress.com`, publicités Automattic, stockage limité |
| **Personal** | Domaine custom, sans pub, support email |
| **Explorer** | Thèmes premium, outils de monétisation basiques |
| **Creator** | Plugins illimités, thèmes premium, outils avancés |
| **Business** | SEO avancé, plugins illimités |
| **Commerce** | WooCommerce complet, paiements en ligne |

---

## Scénario 1 — Client qui ne peut pas installer un plugin

### Ticket client type

> "Je veux installer WooCommerce mais je ne vois pas Plugins dans mon menu."

### Plan minimum requis

**Creator** — en dessous de Creator, l'installation de plugins est bloquée.

### Réponse HE

1. Demander : "Quel est votre plan actuel ?"
2. Si Free ou Personal : *"Malheureusement, l'installation de plugins n'est pas possible avec votre plan actuel. Il vous faut au minimum le plan Creator. Êtes-vous disposé à migrer vers ce plan ?"*

---

## Scénario 2 — Client qui veut un domaine custom

### Ticket client type

> "Je veux utiliser mon propre domaine monsite.com sur WordPress.com."

### Plan minimum requis

**Personal** — le plan Free impose un sous-domaine `.wordpress.com`.

### Réponse HE

*"Malheureusement, il n'est pas possible d'utiliser un domaine custom avec le plan Free. Avec le plan Personal, c'est possible. Voulez-vous migrer vers ce plan ?"*

---

## Scénario 3 — Client mécontent des publicités

### Ticket client type

> "Il y a des publicités sur mon site que je n'ai pas mises, comment les enlever ?"

### Cause

Le plan Free inclut des publicités Automattic affichées automatiquement sur le site.

### Plan minimum pour enlever les pubs

**Personal**

### Réponse HE

*"Les publicités font partie intégrante du plan Free dont vous disposez. Il est possible de les faire disparaître en optant pour un plan Personal au minimum. Êtes-vous disposé à changer de plan ?"*

---

## Structure d'une réponse HE sur les plans

Toujours inclure ces 3 éléments :

1. **Expliquer la limitation** — pourquoi ce n'est pas disponible sur le plan actuel
2. **Donner la solution** — quel plan minimum résout le problème
3. **Proposer l'action** — inviter le client à upgrader

---

## Tableau récapitulatif des limitations par plan

| Fonctionnalité | Free | Personal | Creator+ |
|---------------|------|----------|----------|
| Domaine custom | ❌ | ✅ | ✅ |
| Sans publicités | ❌ | ✅ | ✅ |
| Plugins | ❌ | ❌ | ✅ |
| WooCommerce | ❌ | ❌ | ✅ (Commerce) |

## Résultat

3 scénarios de tickets liés aux plans WordPress.com maîtrisés. Structure de réponse HE acquise.

**Prochaine étape** : WPC-02 — Éditeur Gutenberg et blocs.
