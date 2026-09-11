# Unit: Index

Eine Unit besteht normalerweise aus mehreren Dateien. Die Index-Datei (Konvention für Dateiname: \*.ix.json) ist zwingend erforderlich und enthält Referenzen auf die anderen Dateien. Außerdem werden Abhängigkeiten deklariert.

# Spezifikation

## Allgemeine Daten

- `id`: Id der Unit - wird durch den User vergeben
- `uuid`: Optionale universelle Id der Unit. Sie wird für neu erzeugte Units typischerweise automatisch vergeben. Die gesicherte Einmaligkeit hilft beim Wiederfinden von Varianten/Versionen einer Unit
- `modifiedAt`: Markiert den Zeitpunkt der letzten Änderung der Index-Daten. Sollte sich ein externer Datenblock ändern, wird dessen `modifiedAt`-Wert neu gesetzt, aber der `modifiedAt`-Wert für den Index ändert sich nicht.
- Die optionalen Werte für `label` und `description` können Texte enthalten, die bei der Anzeige und der Verarbeitung der Unit genutzt werden.
- Hinweis: Über den externen Datenblock `metadata` können weitere Unit-Metadaten übergeben werden (s. u.).

## Abhängigkeiten/dependencies

In diesem Dokument werden an mehreren Stellen Abhängigkeiten definiert. Das sind Referenzen zu Ressourcen, die notwendig sind für die entsprechende Funktion. Es handelt sich stets um ein Array, das mehrere Typen als Einträge haben kann:

- **Datei**: Folgende Eigenschaften spezifizieren diese Datei:
  - `fileName`: Name der Datei relativ zur Index-Datei (erforderlich)
  - `unpackBeforeProviding`: Wenn `true`, dann erfolgt der Zugriff (Request) nicht auf die Datei selbst, sondern es handelt sich um ein Datei-Archiv mit mehreren Dateien. Dieses Archiv (z. B. ZIP) muss erst ausgepackt werden, und der Zugriff erfolgt dann auf eine dieser Dateien. Der Typ des Archives ist standardmäßig zip (Steuerung ggf. über Dateiendung).
  - `httpResponseMode`: Bei der Auslieferung können Sets von Parametern die Performance verbessern. Beim Modus `STREAM` sollte beispielsweise eine variable Bitrate bei der multipart-Auslieferung eingestellt werden. `STANDARD` wäre ein normales GET.
- **Widget**: Es wird nur ein String übergeben, der den Typ des [Verona-Widgets](https://verona-interfaces.github.io/widget-docs/) angibt. Es wird bei Widgets kein spezifisches Modul angegeben, sondern es muss irgendein Modul dieser Art verfügbar sein.

## UI-Definition `userInterface`

Eine Unit benötigt eine Definition für die Darstellung und ggf. Interaktion durch die Testperson.

- `player`: Zur Anzeige in einem Verona-System muss ein zu der UI-Definition passender Player zur Verfügung stehen. Mit diesem Attribut wird die Id und ggf. die Version des Players angegeben.
- `editor`: Zum Editieren in einem Verona-System kann ein zu der UI-Definition passender Editor genutzt werden. Mit diesem Attribut wird die Id und ggf. die Version des Editors angegeben.
- `type`: Wenn die UI-Definition einer Spezifikation folgt, kann die Id und die Version dieser Spezifikation angegeben werden. Dadurch kann ein alternativer Player oder Editor zugewiesen werden, falls der oben Genannte nicht gefunden wird. Außerdem kann diese Angabe helfen, Kompatibilitätsprobleme zu finden und zu beheben.
- `definition`: Hier ist die eigentliche UI-Definition zu finden, die der Player für die Präsentation und ggf. Interaktion bekommen soll. Es kann sich hier um einen Dateinamen handeln mit diesem Inhalt (Standardname `*.ui.json`) oder um die (maskierte/stringified) Definition selbst. Die Unterscheidung wird über den Schalter `isDefinitionInline` getroffen.
- `isDefinitionInline`: Legt fest, wie die Eigenschaft `definition` zu interpretieren ist. Wenn false (Default-Wert), dann ist dort ein Key als externer Datenblock gespeichert. Wenn true, dann ist der Inhalt von `definition` direkt die UI-Definition. Hinweis: Es kann auch Player geben, die keine UI-Definition benötigen, wie z. B. kleine Spiele, die in den Testverlauf eingestreut werden. Dann fehlt `definition` oder ist leer.
- `modifiedAt`: Zeitpunkt der letzten Änderung
- `playerDependencies`, `editorDependencies`: Abhängigkeiten, die für die Funktionalität des Players bzw. Editors bereitgestellt werden müssen.

## Externe Datenblöcke

Die folgende Tabelle listet alle möglichen externen Datenblöcke. Es handelt sich jeweils um dieselbe Datenstruktur:

- `id`: Dieser String wird genutzt, um den Datenblock zu finden. Es handelt sich üblicherweise um den Namen einer Datei, die im selben Verzeichnis liegt wie die Index-Datei.
- `type`: Die externe Datei folgt i.d.R. einer Spezifikation. Die Id und die Version können im `type`-Attribut angegeben werden und helfen, Kompatibilitätsprobleme zu finden und zu beheben. Bei allen Datenblöcken gibt es einen Default-Typ (s. u.).
- `modifiedAt`: Zeitpunkt der letzten Änderung
- `dependencies`: Abhängigkeiten, die ggf. für die Verwendbarkeit des Datenblockes bereitgestellt werden müssen.

Folgende Datenblöcke werden auf diese Art referenziert:

| Tag | Erläuterung | Relevante Spec | Konvention Dateiname |
|----|----|----|----|
| `codingScheme` | **Kodieranweisungen/Kodierschema**: Vorschriften, wie die Antworten der Unit zu kodieren sind. | [iqb-coding-scheme](https://iqb-specifications.github.io/coding-scheme/) | \*.cs.json |
| `comments` | **Kommentare**: Formatierte hierarchische Texte (Html ggf. mit eingebetteten Bildern) zur Diskussion während der Entwicklungszeit | [iqb-unit-comments](https://iqb-specifications.github.io/unit-comments/) | \*.co.json |
| `richNotes` | **Formatierte Texte/ Begleitmaterial**: Formatierte Texte (Html ggf. mit eingebetteten Bildern) mit unterschiedlichen Verwendungszwecken (z. B. didaktische Kommentare, Transcript) und Links. | [iqb-unit-rich-notes](https://iqb-specifications.github.io/unit-rich-notes/) | \*.rn.json |
| `metadata` | **Metadaten der Unit**: In einem standardisierten JSON-Format werden Verweise auf Vokabulare und Metadatenprofile gespeichert. | [metadata-values](https://iqb-specifications.github.io/metadata-values/) | \*.md.json |
| `items` | **Items**: Liste von Items mit Metadaten und Zuordnung von Variablen | [iqb-unit-items](https://iqb-specifications.github.io/unit-items/) | \*.it.json |
| `variables` | **Variablen**: Es werden alle möglichen Variablen aufgeführt, die die Antwortwerte enthalten. Die JSON-Datei enthält zwei Einträge `baseVariables` und `derivedVariables`. | [unit-variables](https://iqb-specifications.github.io/unit-variables/) | \*.va.json |

Zurück nach oben
