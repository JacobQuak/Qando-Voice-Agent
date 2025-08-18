## 📞 Twilio-nummer kopen dat naar Nederland kan bellen

Om een voice agent te gebruiken die kan bellen naar Nederlandse telefoonnummers (`+31`), kun je een **Twilio-nummer** aanschaffen en configureren. Upgrade eerst het account. 

### 1. Koop een Twilio-nummer
1. Ga naar de [Twilio Console – Buy a Number](https://console.twilio.com/us1/develop/phone-numbers/incoming).
2. Klik op **Buy a number**.
3. Selecteer en kies **buy**:
   - **Country**: Netherlands.
   - **Capabilities**: zorg dat **Voice** aangevinkt staat.
4. Voeg het adres (van kantoor) toe. 
5. Bevestig de aankoop.  
   Je betaalt ± **$6/maand** voor het Nederlandse nummer.

### 2. Zet internationale belrechten aan.
Standaard kan een nieuw Twilio-nummer niet naar alle landen bellen.  
Om bellen naar Nederland toe te staan:

1. Ga in Twilio naar **Voice → Geo Permissions**.
2. Zoek de regio **Europe**.
3. Zet het vinkje aan bij **Netherlands**.
4. Klik op **Save**.

> 💡 Zonder deze instelling krijg je een `403 Forbidden` of `permission_denied` foutmelding bij calls naar `+31`.

### 3. Koppel het nummer aan je Voice Agent in Vapi
Nu je een Twilio-nummer hebt dat internationaal (ook naar NL) kan bellen, kun je dit koppelen aan je agent in **[Vapi](https://vapi.ai)**:

1. Ga in Vapi naar je project of agent.
2. Ga naar de **Phone Settings** of **Telephony settings**.
3. Voeg je Twilio-account toe:
   - **Account SID** en **Auth Token** (te vinden in Twilio Console → Account).
4. Selecteer het zojuist gekochte Twilio-nummer als **Outbound Caller ID**.
5. Sla de instellingen op.

### 4. Test je agent
- Maak een testcall naar een Nederlands mobiel nummer (`+316...`).
- Controleer of de call succesvol wordt opgebouwd.
- Bekijk in Vapi of de agent de ingestelde flow correct uitvoert.

---

## ✅ Samenvatting
- Koop een Twilio-nummer met voice-capabilities.
- Zet **Geo Permissions** voor Nederland aan.
- Koppel Twilio aan je agent in **Vapi** met je SID + Auth Token.
- Selecteer je nummer als outbound caller ID en start een testcall.

Zo kun je een **Vapi voice agent** inzetten die ook daadwerkelijk naar Nederlandse telefoonnummers kan bellen.
