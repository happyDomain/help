---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Délivrabilité sortante (happyDeliver)
description: "Envoie un véritable message de test depuis votre domaine vers une instance happyDeliver et évalue la qualité de sa délivrabilité jusqu'à une boîte de réception réelle."
weight: 210
---

Le vérificateur de **délivrabilité sortante** (nommé *Outbound deliverability (via happyDeliver)* dans happyDomain) mesure le sort que connaîtrait un message envoyé **depuis votre domaine** sur le chemin de la boîte de réception du destinataire. Au lieu d'examiner les enregistrements DNS isolément, il pilote une instance externe [happyDeliver](https://git.nemunai.re/happyDomain/happyDeliver) pour réaliser un test de bout en bout.

Ce vérificateur s'applique au **niveau service** : il est rattaché à un service (typiquement la configuration de messagerie d'un sous-domaine) et a besoin d'identifiants SMTP pour envoyer le message de test en votre nom.

## Fonctionnement

À chaque exécution, le vérificateur :

1. Alloue une adresse de réception neuve sur l'instance happyDeliver configurée.
2. Envoie un **véritable courriel de test** depuis le domaine testé vers cette adresse, en utilisant le serveur de soumission SMTP et les identifiants que vous fournissez.
3. Interroge happyDeliver jusqu'à ce que le message ait été reçu et analysé.
4. Conserve le rapport de happyDeliver et expose le score de chaque section sous forme de métrique.

happyDeliver note le message sur plusieurs sections (DNS, authentification, filtres anti-spam, listes noires, en-têtes, contenu) et calcule un score global. Le vérificateur transforme chaque section en une règle.

{{% notice style="info" title="Une instance happyDeliver externe est requise" %}}
Ce vérificateur ne fait rien seul : il lui faut une instance happyDeliver accessible, identifiée par l'URL de son API et un jeton d'authentification. Ceux-ci sont en général configurés une fois par l'administrateur et peuvent être redéfinis par domaine.
{{% /notice %}}

## Ce qui est vérifié

Chaque règle de section compare le score happyDeliver de cette section à un minimum configurable. La vérification est **Critique** lorsque le score passe sous le minimum, sinon **OK**.

| Règle | Ce qui est vérifié | Minimum par défaut |
|---|---|---|
| `happydeliver.score.overall` | Score global de happyDeliver | 70 |
| `happydeliver.score.dns` | Score de configuration DNS | 70 |
| `happydeliver.score.authentication` | Score d'authentification (SPF / DKIM / DMARC) | 80 |
| `happydeliver.score.spam` | Score des filtres anti-spam | 70 |
| `happydeliver.score.blacklist` | Score des listes noires | 90 |
| `happydeliver.score.header` | Score des en-têtes | 70 |
| `happydeliver.score.content` | Score du contenu | 60 |

Une règle distincte, `happydeliver.lifecycle`, rapporte le déroulement de l'exécution elle-même : **OK** lorsque le message a été analysé, **Critique** lorsque l'adresse de test n'a pu être allouée, que le message n'a pu être envoyé, ou que happyDeliver a renvoyé une erreur d'attente, de récupération ou d'analyse, et **Avertissement** lorsque le message n'a pas été analysé avant l'expiration du délai.

Chaque minimum de section se règle via son option de règle `min_score_<section>` dans l'interface de happyDomain.

## Options

### Envoi (par domaine)

| Option | Signification | Défaut |
|---|---|---|
| Serveur SMTP d'envoi (`smtp_host`) | Nom d'hôte ou IP du serveur de soumission utilisé pour envoyer le courriel de test. **Obligatoire.** | (aucun) |
| Port SMTP d'envoi (`smtp_port`) | Port de soumission (587 pour STARTTLS, 465 pour TLS implicite, 25 pour le texte clair). | 587 |
| Identifiant SMTP (`smtp_username`) | Identifiant du serveur de soumission (à omettre pour une soumission anonyme). | (aucun) |
| Mot de passe SMTP (`smtp_password`) | Mot de passe du serveur de soumission. | (aucun) |
| Mode TLS (`smtp_tls`) | Mode de négociation TLS : `starttls`, `tls` ou `none`. | `starttls` |
| Adresse d'expéditeur (`from_address`) | Adresse utilisée dans l'en-tête `From` ; elle doit appartenir au domaine testé. | `no-reply@<domaine>` |

### Contenu du message (par domaine)

| Option | Signification | Défaut |
|---|---|---|
| Sujet (`subject_override`) | Remplace le sujet de test par défaut. | (intégré) |
| Corps texte (`body_text_override`) | Remplace le corps texte par défaut. | (intégré) |
| Corps HTML (`body_html_override`) | Remplace le corps HTML par défaut. | (intégré) |

### Temporisation (par domaine)

| Option | Signification | Défaut |
|---|---|---|
| Délai d'attente (`wait_timeout`) | Secondes d'attente pour que happyDeliver reçoive et analyse le message. | 900 |
| Intervalle d'interrogation (`poll_interval`) | Secondes entre deux interrogations de statut (borné à la plage 2 à 60). | 5 |

### Instance (admin, redéfinissable par domaine)

| Option | Signification | Défaut |
|---|---|---|
| URL de l'instance happyDeliver (`happydeliver_url`) | URL de base de l'API happyDeliver. | (admin) |
| Jeton d'API happyDeliver (`happydeliver_token`) | Jeton d'authentification de l'API happyDeliver. | (admin) |

## Dans happyDomain

Activez ce vérificateur depuis l'onglet **Vérifications** du service et fournissez les détails de soumission SMTP afin que happyDomain puisse envoyer le message de test. Voir {{< relref "/pages/checks" >}} pour le fonctionnement général de la planification et de la lecture des vérifications.

La délivrabilité dépendant fortement de votre posture anti-usurpation, associez ce vérificateur à une configuration {{< relref "/reference/services/email" >}} bien réglée (SPF, DKIM et DMARC). Pour la partie DNSSEC de la chaîne de confiance de votre domaine, voir {{< relref "/reference/checkers/dnssec" >}}.
