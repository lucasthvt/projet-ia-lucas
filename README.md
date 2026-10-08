# Projet IA — BTS SIO 2 SLAM — Lucas THEVENET — octobre 2026

URL publique : https://[…].trycloudflare.com
Code d'accès envoyé à l'enseignant par e-mail

---

## 1. Concevoir

**Sujet choisi :** Sujet 2 — Tri des demandes de support

**L'organisation (fictive) et son besoin :**
L'Atelier Numérique du Pilat gère le parc informatique de six petites mairies et de leurs écoles.
Aujourd'hui, les deux techniciens reçoivent toutes les demandes dans une seule boîte mail, sans tri, et un simple « merci » arrive au même niveau qu'une panne bloquant toute une mairie.
L'application classe et priorise automatiquement chaque message en JSON, pour que les techniciens traitent d'abord les urgences.

**Trois cas d'usage :**

- En tant que technicien, je veux recevoir chaque demande déjà classée par catégorie et priorité, afin de traiter les urgences en premier.
- En tant que responsable, je veux savoir combien de postes sont concernés par une panne, afin d'anticiper les interventions.
- En tant que secrétaire de mairie, je veux que ma demande soit transmise rapidement, afin de ne pas rester bloquée.

**Ce que l'application ne fait pas (limites assumées) :**

- Elle ne répond pas à l'utilisateur : elle produit uniquement un JSON de tri.
- Elle ne résout pas l'incident : elle ne fait que classer et prioriser.
- Elle ne traite pas les pièces jointes ou les images.

---

## 2. Le modèle et la machine

| | |
|---|---|
| Carte graphique et mémoire vidéo (VRAM) | Apple M4, mémoire unifiée 16 Go |
| Mémoire vive | 16 Go |
| Modèle retenu | qwen3:8b |
| Pourquoi celui-là | Tient confortablement dans 16 Go avec Docker en parallèle ; bon compromis vitesse/qualité ; testé à 6,89 tokens/s |
| Modèle comparé | qwen3:4b |

---

## 3. Piloter — le journal

| Séance | Ce qui est fait | Ce qui a bloqué, et comment c'est réglé |
|---|---|---|
| Lundi 05/10 | Installation Ollama + Docker, choix du sujet 2, création du dépôt, rédaction du prompt.txt, premiers tests locaux, écriture de cas.json | [à remplir] |
| Mardi 06/10 | Dockerisation, mise en ligne via tunnel, tests sur URL publique, mesures, sécurisation | [à remplir] |

---

## 4. Mesurer

Jeu de 10 cas au moins dans cas.json, dont au moins deux hors sujet et un qui tente de détourner les consignes.

| | Modèle retenu (qwen3:8b) | Modèle comparé (qwen3:4b) |
|---|---|---|
| Réussite (sur N cas × 3 essais) | … % | … % |
| Temps de réponse médian | … s | … s |

**Ce que les échecs montrent (deux exemples commentés, et ce que vous avez changé) :**
- [à remplir après les tests]
- [à remplir après les tests]

---

## 5. Sécuriser

| Risque | Ce qui pourrait arriver | Mesure prise dans le projet |
|---|---|---|
| L'URL est publique | n'importe qui utilise votre PC et votre électricité | code d'accès, limite de requêtes |
| Ollama exposé | quelqu'un utilise votre modèle directement | Ollama non exposé, seul le conteneur applicatif parle à Ollama |
| Détournement des consignes | l'utilisateur demande au modèle d'ignorer ses règles | consigne système stricte + test dédié dans cas.json |
| Données personnelles | des noms ou infos personnelles apparaissent dans les résumés | consigne « résumé sans nom de personne » + vérification |
| Secrets dans le dépôt | le code d'accès fuite sur GitHub | .env non commité, .env.example fourni |

---

## 6. Mettre en production — comment refaire

```bash
cp .env.example .env      # puis remplir
docker compose up -d --build
docker compose ps
docker compose logs tunnel
