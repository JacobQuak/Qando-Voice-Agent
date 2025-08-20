# Qando-Voice-Agent

Deze workflow automatiseert het verwerken van data die afkomstig is van een voice call binnen Vapi AI, zoals gegenereerd door een virtuele agent. De workflow wordt geactiveerd zodra een gesprek is afgerond en de data beschikbaar komt.

## ⚙️ Functionaliteit

### Webhook Trigger
Ontvangt een `POST`-verzoek van de voice agent zodra een call is afgelopen. Deze bevat het volledige transcript van het gesprek.

### Survey Data Ophalen
Haalt de originele surveyvragen op uit SurveyMonkey via een HTTP Request, inclusief toegangsheader.

### Structureren van Antwoorden
Een AI-agent ontvangt de ruwe transcriptie, analyseert deze en structureert de antwoorden van de gebruiker in het volgende JSON-format:

```json
{
  "survey_data": [
    {
      "question": "Welke AI-tools gebruik je?",
      "user_answer": ["ChatGPT"]
    },
    ...
  ]
}
```
### Antwoorden Koppelen aan Vragen
De tweede AI-agent vergelijkt de user-antwoorden uit de call met de vragen uit de originele survey, en koppelt ze zo goed mogelijk aan elkaar.

### Verwerken van Output
Een stukje custom JavaScript haalt hieruit de uiteindelijke ‘vraag & antwoord’-paren voor verdere verwerking of opslag.



## 📦 Gebruikte Tools

- `n8n Webhook`
- `SurveyMonkey API`
- Twilio
- Custom `JavaScript`-code voor parsing

## 🔐 Veiligheid

Alle API-aanroepen zijn voorzien van juiste authenticatieheaders. Gevoelige data blijft binnen de veilige omgeving van n8n.

## 📈 Use Case

Deze workflow is ideaal voor het automatisch verwerken van telefonische interviews of voice-based surveys in zakelijke contexten, met als doel inzichten te verkrijgen over AI-gebruik in organisaties.
