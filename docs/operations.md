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
