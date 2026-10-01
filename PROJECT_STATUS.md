# PROJECT_STATUS — K RENTING (EURL K-RENTIGN)

_Dernière mise à jour : 29 septembre 2026_

## 1. Projet
- **Nom :** K RENTING (marque affichée sur le site : K-RENTIGN)
- **Objet :** plateforme de location de matériel lourd et d'engins en Algérie (matériel K-RENTIGN + matériel de propriétaires tiers validé par l'administration).
- **Langue :** français (structure prête pour l'arabe plus tard, sans traduction automatique).

## 2. Technologies
- TanStack Start v1 (React 19, SSR, server functions) + TanStack Router (routage par fichiers) + TanStack Query
- TypeScript (mode strict, `exactOptionalPropertyTypes`)
- Vite 8, Tailwind CSS v4 (configuré dans `src/styles.css`), shadcn/ui (Radix UI), lucide-react
- Zod (validation des formulaires côté serveur), sonner (notifications)
- Supabase (base Postgres, stockage, auth) — hébergé via Lovable Cloud
- Cible de déploiement : Cloudflare Workers (via Nitro) ; fonctionne aussi en Node pour le dev
- Gestionnaire de paquets : bun (`bun.lock`) ; npm fonctionne aussi

## 3. Structure
```text
public/                  favicon.ico, robots.txt
src/
  assets/                k-rentign-hero.jpg (image du Hero)
  components/            site-layout (en-tête, menu mobile, pied de page),
                         page-primitives, form-parts, equipment-card
  components/ui/         composants shadcn/ui
  hooks/                 use-mobile
  integrations/supabase/ clients Supabase (navigateur + serveur), types générés, middlewares auth
  integrations/lovable/  aide OAuth Lovable Cloud (Google)
  lib/                   public.functions.ts (server functions), equipment-display.ts
                         (58 wilayas, catégories, libellés), utils, gestion d'erreurs
  routes/                pages (voir §5) + __root.tsx (shell, 404, erreurs)
  router.tsx, start.ts, server.ts, styles.css, routeTree.gen.ts (généré)
supabase/
  config.toml
  migrations/            4 fichiers SQL (schéma, RLS, stockage)
vite.config.ts, tsconfig.json, eslint.config.js, components.json, package.json, bun.lock
```

## 4. Fonctionnalités existantes
- En-tête fixe avec navigation (ACCUEIL, ÉQUIPEMENTS, LOUER MON MATÉRIEL, COMMENT ÇA MARCHE, À PROPOS, CONTACT) + bouton DEMANDER UNE LOCATION + menu mobile.
- Accueil : Hero, 2 boutons, barre de recherche (texte + wilaya), aperçu des équipements publiés.
- Catalogue avec recherche texte et filtres (type, wilaya, disponibilité, chauffeur, prix max) stockés dans l'URL ; message « Aucun équipement ne correspond à votre recherche. » + lien vers la demande personnalisée.
- Fiche équipement : galerie, caractéristiques, badge K-RENTIGN / PARTENAIRE, formulaire DEMANDER UNE LOCATION.
- Formulaires enregistrés en base : demande de location, contact, demande personnalisée, soumission de matériel (avec envoi de photos, 8 max, 8 Mo).
- Message de confirmation : « Votre demande a bien été envoyée. Notre équipe vous contactera prochainement. »
- Seuls les équipements au statut `published` sont visibles publiquement (règle RLS).
- Page 404 et page d'erreur en français ; metadata SEO par page.

## 5. Pages / routes
| URL | Fichier |
|---|---|
| `/` | `src/routes/index.tsx` |
| `/equipements` | `src/routes/equipements.index.tsx` |
| `/equipements/:slug` | `src/routes/equipements.$slug.tsx` |
| `/louer-mon-materiel` | `src/routes/louer-mon-materiel.tsx` |
| `/demande-personnalisee` | `src/routes/demande-personnalisee.tsx` |
| `/comment-ca-marche` | `src/routes/comment-ca-marche.tsx` |
| `/a-propos` | `src/routes/a-propos.tsx` |
| `/contact` | `src/routes/contact.tsx` |
| 404 | `notFoundComponent` dans `src/routes/__root.tsx` |

## 6. Base de données (Supabase / Postgres)
Enums : `app_role`, `publication_status`, `availability_status`, `request_status`, `price_unit`, `driver_mode`, `owner_type`.

| Table | Rôle | Accès public (RLS) |
|---|---|---|
| `equipment` | catalogue | lecture des lignes `published` uniquement |
| `rental_requests` | demandes de location | insertion (statut `new`) |
| `equipment_submissions` | matériel proposé par des tiers | insertion (statut `pending`) |
| `custom_requests` | demandes personnalisées | insertion |
| `contact_messages` | messages de contact | insertion |
| `user_roles` | rôles (admin…) | chaque utilisateur lit ses propres rôles |

- Les administrateurs gèrent tout via la fonction `private.has_role(uid, 'admin')` (schéma `private`, non exposé).
- Triggers `updated_at` sur toutes les tables métier.
- Stockage : buckets privés `equipment-submissions` (photos envoyées par le public) et `equipment-media` (photos du catalogue, lecture publique via politique). Les politiques de stockage sont dans les migrations.
- Toutes les migrations sont dans `supabase/migrations/` (à appliquer dans l'ordre).

## 7. Authentification
- Supabase Auth configuré (email/mot de passe + Google) côté backend.
- **Aucune page de connexion ni espace admin n'existe encore dans le site.**
- Les rôles sont dans `user_roles` ; aucun administrateur n'est encore attribué.

## 8. Services externes
- Supabase (base, stockage, auth).
- Google Fonts (Archivo, Work Sans).
- `@lovable.dev/cloud-auth-js` et `src/integrations/lovable/` : aide de connexion Google propre à Lovable Cloud (non utilisée par les pages actuelles).
- `@lovable.dev/vite-tanstack-config` : configuration Vite (paquet npm public, fonctionne hors Lovable).
- Aucun paiement, aucun e-mail transactionnel, aucune IA.

## 9. Réalisé
- Base de données, RLS, stockage, sécurité des rôles.
- Identité visuelle (palette industrielle orange/marine, typographies), en-tête/pied de page, menu mobile.
- Les 8 pages publiques, formulaires reliés à la base, recherche et filtres.
- Build OK, typecheck OK, vérification mobile/desktop, envoi d'une demande personnalisée testé.

## 10. Non réalisé
- Espace d'administration (ajout/modification/suppression d'équipements, disponibilité, suivi des demandes, validation/refus des soumissions).
- Page de connexion admin et attribution du premier administrateur.
- Pages légales (en attente du contenu officiel).
- Coordonnées réelles (téléphone, e-mail, adresse) : affichées « À compléter ».
- Version arabe.

## 11. Problèmes connus
- Catalogue vide : aucun équipement publié → formulaire de location sur fiche non testé en réel.
- Envoi de photos dans « Louer mon matériel » non testé de bout en bout.
- Sans espace admin, les soumissions ne peuvent être validées que directement en base.
- Les photos soumises sont dans un bucket privé : leur affichage nécessitera l'espace admin (URLs signées).

## 12. Dernières modifications
- Création des pages Équipements, fiche, Louer mon matériel, Demande personnalisée, Comment ça marche, À propos, Contact.
- Correction des erreurs TypeScript (paramètres de recherche).
- Ajout de ce fichier et du README (aucune modification fonctionnelle).

## 13. Lancer hors Lovable (résumé)
1. Installer Node.js 22 LTS.
2. `npm install`
3. Créer `.env` (voir README) avec l'URL et la clé publique Supabase.
4. Nouveau projet Supabase : appliquer `supabase/migrations/*.sql` dans l'ordre, créer les buckets s'ils n'existent pas, configurer l'auth.
5. `npm run dev` → http://localhost:8080 ; production : `npm run build`.
Détails complets : `README.md`.
