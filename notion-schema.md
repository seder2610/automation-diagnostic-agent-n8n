# Notion — base « Diagnostics clients »

Cette base sert de **livrable lisible** pour le prospect, en parallèle du log interne n8n (`diagnostic_logs`).

## Créer la base dans Notion

1. Créer une **database** (pleine page ou inline).
2. La nommer, par ex. `Diagnostics clients`.
3. Partager la base avec l’**intégration** que tu as connectée à n8n (mêmes droits que pour les autres workflows Notion).
4. Récupérer l’**ID** de la base (URL : `https://notion.so/WORKSPACE/XXXXXXXX` — l’ID est le segment de 32 caractères, à reformater en UUID si besoin).

## Propriétés obligatoires (noms exacts)

Ces noms doivent correspondre au node **Create diagnostic page** (export JSON). Si tu renommes une colonne dans Notion, mets à jour le node (`Name|title`, `Rapport client|rich_text`).

| Propriété Notion   | Type        | Rôle |
|--------------------|------------|------|
| `Name`             | **Title**  | Titre de la page (généré : `Diagnostic — {entreprise} — {date}`) |
| `Rapport client`   | **Text**   | Contenu structuré (synthèse, quick wins, ROI, plan 14j, offre, CTA) |

Si ton interface Notion a créé la première colonne en **« Nom »** au lieu de « Name », soit tu renommes la propriété en `Name`, soit tu changes la clé dans n8n en `Nom|title`.

> Astuce : le type **Text** (plain) suffit ; en **Rich text** n8n mappe aussi via `rich_text`.

## Configurer n8n après import

1. Ouvrir le node **Create diagnostic page**.
2. Remplacer l’ID base factice `aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa` par l’ID réel de ta database (ou sélectionner la base dans la liste).
3. Vérifier que les **clés de propriétés** chargées correspondent (`Name`, `Rapport client`). Sinon, réassigner dans le dropdown du node.
4. Associer les **credentials Notion** si l’import les a perdus.

## Récupérer le lien à envoyer au prospect

Après exécution, le node Notion retourne l’objet page (notamment `url` / `id` selon la version n8n). Copie l’**URL publique** ou partage la page (Notion : Share → partage web selon ton choix).

## Optionnel (évolution)

- Colonne `Statut` (Select) : `Brouillon` / `Envoyé`
- Colonne `Valeur mensuelle EUR` (Number) — dupliquer `roi_value_monthly_eur` pour des vues filtrées
- Colonne `Option` (Select) : `A`, `B` — aligner les options sur le LLM

Ajouter ces champs dans le node n8n **Create diagnostic page** une fois les colonnes créées dans Notion.
