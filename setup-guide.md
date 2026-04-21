# Setup n8n - Diagnostic Agent (pas a pas)

## 1) Nodes a creer

1. `When chat message received`
2. `Basic LLM Chain`
3. `OpenAI Chat Model`
4. `Structured Output Parser`
5. `Code in JavaScript`
6. `Data Table -> Insert row`

## 2) OpenAI Chat Model

- Model: `gpt-4o-mini`
- Temperature: `0.3`
- Max tokens: `1200`

## 3) Basic LLM Chain

- Coller le contenu de `prompt-template-v1.md`
- Connecter:
  - `OpenAI Chat Model` au port `Model`
  - `Structured Output Parser` au port `Output Parser`

## 4) Structured Output Parser (JSON example)

```json
{
  "diagnostic_summary": "Le goulot principal est...",
  "quick_wins": [
    {
      "title": "Automatiser le reporting hebdo",
      "impact_level": "high",
      "effort_level": "medium",
      "expected_hours_saved_weekly": 4,
      "implementation_note": "Connecter sources et generer un rapport auto chaque lundi."
    },
    {
      "title": "Qualification leads automatique",
      "impact_level": "high",
      "effort_level": "low",
      "expected_hours_saved_weekly": 3,
      "implementation_note": "Scoring simple base sur source, budget, urgence."
    },
    {
      "title": "Relances prospects semi-auto",
      "impact_level": "medium",
      "effort_level": "low",
      "expected_hours_saved_weekly": 2,
      "implementation_note": "Relances J+2/J+5 avec templates dynamiques."
    },
    {
      "title": "Onboarding client checklists auto",
      "impact_level": "medium",
      "effort_level": "medium",
      "expected_hours_saved_weekly": 2,
      "implementation_note": "Creation de taches et docs a la signature."
    },
    {
      "title": "Alertes KPI hebdo",
      "impact_level": "low",
      "effort_level": "low",
      "expected_hours_saved_weekly": 1,
      "implementation_note": "Notifier seulement les ecarts importants."
    }
  ],
  "roi_estimate": {
    "hours_saved_weekly": 12,
    "hours_saved_monthly": 48,
    "estimated_value_monthly_eur": 1200,
    "payback_period_weeks": 3
  },
  "plan_14_days": [
    "Jour 1-2: cadrage et mapping process",
    "Jour 3-5: build workflow principal",
    "Jour 6-10: integrations + tests",
    "Jour 11-14: passation + optimisation"
  ],
  "offer_recommendation": {
    "recommended_option": "A",
    "reason": "priorite au process qui fait perdre le plus d'heures",
    "option_a_price_eur": 900,
    "option_b_price_eur": 1500
  },
  "cta_message": "Si tu veux, je te propose un sprint de 2 semaines pour implementer les quick wins prioritaires."
}
```

## 5) Code node (normalisation)

```javascript
const o = $json.output || {};

const quickWins = Array.isArray(o.quick_wins) ? o.quick_wins : [];
const roi = o.roi_estimate || {};
const offer = o.offer_recommendation || {};

return [{
  json: {
    timestamp: new Date().toISOString(),
    diagnostic_summary: o.diagnostic_summary || "",
    quick_wins_count: quickWins.length,
    quick_wins: quickWins,
    roi_hours_monthly: roi.hours_saved_monthly || 0,
    roi_value_monthly_eur: roi.estimated_value_monthly_eur || 0,
    recommended_option: offer.recommended_option || "A",
    cta_message: o.cta_message || ""
  }
}];
```

## 6) Data Table insert (table: `diagnostic_logs`)

Colonnes conseillees:
- timestamp (string)
- diagnostic_summary (string)
- quick_wins_count (number)
- roi_hours_monthly (number)
- roi_value_monthly_eur (number)
- recommended_option (string)
- cta_message (string)

## 7) Tests a coller dans chat n8n

### Test marketing
```json
{"niche":"marketing","company_name":"Agence Nova","team_size":"8","tools":["Notion","HubSpot","Google Sheets"],"current_pains":["reporting manuel","prospection lente","onboarding repetitif"],"hours_lost_per_week":12}
```

### Test odoo
```json
{"niche":"odoo","company_name":"Integrateur Alpha","team_size":"6","tools":["Odoo v16","GitLab","WhatsApp"],"current_pains":["support tickets repetitifs","devis non standardises","suivi migration v17"],"hours_lost_per_week":10}
```
