# Plug-ins - Kommunikation

Die Interaktion zwischen der Hostanwendung und dem Plug-in folgt einem einheitlichen Schema:

``` mermaid
sequenceDiagram
    participant HA as Host-Anwendung

    participant VM as Plug-in
    HA->>VM:Initialisierung
    VM->>HA:vo_ReadyNotification
    HA->>VM:vo_StartCommand
    loop
        Actor TP as Person
        VM->TP:Interaktion
        VM->>HA:vo_StateChangedNotification
    end
```

- **Initialisierung**: Der `iframe` wird angelegt und der Plug-in-Code z. B. über das Attribute `srcdoc` darin platziert.
- **vo_ReadyNotification**: Das Plug-in schickt eine Meldung an den Host, dass alle erforderlichen Initialisierungen abgeschlossen sind.
- **vo_StartCommand**: Der Host schickt spezifische Daten zum Plug-in (z. B. die UI-Definition der Unit oder das Kodierschema) und startet damit die Darstellung und Interaktion.
- Danach meldet das Plug-in üblicherweise Zustandsänderungen **vo_StateChangedNotification**.
- Im Verlauf der Kommunikation können dann noch von beiden Seiten weitere Informationen ausgetauscht werden.

> **NOTE:**
>
> Das Zeichen `_` ist ein Platzhalter für die jeweilige konkrete Bezeichnung der Operation. `vopStartCommand` ist z. B. die Bezeichnung des Start-Kommandos des Players.

# Nachrichten empfangen

Die Kommunikation kann nur erfolgen, wenn die Host-Anwendung Nachrichten an das `window`-Objekt abfängt. Das Verona-Modul kann nur allgemein an seine Host-Anwendung Nachrichten schicken, nicht gezielt an eine bestimmte Komponente oder ein bestimmtes DOM-Element. Auf höchster Ebene muss also z. B. so etwas einmalig aufgerufen werden:

``` typescript
 window.addEventListener('message', (event: MessageEvent) => {
      const msgData = event.data;
      const msgType = msgData.type;
      if ((msgType !== undefined) &&
            (msgType.substr(0, 2) === 'vo')) {
        this.tcs.postMessage$.next(event);
      }
    });
```

In diesem Beispiel prüft die Funktion, ob der Typ der Nachricht mit ‘vo’ beginnt. In diesem Fall wird angenommen, dass es sich um eine Nachricht von einem Plug-in handelt und ein Observable wird auf einen neuen Wert gesetzt. An passenden Stellen im Host kann dann darauf reagiert werden.

An das `<iframe>`-Element können Nachrichten beispielsweise so geschickt werden:

``` typescript
  this.postMessageTarget.postMessage({
        type: 'vopStartCommand',
        sessionId: this.itemPlayerSessionId,
        unitDefinition: pendingUnitDef,
        unitState: {
          dataParts: pendingUnitDataToRestore
        },
        playerConfig: this.tcs.fullPlayerConfig
      }, '*');
```

> **WARNING:**
>
> Bei schnellen Tastatureingaben oder ambitionierter Beobachtung des Userverhaltens (Logging) fallen sehr viele Daten an. Beim Player-Plug-in kann man die Daten aufteilen (sog. `dataParts`), aber auch sonst könnte es viel werden. Die Hostanwendung sollte einen Puffer einrichten und nicht alles sofort wegspeichern. Bei `rxjs` gibt es beispielsweise den Operator `debounceTime`.

# Gemeinsame Parameter `sharedParameters`

Plug-ins kennen einander nicht. Sie werden nach Bedarf geladen in einer nicht vorhersehbaren Reihenfolge. Allerdings gibt es Situationen, in denen Daten von einem Plug-in zum anderen weitergegeben werden soll. Beispiele:

- Ein Player kann bei einem Convertible nicht verlässlich erkennen, ob eine Tastatur vorhanden ist. Vorsichtshalber wird der Player die eigene Tastatur einblenden, und die Testperson wird diese virtuelle Tastatur ausblenden, um die des Gerätes zu verwenden. Das passiert jedoch bei jeder Unit neu, wenn der Player geladen wird. Besser wäre, dass die Einstellung “bitte keine virtuelle Tastatur” an folgende Player weitergegeben wird.
- Ein Schemer hat verschiedene UI-Modi - je nachdem, ob automatische oder manuelle Kodierung definiert werden soll. Den gewählten Modus an folgende Schemer weiterzugeben, wäre eine gute UX.
- Zu Beginn der Testdurchführung wählt die Testperson einen Begleiter (Avatar). Wenn diese Figur in jeder Unit auftauchen soll wäre es klug, dem Player die Auswahl mitzuteilen und damit die richtige Figur einzublenden. Es wäre alternativ sehr aufwändig, für jede Unit getrennte Varianten für jeden Avatar zu entwickeln und dann über adaptiven Testverlauf die korrekte Anzeige zu erreichen.

Unter `sharedParameters` sind also Daten zu verstehen, die ein Plug-in während der Laufzeit einem anderen Modul weitergeben kann. Es kann sich um ein Modul derselben Art handeln. Wenn man übergreifende Konventionen für die Parameter entwickelt, können auch unterschiedliche Module aus verschiedenen Quellen diese Informationen nutzen. Daher enthalten alle Plug-ins in ihrem Kommunikationsmodell diese Art von Datenaustausch.

Die Spezifikation eines Plug-ins nutzt eine einfache Struktur `key` und `value`.

``` json
[
  {
    "key": "AVATAR",
    "value": "CRAB"
  },
  {
    "key": "BACKGROUND_COLOR",
    "value": "#34F"
  },
  {
    "key": "UI_MODE",
    "value": "RULES_ONLY"
  }
]
```

# Korrekte Zuordnung der Modul-Instanz bzw. der Daten

Bei den Nachrichten gibt es zwei Eigenschaften, die eine korrekte Zuordnung unterstützen:

- `sessionId`: Diese ID wird beim Startkommando vom Host vergeben und wird vom Plug-in bei jeder Nachricht mitgeschickt. Diese ID wird durch das Plug-in nicht verändert. Der Host kann darüber das Plug-in-Element identifizieren und daher sicherstellen, dass bei den überwiegend asynchronen Vorgängen im Browser die Daten stets z. B. der richtigen Unit zugeordnet werden.
- `timeStamp`: Auch diese Eigenschaft wird stets vom Plug-in mitgeschickt. Darüber kann sichergestellt werden, dass die Reihenfolge der Nachrichten nicht vertauscht wird. Es ist stets die jeweils letzte Änderung feststellbar.

# Beenden: unnötig

Da Änderungen sofort gemeldet werden, ist ein formelles Beenden des Plug-ins unnötig. Es gibt bei keinem Plug-in Bedarf, vor Entfernen des Moduls bestimmte Funktionen aufzurufen. Wenn das Modul nicht mehr benötigt wird, kann das `<iframe>`-Element entfernt oder geleert werden (`srcdoc=""`).

#  Sicherheitshinweis

Über die Verwendung von postMessage() in den [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage#notes):

> Any window may access this method on any other window, at any time, regardless of the location of the document in the window, to send it a message. Consequently, any event listener used to receive messages **must** first check the identity of the sender of the message, using the `origin` and possibly `source` properties. This cannot be overstated: **Failure to check the `origin` and possibly `source` properties enables cross-site scripting attacks.**

Host-Anwendung und Plug-in sollten also alles unternehmen, die Identität des Senders festzustellen. Dazu kann man die Html-Elemente mit IDs versehen und die sessionId verwenden.

Zurück nach oben
