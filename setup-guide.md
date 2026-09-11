# Setup n8n - Diagnostic Agent (pas a pas)

## 1) Nodes a creer

1. `When chat message received`
2. `Basic LLM Chain`
3. `OpenAI Chat Model`
4. `Structured Output Parser`
5. `Code in JavaScript`
6. `Data Table -> Insert row`
7. `Notion` → **Database Page** → creer une page (base Diagnostics clients — voir `notion-schema.md`)

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
    "option_a_price_eur": 2000,
    "option_b_price_eur": 3000
  },
  "cta_message": "Si tu veux, je te propose un sprint de 2 semaines pour implementer les quick wins prioritaires."
}
```

## 5) Code node (normalisation + texte client Notion)

- Parse le `chatInput` JSON pour `company_name` / `niche` (titre de page + contexte).
- Construit `notion_rapport` (texte structuré : synthèse, 5 quick wins, ROI, plan 14j, offre, CTA).
- Garde les champs pour `Insert row` inchanges.

```javascript
const o = $json.output || {};

const trigger = $items("When chat message received", 0, 0)[0]?.json || {};
const rawIn = trigger.chatInput || "";
let companyName = "";
let niche = "";
try {
  const parsed = typeof rawIn === "string" ? JSON.parse(rawIn) : {};
  companyName = parsed.company_name || parsed.companyName || "";
  niche = parsed.niche || "";
} catch (_) {}

const quickWins = Array.isArray(o.quick_wins) ? o.quick_wins : [];
const roi = o.roi_estimate || {};
const offer = o.offer_recommendation || {};

const d = new Date();
const dateStr = d.toISOString().slice(0, 10);
const pageTitle = companyName
  ? `Diagnostic — ${companyName} — ${dateStr}`
  : `Diagnostic — ${dateStr}`;

const impactLabel = { high: "Élevé", medium: "Moyen", low: "Faible" };
const effortLabel = { low: "Faible", medium: "Moyen", high: "Élevé" };

const qwLines = quickWins.map((w, i) => {
  const imp = impactLabel[w.impact_level] || w.impact_level || "";
  const ef = effortLabel[w.effort_level] || w.effort_level || "";
  const h = w.expected_hours_saved_weekly || 0;
  const note = (w.implementation_note || "").trim();
  return `${i + 1}. ${w.title || "Quick win"}\n   Impact: ${imp} · Effort: ${ef} · ~${h} h/sem.\n   ${note}`;
});

const plan = Array.isArray(o.plan_14_days) ? o.plan_14_days : [];
const planLines = plan.map((line) => `- ${line}`).join("\n");

const opt = offer.recommended_option || "A";
const rapportParts = [
  "Mini-diagnostic automation",
  niche ? `Contexte: ${niche}` : null,
  "",
  "Synthèse",
  o.diagnostic_summary || "",
  "",
  "5 quick wins",
  qwLines.join("\n\n"),
  "",
  "Estimation ROI (ordre de grandeur)",
  `Heures économisées / mois: ${roi.hours_saved_monthly ?? 0}`,
  `Valeur estimée / mois: ${roi.estimated_value_monthly_eur ?? 0} EUR`,
  `Retour sur investissement (indicatif): ~${roi.payback_period_weeks ?? 0} semaine(s)`,
  "",
  "Plan 14 jours (type sprint)",
  planLines,
  "",
  "Recommandation",
  `Option ${opt} — ${offer.reason || ""}`,
  `Sprint essentiel: ${offer.option_a_price_eur ?? 2000} EUR · Sprint complet: ${offer.option_b_price_eur ?? 3000} EUR`,
  "",
  "Prochaine étape",
  o.cta_message || "",
];
const notionRapport = rapportParts.filter((x) => x !== null).join("\n");

return [{
  json: {
    timestamp: d.toISOString(),
    diagnostic_summary: o.diagnostic_summary || "",
    quick_wins_count: quickWins.length,
    quick_wins: quickWins,
    roi_hours_monthly: roi.hours_saved_monthly || 0,
    roi_value_monthly_eur: roi.estimated_value_monthly_eur || 0,
    recommended_option: opt,
    cta_message: o.cta_message || "",
    company_name: companyName,
    page_title: pageTitle,
    notion_rapport: notionRapport,
  },
}];
```

## 6) Notion — Create diagnostic page

- Creer la database selon `notion-schema.md` (colonnes `Name`, `Rapport client`).
- Node **Notion** → Resource **Database Page** → **Create** (ou importer `diagnostic-agent-v1.json`).
- Mapper `Name` (title) ← `page_title`, `Rapport client` ← `notion_rapport`.
- Brancher **en parallele** de `Insert row` depuis le meme node Code (deux fleches sortantes).

## 7) Data Table insert (table: `diagnostic_logs`)

Colonnes conseillees:
- timestamp (string)
- diagnostic_summary (string)
- quick_wins_count (number)
- roi_hours_monthly (number)
- roi_value_monthly_eur (number)
- recommended_option (string)
- cta_message (string)

## 8) Tests a coller dans chat n8n

### Test marketing
```json
{"niche":"marketing","company_name":"Agence Nova","team_size":"8","tools":["Notion","HubSpot","Google Sheets"],"current_pains":["reporting manuel","prospection lente","onboarding repetitif"],"hours_lost_per_week":12}
```

### Test odoo
```json
{"niche":"odoo","company_name":"Integrateur Alpha","team_size":"6","tools":["Odoo v16","GitLab","WhatsApp"],"current_pains":["support tickets repetitifs","devis non standardises","suivi migration v17"],"hours_lost_per_week":10}
```
