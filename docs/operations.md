# Betriebs-Notizen

Gelernte Lektionen aus dem laufenden Betrieb — anonymisiert, ohne konkrete
Dokumenteninhalte oder Namen.

## Nebenläufigkeit

Timer-Lauf und ein manueller Testlauf dürfen nie gleichzeitig auf den
Zustandsspeicher und die Mayan-Cabinets schreiben — das führte einmal zu
doppelten Cabinet-Anlagen für denselben Korrespondenten und einem
beschädigten Zustandsspeicher. Seitdem verhindert ein Lockfile (Dateisperre,
die bis Prozessende gehalten wird) parallele Läufe; ein zweiter Lauf bricht
sofort sauber ab, statt zu warten oder zu kollidieren.

## Cabinet-Auswahl darf nicht zu grob sein

Das LLM schlug gelegentlich ein generisches Sammel-Cabinet (die reine
Personen-Wurzel ohne Kategorie, oder ein fachfremdes Sammelbecken) mit hoher
Zuversicht als "guten bestehenden Treffer" vor — obwohl das keine sinnvolle,
korrespondentenspezifische Ablage ist. Solche Root-/Sammel-Pfade sind seither
explizit aus der Liste wählbarer Cabinets ausgeschlossen.

## Dubletten-Vermeidung bei neuen Cabinets

Wird innerhalb eines einzelnen Laufs mehrfach dasselbe neue Cabinet für
denselben Korrespondenten benötigt (z. B. mehrere Dokumente desselben
Absenders im selben Batch), muss die Liste der bekannten Cabinets sofort
nach Anlage aktualisiert werden — sonst legt der Lauf mehrere leicht
unterschiedliche Cabinets für denselben Korrespondenten an, bevor die erste
Anlage "sichtbar" wird.

Zusätzlich wird vor jeder Neuanlage der *gesamte* bestehende Cabinet-Baum
(nicht nur Geschwister unter derselben Kategorie) nach einem passenden Blatt
für den Korrespondenten durchsucht — auch unter einer anderen Kategorie als
vom LLM vorgeschlagen.

## Korrespondent ≠ Dokumenteninhaber

Das LLM extrahierte gelegentlich den Namen des Archiv-Eigentümers selbst
(statt der externen Gegenseite) als "Korrespondenten", z. B. wenn der eigene
Name im OCR-Text prominent als Adressat auftaucht. Ergebnisse, die auf einen
bekannten Eigentümer-Alias matchen, werden verworfen, statt daraus ein
Cabinet oder Metadatum zu erzeugen.

## OCR-Timeouts nicht stillschweigend verschwinden lassen

Siehe [Dokument-Lebenszyklus](document-lifecycle.md) — Dokumente, deren
OCR-Text dauerhaft ausbleibt (unsupportete Dateiformate, leerer OCR-Inhalt),
bekommen inzwischen einen eigenen Tag und werden periodisch statt gar nicht
mehr erneut geprüft.

## Konfiguration ausgelagert, eigene Web-GUI statt SSH/Editor

Schwellwerte (Confidence, OCR-Warte-/Recheck-Intervalle), die Kategorienliste
für Person B, die Eigentümer-Aliase und der Ollama-System-Prompt waren
ursprünglich in `classify.py` selbst hartkodiert. Sie liegen jetzt in einer
externen `config.json` (analog zu `keyword_rules.csv` für die Stichwort-Regeln)
und werden bei jedem Lauf frisch geladen — Änderungen greifen ohne Neustart
des Timers.

Zur Bearbeitung dient eine kleine, separate Web-Anwendung (eigener
systemd-Dienst, eigener Login) auf demselben Host wie `classify.py` — bewusst
**kein** Mayan-Plugin: Ein echtes Plugin würde Custom-Code im produktiven
Mayan-Container erfordern, inklusive Neustart-Risiko dort und Pflegeaufwand
bei künftigen Mayan-Upgrades (siehe
[Mayan-Workflow-Machbarkeit](mayan-workflow-machbarkeit.md)). Die separate
Web-GUI deckt drei Bereiche ab: Stichwort-Regeln (Tabellen-Editor mit
Hinzufügen/Bearbeiten/Löschen), Schwellwerte/Listen, und den Ollama-Prompt —
plus einen Knopf, um `classify.py` sofort einmal auszulösen, statt bis zu
5 Minuten auf den nächsten Timer-Tick zu warten.

Absicherung: eigener Passwort-Login (gehashtes Passwort, kein Klartext auf
Platte) plus zusätzlich vor die zentrale SSO/2FA-Lösung des Heimnetzes
gehängt — konsistent mit allen anderen internen Admin-Tools dort, nur intern
erreichbar (kein öffentlicher DNS-Eintrag). Beide Schutzschichten bleiben
unabhängig voneinander bestehen (Defense-in-Depth), wie bei den anderen
SSO-gegateten Diensten mit eigenem Fallback-Login üblich.
