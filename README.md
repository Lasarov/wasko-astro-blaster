# WASKO ASTRO BLASTER

Ein kleines Retro-Arcade-Spiel im Stil der späten 80er, überarbeitet von jemandem
mit Windows 98, Paint und mittelmäßigen Multimedia-Kenntnissen.

Wasko verteidigt seine Party gegen schlechten Käse, schlechte Musik, schlechte
Laune und schlechte Drinks — bewaffnet mit Weinkorken.

**Spielen:** einfach die Seite öffnen. Kein Login, keine Installation, keine App.

## Technik

Alles steckt in einer einzigen `index.html`: HTML, CSS, Canvas-Spiel, Pixel-Font,
Sound und Musik (WebAudio, komplett synthetisiert) sowie die Grafik als Base64.
Keine externen Bibliotheken, keine CDN-Abhängigkeiten, keine Schriftarten von außen.

- Interne Auflösung 480 × 800, per CSS responsiv skaliert (`image-rendering: pixelated`)
- Steuerung: Finger bewegen (mobil) oder Pfeiltasten (Desktop), Feuer läuft automatisch
- Ziel: 40 Gegner in 60 Sekunden
- Punkte: 100 je Gegner (150 mit Champagnerkorken), multipliziert mit der
  Kombo-Kette (×1 bis ×5, reißt wenn ein Gegner durchkommt oder man
  zusammenstößt), 250 je Power-up, 100 pro verbleibender Sekunde Zeitbonus
- Extra-Dinger fliegen durchs Bild: Gutes einsammeln (+400 bis +800),
  Mieses ausweichen (−350 bis −600)

Das Spiel selbst ist bewusst leicht, damit jeder gewinnt und die Einladung
sieht. Kombo und Extra-Dinger sorgen für den Unterschied in der Bestenliste,
ohne die Rundenlänge zu verändern — sie liegt weiter bei 35 bis 50 Sekunden.

## Bestenliste

Die Bestenliste läuft über [Supabase](https://supabase.com). Die beiden Werte stehen
oben im `<script>`-Block von `index.html`:

```js
const SUPABASE_URL = "";
const SUPABASE_KEY = "";
```

Solange sie leer sind, funktioniert das Spiel vollständig — die Bestenliste bleibt
dann nur lokal im jeweiligen Browser und wird nicht geteilt.

Gemeint ist der **Publishable key** (`sb_publishable_...`) aus *Settings → API Keys*.
Er ist ausdrücklich für den Einsatz im Browser gedacht und darf öffentlich sein.
Geschützt werden die Daten nicht durch den Key, sondern durch Row Level Security
und die `CHECK`-Regeln der Tabelle.

Ältere Projekte nutzen noch den `anon`-Key (ein JWT, beginnt mit `eyJ`). Auch der
funktioniert — das Spiel erkennt am Format selbst, welche Header es senden muss.

### Tabelle anlegen

```sql
create table public.scores (
  id         bigint generated always as identity primary key,
  name       text        not null check (char_length(name) between 2 and 14),
  score      int         not null check (score between 0 and 40000),
  kills      int         not null check (kills between 0 and 40),
  secs       numeric(4,1) not null check (secs between 0 and 60),
  won        boolean     not null default false,
  created_at timestamptz not null default now()
);

alter table public.scores enable row level security;

create policy "jeder darf lesen"    on public.scores for select to anon using (true);
create policy "jeder darf eintragen" on public.scores for insert to anon with check (true);

create index scores_rang on public.scores (score desc, secs asc);
```

Es zählt je Name nur der beste Lauf; die Auswahl passiert beim Anzeigen.
