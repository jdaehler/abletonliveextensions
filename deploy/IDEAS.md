# MIDI Device Lookup — Ideas Backlog

Ideas collected from community feedback, videos, and discussions.
Not yet implemented — revisit when relevant.

---

## Export für Hardware-Controller

**Quelle:** YouTube-Video "Jetzt greift AKAI..." (~min 60), Hosts diskutieren Candct-App, SL Mark 3, Oxi

**Idee:** CC-Mapping eines Geräts in einem Format exportieren, das direkt in Hardware-Controller-Apps importiert werden kann (Candct, SL Mark 3, Oxi, Roto-Control etc.).

**Warum noch nicht:** Jede App hat ein anderes Format — es gibt keinen Standard. Müsste gerätespezifisch implementiert werden.

**Möglicher Ansatz wenn relevant:** Einen generischen JSON/CSV-Export anbieten der Name, CC-Nummer und Range enthält — viele Apps können das importieren. Oder direkt ein Candct-Format implementieren wenn die App an Relevanz gewinnt.

---

## Funktionale Parameter-Gruppierungen / Templates

**Quelle:** YouTube-Video "Jetzt greift AKAI..." (~min 63), Host: "wenn du Gruppierungen machst und sagst, dieses hier ADSR sind vier Werte"

**Idee:** Über die Sektions-Chips hinaus vordefinierte funktionale Templates anbieten — z.B. "Hüllkurve" fasst automatisch alle ADSR-Parameter zusammen, egal in welcher Sektion sie liegen. User wählt "Filter" und sieht sofort die relevanten CCs ohne zu scrollen.

**Warum noch nicht:** Erfordert entweder Keyword-Matching über Parameternamen (fehleranfällig) oder manuelle Annotation der Daten in midi.guide.

**Möglicher Ansatz wenn relevant:** midi.guide bitten, funktionale Tags/Kategorien in die API aufzunehmen. Dann könnten wir direkt darauf filtern.
