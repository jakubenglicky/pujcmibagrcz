# n8n Workflow Setup - AI Chat pro Rezervaci Minibagru

## Přehled Workflow

Tento workflow slouží jako backend pro AI chat, který pomáhá zákazníkům s objednávkou minibagru.

## Struktura Workflow

### 1. Webhook Node (vstup)
- **Type:** Webhook
- **HTTP Method:** POST
- **Path:** `/ai-chat` (nebo vlastní)
- **Response Mode:** "When Last Node Finishes"
- **Response Data:** "First Entry JSON"

**Přijímá data:**
```json
{
  "sessionId": "unique-session-id",
  "message": "Zpráva od uživatele",
  "conversationHistory": []
}
```

### 2. Function Node - Prepare Context
**Purpose:** Připraví kontext pro Claude AI

**Code:**
```javascript
// Získat data z webhooku
const sessionId = $input.item.json.sessionId;
const userMessage = $input.item.json.message;
const history = $input.item.json.conversationHistory || [];

// Systémový prompt pro Claude
const systemPrompt = `Jsi asistent pro půjčovnu minibagru v Kněževsi u Rakovníka. Tvým úkolem je pomoci zákazníkovi s rezervací.

CENÍK:
- 1-4 dny: 1 600 Kč/den
- 5-30 dní: 1 400 Kč/den
- 31+ dní: 1 000 Kč/den
- Vratná kauce: 10 000 Kč
- Doprava: 15 Kč/km (4 cesty celkem: dovoz tam+zpět, odvoz tam+zpět)
- Svépomocí: 500 Kč (potřeba auto pro 1200 kg)

DOSTUPNÉ PŘÍSLUŠENSTVÍ (vše v ceně):
- Lžíce 38 cm
- Lžíce 20 cm
- Hydraulická svahovka 80 cm
- Vrták 20 cm
- Kleště
- Rozrývací trn

TVŮJ ÚKOL:
1. Přivítej zákazníka a zeptej se, jak mu můžeš pomoci
2. Zjisti potřebné informace pro rezervaci:
   - Jméno
   - Telefon (formát: +420... nebo 9 číslic)
   - Email
   - Místo realizace (adresa)
   - Datum pronájmu
   - Doba pronájmu (dny)
   - Potřeba dopravy (ano/ne)
   - Požadované příslušenství
   - Poznámka (volitelné)

3. Buď vstřícný, ale stručný. Ptej se na 1-2 věci najednou.
4. Když máš všechny POVINNÉ údaje, potvrď je a zeptej se: "Mám odeslat poptávku?"
5. Po potvrzení odpověz POUZE: "ORDER_READY" a nic víc

DŮLEŽITÉ:
- Neodhaduj data, která ti zákazník neřekl
- Buď příjemný a profesionální
- Vysvětli ceník pokud se zákazník ptá
- Pro příslušenství: pokud se zákazník nezeptá, nemusíš to nabízet aktivně`;

// Sestavit conversation history pro Claude
const messages = [];

// Přidat historii
history.forEach(msg => {
  messages.push({
    role: msg.role,
    content: msg.content
  });
});

// Přidat novou zprávu od uživatele
messages.push({
  role: 'user',
  content: userMessage
});

return {
  json: {
    sessionId,
    systemPrompt,
    messages,
    userMessage
  }
};
```

### 3. OpenAI/Anthropic Node (Claude)
- **Type:** @n8n/n8n-nodes-langchain.lmChatAnthropic (nebo podobný Claude node)
- **Model:** claude-3-5-sonnet-20241022 (nebo nejnovější)
- **Temperature:** 0.7
- **Max Tokens:** 1000

**Settings:**
- System Message: `{{ $json.systemPrompt }}`
- Messages: `{{ $json.messages }}`

### 4. Function Node - Parse Response
**Purpose:** Zkontroluje jestli je objednávka ready

**Code:**
```javascript
const aiResponse = $input.item.json.response; // Upravte podle struktury vašeho Claude node outputu
const sessionId = $input.item.json.sessionId;

// Zkontrolovat jestli AI řekla ORDER_READY
const isOrderReady = aiResponse.includes('ORDER_READY');

return {
  json: {
    sessionId,
    aiResponse,
    isOrderReady,
    messages: $input.item.json.messages
  }
};
```

### 5. IF Node - Check if Order Ready
**Condition:**
- Value 1: `{{ $json.isOrderReady }}`
- Operation: Equal
- Value 2: `true`

### 6A. Function Node - Extract Order Data (IF TRUE)
**Purpose:** Extrahuje data z konverzace a připraví POST body

