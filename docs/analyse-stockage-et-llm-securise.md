# Analyse — Stockage des « cerveaux » clients et LLM maison

Note interne IA avec Ugo. Objectif : évaluer (1) un stockage multi-fournisseur
(DigitalOcean, GitHub, bientôt AWS) où chaque client choisit où réside son
« cerveau », et (2) la construction d'un LLM maison pour la sécurité.

Demande d'origine : « For the storage of data we use DigitalOcean and GitHub
and soon moving to AWS so the storage of clients' brains can be where they
want. Also building our own LLM too for more security. »

---

## 1. Résumé exécutif

| Sujet | Verdict | Pourquoi |
|---|---|---|
| Stockage au choix du client (DO / AWS) | ✅ Bonne idée, faisable | Les deux offrent un API compatible S3 et une région canadienne. Une couche d'abstraction unique suffit. |
| GitHub comme lieu de stockage des cerveaux clients | ❌ À éviter | GitHub est un outil de code, pas un stockage de données personnelles. Pas de choix de région pour un compte standard, historique Git impossible à purger proprement (droit à l'effacement, Loi 25). |
| « Construire notre propre LLM » | ⚠️ Reformuler | Entraîner un modèle de zéro est hors de portée (coût, équipe, qualité). Ce qui est réaliste et sécuritaire : **héberger soi-même un modèle à poids ouverts** ou utiliser une **inférence en région canadienne** avec zéro rétention. |

---

## 2. Stockage : où vivent les cerveaux clients

### 2.1 Ce qu'un « cerveau » contient

Documents internes du client, courriels, procédures, données de clients finaux
→ souvent des **renseignements personnels** au sens de la Loi 25. Donc :

- Registre des traitements et lieu d'hébergement documenté par client.
- **ÉFVP** (évaluation des facteurs relatifs à la vie privée) obligatoire avant
  tout transfert de renseignements personnels hors Québec (art. 17 Loi 25).
  Une région canadienne hors Québec (Toronto, Calgary) reste un transfert
  « hors Québec » à documenter, même si le risque est faible.
- Contrat de traitement avec chaque fournisseur infonuagique (déjà couvert par
  leurs DPA standard, à conserver au dossier).

### 2.2 Régions disponibles

| Fournisseur | Région canadienne | Stockage objet | Chiffrement clés client |
|---|---|---|---|
| DigitalOcean | Toronto (`tor1`) | Spaces (API S3) | Limité (SSE côté serveur, pas de BYOK complet) |
| AWS | Montréal (`ca-central-1`), Calgary (`ca-west-1`) | S3 | KMS, clés gérées par le client (BYOK), journalisation CloudTrail |
| GitHub | Aucun choix de région pour un compte standard | n/a | n/a |

**Montréal est la seule région qui permet de dire « vos données restent au
Québec ».** C'est un argument commercial fort pour une PME québécoise.
À vérifier avant de promettre : les régions et fonctionnalités évoluent;
confirmer sur les pages officielles au moment de signer.

### 2.3 Architecture recommandée

```
Client A ──┐                       ┌── DO Spaces tor1  (bucket a-…)
Client B ──┼─► API IA avec Ugo ───►│── AWS S3 ca-central-1 (bucket b-…)
Client C ──┘   (couche stockage)   └── AWS S3 ca-central-1 (bucket c-…)
```

Principes :

1. **Une seule interface de stockage** dans le code (interface S3). Le
   fournisseur et la région sont une **configuration par client**, pas du
   code différent. Ajouter AWS = ajouter un adaptateur, pas réécrire.
2. **Un bucket (ou préfixe) par client**, jamais de mélange. Politique IAM
   qui interdit l'accès croisé.
3. **Chiffrement** : TLS en transit, chiffrement au repos partout. Sur AWS,
   une clé KMS par client; offrir le BYOK aux clients exigeants.
4. **Effacement** : procédure documentée et testée pour supprimer un cerveau
   complet (données, index vectoriel, sauvegardes, journaux) sur demande.
5. **Sauvegardes** dans la même région que la donnée primaire, sinon on
   annule la promesse de résidence.
6. **Journal d'accès** conservé (qui a lu quoi, quand) — requis pour répondre
   à un incident (obligation de notification, Loi 25).
7. **Index vectoriel / base de données** : il doit suivre la même règle de
   résidence que les fichiers. Un cerveau à Montréal avec un index vectoriel
   aux États-Unis n'est pas « au Canada ».

### 2.4 Rôle de GitHub

- Code source, infrastructure-as-code, documentation publique : oui.
- Cerveaux clients, exports, fichiers de formation : **non**.
- Activer : secret scanning, push protection, 2FA obligatoire, revue des
  accès des collaborateurs. Vérifier qu'aucun cerveau client n'est déjà
  commité dans un dépôt (historique inclus).

### 2.5 Plan de migration vers AWS

1. Créer l'organisation AWS (Organizations, comptes séparés prod/dev),
   région par défaut `ca-central-1`.
2. Implémenter l'adaptateur S3 + KMS derrière l'interface existante.
3. Migrer un client pilote (copie, vérification d'intégrité, bascule, purge
   de l'ancien emplacement, preuve de purge au dossier).
4. Ajouter la question « où voulez-vous héberger vos données? » au processus
   d'intégration client, avec les options : Toronto (DO), Montréal (AWS).
5. Mettre à jour la politique de confidentialité et les contrats clients.

---

## 3. « Notre propre LLM » : ce que ça veut dire vraiment

### 3.1 Trois options, très différentes

| Option | Coût / effort | Sécurité | Qualité | Recommandation |
|---|---|---|---|---|
| A. Entraîner un modèle de fondation de zéro | Millions $, équipe ML, mois | Totale | Très inférieure aux modèles de pointe | ❌ Non |
| B. Héberger soi-même un modèle à poids ouverts (Llama, Mistral, Qwen…), éventuellement affiné (fine-tuning) sur nos données | GPU loués (~2 000–10 000 $/mois selon la taille), DevOps | Très forte : aucune donnée ne sort de notre infra | Bonne pour RAG et tâches ciblées, sous les meilleurs modèles commerciaux | ✅ Pour les clients qui l'exigent |
| C. Inférence commerciale en région canadienne, zéro rétention (ex. Claude via Amazon Bedrock dans `ca-central-1`) | Paiement à l'usage, aucune infra | Forte : données traitées au Canada, non utilisées pour l'entraînement, non conservées | Meilleure qualité disponible | ✅ Par défaut |

### 3.2 Ce que la sécurité exige réellement

Le risque principal n'est pas « le modèle », c'est **le chemin que prennent
les données** : où elles sont envoyées, si elles sont conservées, qui peut
les voir, si elles servent à entraîner un autre modèle. Les options B et C
répondent toutes deux à ces questions. L'option A n'apporte rien de plus en
sécurité et coûte énormément.

À exiger quel que soit le choix :

- Contrat qui interdit l'entraînement sur nos données et fixe la rétention à
  zéro (ou quelques jours pour la détection d'abus, documenté).
- Résidence de l'inférence au Canada, ou sur nos propres serveurs.
- Journalisation des requêtes chez nous (pas chez le fournisseur).
- Tests de fuite entre clients : un cerveau ne doit jamais apparaître dans
  la réponse d'un autre client (isolation par client au niveau du RAG).
- Défenses contre l'injection de prompt dans les documents clients.

### 3.3 Plan recommandé

1. **Maintenant** : option C. Inférence en région canadienne, contrat sans
   entraînement, zéro rétention. Argument marketing valide : « Vos données
   sont traitées au Canada et ne servent jamais à entraîner un modèle. »
2. **Dans 6 à 12 mois, si des clients l'exigent** (secteur public, santé,
   juridique) : option B en offre premium. Un modèle à poids ouverts hébergé
   sur nos serveurs (DO GPU Droplets à Toronto ou AWS à Montréal), affiné au
   besoin. Facturer ce surcoût.
3. **Ne pas** communiquer « nous construisons notre propre LLM ». Dire plutôt
   « modèle hébergé de façon privée » ou « inférence souveraine ». C'est
   précis, vérifiable, et ça n'engage pas à un chantier impossible.

---

## 4. Questions ouvertes pour Ugo

1. Combien de clients actifs et quel volume par cerveau (Go)? Ça détermine si
   le multi-fournisseur vaut la complexité tout de suite.
2. Des clients ont-ils déjà demandé une résidence précise (Québec vs Canada)?
3. Y a-t-il des cerveaux clients dans un dépôt GitHub aujourd'hui?
4. L'index vectoriel actuel est hébergé où?
5. Budget mensuel acceptable pour une offre « modèle privé »?

## 5. Prochaines étapes concrètes

- [ ] Audit : lister chaque emplacement actuel de données clients (DO, GitHub,
      index vectoriel, sauvegardes, outils tiers).
- [ ] Créer le compte AWS, région `ca-central-1`, un bucket pilote + KMS.
- [ ] Implémenter l'interface de stockage unique + adaptateur AWS.
- [ ] Rédiger le modèle d'ÉFVP par client et le gabarit de contrat.
- [ ] Choisir et contractualiser l'inférence en région canadienne.
- [ ] Rédiger la procédure d'effacement complet et la tester sur un client
      test.

---

## 6. Comparaison des options sous l'angle de la Loi 25

Loi 25 = Loi modernisant des dispositions législatives en matière de
protection des renseignements personnels (Québec). Pour une entreprise privée
comme IA avec Ugo, les articles qui comptent sont ceux de la **Loi sur la
protection des renseignements personnels dans le secteur privé (LPRPSP)**.
Les articles cités ci-dessous sont ceux de la LPRPSP telle que modifiée.
Vérifier le texte à jour sur legisquebec.gouv.qc.ca avant toute citation
externe.

### 6.1 Les obligations qui touchent directement ce projet

| Article | Obligation | Impact sur le stockage / LLM |
|---|---|---|
| art. 3.1 | Responsable de la protection des renseignements personnels (RPRP) désigné, coordonnées publiées | Ugo par défaut. À nommer par écrit. |
| art. 3.2 | Registre des incidents; notification à la CAI et aux personnes en cas d'incident présentant un risque de préjudice sérieux | Exige des journaux d'accès chez nous, pas seulement chez le fournisseur. |
| art. 3.3 | ÉFVP pour tout projet d'acquisition, de développement ou de refonte d'un système d'information impliquant des renseignements personnels | La migration vers AWS et le déploiement d'un LLM sont **chacun** un projet visé. |
| art. 3.5 | Confidentialité par défaut (paramètres les plus protecteurs) | Le choix de région le plus protecteur (Québec) devrait être le défaut, pas une option payante. |
| art. 8 | Information à la personne : finalités, moyens de collecte, **possibilité que les renseignements soient communiqués hors Québec** | Politique de confidentialité et contrats clients à mettre à jour, quelle que soit l'option. |
| art. 10 | Mesures de sécurité raisonnables, proportionnées à la sensibilité | Chiffrement, isolation par client, contrôle d'accès. |
| art. 12.1 | Décision fondée exclusivement sur un traitement automatisé : information et droit de faire valoir ses observations | Si le LLM prend des décisions sur des personnes (tri de candidatures, scoring), obligation d'informer. |
| art. 17 | Communication hors Québec : ÉFVP tenant compte de la sensibilité, des finalités, des mesures de protection **et du régime juridique de l'État**; entente écrite; protection adéquate exigée | **L'article central pour la comparaison ci-dessous.** |
| art. 27 et 28.1 | Droit d'accès, de rectification, **portabilité** (en vigueur depuis sept. 2024) et **cessation de diffusion / désindexation** | Le stockage doit permettre d'extraire et d'effacer un dossier complet, sauvegardes et index compris. |
| art. 90.1 et s. | Sanctions administratives jusqu'à 10 M$ ou 2 % du chiffre d'affaires mondial; pénales jusqu'à 25 M$ ou 4 % | Le risque financier justifie de documenter chaque choix. |

### 6.2 Les options de stockage face à la Loi 25

Point de départ : **la Loi 25 n'interdit pas d'héberger hors Québec**. Elle
exige une ÉFVP (art. 17) qui conclut à une protection adéquate, une entente
écrite, et l'information des personnes (art. 8). Toronto, Montréal chez un
fournisseur américain, et un fournisseur québécois n'ont donc pas le même
poids dans l'ÉFVP.

| Critère Loi 25 | DigitalOcean Toronto (`tor1`) | AWS Montréal (`ca-central-1`) | GitHub | Hébergeur québécois / serveur dédié au Québec |
|---|---|---|---|---|
| Données physiquement au Québec | ❌ Ontario | ✅ Québec | ❌ (aucun contrôle de région) | ✅ |
| Communication « hors Québec » (art. 17) | Oui : ÉFVP obligatoire, entente écrite | **Débat.** Données au Québec, mais fournisseur soumis au droit américain (CLOUD Act). La CAI recommande d'évaluer le régime juridique du fournisseur, pas seulement l'emplacement. ÉFVP à faire quand même. | Oui, et sans pouvoir garantir le lieu | Non, si l'entreprise est québécoise et n'a pas de société mère étrangère |
| Régime juridique étranger (CLOUD Act, FISA 702) | Exposé (société américaine) | Exposé (société américaine) | Exposé (Microsoft) | Non exposé |
| Mesures de sécurité (art. 10) | Chiffrement au repos et en transit; BYOK limité | Chiffrement, KMS par client, BYOK complet, CloudTrail, conformité SOC 2 / ISO 27001 | Chiffrement, mais conçu pour du code, pas des données personnelles | Variable selon le fournisseur; à auditer |
| Journalisation pour le registre d'incidents (art. 3.2) | Journaux Spaces limités | Complète (CloudTrail, S3 access logs) | Journal d'audit orienté code | Variable |
| Effacement complet, sauvegardes incluses (art. 28.1) | Faisable : versioning Spaces à gérer | Faisable : versioning + cycle de vie S3 + suppression des sauvegardes | **Difficile** : l'historique Git conserve les données; réécriture d'historique et forks hors de contrôle | Faisable si prévu au contrat |
| Portabilité (art. 27) | Export S3 standard | Export S3 standard | Export possible mais non structuré | Selon le fournisseur |
| Entente écrite conforme (art. 17, 18.3) | DPA standard DigitalOcean | DPA AWS + addendum canadien | Conditions GitHub, non conçues pour des sous-traitants de données personnelles | Contrat négociable, sur mesure |
| Confidentialité par défaut (art. 3.5) | Défaut acceptable | Meilleur défaut disponible chez un hyperscaler | Ne peut pas être le défaut | Meilleur défaut absolu |
| Verdict Loi 25 | **Acceptable** avec ÉFVP documentée. Bon niveau, région hors Québec. | **Recommandé** comme défaut : le meilleur compromis conformité / outils / coût. ÉFVP à documenter sur le point CLOUD Act. | **Non conforme** pour des cerveaux clients. Code seulement. | **Le plus fort** pour les clients à haute sensibilité (santé, juridique, public). Plus cher, moins d'outils. |

Lecture pratique :

- **Défaut pour tous les clients : AWS Montréal.** Données au Québec,
  outillage complet pour l'art. 10 et l'art. 3.2, effacement maîtrisable.
- **DigitalOcean Toronto** reste conforme pour les clients existants, à
  condition que l'ÉFVP art. 17 soit faite et classée, et que les clients
  soient informés (art. 8) que leurs données sont en Ontario.
- **Sortir tout cerveau client de GitHub**, y compris de l'historique. Cette
  action est prioritaire : c'est le seul emplacement actuel qui ne peut pas
  être rendu conforme.
- **Offrir un palier « hébergement québécois »** (fournisseur québécois ou
  serveur dédié) pour les clients dont l'ÉFVP exclut un fournisseur soumis au
  CLOUD Act. Pas besoin de le construire tout de suite; il faut pouvoir le
  proposer.

### 6.3 Les options de LLM face à la Loi 25

Le LLM est un **sous-traitant qui lit les renseignements personnels** à chaque
requête. Les mêmes articles s'appliquent, plus l'art. 12.1 si le modèle
décide.

| Critère Loi 25 | A. LLM entraîné de zéro | B. Modèle à poids ouverts hébergé chez nous (Canada / Québec) | C. API commerciale en région canadienne, zéro rétention (ex. Bedrock `ca-central-1`) | D. API commerciale grand public (hors Canada, rétention par défaut) |
|---|---|---|---|---|
| Communication hors Québec (art. 17) | Non | Non si hébergé au Québec; ÉFVP si à Toronto | Oui au sens large (fournisseur américain), même si l'inférence est à Montréal. ÉFVP requise. | Oui, ÉFVP requise et difficile à conclure favorablement |
| Rétention des requêtes | Contrôlée | Contrôlée | Zéro ou courte, contractuelle | Souvent 30 jours ou plus; parfois utilisées pour l'entraînement |
| Utilisation pour entraîner un autre modèle | Non | Non | Non (contrat commercial) | Possible selon les conditions |
| Mesures de sécurité (art. 10) | À construire entièrement | À notre charge : correctifs, isolation, GPU | Fournisseur certifié + notre couche applicative | Fournisseur certifié, mais flux de données non maîtrisé |
| ÉFVP art. 3.3 requise | Oui | Oui | Oui | Oui |
| Effort et coût | Irréaliste | Élevé, récurrent | Faible | Faible |
| Qualité des réponses | Faible | Bonne | Meilleure disponible | Meilleure disponible |
| Verdict Loi 25 | Aucun avantage de conformité par rapport à B; coût prohibitif | **Le plus solide** : aucune communication à un tiers. Idéal pour les clients à haute sensibilité. | **Conforme et pragmatique** avec ÉFVP documentée et contrat sans entraînement / zéro rétention. Défaut recommandé. | À proscrire pour des cerveaux clients contenant des renseignements personnels. |

### 6.4 Ce que la Loi 25 impose peu importe l'option choisie

1. **Une ÉFVP par projet** : une pour la migration AWS, une pour le
   déploiement du LLM, et une par client dont le cerveau contient des
   renseignements sensibles. Conserver les ÉFVP : la CAI peut les demander.
2. **Un registre par client** : où sont les données, chez qui, sous quel
   contrat, dans quelle région, depuis quand.
3. **Ententes écrites** avec chaque fournisseur (DO, AWS, fournisseur LLM),
   contenant : finalités limitées, mesures de sécurité, notification
   d'incident, destruction en fin de contrat, interdiction de sous-traiter
   sans accord.
4. **Information des personnes** (art. 8) : la politique de confidentialité
   d'IA avec Ugo, et celle de chaque client utilisateur, doivent mentionner
   l'hébergement hors Québec lorsqu'il a lieu.
5. **Procédure d'effacement et de portabilité** testée, sauvegardes et index
   vectoriel inclus.
6. **Registre des incidents** et procédure de notification à la CAI.
7. **Encadrement des décisions automatisées** (art. 12.1) si un cas d'usage
   client s'y rapproche.

### 6.5 Recommandation finale

| Palier | Stockage | Inférence | Clientèle visée |
|---|---|---|---|
| Standard (défaut) | AWS Montréal, KMS par client | API commerciale en région canadienne, zéro rétention, contrat sans entraînement | PME générales |
| Existant | DigitalOcean Toronto, ÉFVP classée | Idem | Clients déjà en place, migration proposée |
| Souverain (premium) | Hébergeur québécois ou serveur dédié au Québec | Modèle à poids ouverts hébergé chez nous | Santé, juridique, secteur public, données très sensibles |

GitHub : code et documentation uniquement, jamais de données clients.
« Construire notre propre LLM » : remplacé par « modèle hébergé de façon
privée » au palier souverain.

---

Sources à citer dans toute communication externe tirée de ce document :
Commission d'accès à l'information du Québec (cai.gouv.qc.ca) pour la Loi 25;
pages officielles de régions AWS et DigitalOcean pour la résidence des
données. Aucun chiffre de ce document ne doit être publié sans vérification.
