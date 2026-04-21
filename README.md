# Demo #2 - Diagnostic Automation Agent (n8n)

Deuxieme demo orientee vente: produire un mini-audit automation a partir d'un contexte entreprise.

## Objectif business

Transformer un prospect "curieux" en prospect "chaud" avec un livrable concret:
- 5 quick wins classes par impact
- estimation ROI simple
- plan d'action 14 jours
- proposition de sprint payant

Cette demo sert d'entree naturelle vers l'offre:
- Option A: Sprint essentiel (900 EUR)
- Option B: Sprint complet (1500 EUR)

## Entree attendue (chat n8n)

JSON recommande:

```json
{
  "niche": "marketing",
  "company_name": "Agence X",
  "team_size": "8",
  "tools": ["Notion", "HubSpot", "Google Sheets"],
  "current_pains": [
    "reporting manuel",
    "prospection lente",
    "onboarding client repetitif"
  ],
  "hours_lost_per_week": 12
}
```

## Sortie attendue

- diagnostic_summary (string)
- quick_wins (array 5 objets)
- roi_estimate (objet)
- plan_14_days (array)
- offer_recommendation (objet)
- cta_message (string)

## Architecture n8n (V1)

1. `When chat message received`
2. `Basic LLM Chain`
3. `OpenAI Chat Model`
4. `Structured Output Parser`
5. `Code in JavaScript` (normalisation + fallback)
6. `Data Table -> Insert row` (`diagnostic_logs`)

## Pourquoi cette demo vend mieux que la demo #1

- Demo #1 montre "capabilite technique"
- Demo #2 montre "valeur business + argent/temps gagne"
- Le prospect visualise directement son ROI

## Tests minimum avant usage client

Tester sur 3 cas:
1. Agence marketing (5-15 personnes)
2. Integrateur Odoo (3-10 personnes)
3. PME service (operations manuelles)

Definition de done:
- JSON propre
- Quick wins actionnables (pas de blabla)
- ROI coherent
- CTA naturel vers sprint