**Code:**
```javascript
const messages = $input.item.json.messages;

// Funkce pro extrakci informací z konverzace
function extractInfo(messages, patterns) {
  for (let i = messages.length - 1; i >= 0; i--) {
    const msg = messages[i];
    if (msg.role === 'user') {
      for (const pattern of patterns) {
        const match = msg.content.match(pattern);
        if (match) return match[1] || match[0];
      }
    }
  }
  return null;
}

// Extrakce jednotlivých polí (můžete vylepšit regex podle potřeby)
const jmeno = extractInfo(messages, [/jmenuji se (.+)/i, /jsem (.+)/i]) || '';
const telefon = extractInfo(messages, [/telefon[:\s]+(\+?\d+)/i, /(\+?\d{9,})/]) || '';
const email = extractInfo(messages, [/email[:\s]+([^\s]+@[^\s]+)/i, /([^\s]+@[^\s]+)/]) || '';
const adresa = extractInfo(messages, [/adresa[:\s]+(.+)/i, /místo[:\s]+(.+)/i]) || '';
const datum = extractInfo(messages, [/datum[:\s]+(\d{4}-\d{2}-\d{2})/i, /(\d{1,2}\.\s*\d{1,2}\.?\s*\d{0,4})/]) || '';
const doba = extractInfo(messages, [/(\d+)\s*dn/i]) || '1';
const doprava = extractInfo(messages, [/doprava[:\s]+(ano|ne)/i]) === 'ano' ? 'true' : 'false';

// Příslušenství - hledat zmínky
let prislusenstvi = '';
const fullText = messages.map(m => m.content).join(' ').toLowerCase();
if (fullText.includes('lžíce 38') || fullText.includes('lzice 38')) prislusenstvi += 'lzice, ';
if (fullText.includes('lžíce 20') || fullText.includes('lzice 20')) prislusenstvi += 'lzice-20, ';
if (fullText.includes('svahovka')) prislusenstvi += 'svahovka, ';
if (fullText.includes('vrták') || fullText.includes('vrtak')) prislusenstvi += 'vrtak, ';
if (fullText.includes('kleště') || fullText.includes('kleste')) prislusenstvi += 'kleste, ';
if (fullText.includes('trn')) prislusenstvi += 'trn, ';
prislusenstvi = prislusenstvi.slice(0, -2); // Odstranit poslední čárku

const poznamka = 'Objednávka vytvořená přes AI chat';

return {
  json: {
    jmeno,
    telefon,
    email,
    adresa,
    datum,
    doba,
    doprava,
    prislusenstvi,
    poznamka,
    'g-recaptcha-response': 'ai-chat-bypass' // Možná budete muset upravit Make.com webhook
  }
};
```

**POZNÁMKA:** Tato extrakce je zjednodušená. Pro lepší výsledky doporučuji:
- Použít structured output z Claude (JSON mode)
- Nebo přidat další Claude AI krok který strukturuje data do JSON

### 7. HTTP Request Node - Send to Make.com
**Method:** POST
**URL:** `https://hook.eu2.make.com/rw4a1vvutigc444ysq43cgyr5nd3mt9i`
**Body:** `{{ $json }}`
**Content-Type:** application/json

### 8A. Respond to Webhook (Success)
**Status Code:** 200
**Body:**
```json
{
  "success": true,
  "message": "Děkuji! Vaše poptávka byla odeslána. Budeme vás kontaktovat telefonicky nebo e-mailem pro potvrzení.",
  "orderComplete": true
}
```

### 6B. Respond to Webhook (Continue Chat - IF FALSE)
**Status Code:** 200
**Body:**
```json
{
  "success": true,
  "message": "{{ $('Parse Response').item.json.aiResponse }}",
  "orderComplete": false
}
```

## Diagram Workflow

```
Webhook
  ↓
Prepare Context
  ↓
Claude AI
  ↓
Parse Response
  ↓
IF Order Ready?
  ├─ YES → Extract Order Data → HTTP to Make.com → Respond (Success)
  └─ NO → Respond (Continue Chat)
```

## Vylepšení (volitelné)

### 1. Lepší extrakce dat pomocí Structured Output
Místo regex parsing, můžete použít Claude s JSON schema:

```javascript
// V Prepare Context node přidejte do system promptu:
"Když máš všechny údaje, odpověz ve formátu JSON:
{
  \"orderReady\": true,
  \"data\": {
    \"jmeno\": \"...\",
    \"telefon\": \"...\",
    \"email\": \"...\",
    \"adresa\": \"...\",
    \"datum\": \"YYYY-MM-DD\",
    \"doba\": \"číslo\",
    \"doprava\": \"true/false\",
    \"prislusenstvi\": \"...\",
    \"poznamka\": \"...\"
  }
}"
```

### 2. Session Management
Pro ukládání konverzací můžete přidat:
- Redis node pro cache session dat
- Database node pro dlouhodobé ukládání

### 3. Validace
Přidat validation node který zkontroluje:
- Email formát
- Telefon formát
- Datum není v minulosti

## Testování

1. Aktivujte workflow v n8n
2. Zkopírujte Production Webhook URL
3. Vložte URL do frontendu (viz index.html)
4. Otestujte konverzaci

## Troubleshooting

- **Claude neodpovídá:** Zkontrolujte API credentials
- **Data se neextrahují správně:** Použijte structured output approach
- **Make.com webhook vrací error:** Zkontrolujte formát dat, možná musíte upravit g-recaptcha-response

---

**URL vašeho n8n webhooku vložte do souboru index.html na řádku s:**
```javascript
const N8N_WEBHOOK_URL = 'https://vase-n8n-instance.com/webhook/ai-chat';
```
