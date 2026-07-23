# Kréyòl gwiyanè — collecte de traductions

Micro-application statique (HTML / CSS / JS vanilla, sans build, sans npm) pour recueillir
des traductions français → créole guyanais auprès d'un petit groupe de locuteurs.

```
index.html     entrée par code + profil (une seule fois)
collecte.html  écran de traduction, une phrase à la fois
style.css      styles communs (clair / sombre)
config.js      URL Supabase + clé publiable
```

Les données vont dans Supabase (tables `kreyol_participant`, `kreyol_item`, `kreyol_reponse`)
via l'API REST, en `fetch` direct.

## Publier sur GitHub Pages

1. Créer un dépôt GitHub et y pousser ces fichiers à la racine (pas dans un sous-dossier).
2. Dans le dépôt : **Settings → Pages**.
3. *Source* : **Deploy from a branch**. *Branch* : **main**, dossier **/ (root)**. **Save**.
4. Au bout d'une minute, le site est en ligne sur `https://<compte>.github.io/<depot>/`.

Vérifier avant de diffuser le lien :

- `config.js` contient bien l'URL du projet et la clé **publiable** (`anon`), jamais la clé
  `service_role`.
- Les politiques RLS autorisent, pour le rôle `anon` : lecture de `kreyol_item`, lecture de
  `kreyol_participant`, lecture / insertion / mise à jour de `kreyol_reponse`.
  Sans cela, l'application affiche « Code inconnu » ou ne charge aucune phrase.

Test local, sans rien installer :

```sh
python3 -m http.server 8000   # puis ouvrir http://localhost:8000
```

(Ouvrir `index.html` par double-clic ne suffit pas : `fetch` a besoin d'un vrai serveur.)

## Créer les comptes participants

Un participant = une ligne dans `kreyol_participant` avec un `code` unique. Le reste du profil
(prénom, âge, origine…) est demandé à la personne à sa première connexion, puis n'est plus
jamais redemandé.

Dans le SQL Editor de Supabase :

```sql
insert into kreyol_participant (code) values
  ('GWIYAN-01'), ('GWIYAN-02'), ('GWIYAN-03'), ('GWIYAN-04');
```

Communiquer à chaque personne : le lien du site et son code (écrit en gros, avec le tiret).
Le code se saisit sans tenir compte de la casse : la page passe tout en majuscules.

## Ce qu'il faut savoir sur le fonctionnement

- La progression est reprise automatiquement à la première phrase sans réponse : la personne
  peut fermer l'onglet et revenir des jours plus tard.
- La réponse est enregistrée à chaque changement de phrase, et automatiquement toutes les
  5 secondes si le texte a changé.
- Si le réseau tombe, la réponse est gardée dans le navigateur (`localStorage`) et repartie
  toute seule au retour de la connexion. L'utilisateur n'est jamais bloqué par une erreur.
- Le champ `cible_grammaticale` des items est une note interne : il n'est jamais affiché.
- L'appareil garde le participant connecté. Sur un téléphone partagé, utiliser
  « Ce n'est pas moi, changer de code » sur la page d'entrée.

## Relire les réponses

```sql
select p.code, i.id, i.fr, r.creole, r.registre, r.certitude, r.passe, r.note
from kreyol_reponse r
join kreyol_participant p on p.id = r.participant_id
join kreyol_item i on i.id = r.item_id
order by p.code, i.id;
```

`certitude` : 5 = sûr, 3 = à peu près, 1 = pas sûr. `passe = true` = « je ne sais pas ».
