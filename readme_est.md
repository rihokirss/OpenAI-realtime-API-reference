
# OpenAI Realtime API Juhend

## Ülevaade

OpenAI Realtime API võimaldab vestlustes ja sündmustes osalemist, kasutades reaalajas protokolli. Selle API kaudu saab saata kasutaja sisendeid, funktsioonikutseid ning funktsiooniväljundeid. Järgnevalt on esitatud juhised ja näited korrektsete päringute tegemiseks ning vigade vältimiseks.

---

## Olulised Reeglid ja Struktuur

### 1. Üldine Sündmuse Struktuur

Kõik päringud API-le järgivad üldist sündmuse struktuuri:
```json
{
  "type": "<event_type>",
  "item": {
    "type": "<item_type>",
    "call_id": "<unique_call_id>",
    "output": "<output_data>"
  }
}
```

### 2. Kehtivad `type` Väärtused

Sündmuse tüübid, mida API aktsepteerib:

- `session.update`: Seansi konfigureerimiseks (nt tööriistade lisamine, juhised jms).
- `conversation.item.create`: Kasutatakse uue vestluselementi loomiseks, näiteks:
  - Tekstisisend (`message` tüüpi).
  - Funktsioonikutse väljund (`function_call_output` tüüpi).
- `response.create`: Käsk GPT-l vastust genereerida, kasutades eelnevalt saadetud sisendeid ja tulemusi.

---

## Näited

### `conversation.item.create` Näide

#### Teksti saatmine GPT-le

Kasutaja sisendi saatmine:
```json
{
  "type": "conversation.item.create",
  "item": {
    "type": "message",
    "role": "user",
    "content": "Hello, can you provide a color palette?"
  }
}
```

#### Funktsioonikõne väljund

Kui mudel palub funktsioonikõne tulemust, tuleb kasutada järgmisi välju:
```json
{
  "type": "conversation.item.create",
  "item": {
    "type": "function_call_output",
    "call_id": "call_lWPIvNlRxZnHNjcB",
    "output": "{"result": [{"color": "#FFFFFF"}]}"
  }
}
```

---

## Tõrked

### Vigade Näited ja Lahendused

#### Viga:
```json
{"code": "unknown_parameter", "message": "Unknown parameter: 'item.name'."}
```
**Põhjus:** Väli `name` ei ole toetatud `item` objekti sees.  
**Lahendus:** Kasutage ainult `type`, `call_id`, ja `output` välju.

#### Viga:
```json
{"code": "missing_required_parameter", "message": "Missing required parameter: 'item.output'."}
```
**Põhjus:** Väli `output` on `function_call_output` jaoks kohustuslik.  
**Lahendus:** Lisage `output` JSON-stringiga vastuseandmetest.

---

## Täiendavad Näited

### `session.update` Näide

Seansi konfigureerimiseks (nt tööriistade lisamine või juhiste uuendamine):
```json
{
  "type": "session.update",
  "session": {
    "tools": [
      {
        "type": "function",
        "name": "get_color_palette",
        "description": "Provides a color palette based on the given theme.",
        "parameters": {
          "type": "object",
          "properties": {
            "theme": {
              "type": "string",
              "description": "Theme for the color palette."
            }
          },
          "required": ["theme"]
        }
      }
    ]
  }
}
```

### `response.create` Näide

Kui soovite anda mudelile juhiseid vestluse jätkamiseks pärast funktsioonikõne väljundit:
```json
{
  "type": "response.create",
  "response": {
    "instructions": "Please respond to the user with the color palette result."
  }
}
```

---

## Vigade Tõrjumine

### Tavalised Probleemid

#### Tundmatu Väli:

Kontrollige, et ainult lubatud väljad oleksid kaasatud.

#### Vigane JSON:

Kontrollige, et kõik JSON-stringid oleksid õigesti põgenemismärkidega (`"`), kui kasutate neid teise JSON-i sees.

#### Tundmatu `type`:

Veenduge, et `type` väärtus oleks üks toetatutest: `session.update`, `conversation.item.create`, jne.

---

## Logimine ja Diagnostika

Logige kõik sündmused enne nende saatmist:
```javascript
console.log("Sending event:", JSON.stringify(event));
```

Kontrollige serveri vastuseid ja käsitlege tõrked:
```javascript
dataChannel.onerror = (error) => {
    console.error("Data channel error:", error);
};
```

---

## Kokkuvõte

See juhend aitab tagada OpenAI Realtime API korrektse kasutamise. Järgige kehtestatud struktuure ja vältige üleliigsete väljade lisamist, et vältida tõrkeid. Logige kõik sündmused ja vajadusel testige väljundeid enne saatmist.
