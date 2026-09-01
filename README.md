# ChessBall

Chess + football hybrid 1v1 prototype. Play in browser: [index.html](./index.html)

## Documentation

- 📋 **[Project Specification](./PROJECT_SPEC.md)** — Complete game rules, architecture, and roadmap
- 📝 **[Coding Rules](./CODING_RULES.md)** — Development standards and best practices

## How to Play
- You are white, opponent is AI
- Click piece, then click highlighted square
- Pass: Only to your pieces, within range; opponent pieces intercept
- Shot: Within shot range, click red goal square if path is clear
- Goal: Shoot OR carry ball into goal square (`D/E/F` × 1 or 11)
- Press: Walk onto opponent ball carrier. Loser retreats; retreat counts as their turn, then presser's side continues

## Pieces
| Piece | Walk | Carry | Pass | Shot |
|---|---|---|---|---|
| KAL (GK) | 1, penalty area | 1 | 2 any | 1 any |
| STP (DF) | knight | knight | 2 straight | 2 straight |
| KNT (WG) | 4 straight | 2 straight | 4 straight | 3 straight |
| ON (AM) | 3 diagonal | 1 diagonal | 3 diagonal | 3 diagonal |
| FRV (FW) | 2 any | 2 any | 3 any | 4 any |

Repo: https://github.com/CanBASCI/ChessBall
