Role: tu es un expert en redaction de plan de formation ORSYS.
Objectif: tu vas faire la commande xxxPF décrite dans `./.github/prompts/xxxPF.prompt.md`
Contraintes : le client desire modifier le plan actuel. Voici le contenu de désidératas:

Format de sortie:

- Markdown
- Ne pas mettre l'indication des jours dans le plan mais seulement les chapitres.
- Ne pas expliquer les contraintes de formattage (ex: 8 points) dans le document de sortie.

```
Aulieu de 5 bullets points par chapitre, il faut en mettre 8.

Ce sont des experts en securité. Mais ils ne connaissent rien à l'IA. Donc ils ne veulent pas avoir l'impression que on leur donne un cours sur ISO27001, NIS2, ANSSI, etc. Ils pensent être au top dans ces aspects. Mais ils veulent savoir ce que l'IA aujourd'hui peut causer comme problèmatique en sécurité informatique dans une DSI telles que celles des Hauts-de-France.

Ils veulent que le cours reste sur 3 jours.
Le public sera des gens de la DSI mais pas que des developpeurs. Il y aura des managers, le RSSI, des gens au type plus "responsable de projet".

Ils veulent dans le contenu :
- 1 journee a passer a comprendre ce qu'est techniquement une IA, avec les aspects ethiques, ils veulent aussi un panorma des outils IA du web (generation image, video, codage de site, agent codex, generation de slides, etc.)
- 1 journee dediee à la securité et l'avenement des IA, appliquee a leur problematique (DSI des Haut-de-France)
- 1 journee a passer à apprendre a prompter l'IA (prompt engineering).


Ils ont aussi des aversions:
- ils veulent passer entierement sous LibreOffice et se detacher des outils Microsoft bureautique payant.
- ils ne veulent plus d'IBM.
- ils ne codent pas en python
- ils comprennent les technologies du web (Front-end, Back-end, etc.)

Ne pas les facher avec les mots IBM, Microsoft, Python, ou les termes qu'ils pensent maitriser (ISO27001, etc.)

Ils ont des nouveaux sujets qui les interessent :
ethique de l'IA, ecologie, droit d'auteur, respect des donnees
Architecture RAG et embedding

Attention, il y a des sujets a garder de l'ancien plan mais reformule les si besoin de garder une coherence :
•	Enjeux stratégiques de l’IA pour la cybersécurité dans les collectivités locales : protection des données publiques, continuité de service, conformité réglementaire.
•	Vocabulaire essentiel : apprentissage automatique, modèles prédictifs, détection d’anomalies, IA générative.
•	Applications concrètes de l’IA pour la cybersécurité : supervision, SOC augmentés, automatisation des alertes, classification d’incidents.
•	Panorama des menaces émergentes : deepfakes, attaques par IA, code vulnérable généré par IA.
•	Cadre de l’IA Act
•	Exemple d’utilisation de l’IA autres collectivités
•	Cas d’usages réussis de réaction suite à une Cyber-attaque impliquant l’IA
•	Panorama des principaux outils que les usagers peuvent utiliser au quotidien
o	(Memento sous format tableau des LLMs – veille technologique des outils selon usage)
•	Collecte et préparation de données issues du SI public : logs, flux réseaux, incidents de sécurité.
•	Méthodes d’analyse de données pour la cybersécurité : classification, corrélation, scoring de risque.
•	Algorithmes de machine learning pour la détection d’anomalies et de comportements suspects.
•	Automatisation de la réponse aux incidents via des outils d’IA (SOAR, LLM assistés).
•	Intégration de l’IA dans le dispositif de sécurité existant : contraintes techniques et réglementaires propres aux collectivités.
•	Développement et déploiement de modèles prédictifs pour anticiper les cyberattaques.
•	Gestion des risques et des biais liés à l’IA : transparence, fiabilité, sécurité des modèles.
•	Gouvernance, responsabilités et enjeux éthiques dans le secteur public.
•	Méthodologie d’audit et de tests de sécurité pour les applications intégrant de l’IA.
•	Élaboration d’une feuille de route IA & cybersécurité pour la collectivité.


Travaux pratique sur le prompt:
construction d’un plan d’action pour intégrer l’IA de manière sécurisée et éthique dans la DSI territoriale.
```
