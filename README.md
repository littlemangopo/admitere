# Admitere Liceu București

Site neoficial cu statistici de admitere la liceu în București, construit cu AI (Claude). Poate conține erori — verificați întotdeauna cu datele oficiale.

## Surse oficiale

| Resursă | URL |
|---|---|
| ISMB Admitere | http://ismb.edu.ro/index.php/admitere |
| Date oficiale 2026 | https://static.admitere.edu.ro/2026/ |
| Date oficiale 2025 | https://static.admitere.edu.ro/2025/ |
| Date oficiale 2024 | https://static.admitere.edu.ro/2024/ |
| Broșură admitere 2026 | http://www.ismb.edu.ro/documente/examene/admitere/2026_2027/BROSURA_ADMITERE_2026_1.pdf |
| Repartizare 2025 (JSON) | https://static.admitere.edu.ro/2025/repartizare/B/data/candidate.json |
| Repartizare 2024 (JSON) | https://static.admitere.edu.ro/2024/repartizare/B/data/candidate.json |
| Ierarhie 2025 (JSON) | https://static.admitere.edu.ro/2025/ierarhie/B/data/candidate.json |
| Ierarhie 2024 (JSON) | https://static.admitere.edu.ro/2024/ierarhie/B/data/candidate.json |

## Date locale

| Fișier | Conținut |
|---|---|
| `stats.json` | Date agregate: licee, medii, distribuții, locuri oficiale 2025–2026 |
| `students.json` | Candidați repartizați (2024–2025), anonimi, ordonați după medie |
| `2025/brosura.json` | Date extrase din broșura oficială 2025 |
| `2026/brosura.json` | Date extrase din broșura oficială 2026 |
| `2025/repartizareB.json` | Date brute repartizare 2025, candidați din București |
| `2024/repartizareB.json` | Date brute repartizare 2024, candidați din București |

## Rulare locală

```bash
python3 -m http.server 8765
# deschide http://localhost:8765
```

## Discrepanțe cunoscute între broșură și date agregate

Totalul locurilor din broșură (`officialTotalPlaces`) vs suma locurilor per specializare (`sum(officialPlaces)`) nu se potrivesc la câteva licee. Cauze identificate:

### Licee tehnice/vocaționale (~35 școli)
**Așteptat.** Totalul din broșură include clase vocaționale (Electric, Mecanică, Comerț etc.) pe care aplicația nu le urmărește. Discrepanța = locuri vocaționale neextrase.

### Sub-numărare la licee teoretice

| Liceu | An | Oficial | Sumă | Diff | Cauză |
|---|---|---|---|---|---|
| Liceul Teoretic „Alexandru Ioan Cuza" | 2025 | 234 | 182 | −52 | Științe Sociale neextrase din broșură |
| Colegiul Național „Aurel Vlaicu" | 2025 | 156 | 104 | −52 | Științe ale Naturii neextrase din broșură |
| Liceul Teoretic „Dante Alighieri" | 2025 | 182 | 156 | −26 | O specializare neextrasă (probabil Științe Sociale) |
| Liceul Teoretic „Dante Alighieri" | 2026 | 196 | 168 | −28 | Idem 2026 |
| Liceul Teoretic „Alexandru Vlahuță" | 2025 | 78 | 65 | −13 | Clasă de 0.5 Filologie bilingv DE neextrasă |
| Liceul Teoretic „Alexandru Vlahuță" | 2026 | 84 | 70 | −14 | Idem 2026 |

**Rezolvate:** Colegiul Național „Școala Centrală" — clasele duble bilingv FR (cu/fără probă de verificare) sunt acum sumate corect (56 locuri în loc de 28 per specializare bilingv). Colegiul Național „Iulia Hașdeu" — potrivire greșită limbă bilingv EN/ES rezolvată.

### Supra-numărare la licee cu limbi minoritare

