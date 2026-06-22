---
layout: post
title: "J-01 : Jetpack — Journal Happiness Engineer"
date: 2026-06-22
categories: [wordpress, jetpack, happiness-engineer]
tags: [jetpack, wordpress-com, connexion, xml-rpc, backup, happiness-engineer]
---

## Objectif

Comprendre Jetpack et diagnostiquer les scénarios de support les plus fréquents.

---

## Qu'est-ce que Jetpack ?

Jetpack est un plugin WordPress créé par Automattic qui regroupe plusieurs services en un seul plugin. C'est le pont entre un WordPress self-hosted et les services Automattic.

| Fonctionnalité | Description |
|---------------|-------------|
| **Backup** | Sauvegarde automatique du site (VaultPress) |
| **Security** | Scan malware, protection brute force, WAF |
| **Stats** | Statistiques de visites du site |
| **CDN** | Accélération des images via le CDN Automattic |
| **Boost** | Optimisation des performances |
| **Social** | Partage automatique sur les réseaux sociaux |

---

## Scénario 1 — Jetpack ne se connecte pas à WordPress.com

### Ticket client type

> "J'ai installé Jetpack mais il affiche 'Jetpack is not connected to WordPress.com'."

### Causes principales

| Cause | Explication | Solution |
|-------|-------------|----------|
| **Firewall** | Le serveur bloque les connexions sortantes vers `wordpress.com` | Autoriser les IPs WordPress.com dans le firewall |
| **XML-RPC désactivé** | Jetpack utilise `xmlrpc.php` pour communiquer | Autoriser l'accès à xmlrpc.php pour WordPress.com |
| **Conflit plugin** | Plugin sécurité (Wordfence, iThemes) bloque Jetpack | Désactiver temporairement les plugins de sécurité |
| **IP privée** | Site sur réseau local — WordPress.com ne peut pas l'atteindre | Exposer le site publiquement |

### Erreur rencontrée en lab

```
Your site host "192.168.11.103" is on a private network.
Jetpack can only connect to public sites. (Status 500)
```

Comportement normal en lab local — WordPress.com ne peut pas atteindre une IP privée depuis internet.

### Conflit XML-RPC — Le cas classique

Jetpack utilise `xmlrpc.php` pour se connecter à WordPress.com. Si on bloque XML-RPC pour des raisons de sécurité (bonne pratique), Jetpack ne peut plus se connecter.

**Solution** : autoriser uniquement les IPs Automattic/WordPress.com à accéder à xmlrpc.php dans Nginx :

```nginx
location = /xmlrpc.php {
    allow 192.0.64.0/18;  # IPs WordPress.com
    deny all;
}
```

---

## Scénario 2 — Jetpack sur WordPress.com

En production sur WordPress.com, Jetpack est **pré-intégré** — le client n'a pas besoin de le connecter manuellement. Les fonctionnalités disponibles dépendent du plan :

| Plan | Fonctionnalités Jetpack incluses |
|------|----------------------------------|
| Free | Stats basiques |
| Personal | Backup journalier, Stats |
| Creator+ | Backup temps réel, Scan malware, CDN |

---

## Concepts clés appris

| Concept | Explication |
|---------|-------------|
| Jetpack | Plugin Automattic — pont entre WordPress self-hosted et WordPress.com |
| XML-RPC | Protocole de communication WordPress utilisé par Jetpack |
| Conflit sécurité/fonctionnalité | Bloquer XML-RPC = sécurité mais casse Jetpack |
| IP privée | Jetpack nécessite un site accessible publiquement |

## Résultat

Jetpack compris — causes de connexion échouée identifiées et diagnostiquées.

**Prochaine étape** : J-02 — Jetpack Backup & Restore.
