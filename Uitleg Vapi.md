# Qando AI Interview Agent – VAPI → n8n Integratie

Deze repository documenteert hoe we een **AI-voice-agent** hebben opgezet in [VAPI](https://vapi.ai) die een vastgestelde vragenlijst (interview) afneemt, antwoorden opslaat in **Structured Data** en na afloop automatisch naar een **n8n workflow** stuurt.

## 📋 Overzicht

- **Doel**: Een gestructureerd interview uitvoeren en antwoorden opslaan als gestructureerde JSON-variabelen.
- **Platform**: [VAPI](https://vapi.ai) voice assistant.
- **Integratie**: [n8n](https://n8n.io) webhook (Production URL).
- **Output**: JSON met ingevulde velden + transcript van het gesprek.

---

## 1️⃣ Interview Script in VAPI

We hebben in **Model → Instructions** van VAPI een duidelijke system prompt gezet die:
- De AI vraagt om een vaste reeks vragen te stellen.
- Eén vraag per beurt behandelt.
- Antwoorden bevestigt voor doorgaan.
- De taal in het Nederlands houdt.
- Geen variabelen noemt tijdens het gesprek.

> **Tip**: Zet al je vragen in een vaste volgorde in de prompt. Gebruik bevestigingen (“Ik noteer: … Klopt dat?”) voor duidelijkheid.

---

## 2️⃣ Structured Data instellen

In **Advanced → Structured Data**:
1. **Enable Structured Data** = ✅  
2. **Data Schema**: JSON-schema met alle velden, bijv.:
