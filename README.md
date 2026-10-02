<p align="center">
  <img src="assets/styles.gif" alt="Claude, das kleine orange Wesen, spielt Coding-Momente in allen elf Stilen nach: Pixel Art, das Original, Höhlenmalerei, Blaupause, Mosaik, Frutiger Aero, Kupferstich, Millefleur-Wandteppich, Golden-Age-Comic, Ukiyo-e und Kamon" width="960">
</p>

<h1 align="center">Claude Fables</h1>

<p align="center">
  <b>Deine Arbeit, als Cartoon erzählt, während Claude programmiert.</b><br>
  Eine Claude-Code-Mod für die Desktop-App, die in der Leiste über dem Prompt eine kleine animierte Geschichte abspielt.
</p>

<p align="center">
  <img alt="Claude Code mod" src="https://img.shields.io/badge/Claude_Code-mod-d97757">
  <img alt="Desktop app" src="https://img.shields.io/badge/runs_in-the_desktop_app-3b3a36">
  <img alt="11 styles" src="https://img.shields.io/badge/styles-11-6a5acd">
  <img alt="20 actions" src="https://img.shields.io/badge/actions-20-2e8b57">
  <img alt="7 scenes" src="https://img.shields.io/badge/scenes-7-c46a2a">
</p>

<p align="center">
  <a href="#schnellstart">Schnellstart</a> ·
  <a href="#so-funktioniert-es">So funktioniert es</a> ·
  <a href="#benutzung">Benutzung</a> ·
  <a href="#die-szenen">Die Szenen</a> ·
  <a href="#captions">Captions</a> ·
  <a href="#stile">Stile</a> ·
  <a href="#entwickeln">Entwickeln</a>
</p>

> [!NOTE]
> Dies ist ein deutscher Fork von [Claude Fables](https://github.com/henrik-thevibe/Claude-Fables): Der Narrator schreibt seine Captions auf Deutsch, und diese README ist übersetzt. Änderungen des Originals kommen wöchentlich als Pull Request (`.github/workflows/sync-upstream.yml`).

> [!NOTE]
> Claude Fables wurde komplett in Claude-Code-Cloud-Umgebungen entwickelt und zusätzlich lokal in der Desktop-App getestet. Trotzdem kann es Fehler und Ecken und Kanten geben. [Issues](https://github.com/henrik-thevibe/Claude-Fables/issues) und Pull Requests sind sehr willkommen.

Während Claude arbeitet, beobachtet Fables jeden Tool-Aufruf und jede Zeile, die Claude sagt. Alle paar Sekunden bittet es Sonnet (oder auf Wunsch Haiku), den letzten Moment als Szene nachzuerzählen. Aus der Fehlersuche wird eine Tierdokumentation, und schlechte Regexe werden von der Zugpolizei rausgewinkt. Claude erscheint als kleines orangefarbenes Wesen, das durch die Geschichte läuft, schleicht oder fliegt. Am Ende eines Durchgangs gibt es eine Schlussszene, die 30 Sekunden stehen bleibt.

- **Alle paar Sekunden eine Szene:** Jeder Tool-Aufruf und jede Zeile von Claude wird zu einem Moment der Geschichte.
- **20 Aktionen:** Claude läuft, schleicht, gräbt, stolpert, zuckt mit den Schultern, schläft und feiert, passend zur Arbeit.
- **7 handgeleuchtete Szenen:** Wald, Weltraum, Stadt, Wüste, Vulkan, Labor und Dorf bei Nacht.
- **11 Stile:** von Pixel Art bis Ukiyo-e, jedes Element im jeweiligen Medium neu gezeichnet.
- **Captions, die wie ein Terminal lesen:** Dateien, Funktionen, Zahlen, Fehlschläge und Erfolge setzen sich jeweils ab.
- **Sicher gebaut:** Das Modell schreibt Daten, nie Code, und eine schlechte Antwort wird einfach übersprungen.

## Schnellstart

<p align="center"><img src="assets/install.gif" alt="Claude trägt einen Karton durch die Wüste: Umzug nach ~/code/Claude-Fables..." width="960"></p>

1. Klone dieses Repo an einen festen Ort, zum Beispiel nach `~/code/Claude-Fables`. Für diesen Fork:

   ```sh
   git clone -b deutsch https://github.com/Luxx1993/Claude-Fables-Plugin ~/code/Claude-Fables
   ```

2. Trage es im `env`-Block von `~/.claude/settings.json` ein:

   ```json
   {
     "env": {
       "CLAUDE_CODE_PLUGIN_DIRS": "/Users/du/code/Claude-Fables"
     }
   }
   ```

   Um an der Mod mit Hot Reload zu arbeiten, füge zusätzlich `"CLAUDE_CODE_PLUGIN_DIR_WATCH": "1"` hinzu.
3. Starte die Desktop-App neu und gib Claude im Code-Tab eine Aufgabe. Der Cartoon erscheint nach den ersten Sekunden über dem Prompt.

Um es nur für eine Sitzung im Terminal auszuprobieren, starte `claude --plugin-dir ~/code/Claude-Fables`. Beachte: Im Terminal beobachtet die Mod nur und zeichnet nichts.

## So funktioniert es

<p align="center"><img src="assets/how-it-works.gif" alt="Claude prüft das Aktivitätsprotokoll in einem Labor spät in der Nacht" width="960"></p>

```
Tool-Aufrufe, Claudes eigene Worte ──► Aktivitätsprotokoll (letzte 14 Zeilen)
                                         │  wenn eine Szene gelesen wird, ≥5 s Abstand
                                         ▼
        Sonnet oder Haiku ($.model.complete) ──► JSON-Szene ──► parseScene (prüfen + begrenzen)
                                                                  │
                                                                  ▼
                                    sceneToSvg ──► ein selbstanimiertes SVG (SMIL)
                                                                  │
                                                                  ▼
                                        <Svg isInteractive/> in der AbovePrompt-Leiste
```


- **Das Modell schreibt Daten, keinen Code.** Jede Szene ist ein kleines deklaratives JSON-Objekt: ein Hintergrund, Claudes Aktion, Partikel, eine Caption und ihr Ton. `hooks/scene.ts` prüft es streng: Unbekannte Felder fallen weg, Zahlen werden begrenzt, Strings geglättet und gekürzt, Farben müssen 3- oder 6-stellige Hex-Werte sein. Eine schlechte Antwort kann nichts kaputt machen; sie wird nur nicht angezeigt.
- **Die Animation läuft im SVG.** `hooks/svg.ts` übersetzt eine Szene in ein einziges SVG-Dokument, das sich per SMIL selbst animiert: Gehzyklen, Wippen, ziehende Züge, funkelnde Sterne und eine Sprechblase, die sich selbst tippt. Ist die Szene einmal gezeichnet, braucht der Desktop dafür keine Neuzeichnungen mehr.
- **Jede Zeile wird gelesen.** Szenen laufen nacheinander aus einer Warteschlange. Jede bleibt stehen, bis ihre Blase fertig getippt und lange genug zum Lesen zu sehen war, was bei längeren Captions länger dauert. Nichts drängelt sich dazwischen: kein Fehlschlag, nicht das Ende des Durchgangs, nicht der nächste Prompt. Der Narrator fragt die nächste Szene etwas vor dem Ende der aktuellen an, abgestimmt darauf, wie schnell das Modell zuletzt geantwortet hat, damit sie rechtzeitig bereit ist. Eine Szene, die früh ankommt, wartet in der Schlange. Neuigkeiten (ein Fehlschlag, das Ende des Durchgangs) werden sofort angefragt und reihen sich hinter der aktuellen Szene ein. Dem Narrator wird gesagt, was der Held gerade gesagt hat, damit die Neuigkeit innerhalb der Geschichte hereinplatzt („Moment-“, „Oh!“) statt auf dem Bildschirm. Kommt ein neuer Prompt, wird die aktuelle Szene noch zu Ende gelesen, und eine noch unterwegs befindliche Schlussszene läuft, bevor die neue Geschichte beginnt. Ist ein Durchgang vorbei, bleibt seine letzte Szene 30 Sekunden stehen, dann wird die Leiste leer.

<details>
<summary><b>Unter der Haube:</b> Warteschlange, sanfte Übergänge, Caption-Anpassung, Grenzen, Größe</summary>

- **Übergänge sind sanft.** Eine Szene in neuer Umgebung blendet aus der letzten ein; eine in derselben Umgebung läuft direkt weiter. Jede Szene wird einmal gezeichnet und behalten, sodass der Desktop sie durchspielt. Der Desktop zeichnet die Leiste neu, sobald sich ihre Props ändern, und das Ende eines Durchgangs ändert sie immer. Deshalb wird jede Neuzeichnung einer sichtbaren Szene (Ende des Durchgangs, Größenänderung, neuer Stil) auf die eigene Uhr der Szene gesetzt: Die Caption ist genau so weit getippt wie zuvor, und Claude ist genau so weit in seiner Bewegung.
- **Captions passen in die Blase.** Der Narrator soll höchstens 70 Zeichen schreiben. Die harte Grenze liegt bei 80, so viel zeigt die Blase in vier Zeilen vollständig. Eine längere Caption wird nach dem letzten ganzen Satz abgeschnitten oder sonst nach einem ganzen Wort mit Auslassungspunkten, nie mitten im Wort. Die Blase zeigt immer jedes Wort, das sie bekommt.
- **Es gibt Grenzen.** Es läuft nur eine Modellanfrage gleichzeitig, Szenen kommen mindestens 5 Sekunden auseinander, und nach Fehlern wartet es exponentiell länger, bis zu 60 Sekunden. Der Prompt bleibt immer begrenzt (die letzten 14 Aktivitätszeilen und die letzten 4 Szenen), sodass eine lange Sitzung das Kontextfenster nicht sprengt.
- **Es passt ins Fenster.** Die Leiste bekommt immer einen Cartoon, der so breit ist wie sie selbst und 192 px hoch. Ein breiteres Fenster zeigt mehr von der Szene, nicht eine größere, sodass Bild und Text gleich groß bleiben. Eine Größenänderung des Fensters zeichnet neu.
- **Nur Desktop.** Im Terminal bleibt die Leiste genau so, wie die Engine sie zeichnet.

</details>

<details>
<summary><b>Die Dateien</b></summary>

| Datei | Was sie tut |
| --- | --- |
| `hooks/register.tsx` | Die Hooks: beobachtet Tool-Aufrufe, Antworten und Durchgänge, zeichnet die Leiste, behandelt `/fables` |
| `hooks/narrator.ts` | Der System-Prompt des Narrators, Prompt-Aufbau, Antwort-Parsing, Backoff |
| `hooks/director.ts` | Die Schleife des Narrators: was er sich merkt, wann er fragt, was er mit einer Antwort tut. Plugin und Viewer führen dieselbe aus |
| `hooks/activity.ts` | Kocht jeden Tool-Aufruf auf eine lesbare Zeile ein |
| `hooks/scene.ts` | Das Szenenformat und sein Validator |
| `hooks/sprites.ts` | Die Pixel-Art-Bibliothek: Claude in zwei Gehbildern plus über 30 Props |
| `hooks/svg.ts` | Szene → animiertes SVG: Hintergründe, Partikel, Props, Held, Caption |
| `hooks/looks.ts` | Die Looks: Pixel Art, das Original und das Register der Stile |
| `hooks/styles/*.ts` | Die neun Galerie-Stile, jeder eine Art Bible: Tinten, wie jede Art von Element gezeichnet wird, Claude, Caption, Tag, Rahmen |
| `hooks/art/roles.ts`, `hooks/art/painter.ts`, `hooks/art/ink.ts` | Die Rollen, unter denen Szenen ihre Elemente malen, und der Maler, mit dem ein Stil sie neu zeichnet |
| `hooks/grade.ts` | Das Medium, in das ein Raster-Stil die gezeichnete Bühne setzt: Tesserae oder ein Gewebe |
| `hooks/clawd3d.ts` | Das 3D-Claude: das Boxmodell, seine Bewegungen und seine Projektion (aus der Galerie portiert) |
| `hooks/hero3d.ts` | Backt das posierte 3D-Modell in SVG-Bilder, die SMIL nacheinander abspielt |
| `hooks/scenery.ts` | Die sieben gestalteten Szenen: Lichtkarten, Materialien, Spiegelungen |
| `types/index.d.ts` | Die Szenentypen und der `$.state`-Vertrag der Mod |

</details>

## Benutzung

<p align="center"><img src="assets/use.gif" alt="Claude winkt im Wald: /fables on, und hallo!" width="960"></p>

- `/fables`: schaltet die Mod an oder aus. `/fables on` und `/fables off` funktionieren auch. Die Einstellung wird über Sitzungen hinweg gemerkt.
- `/fables style <name>`: zeichnet jede Szene in einem der unten genannten Stile, zum Beispiel `/fables style ukiyo-e` oder `/fables style golden age`. `/fables style` listet sie auf, und `/fables style off` geht zurück zum Standard, Pixel Art. Wird über Sitzungen hinweg gemerkt.
- `/fables pixel off` und `/fables pixel on` funktionieren weiterhin: Sie wechseln zwischen dem Original-Look glatt gezeichnet und seiner Pixel-Art-Fassung.
- `/fables model haiku` und `/fables model sonnet`: wählen, wer die Geschichte schreibt. Sonnet ist der Standard und schreibt witzigere Szenen; Haiku antwortet in etwa einer Sekunde und kostet weniger, schreibt aber schlichtere Captions und patzt etwas öfter (eine schlechte Antwort wird einfach übersprungen). `/fables model` zeigt, welches aktiv ist. Die Wahl wird über Sitzungen hinweg gemerkt; **Scene model** im Konfigurationsmenü (`pluginConfigs.fables.model`) legt den Standard fest.

Jede Szene ist eine kleine Modellanfrage, das macht also ein paar Anfragen pro Minute, solange Claude arbeitet.

## Die Szenen

<p align="center"><img src="assets/scenes.gif" alt="Ein Rundgang durch die sieben Szenen: Wald, Weltraum, Stadt, Wüste, Vulkan, Labor und Dorf bei Nacht" width="960"></p>

Claude wird als das 3D-Modell der Galerie gezeichnet: derselbe Boxkörper, dieselben Arme, Beine und Pillenaugen, beleuchtet und nach Tiefe sortiert. Der Rahmen der Leiste führt kein Skript aus, das Modell kann also nicht live gezeichnet werden. `hooks/clawd3d.ts` bringt es pro Bewegung in 6 bis 12 Posen, `hooks/hero3d.ts` backt jede Pose in flache SVG-Polygone, und die Szene blättert per SMIL durch sie. Das Modell kann dem Cursor nicht folgen und nicht gezogen werden; dafür braucht es die Live-Engine.

Der Narrator wählt für jede Szene eine von 20 Aktionen, jede an eine Art von Arbeit geknüpft:

| spielt | Aktionen |
|---|---|
| quer über die Bühne, von `from` nach `to` | walk, run, fly, carry (Dateien verschieben), sneak (Fehlersuche), jump, tumble (Hindernisse, Wiederholungen) |
| an Ort und Stelle, in Schleife | dig (Suchen), inspect (Code lesen und bearbeiten), think, point (gefunden), peek, spin (Refactorings), wave (Hallo), sleep (lange Wartezeiten), panic (Fehler), dance, celebrate (Meilensteine) |
| an Ort und Stelle, einmal | trip (ein Test schlägt fehl), shrug (nichts gefunden) |

Eine Szene kann mit `then` eine zweite Aktion anhängen, die dort an Ort und Stelle läuft, wo die erste endet: ein Sneak und dann ein Peek, ein Trip und dann ein Shrug, ein Jump und dann ein Celebrate. Ohne eine solche steht Claude nach dem Queren oder nach einer einmaligen Aktion untätig da, und ein untätiger Claude sieht sich um, blinzelt, verlagert sein Gewicht und streckt sich. Dabei werden seine Augen groß, verengen sich vor Konzentration, schielen bei Schwindel und sorgen sich unter einer Braue.

Jeder der sieben Hintergründe in `hooks/scenery.ts` ist eine gestaltete Szene mit einem Briefing, einer Lichtquelle und einer kleinen Palette:

| Hintergrund | Die Szene |
| --- | --- |
| forest | Morgendämmerung in einem alten Wald: eine tiefe Sonne hinter zwei Reihen Tannen mit braunen Stämmen, Nebel dazwischen, Licht, das in Strahlen einfällt und sich auf einer Lichtung sammelt, zwei mächtige Stämme rahmen die Ränder |
| space | Erdaufgang über einem Mondaußenposten: eine tiefe Sonne streift den Regolith, sodass jede Wölbung einen hellen Kamm hat, jeder Krater eine schwarze Schale mit heller ferner Wand, jeder Fels einen langen Schatten |
| city | Blaue Stunde nach dem Regen: Türme mit hell beleuchteten Westkanten und dunklen Ostseiten, Büros, die Stockwerk für Stockwerk leuchten, eine Turmspitze, eine Hochbahn, und die ganze Skyline spiegelt sich in der nassen Straße |
| desert | Sonnenuntergang an der Mesa: Die Sonne geht in einer Lücke unter, die die Bergketten freilassen, sodass sie uns im violetten Schatten zugewandt sind und ihre Sonnenseiten brennen, ihre Schatten fächern sich über den Sand auf uns zu |
| volcano | Ein nächtlicher Ausbruch: Der Krater beleuchtet seine eigene Aschesäule von unten, Lava fließt einen zerfurchten Kegel hinab, und ein Bach quert eine schwarze Kruste voller glühender Risse |
| lab | Spät noch am Arbeiten: Eine Architektenlampe wärmt den schalungsrauen Beton und die Werkbank, Staub dreht sich in ihrem Kegel, Regen perlt am Fenster über einer Stadt, die sich zu Bokeh öffnet, und der polierte Boden spiegelt den Raum |
| night | Ein schlafendes Dorf: Hügel unter einem hohen Mond, Häuschen mit einem beleuchteten Fenster und einem Rauchfaden, eine große Eiche rahmt die Aussicht, Glühwürmchen |

<details>
<summary><b>Wie sie beleuchtet werden</b></summary>

Jede Fläche wird zweimal gemalt. Zuerst als Licht: ein warmes Hauptlicht, wo die Lichtquelle hinreicht, kühler Schatten auf der abgewandten Seite, tiefe Töne, wo Flächen aufeinandertreffen. Dann wird diese Lichtkarte mit einem Material multipliziert, das in einem SVG-Filter aus gesetztem Rauschen entsteht und in eine kleine Palette verwandter Farben geschnitten wird: Nadeln, Rinde, Gras, Basalt, Sandstein, Sand, Regolith, Beton, Holz, Asphalt. Licht, das ein dunkles Material aufhellen muss (Strahlen, Lavaglut, Lampenpfützen), wird stattdessen mit einem Screen-Blend obendrauf addiert. Leuchten blühen auf, ferne Ebenen liegen leicht unscharf, Lichtränder erscheinen nur dort, wo das Licht tatsächlich hinreicht, und ein Linsendurchgang legt feines Korn und eine Vignette über das ganze Bild, Claude eingeschlossen.

Nasse und polierte Böden spiegeln: Die Skyline der Stadt und der Raum des Labors werden einmal gezeichnet und dann kopfüber, unscharf und von Wellen gebrochen noch einmal platziert, am stärksten in den Pfützen.

Tiefe entsteht durch Luftperspektive: Weiter entfernte Ebenen sind heller und näher an der Farbe des Himmels. Brennpunkte liegen außerhalb der Mitte, damit die Mitte der Bühne, wo Claude und die Caption sind, ruhig bleibt. Silhouetten kommen aus weichem Rauschen statt aus wiederholten Kacheln, es gibt also bei keiner Breite eine Naht. Alles wird auf die Bühne beschnitten, und die Bewegung ist langsam und gehört zur Geschichte.

Der Rahmen der Leiste nimmt höchstens 131.072 Zeichen auf, deshalb werden wiederkehrende Dinge einmal gezeichnet und vielfach platziert: Die Tannen, die Grasbüschel und die fernste Baumlinie sind Vorlagen. Eine Szene, die dennoch darüber läge, wird schlanker neu gezeichnet, dann ohne Partikel, und als letztes Mittel auf der flachen Bühne.

</details>

## Captions

<p align="center"><img src="assets/captions.gif" alt="Claude zeigt auf eine Caption: npm test auf dates.ts: 42/42 bestanden, 0 Fehler" width="960"></p>

Die Geschichte wird in Worten erzählt. Props werden nicht in die Szene gezeichnet, damit nichts mit Claude und der Caption konkurriert.

Die Caption ist eine normale Sprechblase aus Cartoonpapier mit einem Schwänzchen, das auf Claude zeigt, gesetzt in [Monocraft](https://github.com/IdreesInc/Monocraft) von Idrees Hassan (SIL Open Font License, `fonts/Monocraft-OFL.txt`), eingebettet als 5-KB-Subset, damit sie überall gleich aussieht. In diesem Fork enthält das Subset zusätzlich ä, ö, ü, Ä, Ö, Ü und ß. Die Blase bleibt bei Claude und verdeckt ihn nie: Sie nimmt einen Platz direkt daneben, darüber oder (bei einem fliegenden Claude) darunter, geprüft gegen Claudes ganzen Weg, Sprünge und Schwanken eingeschlossen. Während Claude läuft, läuft die Blase im gleichen Abstand mit, sodass ihr Schwänzchen immer auf Claude zeigt. Über Claude kann sie an jeder Stelle seines Kopfes sitzen, und ein engerer Umbruch wird probiert, um einen Platz zu finden, den die Ränder der Bühne nie behindern. Ist ein Weg für jeden Platz zu lang, wartet die Blase unsichtbar, bis Claude weit genug drin ist, erscheint dann neben Claude und läuft mit ihm weiter. Diese Wartezeit wird zur Lesezeit der Szene addiert. Darin werden Arten von Wörtern abgesetzt, sodass eine Caption wie ein Terminal liest:

| Art | Beispiel | Sieht aus wie |
| --- | --- | --- |
| Code und Befehle (in Backticks oder ein bekannter Befehl) | `npm test` | türkis |
| Dateien und Pfade | dates.ts, src/auth | blau |
| Funktionen | daysInMonth() | violett |
| Zahlen und Zeiten | 312, 2.41s, 42/42 | orange, fett |
| Fehlschläge | failed, fehlgeschlagen, Fehler, TypeError, N+1 | rot, fett |
| Erfolge | passed, bestanden, erfolgreich, sauber, green | grün, fett |
| ASCII-Gesichter und Symbole | ^_^ >_< \o/ -> [OK] | warmer Akzent |

Der Narrator wählt außerdem einen Ton für den Moment. Ärger setzt der Caption ein rotes ✗ voran und ein Meilenstein ein grünes ✓; ein Celebrate ist ein Meilenstein, sofern nichts anderes dasteht. Der Narrator soll wie ein Entwickler schreiben, mit Backticks um Code und dem einen oder anderen ASCII-Gesicht.

<details>
<summary><b>Der Pixel-Art-Durchgang</b></summary>

Im Pixel-Art-Look (dem Standard) läuft die ganze Bühne, Szenerie und Claude zusammen, durch einen Pixelizer: ein SVG-Filter, der die Zeichnung in der Mitte jedes Zwei-Einheiten-Quadrats abtastet und jede Probe über ihr Quadrat verteilt, sodass alles wie Pixel Art liest, ohne dass etwas neu gezeichnet wird, und kein Weichzeichner die Farben auswäscht. Das Raster beginnt an der Ecke der Bühne, die Pixel liegen also deckungsgleich. Claude bekommt auf demselben Raster einen eigenen Sprite-Filter: sein Körper scharf abgetastet, seine Augen (feiner als ein Pixel) an ihrer Dunkelheit erkannt und gerade so verdickt, dass sie auf ganzen Pixeln landen, und eine dunkle Ein-Pixel-Kontur drumherum, wie eine Pixel-Art-Figur sie hätte. Die Caption liegt über allem, schon in Pixelschrift, und bleibt so scharf. Im Original-Look werden Szenen glatt gezeichnet, und die Szenerie bekommt stattdessen eine leichte Unschärfe, damit Claude und die Caption zuerst gelesen werden.

</details>

## Stile

<p align="center"><img src="assets/styles-grid.png" alt="Ein Standbild je Stil der elf Stile, jeweils mit seinem /fables-style-Befehl beschriftet" width="960"></p>

Zwei Looks zeichnen die gestalteten Szenen, wie sie sind: **Pixel Art** (`pixel`, der Standard) und das **Original** (`original`), dieselben Szenen glatt gezeichnet.

Neun weitere Stile aus der [Claude Mascot Style Gallery](https://github.com/henrik-thevibe/Claude-Mascot-Style-Gallery) sind eigene Kunstwerke. Jeder zeichnet jedes Element jeder Szene neu, Claude, die Caption-Blase, das Kapitel-Tag und den Rahmen, in seinem eigenen Medium, übersetzt aus der Originaltafel der Galerie. Sie sind rein kosmetisch: Die Geschichte, die Komposition der Szenerie und Claudes Weg bleiben, wie sie sind, und der Szene wird nichts hinzugefügt.

| Stil | `/fables style …` | Das Kunstwerk |
| --- | --- | --- |
| Höhlenmalerei | `cave` | Ocker, Ruß und heller Lehm, dünn auf fackelbeleuchteten Kalkstein gerieben; kein Himmel, nur die Wand; gebrochene Rußkonturen; Claude in rotem Ocker |
| Blaupause | `blueprint` | Weiße Linien auf einem Cyanotypie-Blatt; Schatten schraffiert wie im Schnitt, Luft und Licht als Phantomlinien; Claude als Patentzeichnung mit gestrichelten verdeckten Kanten |
| Mosaik | `mosaic` | In die Steine des Bodens gelegt und als Tesserae in Fugenmasse gesetzt, in Reihen dunkler Steine umrandet; eine Mäanderbordüre |
| Frutiger Aero | `aero` | Glänzende Verläufe mit weißem Rand auf jeder Fläche, nach den Hintergründen von Vista und 7: ein azurblauer Himmel mit Sonnenreflex, Bliss-grünes Gras, aquafarbene Glastürme, perlmuttfarbene Räume; Frutiger Aurora bei Nacht mit türkisen Wiesen, mondbeschienenen Wolken und Bokeh-Sternen; Claude als Mandarinen-Gelee |
| Kupferstich | `engraving` | Eine sepiafarbene Tinte auf Büttenpapier, jeder Ton als Schraffur entlang der Maserung dessen geschnitten, was er ist; eine Plattenkante |
| Millefleur-Wandteppich | `tapestry` | Gewebt in Krapp-, Waid-, Wau- und Walnuss-Wolle auf dem Raster des Webstuhls; Gras wird zum Feld der tausend Blumen |
| Golden-Age-Comic | `golden` | Flache Zeitungsdruck-Farben, Schatten in Ben-Day-Punkten, schwere Konturlinien; eine gesetzte Sprechblase |
| Ukiyo-e | `ukiyoe` | Flache Holzschnitt-Farben über einer Konturlinie, Bokashi-Himmel, Kasumi-Dunst, eine zinnoberrote Sonne; eine Kartusche und ein Siegel |
| Kamon | `kamon` | Cremefarbene Flächen auf schwarzer Seide, durch Schnitte gleicher Breite getrennt; Claude als Wappen; der zinnoberrote Hanko |

<details>
<summary><b>Wie ein Stil eine Szene neu zeichnet</b></summary>

Jedes Element einer Szene wird einem Maler unter einer *Rolle* übergeben, die sagt, was es ist und wie tief es steht (`hooks/art/roles.ts`): eine Tanne, eine Nebelbank, eine Mesa, der Kegel der Lampe, die nasse Straße. Die ursprünglichen Looks behalten die beleuchtete, fotografische Malerei. Der Maler eines Stils (`hooks/art/painter.ts`, `hooks/styles/*.ts`) liest diese Malerei auf Formen und Töne und zeichnet sie neu:
- Blüten, Materialien und Linse des beleuchteten Looks entfallen;
- jede beleuchtete Farbe wird zu einer der Tinten, Wollen, Steine oder Fäden des Stils, gewählt nach Familie und Tiefe des Elements;
- Formen nehmen die Linie des Stils an;
- was in der Kunstform keine Kante hat (Sonne und Mond, Nebel, Licht, Wasser), wird nach den eigenen Konventionen des Stils neu gezeichnet.

Claude wird vom Helden-Maler jedes Stils gezeichnet, aus den Flächen des 3D-Modells, den Umrisshüllen seiner Teile und seinen Kanten, die wie in der Engine der Galerie als Kontur, Falz oder verdeckt eingestuft werden. Ein Stil auf einem Raster (Mosaik, Wandteppich) nennt dieses Raster sein Medium: Die gezeichnete Bühne wird in Tesserae oder ein Gewebe gesetzt, und seine Fugen oder die Rippen des Gewebes werden darübergelegt. Der Narrator hört die Stimme jedes Stils, sodass eine Caption wie ein Comic von 1938 oder ein Holzschnitt klingen kann und trotzdem von der echten Arbeit handelt.

Die Stile zeichnen so schnell wie der Original-Look oder schneller, da sie die Materialfilter des beleuchteten Looks weglassen, und jeder passt bei jeder Breite in das Größenlimit der Leiste.

</details>

## Entwickeln

<p align="center"><img src="assets/develop.gif" alt="Claude gräbt neben einem ausbrechenden Vulkan: Durchsuche scripts/rehearse.ts" width="960"></p>

```sh
claude plugin validate .              # Manifest, Hooks und State-Vertrag
claude plugin test .                  # Unit-Tests plus Engine-Tests (gestubbtes Sonnet)
bun scripts/preview.ts > gallery.html # Beispielszenen als Seite im Browser rendern
bun scripts/preview.ts --look all > styles.html # jeder Look
```

`scripts/scenarios.ts` enthält sieben ganze Sitzungen: den Prompt, jeden Tool-Aufruf und sein Ergebnis, Claudes Worte und was Sonnet und was Haiku jedes Mal zurückschreiben, wenn der Narrator fragt, als Rohtext mitsamt Macken (Code-Fences, ein Wort Geplauder, eine fehlende Caption, eine leere Antwort). `scripts/rehearse.ts` spielt eine Sitzung durch die eigene Schleife des Narrators auf einer eigenen Uhr, sodass die Anfragen dann landen, wann das Plugin sie stellen würde, genau den Prompt sehen, den es senden würde, und schlechte Antworten genauso übersprungen und mit Backoff behandelt werden. Der Tab **Sessions** des Viewers zeigt das nebeneinander: Claude Codes Aktivität, die Anfragen des Narrators (mit jedem Prompt) und die Antworten des Modells (mit jeder Rohantwort und dem Urteil des Validators), die Leiste, und ein Feld, in das du eine eigene Antwort einfügen kannst, um zu sehen, was das Plugin daraus machen würde. Ein Test spielt jede Sitzung mit beiden Modellen durch. `bun scripts/preview.ts my-scenes.json` rendert deine eigenen Szenen, praktisch zum Feintuning von Sprites oder um auszuprobieren, was Sonnet zurückgeschickt hat.

Die Mod-API ist im Early Access und kann sich zwischen Claude-Code-Versionen ändern. Diese Mod wurde gegen Claude Code 2.1.287 gebaut. Hört etwas auf zu zeichnen, starte `claude --debug`: Die Logzeile nennt, was die Engine abgelehnt hat.

Die Bilder in dieser README zeichnet die Mod selbst: `bun scripts/readme-gifs.ts` rendert sie nach `assets/`. Das Launch-Video entsteht auf dieselbe Weise; siehe [`video/`](video/README.md).

## In der Cloud gebaut

<p align="center"><img src="assets/cloud.gif" alt="Claude fliegt über den Mond: Komplett in der Cloud gebaut. Bugs können lauern." width="960"></p>

Claude Fables wurde komplett in Claude-Code-Cloud-Umgebungen entwickelt: die Mod, ihre Tests, die Szenen und Stile, die Bilder dieser README und das Launch-Video. Es wurde auch lokal in der Desktop-App getestet, aber es kann trotzdem Fehler geben, und manche zeigen sich vielleicht nur bei deinem Setup. Sieht etwas komisch aus, starte `claude --debug` und [öffne ein Issue](https://github.com/henrik-thevibe/Claude-Fables/issues) mit dem, was das Log sagt.

## Danksagung

- Claudes 3D-Modell und seine Projektion sind von [ChetasLua](https://github.com/ChetasLua) aus der Engine der Galerie portiert, unter der MIT-Lizenz.
- Die neun Galerie-Stile stammen aus der [Claude Mascot Style Gallery](https://github.com/henrik-thevibe/Claude-Mascot-Style-Gallery).
- Captions sind in [Monocraft](https://github.com/IdreesInc/Monocraft) von Idrees Hassan gesetzt, unter der SIL Open Font License (`fonts/Monocraft-OFL.txt`).
