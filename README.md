# ✉️ E-Mail-Automatisierung mit n8n und lokalem LLM

> Prototyp einer E-Mail-Pipeline, die eingehende Kundenanfragen regelbasiert klassifiziert und mit einem lokal betriebenen LLM (Ollama, Llama 3 8B) Antwortentwürfe in Gmail anlegt.

---

## Rolle & Entwicklungsansatz

* **Rolle:** Christoph Bauer, Business Analyst: Prozessanalyse, Workflow-Design, Datenschutzkonzept, Prompt-Design und Test.
* **Umsetzung:** Docker-Setup, n8n-Workflow und JavaScript-Logik habe ich mit KI-Coding-Assistenten erstellt und anschließend getestet und angepasst.

---

## Ausgangslage und Lösung

* **Problem:** Im Kundenservice kostet das Sortieren von Standardanfragen und das Schreiben von Erstantworten viel Zeit.
* **Lösung:** Eine n8n-Pipeline erkennt Kundenanfragen über eine regelbasierte Prüfung und lässt ein lokales LLM einen Antwortentwurf schreiben.
* **Datenschutz:** Die Textverarbeitung läuft lokal über Ollama; E-Mail-Inhalte gehen an keinen externen LLM-Anbieter.

---

## Tech Stack

* **Orchestrierung:** n8n (selbst gehostet per Docker)
* **LLM:** Ollama mit Llama 3 (8B), lokal
* **Integration:** Gmail API / IMAP-Trigger
* **Logik:** JavaScript (n8n Code Node)

![Docker Setup](Docs/n8n-docker-container-status.png)

### Warum diese Architektur

1. **Lokale Verarbeitung:** Keine E-Mail-Inhalte bei externen LLM-APIs.
2. **Kosten:** Keine nutzungsabhängigen API-Kosten; die Leistung hängt von der eigenen Hardware ab.
3. **Unabhängigkeit:** Modell, Prompt und Infrastruktur bleiben unter eigener Kontrolle.

---

## Architektur: Regeln für Entscheidungen, LLM für Text

Ein wichtiges Learning war der Wechsel von einer reinen LLM-Klassifizierung zu einem hybriden Ansatz:

* **Problem:** Kleine lokale Modelle wie Llama 3 8B halten feste Ausgabeformate nicht immer ein und antworten teils mit Metasprache („Ich habe den Text analysiert …“). Für Weichenstellungen ist das zu unzuverlässig.
* **Lösung:** Die Entscheidung „Kundenanfrage ja/nein“ trifft deterministischer JavaScript-Code. Das LLM wird nur für das Verstehen des Kontexts und das Formulieren der Antwort eingesetzt.

![Workflow Architecture](Docs/n8n-workflow-pipeline.png)

---

## Umsetzung im Detail

### A. IMAP- / E-Mail-Trigger
Liest neue E-Mails ein und übergibt Text- und HTML-Inhalt an den Workflow.

### B. Code Node (regelbasierte Prüfung)
Prüft den Inhalt auf definierte Begriffe für Kunden- und Supportanliegen, mit Fallback über `textPlain`, `textHtml` und `text`.

![JavaScript Code Logic](Docs/javascript-code-node-logic.png)

### C. LLM Chain (Ollama / Llama 3 8B)
Erstellt aus dem übergebenen `emailText` einen passenden Antwortentwurf.

### D. Weiterleitung und Ausgabe (If Node & Gmail API)
* Der If-Knoten nutzt direkt das Ergebnis des Code-Knotens (`$('Code in JavaScript').item.json.isCustomer`).
* **Mensch entscheidet:** Antworten werden nicht automatisch versendet, sondern als Entwurf in Gmail gespeichert und vor dem Versand geprüft.

![Gmail Draft Result](Docs/gmail-draft-result-preview.png)

---

## Test

Im Testfall hat der Workflow Name, Firma, Anliegen, Adressänderung und Kundennummer korrekt erkannt, den Kundenpfad ausgelöst und in unter 3 Sekunden einen Entwurf angelegt. Getestet wurde mit Beispiel-E-Mails, nicht im Produktivbetrieb.

---

## Learnings

1. **Regeln plus LLM statt nur LLM:** Für eindeutige Entscheidungen ist Code verlässlicher; das LLM spielt seine Stärke bei der Textformulierung aus.
2. **Lokaler Betrieb:** n8n und Ollama im Docker-Netz halten die Verarbeitung im eigenen System.
3. **Entwürfe statt Versand:** Die Freigabe durch einen Menschen begrenzt das Risiko fehlerhafter Antworten.

---

## Repository-Struktur

```text
├── Workflows/
│   └── email_automation_pipeline.json  # Exportierter n8n-Workflow
├── Scripts/
│   └── email_classifier.js             # Regelbasierte Klassifizierung
├── Docs/
│   ├── n8n-docker-container-status.png
│   ├── n8n-workflow-pipeline.png
│   ├── javascript-code-node-logic.png
│   └── gmail-draft-result-preview.png
├── docker-compose.yml                  # Lokales Container-Setup (n8n & Ollama)
└── README.md
```
