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

## Utilitar procesare date

Scripturile de procesare se află în `utils/`:

- `extract_pdf.mjs` — extrage text din broșurile PDF (folosește pdfjs-dist)
- `parse_brosura.py` — parsează textul extras în `brosura.json`
- `enrich_stats.py` — îmbogățește `stats.json` cu date din broșuri (locuri oficiale, coduri)
