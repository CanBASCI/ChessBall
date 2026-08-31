# Şah Saha / ChessBall — Cursor el kitabı

Bu belge başka bir Cursor oturumunda oyunu sıfırdan anlamadan devam ettirmek içindir. Kaynak gerçeklik: çalışan prototip `index.html` / `SahSaha.html`. Bu spec ile kod çelişirse **kodu spec’e çek**, spekülatif kural ekleme.

## 1. Oyun nedir

İsim: **Şah Saha** (repo: ChessBall).

Satranç + futbol hibrit. Üstten bakış, tur tabanlı, 1v1. Amaç rakip kale karesine topu sokmak (yürüyerek veya şutla).

Hedef platform (niyet): iOS, Swift, SpriteKit üstten 2D, Game Center turn-based `GKTurnBasedMatch`, rekabetçi ranked.
Şu anki teslim: tek dosya HTML/JS prototip, insan (beyaz) vs basit PC (siyah).

Repo: https://github.com/CanBASCI/ChessBall
Sahip: CanBASCI
Branch: `main`

## 2. Tahta

- 9 dosya × 11 sıra.
- Dosyalar `A–I` → kodda `f = 0..8`.
- Sıralar `1–11` → kodda `r = 1..11`.
- Beyaz altta (küçük `r`), siyah üstte. Beyaz hücum yönü `r` artışı (kaleye 11). Siyah hücum yönü `r` azalışı (kaleye 1).
- Orta çizgi görsel: `r === 6`.
- Kale kareleri: beyaz `D1 E1 F1` `(3–5,1)`; siyah `D11 E11 F11` `(3–5,11)`.
- Ceza sahası: beyaz `C–G` × `1–3`; siyah `C–G` × `9–11`.
- `isGoal(f,r)`: `(r===1 || r===11) && f>=3 && f<=5`

## 3. Taşlar

Her taraf 5 taş. `K` KAL şah/kaleci, `S` STP at/stoper, `N` KNT kale/kanat, `O` ON fil/on numara, `F` FRV vezir/forvet.

Açılış: W KAL E2, STP C3 G3, ON D4, FRV E5. B KAL E10, STP D9 F9, KNT B8, FRV E8. Top E6. Beyaz başlar. Gizli setup henüz yok.

## 4. Döngü

Tek aksiyon / tur: yürü, götür, pas, şut, pres. İlk 2 gol. `kickPlies=2` açılış ve gol sonrası: şut yok, topla kale karesine giriş yok. 24 tur sonra skor / top sahibi.

## 5. Menziller

```
        yürü         götür        pas           şut
KAL     1*           1*           2 her         1 her
STP     at           at           2 düz         2 düz
KNT     4 düz        2 düz        4 düz         3 düz
ON      3 çap        1 çap        3 çap         3 çap
FRV     2 her        2 her        3 her         4 her
* yalnız kendi ceza sahası
```

Yürüme yolunu her taş keser. Pas/şut çizgisini yalnız rakip keser.
Pas yalnız kendi taşına. İleri pasta ofsayt var (hedefin önünde rakip saha oyuncusu yoksa yasak).
Şut rakip kale karelerine. Hedefte kaleci varsa kurtarış. Boşsa gol.
Topla kale karesine yürümek de gol.

## 6. Pres

Topu tutan rakibe yürüme deseniyle bas. Kurban kendi WALK karesine kaçar (taş seçilmez, kareye basılır). Kaçış kurbanın turudur; sıra presleyene döner. `pressShieldId` 1 tur pres yemez. `staggerId` o tur pres atamaz. Kurban geçici `f=-1` (tıklama çakışmasın).

## 7. Kod

Kanonik prototip: `index.html` (ve `SahSaha.html`).
Fonksiyonlar: `startState`, `legalMoves`, `applyMove`, `applyRetreat`, `scoreGoal`, `endTurnExtras`, `aiPick`.
State: pieces, ball, holder, turn, ply, score, kickPlies, passStreak, lastPair, pressShieldId, staggerId, clearFlagsOnSide, pendingRetreatId, pendingFrom, winner, log.
Tıklama önceliği: shot > press > pass > walk/carry.

## 8. Yok / sonraki

iOS SpriteKit, Game Center `GKTurnBasedMatch`, gizli diziliş, güçlü AI, 3D yok.
PC AI 1-ply heuristic.

Cursor kuralı: SPEC’e aykırı kural ekleme. Kod çelişirse spec’e çek.
