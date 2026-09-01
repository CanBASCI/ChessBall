# ChessBall — Project Specification

## 1. Project Overview

**Name:** ChessBall (Turkish: Şah Saha)  
**Genre:** Turn-based strategy game  
**Concept:** Hybrid of chess and football mechanics  
**Repository:** https://github.com/CanBASCI/ChessBall  
**Owner:** CanBASCI  
**Current Status:** HTML/JS prototype (single-file implementation)

### Vision
A competitive 1v1 turn-based game combining strategic chess-like piece movement with football objective mechanics. The goal is to score by getting the ball into the opponent's goal area, either by carrying or shooting.

### Target Platform
- **Intended:** iOS native app using Swift + SpriteKit (top-down 2D view)
- **Multiplayer:** Game Center turn-based matches (`GKTurnBasedMatch`)
- **Gameplay Mode:** Competitive ranked matches
- **Current:** Single-file HTML/JS prototype (human vs basic AI)

---

## 2. Game Rules

### 2.1 Board Configuration
- **Dimensions:** 9 files × 11 ranks
- **Files:** A-I (internally indexed 0-8)
- **Ranks:** 1-11
- **Orientation:**
  - White plays from bottom (lower ranks), attacks upward (toward rank 11)
  - Black plays from top (higher ranks), attacks downward (toward rank 1)
- **Midline:** Rank 6 (visual reference)
- **Goal Squares:**
  - White goal: D1, E1, F1 (files 3-5, rank 1)
  - Black goal: D11, E11, F11 (files 3-5, rank 11)
- **Penalty Areas:**
  - White: Files C-G (2-6) × Ranks 1-3
  - Black: Files C-G (2-6) × Ranks 9-11

### 2.2 Pieces

Each side has 5 pieces with distinct movement patterns and abilities:

| Code | Full Name | Chess Origin | Football Role | Count |
|------|-----------|--------------|---------------|-------|
| K | KAL | King | Goalkeeper | 1 |
| S | STP | Knight | Stopper/Defender | 2 |
| N | KNT | Rook | Wing/Fullback | 1-2 |
| O | ON | Bishop | #10/Playmaker | 1 |
| F | FRV | Queen | Forward/Striker | 1-2 |

**Starting Formation:**

White:
- KAL (K): E2
- STP (S): C3, G3
- ON (O): D4
- FRV (F): E5

Black:
- KAL (K): E10
- STP (S): D9, F9
- KNT (N): B8
- FRV (F): E8

**Ball:** Starts at E6  
**First Turn:** White always starts

### 2.3 Movement Ranges

Each piece has four distinct range values for different actions:

| Piece | Walk | Carry | Pass | Shot |
|-------|------|-------|------|------|
| KAL | 1* | 1* | 2 any | 1 any |
| STP | knight | knight | 2 straight | 2 straight |
| KNT | 4 straight | 2 straight | 4 straight | 3 straight |
| ON | 3 diagonal | 1 diagonal | 3 diagonal | 3 diagonal |
| FRV | 2 any | 2 any | 3 any | 4 any |

**Notes:**
- `*` KAL can only move/carry within own penalty area
- "any" = orthogonal or diagonal
- "straight" = orthogonal only (no diagonals)
- "diagonal" = diagonal only (no orthogonal)
- "knight" = L-shaped knight moves (2+1 or 1+2)

### 2.4 Action Types

**One action per turn.** Available actions:

#### Walk
- Move piece without ball using Walk range
- Any piece can collide with ball to pick it up
- Walking path blocked by any piece
- Goal squares: only accessible to ball carrier when `kickPlies = 0`

#### Carry
- Move piece with ball using Carry range
- Ball moves with piece
- Walking path blocked by any piece
- Entering goal square with ball = **GOAL**

#### Pass
- Transfer ball to teammate using Pass range
- Only available to ball carrier
- Can only pass to own pieces
- Pass path blocked only by opponent pieces (not teammates)
- **Offside rule:** Forward pass illegal if no opponent field player ahead of receiver
- **Pass repetition limit:** Same two pieces cannot pass back-and-forth more than 4 consecutive times

#### Shot
- Shoot ball toward opponent goal using Shot range
- Only available to ball carrier
- Target: One of three opponent goal squares (D/E/F × 1 or 11)
- Shot path must be clear of opponent pieces
- If goalkeeper occupies target square: **SAVE** (goalkeeper gains possession)
- If goal square empty: **GOAL**
- **Shot restriction:** Disabled for first 2 moves (`kickPlies = 2`) after kickoff/goal

