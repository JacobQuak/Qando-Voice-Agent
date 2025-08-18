## 🧠 Voice Assistant Workflow – Uitleg

Deze workflow automatiseert het verwerken van gesprekken gevoerd door een voice assistant. Het doel is om gespreksdata (zoals transcripties) te analyseren, structureren en om te zetten in bruikbare output zoals vragenlijsten, user input, of inzichten.

---

### 🔄 Algemene Flow

De workflow verloopt globaal als volgt:

1. **Trigger via Webhook**  
   De flow start zodra een voice assistant een webhook activeert, bijvoorbeeld na het afronden van een gesprek.

2. **Transcriptie ophalen**  
   De ruwe gespreksdata (meestal een transcript in tekstvorm) wordt opgehaald of meegeleverd in de webhook.

3. **Structurering via AI Agent**  
   Een AI node ontvangt de ongestructureerde tekst en probeert deze te herstructureren naar een begrijpelijke en logische vorm.

4. **Matchen met originele surveyvragen**
   De antwoorden uit het gesprek worden vergeleken met de originele enquêtevragen, zodat de juiste input op de juiste vraag gematcht wordt. Dit gebeurt met behulp van een AI-agent die probeert semantische overeenkomsten te vinden tussen wat de gebruiker zegt en de oorspronkelijke vraagformulering.

---

### 🛠️ Verwerking via JavaScript node

In deze stap wordt de output van de AI verder:
- geschoond (bijvoorbeeld onnodige whitespace verwijderd),
- genormaliseerd (zoals het herstructureren van array-structuren),
- en eventueel verrijkt met aanvullende metadata.

Dit gebeurt met een **Custom Function Node** in n8n (JavaScript).

---

### 🔍 Gebruikte technieken

| Onderdeel             | Technologie           | Doel                                               |
|------------------------|------------------------|-----------------------------------------------------|
| **Webhook**            | n8n Webhook Node       | Ontvangen van nieuwe call data                      |
| **AI Structurering**   | OpenAI (via Langchain) | Herstructureren van transcripties                   |
| **Survey Matching**    | OpenAI (via Langchain) | Antwoorden koppelen aan bijbehorende surveyvragen   |
| **Custom JavaScript**  | JavaScript Function    | Transformeren/verwerken van output                  |
