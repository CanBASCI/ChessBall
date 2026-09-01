# ChessBall — Şah Saha

Satranç + futbol 1v1 prototip. Tarayıcıda oyna: [index.html](./index.html)

## Dokümantasyon

- 📋 **[Project Specification](./PROJECT_SPEC.md)** — Complete game rules, architecture, and roadmap
- 📝 **[Coding Rules](./CODING_RULES.md)** — Development standards and best practices

## Nasıl
- Sen beyaz, rakip PC
- Taşa bas, parlayan kareye bas
- Pas: yalnız kendi oyuncuna, taşın yönü + menzili; aradaki rakip keser
- Şut: yön + şut menzili, çizgide rakip yoksa kırmızı kale karesine bas
- Gol: şut VEYA topu kale karesine yürüterek (`D/E/F` × 1 veya 11)
- Pres: topu tutan rakibe yürü. Kaybeden kendi hareketiyle kaçar; kaçış onun hamlesidir, sıra presleyene döner

## Taşlar
| Figür | Yürü | Götür | Pas | Şut |
|---|---|---|---|---|
| KAL | 1, ceza sahası | 1 | 2 her | 1 her |
| STP | at | at | 2 düz | 2 düz |
| KNT | 4 düz | 2 düz | 4 düz | 3 düz |
| ON | 3 çapraz | 1 çapraz | 3 çapraz | 3 çapraz |
| FRV | 2 her | 2 her | 3 her | 4 her |

Repo: https://github.com/CanBASCI/ChessBall