#### Press
- Tackle opponent ball carrier
- Move to opponent ball carrier's square using Walk pattern
- Presser takes possession of ball
- Victim must **retreat** to any legal Walk square
  - Victim chooses retreat destination (this counts as victim's turn)
  - If no legal retreat exists, victim placed adjacent to original position
  - Victim temporarily moved to off-board position (f=-1) during retreat selection to prevent click conflicts
- After retreat completes, turn returns to presser's side
- **Press shield:** Piece that successfully pressed cannot be pressed back for 1 turn
- **Stagger:** Victim of press cannot press on their retreat turn

### 2.5 Path Blocking Rules

| Action Type | Blocked By |
|-------------|------------|
| Walk / Carry | Any piece (own or opponent) |
| Pass | Opponent pieces only |
| Shot | Opponent pieces only |

### 2.6 Goalkeeper Special Rules
- KAL restricted to own penalty area at all times
- Can save shots if positioned in targeted goal square
- After save, KAL gains possession

### 2.7 Goal Squares
- Only ball carrier can enter (when `kickPlies = 0`)
- Entering = immediate goal
- KAL cannot enter any goal square (even own)
- Non-carrier pieces cannot enter goal squares

### 2.8 Scoring

**Goal conditions:**
1. Carry ball into opponent goal square (D/E/F × 1 or 11)
2. Shoot ball into empty opponent goal square

**Goal outcomes:**
- Score increments for attacking side
- Ball returns to center (E6)
- All pieces reset to starting positions
- `kickPlies` reset to 2
- Turn switches to defending side (or attacker if own goal)

**Own goal:** If piece scores on own goal, opponent gets point

**Win conditions:**
- First to 2 goals wins immediately
- After 24 plies (12 turns each): Higher score wins
- Tie after 24 plies: Winner = side with ball possession (or ball location if neutral)

### 2.9 Turn Structure

- **Ply counter:** Increments when White completes turn
- **Turn order:** White → Black → White → ...
- **Action resolution:**
  1. Player selects piece
  2. Legal moves highlighted (color-coded by type)
  3. Player selects destination
  4. Move executes
  5. If press occurred: Victim retreats, turn returns to presser
  6. Otherwise: Turn switches to opponent

### 2.10 Kickoff Restriction
- First 2 moves after kickoff or goal: `kickPlies = 2`
- During kickoff:
  - No shots allowed
  - Cannot carry ball into goal squares
- Decrements by 1 each ply until reaches 0

---

## 3. Technical Architecture

### 3.1 Current Implementation (Prototype)

**File:** `index.html`  
**Technology:** Pure HTML + CSS + Vanilla JavaScript  
**Single-file architecture:** All code embedded in HTML

#### Key Components

**State Management:**
```javascript
{
  pieces: [],           // Array of piece objects {id, side, kind, f, r}
  ball: {f, r},        // Ball position
  holder: id|null,     // ID of piece carrying ball
  turn: "W"|"B",       // Current turn
  ply: number,         // Move counter (increments on White's turn)
  score: [W, B],       // Score array
  kickPlies: number,   // Countdown restricting shots
  passStreak: number,  // Consecutive passes between same pair
  lastPair: [id, id],  // Last two pieces that passed
  pressShieldId: id,   // Piece immune to press this turn
  staggerId: id,       // Piece that cannot press this turn
  clearFlagsOnSide: "W"|"B",  // Side that clears shields when turn ends
  pendingRetreatId: id,// Victim awaiting retreat selection
  pendingFrom: {f, r}, // Original position of retreat victim
  winner: "W"|"B"|null,
  log: string          // Last action description
}
```

**Core Functions:**

| Function | Purpose |
|----------|---------|
| `startState()` | Initialize game state with starting positions |
| `legalMoves(state, pieceId)` | Calculate all legal moves for a piece |
| `applyMove(state, move)` | Execute move and return new state |
| `applyRetreat(state, dest)` | Resolve press victim retreat |
| `scoreGoal(state, piece, f, r)` | Process goal scoring |
| `endTurnExtras(state, afterGoal)` | Handle turn transitions and flags |
| `aiPick(state)` | AI decision-making (1-ply heuristic) |

**Move Priority (when multiple actions target same square):**
1. Shot (red highlight)
2. Press (orange highlight)
3. Pass (blue highlight)
4. Walk/Carry (gold highlight)

### 3.2 AI Implementation

**Current:** Simple 1-ply evaluation heuristic

**Evaluation factors:**
- Score differential (×200 weight)
- Ball carrier proximity to opponent goal
- Goal scoring opportunities (+400 bonus)
- Press opportunities (+18 bonus)
- Shot opportunities (+80 bonus)
- Random noise (0-3) for variety

**Future:** Advanced AI with multi-ply lookahead, Monte Carlo tree search, or neural network

### 3.3 UI/UX

**Visual Design:**
- Dark theme with grass-like field colors
- Color-coded legal move indicators
- Piece identification via 3-letter codes
- Ball represented as golden circle
- Goal squares highlighted in distinct color

**Interaction:**
1. Click piece to select (shows legal moves)
2. Click highlighted square to execute action
3. During retreat: Click square to retreat to

**Mobile considerations:**
- Responsive grid layout
- Touch-optimized targets
- No-zoom viewport configuration

---

## 4. Future Development Roadmap

### 4.1 iOS Native App

**Technology Stack:**
- Swift
- SpriteKit (2D top-down view)
- Game Center integration

**Features:**
- Turn-based multiplayer via `GKTurnBasedMatch`
- Ranked competitive play
- Player profiles and statistics
- Match history
- Push notifications for turn alerts

### 4.2 Gameplay Enhancements

**Hidden Setup Phase:**
- Pre-game piece placement customization
- Formation strategies
- Counter-play tactics

**Advanced AI:**
- Multiple difficulty levels
- Opening strategy library
- Endgame pattern recognition
- Training mode with hints

**Game Modes:**
- Quick match (current rules)
- Extended match (higher goal limit)
- Practice mode vs AI
- Puzzle challenges

### 4.3 Additional Features

**Replay System:**
- Save and review past matches
- Share notable games
- Tutorial playback

**Customization:**
- Piece skin variants
- Board themes
- Sound effects and music

**Social Features:**
- Friend challenges
- Leaderboards
- Achievements
- Clans/teams

---

## 5. Design Principles

### 5.1 Strategic Depth
- No randomness in core mechanics (except AI move selection)
- Multiple viable strategies
- Risk/reward decision points
- Counter-play opportunities

### 5.2 Accessibility
- Easy to learn, hard to master
- Clear visual feedback
- Intuitive controls
- Quick match duration (5-15 minutes)

### 5.3 Balance
- Both sides start equal
- No dominant strategy
- Press mechanic provides comeback opportunities
- Offside rule prevents camping

### 5.4 Performance
- Fast state calculations
- Smooth animations
- Minimal network latency (async turn-based)
- Low battery consumption

---

## 6. Testing Requirements

### 6.1 Unit Tests
- Move legality validation
- Path blocking logic
- Offside detection
- Press/retreat mechanics
- Goal detection
- State transitions

### 6.2 Integration Tests
- Full game flow (kickoff to win)
- AI decision consistency
- Press chain resolution
- Pass repetition prevention
- Boundary conditions (24-ply limit, tie scenarios)

### 6.3 Manual Testing
- UI responsiveness
- Visual clarity
- Touch target accuracy
- Error state handling
- Network disconnection (iOS)

---

## 7. Known Limitations (Current Prototype)

1. **No hidden setup phase** — Fixed starting positions
2. **Basic AI** — 1-ply heuristic only
3. **No multiplayer** — Local human vs AI only
4. **No persistence** — State lost on page refresh
5. **No replay/undo** — Cannot review previous moves
6. **No sound/animations** — Static visual feedback
7. **Single language** — Turkish UI text hardcoded

---

## 8. Success Metrics

### 8.1 Gameplay Metrics
- Average match duration
- Move diversity (no repetitive patterns)
- Comeback rate (team behind at midgame wins)
- Goal distribution (walk-in vs shot)

### 8.2 Technical Metrics
- State calculation time < 50ms
- UI response time < 100ms
- Zero game-breaking bugs
- 99%+ network reliability (iOS)

### 8.3 User Engagement (iOS)
- Daily active users
- Match completion rate
- Average matches per user
- Retention rate (D1, D7, D30)

---

## 9. References

- Game Repository: https://github.com/CanBASCI/ChessBall
- Current Prototype: `index.html`
- Rules Specification: `SAH_SAHA_SPEC.md` (Turkish)
- README: `README.md` (Turkish)

---

**Last Updated:** 2026-09-01  
**Version:** 1.0  
**Status:** Prototype Phase
