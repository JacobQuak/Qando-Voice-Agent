# Qando AI Interview Agent – VAPI → n8n Integratie

Deze repository documenteert hoe we een **AI-voice-agent** hebben opgezet in [VAPI](https://vapi.ai) die een vastgestelde vragenlijst (interview) afneemt, antwoorden opslaat in **Structured Data** en na afloop automatisch naar een **n8n workflow** stuurt.

## 📋 Overzicht

- **Doel**: Een gestructureerd interview uitvoeren en antwoorden opslaan als gestructureerde JSON-variabelen.
- **Platform**: [VAPI](https://vapi.ai) voice assistant.
- **Integratie**: [n8n](https://n8n.io) webhook (Production URL).
- **Output**: JSON met ingevulde velden + transcript van het gesprek.

---

## 1️⃣ Interview Script in VAPI

We hebben in **Prompts --> system prompt** een duidelijke system prompt gezet die:
- De AI vraagt om een vaste reeks vragen te stellen.
- Eén vraag per beurt behandelt.
- Antwoorden bevestigt voor doorgaan.
- De taal in het Nederlands houdt.
- Geen variabelen noemt tijdens het gesprek.
- Voor een natuurlijke opening van het gesprek, zet *First Message Mode* op *Assistant speaks first with model generated message*. Vervolgens kan er in het system prompt een vaste opening van het gesprek gezet worden.

> **Tip**: Zet al je vragen in een vaste volgorde in de prompt. Gebruik bevestigingen (“Ik noteer: … Klopt dat?”) voor duidelijkheid.

<img width="2903" height="1117" alt="image" src="https://github.com/user-attachments/assets/6fe50cfe-da82-4489-95fd-74289fe6e94b" />


---

## 2️⃣ Structured Data instellen

In **Advanced → Structured Data**:
1. Voeg het prompt toe dat we onder 'structured data prompt' hebben staan.
2. Voeg elke variabele die je expliciet gestructureerd uit het interview wil halen aan de properties (in dit geval: de variabelen van het system prompt.
  > Zet Type op 'string' bij elke variabele.
<img width="2903" height="1117" alt="image" src="https://github.com/user-attachments/assets/36d4438b-8d79-4de3-955f-67ee7fa17e90" />

# 🔗 Webhook instellen voor VAPI → n8n

Deze handleiding legt uit welke webhook-URL je nodig hebt in VAPI en waar je deze instelt, zodat de **Structured Data** en het **transcript** automatisch naar n8n worden gestuurd.

---

## 1️⃣ Webhook-URL vinden in n8n

1. Open je n8n-project.
2. Voeg een **Webhook**-node toe.
3. Zet:
   - **HTTP Method**: `POST`
   - **Respond**: `Immediately`
4. Kopieer de **Production URL** van de Webhook-node.  
   🔹 **Niet** de Test URL gebruiken, want die werkt alleen tijdens development in *Test Execution*.

---

## 2️⃣ Webhook instellen in VAPI

1. Ga in VAPI naar je **Assistant**.
2. Open **Advanced → Messaging**.
3. **Server URL** = plak hier de **Production URL** van je n8n Webhook-node.
4. In **Server Messages**:
   - Zet **alleen** `end-of-call-report` aan  
     *(optioneel ook `analysis-result` als je realtime data wilt)*.
   - Zet **alle andere events uit** (`conversation-update`, `analysis-plan`, etc.) om onnodige verzoeken te voorkomen.

