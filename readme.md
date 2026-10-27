Raspberry Pico MIDI Lichtorgel Projekt
======================================

Idee: Eine MIDI Lichtorgel. Die Note bestimmt den LED Strip, die Anschlagstärke den Ausschlag des LED Strips. Farbe und weitere Effekte lassen sich programmieren. Wenn mehrere Midi Kanäle gleichzeitig spielen können sie verschiedenen Farben zugeordnet werden.

### 1. Hardware

Hauptkomponenten:
*   **Controller:** Raspberry Pi Pico 2
*   **LED-Streifen:** Eine Matrix aus mind. 72*32 Led's um mehrere Oktaven abbilden zu können (12 Halbtöne * 6 Oktaven = 72 Spalten)
*   **MIDI-Schnittstelle:** Um von einer Tastatur oder Software Signale zu empfangen. Falls möglich sowohl DIN Midi als auch USB
    *   *Option A (Hardware):* Ein optokoppler-basierter MIDI-Eingang (Standard DIN-Buchse) an einem UART-Pin des Pico. Das erfordert etwas Schaltungsaufbau (Optokoppler 6N138, Widerstände).
    *   *Option B (Software/USB):* Pico agiert als als USB-MIDI-Gerät und empfängt direkt die Daten
* **Platine** Breadboard / Platine zum verbinden der LED's mit dem Microcontroller
* **Netzteil**: Externes 5V Netzteil (USB?) um die LED's zu versorgen
    *   *Wichtig:* Verbinde die Masse (GND) des Netzteils zwingend mit dem GND des Picos, damit sie das gleiche Referenzpotential haben.
*   **Datenleitung:** Die Datenleitung vom Pico zum ersten LED-Strip muss einen Vorwiderstand (ca. 330 Ohm) haben, um Signalreflexionen zu minimieren. Bei langen Kabeln kann ein Kondensator (ca. 1000µF) parallel zur Stromversorgung der ersten LEDs helfen, Spannungsspitzen beim Einschalten abzufangen.
* Gehäuse: Soll schön aussehen! Plexiglas Platte über den LED's, der Rest aus Blech, Holz, 3D gedruckt?? Holz Seiten sähen schön nach Synthesizer aus!

### 2. Logik & Algorithmen

Zentral ist die Zuordnung von MIDI-Daten zu LED-Zuständen.
 
**Midi-to-LED Mapping:**
Eine einzelne Note aktiviert nicht nur einen Streifen, sondern definiert eine "Helligkeit" oder "Intensität". Da LEDs nur Werte zwischen 0 und 255 (oder 0.0 bis 1.0) annehmen können, musst du die MIDI-Anschlagsstärke (Velocity, 0-127) auf diesen Bereich abbilden. Die Zuordnung soll aber in der Software definierbar sein. So kann z.B. Velocity auf Farbe und/oder Insensität gemapped werden.

**Der grundlegende Mapping-Ansatz:**
1.  **Note zu Streifen:** Die Streifen bilden einen festen Bereich an Noten z.B. C0-C6 ab. Dieser muss eigentlich nie geändert werden, da er den "normalen" Bereich an Noten abdeckt der hauptsächlich genutzt wird, und wenn man andere Bereiche abbilden will kann man dies über Midi Transpose Effekte im PC machen.
    *   *Beispiel:* Streifen 1 bekommt die Töne C0, Streifen 2 bekommt C#0, Streifen 3 D0 usw.
    *   Alternativ: Wir nutzen Frequenzen. Jede Note hat eine feste Frequenz. Du kannst berechnen, welcher Streifen basierend auf der Frequenz zuständig ist. Dadurch wäre man flexibler was die Anzahl und Aufteilung der LED's angeht, aber es ergeben sich Überlappungen.
2.  **Velocity zu Helligkeit:**
    *   Velocity 0 = Note Off (LED aus oder Fade-Out starten).
    *   Velocity 1-127 = Lineare oder logarithmische Skalierung auf 0-255 Helligkeit.
    *   *Tipp:* Eine logarithmische Skalierung wirkt oft natürlicher für das menschliche Auge, da unsere Wahrnehmung von Helligkeit nicht linear ist.
3.  **Nachhall/Abklingen (Decay):**
    *   Wenn eine Note losgelassen wird (Note Off), sollte das Licht nicht sofort ausgehen. Implementiere einen langsamen Ausblendeffekt (Fade-out), damit es wie ein akustischer Nachhall wirkt.
4.  **Midi Kanal zu Farbe** Wenn mehrere Kanäle gleichzeitig spielen soll jeder Kanal eine eigene Farbe bekommen.
5. **Helligkeitsbegrenzung** Da es zu viel Strom ziehen würde wenn alle LED's gleichzeitig rein weiß leuchten würden muss dies begrenzt werden. Wir können vermutlich insgesamt die Helligkeit auf <50% begrenzen. Zusätzlich können wir per Algorithmus die maximale "Belastung" begrenzen, also Anzahl LED's * Helligkeit < Grenzwert

### 3. Software

Da die Hardware selber so "clean" wie möglich aussehen soll, muss sie per Software konfigurierbar sein um nicht viele Hardware Buttons zu benötigen.
Über die Software sollen mindestens folgende Dinge konfigurierbar sein:

* Mapping von Midi Parametern auf Helligkeit / Farbe / Ausschlag
* Auswahl Midi Kanäle, Mapping auf Farbe

