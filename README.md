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

## Utilitar procesare date

Scripturile de procesare se află în `utils/`:

- `extract_pdf.mjs` — extrage text din broșurile PDF (folosește pdfjs-dist)
- `parse_brosura.py` — parsează textul extras în `brosura.json`
- `enrich_stats.py` — îmbogățește `stats.json` cu date din broșuri (locuri oficiale, coduri)
