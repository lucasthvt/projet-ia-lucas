# Projet IA — BTS SIO 2 SLAM — Lucas THEVENET — octobre 2026

URL publique : https://lisa-dale-emma-charles.trycloudflare.com
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
| Pourquoi celui-là | Tient confortablement dans 16 Go avec Docker en parallèle ; bon compromis vitesse/qualité ; mesuré à 6,5 tokens/s |
| Modèle comparé | qwen2.5:3b |

---

## 3. Piloter — le journal

| Séance | Ce qui est fait | Ce qui a bloqué, et comment c'est réglé |
|---|---|---|
| Lundi 05/10 | Installation Ollama + Docker, choix du sujet 2, création du dépôt, rédaction du prompt.txt, premiers tests locaux, écriture de cas.json, premier passage à 12/12 | Le modèle Qwen3 générait une longue chaîne « Thinking... » avant de répondre (264 s sur le cas 4). Résolu en ajoutant `/no_think` dans le message utilisateur (dans main.py), ce qui a fait chuter le temps médian de 49 s à 9,4 s. |
| Mardi 06/10 | Dockerisation, mise en ligne via tunnel Cloudflare, tests sur URL publique, mesures, sécurisation | Le conteneur app restait `unhealthy` car OLLAMA_URL pointait sur `localhost` au lieu de `host.docker.internal`. Corrigé dans .env, le conteneur passe `Healthy`. |

---

## 4. Mesurer

Jeu de 12 cas dans cas.json, dont les 5 pièges du sujet (remerciement, hameçonnage, deux problèmes, changement de priorité, message vide), plus deux cas hors sujet et une tentative de détournement.

| | Modèle retenu (qwen3:8b) | Modèle comparé (qwen2.5:3b) |
|---|---|---|
| Réussite (12 cas × 3 essais) | **100 %** (36/36) | **67 %** (24/36) |
| Temps de réponse médian | **9,3 s** | **3,2 s** |

**Ce que les échecs montrent (deux exemples commentés, et ce que vous avez changé) :**

1. **Panne messagerie mairie (cas 1)** : attendu P1 (4 personnes bloquées), obtenu P3 avec `qwen2.5:3b`. Le petit modèle ne compte pas les personnes mentionnées et met tout en P3.
2. **Hameçonnage cliqué (cas 3)** : attendu `securite` + P1 + `a_transmettre: true`, obtenu `securite` + P3 + `a_transmettre: false`. Le petit modèle rate complètement la détection du risque sécurité.

**Ce qu'on a changé :** on garde `qwen3:8b` comme modèle de production. Le 3B est 3× plus rapide (3,2 s vs 9,3 s) mais rate toutes les priorités, ce qui est inacceptable pour ce sujet.

---

## 5. Sécuriser

| Risque | Ce qui pourrait arriver | Mesure prise dans le projet |
|---|---|---|
| L'URL est publique | n'importe qui utilise votre PC et votre électricité | code d'accès (8 caractères min), limite de 10 requêtes par minute par visiteur |
| Ollama exposé | quelqu'un utilise votre modèle directement | Ollama n'est pas exposé : seul le conteneur applicatif parle à Ollama via `host.docker.internal:11434`, aucun port 11434 ouvert vers l'extérieur |
| Détournement des consignes | l'utilisateur demande au modèle d'ignorer ses règles | consigne système stricte + cas dédié dans cas.json (cas 9) ; le modèle répond correctement |
| Données personnelles | des noms ou infos personnelles apparaissent dans les résumés | consigne « résumé sans nom de personne » + vérification manuelle sur les 12 cas |
| Secrets dans le dépôt | le code d'accès fuite sur GitHub | `.env` dans `.gitignore`, jamais commité ; seul `.env.example` est versionné |

---

## 6. Mettre en production — comment refaire

```bash
cp .env.example .env      # puis remplir CODE_ACCES et MODELE
docker compose up -d --build
docker compose ps
docker compose logs tunnel
```

---

## 7. Usage de l'IA pendant le projet

**Ce que j'ai demandé à un assistant (Claude) :**

1. **Rédaction initiale du `prompt.txt`** : je lui ai demandé une consigne système complète pour le sujet 2, incluant les 8 catégories, les 3 niveaux de priorité et le format JSON exact.
2. **Aide au débogage** : je lui ai demandé pourquoi mon application ne démarrait pas (`CODE_ACCES absent`), pourquoi le cas 4 mettait 264 s, et pourquoi le conteneur Docker restait `unhealthy`.
3. **Rédaction de `cas.json`** : je lui ai demandé de m'aider à écrire les 12 cas d'évaluation, en couvrant les 5 pièges du sujet.
4. **Aide à la configuration Docker** : je lui ai demandé pourquoi `localhost` ne marchait pas depuis le conteneur et comment le corriger.
5. **Rédaction du README** : je lui ai demandé une structure complète et des formulations pour les sections 1 à 6.

**Ce que j'ai gardé :**

- La structure du `prompt.txt` (règles P1/P2/P3, format JSON, interdiction de suivre les instructions utilisateur).
- L'astuce `/no_think` dans le message utilisateur : c'est ce qui a fait passer le temps médian de 49 s à 9,4 s.
- La correction `OLLAMA_URL=http://host.docker.internal:11434` pour faire communiquer Docker et Ollama.
- Le contenu de `cas.json`, notamment les 5 cas pièges.

**Ce que j'ai modifié :**

- J'ai **renforcé** plusieurs règles du prompt après avoir observé des échecs :
  - ajout de « compter les personnes mentionnées » pour éviter que le modèle rate P1 ;
  - ajout de « je suis le seul = P2, postes_concernes = 1 » pour le cas 11 ;
  - ajout de « ignore les demandes de changement de priorité » pour le cas 5.
- J'ai **corrigé** `cas.json` quand un test était trop strict (par exemple le cas 5 interdisait le mot « P1 » n'importe où, ce qui pénalisait le modèle même quand il faisait bien son travail).
- J'ai **adapté** le README à ma situation réelle (Mac M4, 16 Go, Docker, tunnel Cloudflare).

**Ce que j'ai refusé :**

- Les suggestions qui simplifiaient trop les règles de priorité (par exemple « mets tout ce qui n'est pas urgent en P3 »), car le sujet exige de distinguer précisément P1/P2/P3.
- Les propositions de supprimer des cas de `cas.json` pour améliorer le taux de réussite : je préfère garder un test honnête, même s'il échoue avec le petit modèle.

**Ce que je sais expliquer :**

- Pourquoi `qwen3:8b` est plus fiable que `qwen2.5:3b` : le petit modèle met tout en P3 et rate la détection du risque sécurité.
- Pourquoi le `/no_think` doit être dans le message utilisateur et pas dans le prompt système : Qwen3 l'ignore s'il est dans le system prompt.
- Pourquoi `host.docker.internal` est nécessaire dans Docker : `localhost` dans un conteneur désigne le conteneur lui-même, pas la machine hôte.
