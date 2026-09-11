# Konzepte

Die Liste auf der Startseite dieser Dokumentation bietet eine schnelle Navigation zu spezifischen Definitionen. Viele dieser Einträge haben ein Symbol . Dies ist ein Link zu einer ausführlicheren Dokumentation, sofern die Spezifikation im OpenAPI- bzw. im AsyncAPI-Format nicht ausreicht.

Auf den Seiten im Bereich “Konzepte” finden Sie übergreifende Texte. Mehrere Spezifikationen sind in ihrem Zusammenwirken dargestellt und die Anwendungen, die diese Daten bzw. Schnittstellen verwenden und implementieren, sind erläutert.

Die Zielgruppe für diese Texte sind Personen, die APIs oder Daten genau verstehen sollen - für die Programmierung von Datenauswertungen, Datenumformungen, Datenbanken, Tools, Grafiken einer Ergebnisrückmeldung usw..

| Konzept | Beschreibung | Datum |
|----|----|----|
| [Antwort](../concepts/response/index.llms.md) | Während der Durchführung einer Befragung oder eines Tests entstehen durch die Interaktionen Antwortdaten. Es handelt sich hierbei um eine einfache Datenstruktur, die anschließend in vielen Zusammenhängen verarbeitet wird. |   |
| [Unit](../concepts/unit/index.llms.md) | Die Unit ist Teil einer Befragung oder eines Tests und enthält mindestens ein Interaktionselement. Sie besteht aus mehreren Datenblöcken (z. B. UI-Definition, Kommentare, Metadaten, Items) und wird vor allem bei der Testdurchführung benötigt. | 11.09.2026 |

# Hinweise für Autor\*innen

## Liste der Konzepte

Die obige Liste ist automatisch erzeugt. Dazu muss jedes Konzept in einem separaten Ordner abgelegt werden, der über eine `index.qmd` verfügt. Die Kopfdaten dieser Datei (sog. front matter) werden für die Tabelle ausgelesen: title, description und date.

## Externe README.md

Jede Spezifikation sollte über eine README.md verfügen. Diese kann man in einem Konzept einbinden, da diese Quarto-Seiten die Erweiterung `external` nutzen. Beispiel einer Einbindung:

    {{< external https://raw.githubusercontent.com/iqb-specifications/acp-scale-derived/refs/heads/main/README.md >}}

Quarto ist ein System zum Generieren statischer Webseiten. Die obige Anweisung zum Einbinden wird nur zum Zeitpunkt des Renderns ausgeführt. Um sicherzustellen, dass bei einer Änderung der Spezifikation und dann der README.md auch diese Quarto-Seiten neu gerendert werden, muss das Repository der Spezifikation ein Rendern eine `POST`-Nachricht an Quarto schicken. Siehe hierzu die Definition bei Webhook bzw. die Action-Dateien im Ordner `.github/workflows`!

## Html der Spezifikation einbinden

Soll die Spezifikation selbst eingebunden werden, kann dies über ein iFrame erfolgen. In Quarto wäre folgende Syntax zu wählen:

    ![](https://iqb-specifications.github.io/acp-scale-base/index.html){width="100%" height="600px"}

Zurück nach oben
