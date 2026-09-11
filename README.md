# Diagnostic Automation Agent (n8n)

Deuxième démo orientée vente : mini-audit automation avec 5 quick wins chiffrés.

## Quel fichier importer ?

| Fichier | Contexte | IA |
|---------|----------|-----|
| **`diagnostic-agent-demo-ollama.json`** | **Loom / démo portfolio** | **Ollama local** (0 €) |
| `diagnostic-agent-v1.json` | **Prod / usage réel** | OpenAI GPT-4o-mini (JSON structuré fiable) |

> Vision complète démo vs prod : [`../VISION-CLIENT-COMPLETE.md`](../VISION-CLIENT-COMPLETE.md)

---

## Objectif business

Transformer un prospect « curieux » en prospect « chaud » :

- 5 quick wins classés par impact
- estimation ROI simple
- plan d'action 14 jours
- proposition sprint 2 000 € / 3 000 €

---

## Démo Ollama (5 min)

```bash
ollama serve
ollama list   # qwen2.5:14b
```

1. n8n → Import → `diagnostic-agent-demo-ollama.json`
2. `ollama_url` : `http://host.docker.internal:11434` si n8n Docker
3. ▶️ **Lancer diagnostic démo**
4. Lire **Résumé démo** (+ page Notion si creds configurées)

Entrée exemple déjà dans le workflow (agence SEA 12 personnes). Modifier le nœud **Contexte prospect** pour un autre cas.

---

## Production (`diagnostic-agent-v1.json`)

Architecture LangChain :

1. Chat trigger
2. Basic LLM Chain + OpenAI + Structured Output Parser
3. Code → Notion + Data Table

Voir `setup-guide.md` et `notion-schema.md`.

---

## Tests minimum (prod)

3 cas : agence marketing · intégrateur Odoo · PME service.

Done = JSON propre · 5 quick wins · ROI cohérent · CTA sprint.
