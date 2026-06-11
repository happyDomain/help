---
date: 2020-12-15T01:01:08+01:00
title: E-Mail
description: "Déclarez les serveurs de courrier d'une zone (enregistrements MX) ainsi que les services complémentaires (SPF, DKIM, DMARC) qui encadrent l'envoi et la réception des courriels."
aliases:
    services/email
---

Le service **E-Mail** permet de déclarer les serveurs de courrier responsables d'une zone. Il oriente également vers les services associés, qui régissent la manière dont le courrier est envoyé et authentifié pour le domaine.

Dans happyDomain, ce service porte le nom de **E-Mail servers**. Son unique rôle consiste à publier les enregistrements `MX`, ceux qui indiquent au reste d'Internet *où* livrer les courriels adressés à votre domaine. Tout ce qui protège le courrier *sortant*, c'est-à-dire prouver qu'un message provient réellement de vous, relève de services dédiés et distincts, décrits plus bas.

## Recevoir du courrier : les enregistrements MX

Un enregistrement `MX` (*Mail eXchanger*) désigne une machine qui accepte le courrier du domaine. Une zone en publie généralement plusieurs, afin que la livraison continue de fonctionner si l'un des serveurs devient indisponible.

Chaque entrée comporte deux parties :

- **La priorité** : un nombre qui ordonne les serveurs. Les serveurs émetteurs essaient d'abord le nombre le **plus bas** ; les valeurs plus élevées ne servent que de secours. Deux enregistrements partageant la même priorité sont considérés comme équivalents et se répartissent la charge.
- **Le serveur de courrier** : le nom d'hôte de la machine qui reçoit le courrier (par exemple `mx1.example.com.`). Cet hôte doit lui-même se résoudre vers une adresse ; il ne s'agit pas d'indiquer directement une adresse IP.

Une configuration courante ressemble à ceci :

| Priorité | Serveur de courrier |
|---|---|
| 10 | `mx1.example.com.` |
| 20 | `mx2.example.com.` |

Ici, `mx1` est contacté en premier et `mx2` sert de secours.

{{% notice style="info" title="Un seul service E-Mail par sous-domaine" %}}
Le service E-Mail servers est *unique* : un sous-domaine donné ne peut héberger qu'un seul service de ce type, qui rassemble l'ensemble de ses enregistrements `MX`. Pour déclarer des serveurs de courrier à la fois sur l'apex (`example.com`) et sur un sous-domaine (`mail.example.com`), on ajoute le service à chacun d'eux séparément.
{{% /notice %}}

## Envoyer du courrier : authentification et lutte contre l'usurpation

Publier des enregistrements `MX` suffit pour *recevoir* du courrier, mais ne dit rien sur les serveurs autorisés à en *envoyer* en votre nom. Sans cette indication, n'importe qui peut forger des messages au nom de votre domaine. happyDomain propose plusieurs services dédiés, chacun gérant ses propres enregistrements DNS, pour établir cette protection :

- **SPF** (*Sender Policy Framework*) : un enregistrement `TXT`, placé en général à l'apex de la zone, qui énumère les serveurs autorisés à envoyer du courrier pour le domaine. Les destinataires comparent le serveur émetteur à cette liste.
- **DKIM** (*DomainKeys Identified Mail*) : publie la partie publique d'une clé de signature dans un enregistrement `TXT`, sous un sélecteur `._domainkey`. Vos serveurs signent les messages sortants, et les destinataires vérifient la signature à l'aide de cette clé publiée.
- **DMARC** (*Domain-based Message Authentication, Reporting and Conformance*) : un enregistrement `TXT` placé sur `_dmarc`, qui indique aux destinataires comment traiter les messages échouant aux contrôles SPF ou DKIM (les laisser passer, les mettre en quarantaine ou les rejeter), et où envoyer les rapports agrégés.

Deux autres services couvrent la sécurité du transport et la remontée d'informations :

- **MTA-STS** : déclare que le courrier destiné à votre domaine doit être livré au travers d'une connexion sécurisée (TLS).
- **TLS-RPT** : recueille les rapports relatifs aux problèmes de livraison TLS rencontrés par les serveurs émetteurs.

Ces services sont indépendants du service E-Mail servers. On peut n'ajouter que ceux dont on a besoin, mais une configuration de courrier complète et bien protégée associe le plus souvent `MX`, SPF, DKIM et DMARC.

## Dans l'éditeur de zone

Pour configurer le courrier d'un sous-domaine dans l'éditeur de zone de happyDomain :

1. On ajoute le service **E-Mail servers** au sous-domaine, puis on renseigne une ligne par serveur de courrier, avec sa priorité et son nom d'hôte.
2. On ajoute **SPF**, **DKIM** et **DMARC** en tant que services à part entière sur le sous-domaine concerné (SPF et DMARC se placent généralement à l'apex). Chacun présente un formulaire dédié plutôt que le contenu brut de l'enregistrement.

Comme chacune de ces abstractions constitue un service distinct, on les gère séparément, même si elles concourent toutes à rendre le courrier de votre domaine à la fois livrable et digne de confiance. Le chapitre {{< relref "/reference/services" >}} présente la liste complète des services disponibles.
