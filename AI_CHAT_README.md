# AI Chat pro Rezervaci Minibagru - Návod k Použití

Kompletní řešení pro AI asistenta, který pomáhá zákazníkům s rezervací minibagru.

## Co bylo implementováno

### 1. Frontend (index.html)
✅ Plovoucí chat tlačítko v pravém dolním rohu
✅ Responsivní chat okno s moderním designem
✅ Typing indicator (tři tečky) při načítání odpovědi
✅ Session management pro každou konverzaci
✅ Automatické scrollování
✅ Mobilní optimalizace

### 2. Backend (n8n workflow)
✅ Kompletní workflow struktura v souboru `N8N_WORKFLOW_SETUP.md`
✅ Claude AI integrace
✅ Konverzační logika pro sběr dat
✅ Automatické odeslání na Make.com webhook
✅ Error handling

## Jak to zprovoznit

### Krok 1: Nastavit n8n Workflow

1. **Otevřete n8n** a vytvořte nový workflow
2. **Následujte instrukce** v souboru `N8N_WORKFLOW_SETUP.md`
3. **Důležité kroky:**
   - Přidejte Webhook node (path: `/ai-chat`)
   - Nastavte Claude AI credentials
   - Zkopírujte system prompt ze setup souboru
   - Připojte HTTP Request node na Make.com webhook
   - **Aktivujte workflow!**

4. **Zkopírujte Production Webhook URL** (bude vypadat jako):
   ```
   https://vase-n8n-instance.com/webhook/ai-chat
   ```

### Krok 2: Aktualizovat Frontend

1. **Otevřete `index.html`**
2. **Najděte řádek 1282:**
   ```javascript
   const N8N_WEBHOOK_URL = 'https://VASE-N8N-INSTANCE.com/webhook/ai-chat';
   ```
3. **Nahraďte URL** vaším Production Webhook URL z n8n
4. **Uložte soubor**

### Krok 3: Nasadit na GitHub Pages

```bash
git add index.html
git commit -m "Přidán AI chat pro rezervace"
git push origin main
```

### Krok 4: Testování

1. **Otevřete web** (www.pujcmibagr.cz)
2. **Klikněte na oranžové chat tlačítko** v pravém dolním rohu
3. **Vyzkoušejte konverzaci:**
   - "Ahoj, chtěl bych si půjčit minibagr"
   - Odpovězte na otázky AI asistenta
   - Zkontrolujte jestli se poptávka odešle na Make.com

## Jak AI Chat Funguje

### Flow pro Zákazníka:

1. **Zákazník klikne na chat tlačítko**
2. **AI přivítá** a zeptá se jak může pomoci
3. **AI postupně sebere informace:**
   - Jméno
   - Telefon
   - Email
   - Adresa (místo realizace)
   - Datum pronájmu
   - Doba pronájmu (počet dní)
   - Potřeba dopravy
   - Požadované příslušenství (volitelné)
   - Poznámka (volitelné)
4. **AI potvrdí údaje** a zeptá se na schválení
5. **Odešle poptávku** na Make.com webhook
6. **Zákazník dostane potvrzení**

### Technický Flow:

```
Frontend (JavaScript)
   ↓ POST request
n8n Webhook
   ↓
Function: Prepare Context
   ↓
Claude AI (konverzace)
   ↓
Function: Parse Response
   ↓
IF Order Ready?
   ├─ ANO → Extract Data → HTTP to Make.com → Success Response
   └─ NE → Continue Chat Response
   ↓
Response to Frontend
```

## Vylepšení n8n Workflow (doporučené)

Základní workflow v `N8N_WORKFLOW_SETUP.md` používá **regex parsing** pro extrakci dat z konverzace. To funguje, ale má limity.

### Doporučení: Structured Output

Pro lepší výsledky doporučuji upravit Claude prompt na **JSON mode**:

1. **V Claude AI node** přidejte do system promptu:
```
Když máš všechny údaje, odpověz PŘESNĚ v tomto JSON formátu:
{
  "orderReady": true,
  "confirmationMessage": "Zkontrolujte prosím údaje: ...",
  "data": {
    "jmeno": "Jan Novák",
    "telefon": "+420606123456",
    "email": "jan@example.com",
    "adresa": "Praha 1",
    "datum": "2025-01-15",
    "doba": "3",
    "doprava": "true",
    "prislusenstvi": "lzice, vrtak",
    "poznamka": "..."
  }
}
```

2. **V Parse Response node** pak stačí:
```javascript
const response = JSON.parse($input.item.json.aiResponse);
return { json: response };
```

## Náklady

### Claude API:
- Model: claude-3-5-sonnet-20241022
- Cena: ~$3 za 1M input tokens, ~$15 za 1M output tokens
- **Odhadovaná cena na 1 konverzaci: 0.05 - 0.15 Kč**
- Pro 100 objednávek měsíčně: **~10 Kč/měsíc**

### n8n:
- Self-hosted: zdarma (jen náklady na VPS)
- Cloud: od $20/měsíc

### Make.com:
- Stávající webhook (už používáte)

## Bezpečnost

### CORS
n8n webhook by měl mít povolený CORS pro vaši doménu. V n8n webhook node nastavte:
- Allow Origin: `https://www.pujcmibagr.cz` nebo `*`

### Rate Limiting
Doporučuji přidat do n8n:
- Rate limiting node (max 10 requestů za minutu na session)
- Spam protection

### API Keys
- Claude API key je bezpečně uložen v n8n
- Není viditelný ve frontend kódu

## Možné Problémy a Řešení

### Chat se neotvírá
- Zkontrolujte browser console (F12) na chyby
- Ujistěte se že není konflikt s jinými chat widgety

### n8n webhook neodpovídá
- Zkontrolujte že workflow je **aktivovaný**
- Zkontrolujte URL - musí být Production URL, ne Test URL
- Otevřete workflow execution log v n8n

### Claude neextrahuje data správně
- Použijte structured output (viz Vylepšení výše)
- Nebo upravte regex patterns v Extract Order Data node

### Make.com webhook vrací chybu
- Zkontrolujte že formát dat odpovídá původnímu formuláři
- Možná budete muset upravit `g-recaptcha-response` handling

### Chat překrývá jiné elementy
- Upravte `z-index` v CSS (řádek 986 a 1029 v index.html)

## Monitoring

### n8n Execution Log
- Každá konverzace vytvoří execution v n8n
- Můžete vidět celou historii a debug problémy

### Browser Console
- Frontend loguje chyby do console
- Použijte F12 → Console pro debug

## Další Kroky (volitelné)

### 1. Analytics
Přidejte tracking do chatu:
```javascript
// V sendMessage funkci
gtag('event', 'chat_message_sent', {
  'event_category': 'AI Chat',
  'event_label': 'User Message'
});
```

### 2. Lepší Extrakce Adresy
Integrujte Mapy.cz API přímo do AI chatu pro validaci adresy

### 3. Kalendář Integrace
AI může kontrolovat dostupnost v Google Calendar před potvrzením

### 4. Multi-language
Přidejte podporu pro angličtinu nebo němčinu

### 5. Voice Input
Použijte Web Speech API pro hlasový input

## Support

Pokud máte problémy:

1. **Zkontrolujte tento README**
2. **Podívejte se do `N8N_WORKFLOW_SETUP.md`**
3. **Zkontrolujte n8n execution logs**
4. **Otevřete browser console (F12)**

---

**Happy chatting! 🤖**
