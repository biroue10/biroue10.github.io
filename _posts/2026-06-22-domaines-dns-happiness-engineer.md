---
layout: post
title: "A-01 : Domaines & DNS — Journal Happiness Engineer"
date: 2026-06-22
categories: [wordpress, dns, happiness-engineer]
tags: [dns, domaines, whois, dig, cname, ttl, wordpress-com, happiness-engineer]
---

## Objectif

Maîtriser les outils et scénarios DNS fréquents en tant que Happiness Engineer chez Automattic.

---

## Outils DNS essentiels

### WHOIS — Vérifier l'expiration d'un domaine

```bash
whois nomdudomaine.com | grep -i expir
```

Quand utiliser : client dont le site est soudainement down → vérifier si le domaine a expiré.

```
Registry Expiry Date: 2028-09-14T04:00:00Z
```

Si la date est **passée** → domaine expiré → le client doit renouveler chez son registrar.

### dig — Interroger les enregistrements DNS

```bash
# Enregistrement A (adresse IP)
dig domaine.com A

# Enregistrement CNAME
dig www.domaine.com CNAME

# Nameservers du domaine
dig domaine.com NS

# Interroger directement le serveur DNS autoritaire (bypass cache)
dig @ns1.domaine.com domaine.com A
```

Le flag `aa` dans la réponse signifie **Authoritative Answer** — réponse directe de la source, pas d'un cache.

---

## Scénario 1 — Domaine expiré

### Ticket client type

> "Mon site affiche une page d'erreur depuis ce matin, hier ça marchait."

### Procédure

```bash
# 1. Vérifier l'expiration
whois domaineclient.com | grep -i expir

# 2. Vérifier que le DNS répond encore
dig domaineclient.com A
```

Si le domaine est expiré → demander au client de renouveler chez son registrar.

---

## Scénario 2 — Domaine custom sur WordPress.com ne fonctionne pas

### Ticket client type

> "J'ai configuré mon domaine custom sur WordPress.com mais j'obtiens une erreur DNS."

### Configuration DNS requise pour WordPress.com

| Type | Nom | Valeur |
|------|-----|--------|
| `A` | `@` (domaine racine) | IP fournie par WordPress.com |
| `CNAME` | `www` | `lb.wordpress.com` |

`lb.wordpress.com` est le load balancer de WordPress.com :

```bash
dig www.wordpress.com CNAME
# www.wordpress.com. 5109 IN CNAME lb.wordpress.com.
```

### Diagnostic

```bash
# Vérifier le CNAME de www
dig www.domaineclient.com CNAME

# Vérifier le A record
dig domaineclient.com A

# Interroger directement le nameserver autoritaire
dig domaineclient.com NS
dig @ns1.domaineclient.com domaineclient.com A
```

Si le CNAME est absent ou pointe vers le mauvais endroit → le client doit corriger dans son panneau DNS chez son registrar.

---

## Concept clé — TTL et propagation DNS

Le **TTL (Time To Live)** indique combien de secondes une réponse DNS est mise en cache.

```bash
dig www.wordpress.com CNAME
# www.wordpress.com. 5109 IN CNAME lb.wordpress.com.
#                   ^^^^
#                   TTL en secondes
```

Quand un client demande "pourquoi mon domaine ne marche pas après le changement DNS ?" :

> "Les changements DNS peuvent prendre jusqu'à **48 heures** pour se propager sur tous les serveurs DNS. En pratique, c'est souvent 15-30 minutes, mais la propagation complète peut prendre jusqu'à 48h selon le TTL configuré."

---

## Types d'enregistrements DNS essentiels

| Type | Usage |
|------|-------|
| `A` | Pointe un nom vers une IPv4 |
| `AAAA` | Pointe un nom vers une IPv6 |
| `CNAME` | Pointe un nom vers un autre nom |
| `MX` | Serveurs de messagerie |
| `NS` | Nameservers du domaine |
| `TXT` | Vérification, SPF, DKIM |

> **Règle importante** : un domaine racine (`@`) ne peut **pas** avoir de CNAME — uniquement un enregistrement A.

---

## Concepts clés appris

| Concept | Explication |
|---------|-------------|
| WHOIS | Outil pour voir les infos d'un domaine (expiration, registrar, nameservers) |
| `dig @ns` | Interroger directement le serveur DNS autoritaire — bypass le cache local |
| Flag `aa` | Authoritative Answer — réponse de la source, pas d'un cache |
| CNAME | Enregistrement qui pointe un nom vers un autre nom |
| TTL | Durée de vie du cache DNS — explique la propagation |
| `lb.wordpress.com` | Load balancer WordPress.com — cible du CNAME pour domaines custom |

## Résultat

Maîtrise des outils DNS essentiels (whois, dig) et des scénarios domaines les plus fréquents en support HE.

**Prochaine étape** : WPC-01 — WordPress.com plans et fonctionnalités.