| Liceu | An | Oficial | Sumă | Diff | Cauză |
|---|---|---|---|---|---|
| Liceul Teoretic „Decebal" | 2026 | 196 | 280 | +84 | Potrivire greșită specs în enrich_stats |
| Liceul Teoretic Bulgar „Hristo Botev" | 2025 | 104 | 130 | +26 | Filologie romani (rromani) numărată în plus |
| Liceul Teoretic Bulgar „Hristo Botev" | 2026 | 112 | 140 | +28 | Idem 2026 |

Aceste discrepanțe afectează doar afișarea numărului de locuri în detaliul liceelor respective — mediile de admitere și statisticile generale sunt corecte.

## Algoritmul de alocare simulată 2026

**Script:** `utils/simulate_repartizare2026.py` · Reproducibil cu `random.seed(42)`

### Date de intrare

| Fișier | Rol |
|---|---|
| `2026/ierarhie_EN.json` | Ierarhia oficială EN 2026 (note înainte de contestații), ordonată după mev = (română + matematică) / 2 |
| `2025/repartizareB.json` | Repartizarea reală 2025, în ordinea oficială a sistemului (tiebreaker intern păstrat) |
| `stats.json` | Locuri disponibile 2026 per liceu/specializare (din broșura oficială) |

### Pasul 1 — Tabel de referință 2025

Se sortează `repartizareB.json` descrescător după `madm`. Python `sorted()` stabil păstrează ordinea oficială din JSON pentru ex æquo — rezultând exact aceleași prime 10 poziții ca în clasamentul real 2025. Se salvează `2025/reference_rank_school_spec.json`: **rang → liceu + specializare aleși**.

### Pasul 2 — Injectare preferințe pentru clase noi în 2026

Clasele nou înființate (ex: Ştiinţe Naturii la Tudor Vianu) nu au corespondent în 2025. Algoritmul le injectează sintetic: ultimele `N` poziții (N = locuri noi) din grupul de candidați care au ales aceeași specializare la școli de nivel similar sunt înlocuite cu noua clasă.

### Pasul 3 — Simulare EST1 (note înainte de contestații)

Se parcurge ierarhia 2026 candidat cu candidat:

1. Se preia liceul+specializarea de la același rang din tabelul de referință 2025
2. Dacă există locuri → candidatul este alocat
3. Dacă liceul e plin → se caută **următorul liceu disponibil cu aceeași specializare**, ordonat după media ultimului admis 2025 (Lazăr plin → Sava → Haret → Vianu → ...)
4. **Cutoff EST1** = mev al ultimului candidat admis

### Pasul 4 — Simulare EST2 (după contestații)

Identic cu EST1, dar **10% din candidați** (aleator, `seed=42`) au nota mărită aleator ∈ [nota_curentă, 10]. Lista re-sortată înainte de alocare → cutoffuri EST2 ușor mai ridicate.

### Limitări

- Ierarhia 2026 folosește mev (fără nota școlii); referința 2025 este sortată după madm (cu nota școlii) — aceeași poziție de rang nu corespunde exact aceluiași profil
- ~5.300 candidați nealocați: ierarhia 2026 (~15.000) depășește referința 2025 (~13.000)
- Preferințele sunt presupuse invariante față de 2025; clasele noi pot schimba comportamentul în realitate

## Utilitar procesare date

Scripturile de procesare se află în `utils/`:

- `extract_pdf.mjs` — extrage text din broșurile PDF (folosește pdfjs-dist)
- `parse_brosura.py` — parsează textul extras în `brosura.json`
- `enrich_stats.py` — îmbogățește `stats.json` cu date din broșuri (locuri oficiale, coduri)
- `simulate_repartizare2026.py` — simulează repartizarea 2026 (EST1 + EST2), reproducibil cu `random.seed(42)`
- `scrape_ierarhie2026.py` — descarcă ierarhia oficială EN 2026 de pe evaluare.edu.ro
- `scrape_evaluare2026.py` — descarcă notele EN 2026 per școală de proveniență
