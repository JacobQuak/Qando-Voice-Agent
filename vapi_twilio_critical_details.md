# 📞 Outbound bellen vanuit Vapi met Twilio — Critical Details


---

## 4. Configuratie in Twilio



### Geo permissions
- Twilio Console → **Voice → Geo permissions**  
- Zorg dat **Netherlands (NL)** en alle bestemmingslanden aangevinkt staan.


### Balance
- Check dat er voldoende **saldo** is. Zonder tegoed geeft Twilio direct **403** of **402** errors.

---

## 5. Veelvoorkomende fouten en oplossingen

### ❌ 403 Forbidden / SIP 403
- **Oorzaak**: Caller ID wordt geweigerd door carrier (bijv. KPN).  
- **Fix**: gebruik een lokaal **NL-nummer** (085, 020, …) als outbound `phoneNumber`.

### ❌ 403 Forbidden (trial account)
- **Oorzaak**: trial kan alleen Verified Caller IDs bellen.  
- **Fix**: verifieer het doelnummer in Twilio of upgrade je account.

### ❌ 400 Bad Request: property … should not exist
- **Oorzaak**: verkeerde JSON-structuur (`phoneNumber` in `transport`).  
- **Fix**: zet `phoneNumber` of `phoneNumberId` op **top-level**, niet in `transport`.

### ❌ Geen logs in Vapi
- **Oorzaak**: call faalde vóórdat de agent audio kreeg.  
- **Fix**: check **Twilio Logs → Calls → SIP response**.

---

## 6. Best Practices

- ✅ Gebruik altijd **E.164 formaat** (`+31...`).  
- ✅ **Caller ID policy NL**: gebruik een herkenbaar NL-nummer (geen 097x, geen rare ranges).  
- ✅ **Logging**: check zowel **Vapi Call Logs** als **Twilio Call Logs** voor volledige foutanalyse.  
- ✅ **Testing**: begin met **WebRTC (Talk to Assistant)** in Vapi om je agent te testen, los van PSTN-issues.  
- ✅ **Security**: bewaar je **API keys** niet in scripts, maar als environment variables.  
- ✅ **Failover**: koppel meerdere nummers zodat je kunt testen via verschillende ranges.  

---

## 7. Checklist voor succesvolle call

- [ ] AssistantId correct ingevuld.  
- [ ] phoneNumber/phoneNumberId aanwezig op top-level in payload.  
- [ ] Nummers in E.164.  
- [ ] NL Geo-permission aan in Twilio.  
- [ ] Outbound nummer Voice-capable en niet in “verdachte” ranges (097x vermijden).  
- [ ] Genoeg saldo op Twilio-account.  
- [ ] Bij trial: doelnummer verified in Twilio.

# 📌 Notitie: blokkade van 097x-nummers door providers

In Nederland blokkeren sommige providers (zoals **KPN**) inkomende gesprekken wanneer de **Caller ID** een **097x-nummer** of ander virtueel nummer is. Vodafone accepteert wel 097 nummers. 

