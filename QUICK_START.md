# Quick Start - AI Chat za 5 minut

## Checklist pro Spuštění

### ☐ 1. n8n Workflow (10 min)

1. Otevřete n8n a vytvořte nový workflow
2. Přidejte tyto nodes v tomto pořadí:

```
┌─────────────┐
│  Webhook    │ ← nastavte path: /ai-chat
└──────┬──────┘
       │
┌──────▼─────────────────┐
│  Function Node         │ ← zkopírujte code z N8N_WORKFLOW_SETUP.md
│  "Prepare Context"     │   (sekce 2)
└──────┬─────────────────┘
       │
┌──────▼─────────────────┐
│  Claude AI Node        │ ← použijte system prompt z setup
└──────┬─────────────────┘
       │
┌──────▼─────────────────┐
│  Function Node         │ ← zkopírujte code z setup (sekce 4)
│  "Parse Response"      │
└──────┬─────────────────┘
       │
┌──────▼─────────────────┐
│  IF Node               │ ← condition: isOrderReady === true
└──┬────────────────┬────┘
   │ TRUE           │ FALSE
   │                │
   ▼                ▼
┌────────────┐  ┌──────────────┐
│ Extract    │  │  Respond     │
│ Order Data │  │  (Continue)  │
└─────┬──────┘  └──────────────┘
      │
┌─────▼──────┐
│ HTTP POST  │ ← URL: Make.com webhook
│ to Make    │
└─────┬──────┘
      │
┌─────▼──────┐
│  Respond   │
│ (Success)  │
└────────────┘
```

3. **Aktivujte workflow** (tlačítko Active)
4. **Zkopírujte Production Webhook URL**

### ☐ 2. Aktualizovat index.html (1 min)

Otevřete `index.html` a najděte řádek **1282**:

```javascript
const N8N_WEBHOOK_URL = 'https://VASE-N8N-INSTANCE.com/webhook/ai-chat';
```

Nahraďte `https://VASE-N8N-INSTANCE.com/webhook/ai-chat` vaší Production Webhook URL z n8n.

**Uložte!**

### ☐ 3. Deploy (1 min)

```bash
git add .
git commit -m "Přidán AI chat asistent"
git push
```

### ☐ 4. Test (2 min)

1. Otevřete www.pujcmibagr.cz
2. Klikněte na oranžové chat tlačítko vpravo dole
3. Zkuste: "Ahoj, chtěl bych si půjčit minibagr"
4. Odpovězte na otázky AI
5. Zkontrolujte Make.com jestli přišla poptávka

## Důležité System Prompt pro Claude

Při nastavování Claude AI node v n8n, použijte tento system prompt (najdete ho taky v `N8N_WORKFLOW_SETUP.md` sekce 2):

```
Jsi asistent pro půjčovnu minibagru v Kněževsi u Rakovníka. Tvým úkolem je pomoci zákazníkovi s rezervací.

CENÍK:
- 1-4 dny: 1 600 Kč/den
- 5-30 dní: 1 400 Kč/den
- 31+ dní: 1 000 Kč/den
- Vratná kauce: 10 000 Kč
- Doprava: 15 Kč/km (4 cesty celkem)
- Svépomocí: 500 Kč

DOSTUPNÉ PŘÍSLUŠENSTVÍ (vše v ceně):
- Lžíce 38 cm
- Lžíce 20 cm
- Hydraulická svahovka 80 cm
- Vrták 20 cm
- Kleště
- Rozrývací trn

TVŮJ ÚKOL:
1. Přivítej zákazníka
2. Zjisti: jméno, telefon, email, adresa, datum, doba pronájmu, doprava, příslušenství
3. Buď vstřícný, ale stručný (ptej se na 1-2 věci najednou)
4. Když máš všechno, potvrď údaje a zeptej se: "Mám odeslat poptávku?"
5. Po potvrzení odpověz POUZE: "ORDER_READY"

DŮLEŽITÉ:
- Neodhaduj data
- Buď profesionální
- Pro příslušenství: nabízej jen pokud se zákazník ptá
```

## Co když něco nefunguje?

### Chat se vůbec neobjevuje
→ Zkontrolujte browser console (F12), hledejte červené chyby

### "Nepodařilo se spojit se serverem"
→ n8n webhook není dostupný nebo URL je špatně

### AI neodpovídá správně
→ Zkontrolujte system prompt v Claude AI node

### Objednávka se neodešle na Make.com
→ Zkontrolujte HTTP Request node URL a formát dat

## Need Help?

Přečtěte si:
1. **AI_CHAT_README.md** - kompletní dokumentace
2. **N8N_WORKFLOW_SETUP.md** - detailní workflow setup

## Náklady

- **Claude API:** ~0.10 Kč na konverzaci
- **n8n:** zdarma (self-hosted)
- **Make.com:** stávající webhook (už používáte)

**Celkem: ~10 Kč/měsíc** (při 100 konverzacích)

---

Enjoy! 🚀
