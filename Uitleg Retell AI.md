# Qando Voice Agent – Bellen met Retell AI

Met de **Qando Voice Agent** in [Retell AI](https://www.retellai.com) kun je automatisch gesprekken voeren met een lijst aan telefoonnummers.  
De agent is ontworpen om gestructureerde interviews af te nemen en antwoorden op te slaan voor latere analyse.

## 🔹 Werking van de agent
- De voice agent belt de deelnemer.
- Voert een gestructureerd gesprek op basis van vooraf ingestelde vragen.
- Legt antwoorden vast in variabelen (voor gebruik in bijvoorbeeld webhooks of rapportages).
- Resultaten zijn terug te vinden in de **Call History** van Retell, inclusief gespreksstatus, duur en transcriptie.

## 🔹 Bellen naar een set telefoonnummers
1. Ga in Retell naar **Batch Call**.
2. Upload een CSV-bestand met telefoonnummers in internationaal formaat (E.164), bijvoorbeeld:
   ```csv
   phone_number
   +14155550123
   +14155550124
   ```
3. Selecteer de Qando Voice Agent als **Outbound Call Agent**.

4. Kies of de gesprekken direct moeten worden gestart of ingepland.

5. Start de batch call.

## 🔹 Resultaten bekijken
- Open **Call History** om de status en transcripties van elk gesprek te zien.  
- Je kunt hier ook controleren of antwoorden correct zijn opgeslagen.

## ⚠️ Beperking: bellen naar Nederlandse nummers
Op dit moment ondersteunt Retell AI in de standaardconfiguratie **geen uitgaande gesprekken naar Nederlandse telefoonnummers** (`+31`).

**Nederlands aangevraagd**
- Ik heb begin augustus een mail gestuurd naar **support@retellai.com** met de vraag of ze gesprekken naar Nederlandse nummers kunnen enablen. Hier gingen ze het over hebben. 
