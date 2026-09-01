# ChessBall — Coding Rules & Standards

## 1. Language Requirements

### 1.1 Code Language: ENGLISH ONLY

**All code must be written in English:**
- Variable names
- Function names
- Class names
- Constants
- Type definitions
- Enum values
- Property names
- File names (except localization files)

❌ **INCORRECT:**
```javascript
const tahta = [];
const oyuncu = { isim: "Ali", skor: 0 };
function hamleYap(tas, hedef) { ... }
```

✅ **CORRECT:**
```javascript
const board = [];
const player = { name: "Ali", score: 0 };
function makeMove(piece, target) { ... }
```

### 1.2 Comments: ENGLISH ONLY

**All comments must be in English:**
- Inline comments
- Block comments
- Documentation comments
- JSDoc/TSDoc annotations
- TODO/FIXME notes

❌ **INCORRECT:**
```javascript
// Taşı seçilen kareye taşı
function movePiece(piece, square) {
  // Hamle geçerliliğini kontrol et
  if (!isValidMove(piece, square)) return;
  ...
}
```

✅ **CORRECT:**
```javascript
// Move piece to selected square
function movePiece(piece, square) {
  // Check move validity
  if (!isValidMove(piece, square)) return;
  ...
}
```

### 1.3 Comments: Keep Them Concise

**Guidelines:**
- Comments should explain **WHY**, not **WHAT**
- Avoid obvious statements
- Maximum 80 characters per line
- Use multi-line comments for longer explanations
- Remove obsolete comments immediately

❌ **TOO VERBOSE:**
```javascript
// This function calculates the legal moves for a given piece by first checking
// what type of piece it is, then iterating through all possible squares on the
// board, and for each square it validates whether that square is reachable based
// on the piece's movement pattern, and then it also checks if there are any
// pieces blocking the path to that square, and finally returns an array
function legalMoves(piece) { ... }
```

❌ **TOO OBVIOUS:**
```javascript
// Increment counter by 1
counter++;

// Return the result
return result;

// Loop through pieces
for (const piece of pieces) { ... }
```

✅ **CONCISE & MEANINGFUL:**
```javascript
// Offside: Forward pass illegal if no opponent ahead of receiver
if (isForwardPass && !hasOpponentAhead(receiver)) {
  return false;
}

// Press victim temporarily off-board to prevent click conflicts
victim.f = -1;

// AI noise prevents deterministic play patterns
score += Math.random() * 3;
```

### 1.4 User-Facing Text: USE LOCALIZATION

**All user-visible text must be localized:**
- UI labels and buttons
- Status messages
- Error messages
- Game notifications
- Tutorial text
- Achievement descriptions

❌ **HARDCODED TEXT:**
```javascript
button.textContent = "Yeni Maç";
status.textContent = "Senin sıran";
alert("Gol!");
```

✅ **LOCALIZED TEXT:**
```javascript
button.textContent = i18n.t("game.newMatch");
status.textContent = i18n.t("game.yourTurn");
showNotification(i18n.t("game.goalScored"));
```

---

## 2. Code Organization

### 2.1 File Structure

**Separate concerns clearly:**
```
src/
├── core/               # Game logic (platform-agnostic)
│   ├── state.js       # State management
│   ├── rules.js       # Move validation
│   ├── ai.js          # AI logic
│   └── constants.js   # Game constants
├── ui/                # User interface
│   ├── board.js       # Board rendering
│   ├── controls.js    # Input handling
│   └── animations.js  # Visual effects
├── locales/           # Translation files
│   ├── en.json
│   ├── tr.json
│   └── ...
└── utils/             # Shared utilities
    ├── math.js
    └── helpers.js
```

### 2.2 Module Pattern

**Export explicit public APIs:**
```javascript
// rules.js
export function legalMoves(state, pieceId) { ... }
export function isValidMove(state, move) { ... }

// Private helpers (not exported)
function pathBlocked(state, from, to) { ... }
```

---

## 3. Naming Conventions

### 3.1 Variables & Functions

**Use descriptive, meaningful names:**

| Type | Convention | Example |
|------|-----------|---------|
| Variables | camelCase | `ballPosition`, `currentPlayer` |
| Constants | UPPER_SNAKE_CASE | `MAX_TURNS`, `GOAL_SQUARES` |
| Functions | camelCase (verb) | `calculateMoves()`, `applyPress()` |
| Classes | PascalCase | `GameState`, `PieceController` |
| Private | _prefix | `_internalHelper()` |
| Boolean | is/has/can prefix | `isGoal`, `hasShield`, `canPress` |

**Avoid abbreviations unless universally known:**

❌ **UNCLEAR:**
```javascript
const pos = { x: 4, y: 6 };
function calcDst(a, b) { ... }
```

✅ **CLEAR:**
```javascript
const position = { file: 4, rank: 6 };
function calculateDistance(from, to) { ... }
```

**Exception:** Well-known abbreviations are acceptable:
- `id`, `url`, `html`, `json`, `xml`
- `min`, `max`, `avg`
- `i`, `j` (for short loop indices only)

### 3.2 File Names

**Use kebab-case for files:**
```
game-state.js
move-validator.js
ai-engine.js
piece-renderer.js
```

---

## 4. Function Design

### 4.1 Pure Functions Preferred

**Avoid side effects when possible:**

❌ **IMPURE (modifies input):**
```javascript
function movePiece(state, piece, target) {
  piece.f = target.f;
  piece.r = target.r;
  state.turn = state.turn === "W" ? "B" : "W";
  return state;
}
```

✅ **PURE (returns new state):**
```javascript
function movePiece(state, piece, target) {
  const newState = cloneState(state);
  const movedPiece = findPiece(newState, piece.id);
  movedPiece.f = target.f;
  movedPiece.r = target.r;
  newState.turn = state.turn === "W" ? "B" : "W";
  return newState;
}
```

### 4.2 Function Length

**Keep functions focused and short:**
- Maximum 50 lines per function (guideline, not strict)
- If longer, extract helper functions
- Single responsibility principle

### 4.3 Parameter Count

**Limit parameters:**
- Maximum 4 parameters preferred
- Use object parameters for complex data

❌ **TOO MANY PARAMETERS:**
```javascript
function createPiece(id, side, kind, file, rank, hasBall, isShielded) { ... }
```

✅ **OBJECT PARAMETER:**
```javascript
function createPiece({ id, side, kind, position, status }) { ... }
```

---

## 5. State Management

### 5.1 Immutable State

**Never mutate state directly:**

❌ **MUTATION:**
```javascript
state.score[0]++;
state.pieces[2].f = 5;
```

✅ **IMMUTABLE UPDATE:**
```javascript
const newState = {
  ...state,
  score: [state.score[0] + 1, state.score[1]]
};

const newPieces = state.pieces.map(p =>
  p.id === targetId ? { ...p, f: 5 } : p
);
```

### 5.2 State Structure

**Keep state flat and normalized:**

✅ **GOOD:**
```javascript
{
  pieces: [{ id: "WK", side: "W", f: 4, r: 2 }],
  ball: { f: 4, r: 6 },
  holder: "WF" // ID reference
}
```

❌ **BAD (nested, denormalized):**
```javascript
{
  white: {
    pieces: [
      { 
        id: "WK",
        position: { file: 4, rank: 2 },
        ball: { present: false }
      }
    ]
  }
}
```

---

## 6. Error Handling

### 6.1 Validate Inputs

**Check preconditions explicitly:**

```javascript
function legalMoves(state, pieceId) {
  if (!state) {
    throw new Error("State is required");
  }
  
  if (!pieceId) {
    throw new Error("Piece ID is required");
  }
  
  const piece = findPiece(state, pieceId);
  if (!piece) {
    console.warn(`Piece not found: ${pieceId}`);
    return [];
  }
  
  // Continue with valid inputs
  ...
}
```

### 6.2 Fail Gracefully

**Return safe defaults rather than crashing:**

```javascript
// Prefer empty array over null/undefined
function getAvailableMoves(piece) {
  if (!piece) return [];
  ...
}

// Provide fallback values
function getPieceName(kind) {
  return PIECE_NAMES[kind] || "Unknown";
}
```

### 6.3 Log Errors Appropriately

```javascript
// Development: verbose logging
if (DEV_MODE) {
  console.log("Move attempted:", move);
  console.warn("Invalid path:", path);
  console.error("State corruption detected:", state);
}

// Production: silent or user-friendly messages only
if (!isValid) {
  logError("Invalid move", { moveId, pieceId });
  showMessage(i18n.t("errors.invalidMove"));
}
```

---

## 7. Performance

### 7.1 Avoid Premature Optimization

**Write clear code first, optimize only when needed:**
- Profile before optimizing
- Document why optimization was necessary
- Keep original clear version in comments if significantly different

### 7.2 Common Optimizations

**Memoize expensive calculations:**
```javascript
const legalMovesCache = new Map();

function getLegalMoves(state, pieceId) {
  const key = `${state.ply}-${pieceId}`;
  
  if (legalMovesCache.has(key)) {
    return legalMovesCache.get(key);
  }
  
  const moves = calculateLegalMoves(state, pieceId);
  legalMovesCache.set(key, moves);
  return moves;
}
```

**Break early when possible:**
```javascript
// Stop checking once we find a blocker
function isPathClear(state, from, to) {
  const path = getPath(from, to);
  
  for (const square of path) {
    if (hasPiece(state, square)) {
      return false; // Early exit
    }
  }
  
  return true;
}
```

---

## 8. Testing

### 8.1 Test File Naming

```
src/core/rules.js → tests/core/rules.test.js
src/ui/board.js   → tests/ui/board.test.js
```

### 8.2 Test Structure

**Use descriptive test names:**

```javascript
describe("legalMoves", () => {
  describe("when piece is goalkeeper", () => {
    it("restricts movement to penalty area", () => {
      // Arrange
      const state = createTestState();
      const keeper = findPiece(state, "WK");
      
      // Act
      const moves = legalMoves(state, keeper.id);
      
      // Assert
      moves.forEach(move => {
        expect(isInPenaltyArea("W", move.f, move.r)).toBe(true);
      });
    });
  });
  
  describe("when piece has ball", () => {
    it("uses carry range instead of walk range", () => {
      const state = createStateWithBallCarrier("WF");
      const moves = legalMoves(state, "WF");
      
      expect(getMaxDistance(moves)).toBe(CARRY.F);
    });
  });
});
```

### 8.3 Test Coverage

**Aim for high coverage on core logic:**
- Critical paths: 100% coverage
- Game rules: 100% coverage
- AI logic: 80%+ coverage
- UI components: 60%+ coverage

---

## 9. Version Control

### 9.1 Commit Messages

**Use conventional commit format:**

```
type(scope): description

feat(ai): add Monte Carlo tree search
fix(rules): correct offside detection for diagonal passes
docs(readme): update installation instructions
refactor(state): extract retreat logic into separate function
test(press): add edge cases for press shielding
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Code style (formatting, no logic change)
- `refactor`: Code restructuring (no behavior change)
- `test`: Add or update tests
- `chore`: Maintenance tasks

### 9.2 Branch Naming

```
feature/multiplayer-support
fix/goal-detection-bug
refactor/state-management
docs/api-documentation
```

---

## 10. Localization System

### 10.1 Translation File Structure

**Use nested JSON for organization:**

```json
{
  "game": {
    "title": "ChessBall",
    "newMatch": "New Match",
    "yourTurn": "Your Turn",
    "opponentTurn": "Opponent's Turn"
  },
  "pieces": {
    "KAL": "Goalkeeper",
    "STP": "Stopper",
    "KNT": "Wing",
    "ON": "Playmaker",
    "FRV": "Forward"
  },
  "actions": {
    "walk": "Walk",
    "carry": "Carry",
    "pass": "Pass",
    "shoot": "Shoot",
    "press": "Press"
  },
  "messages": {
    "goal": "Goal!",
    "save": "Save by goalkeeper!",
    "ownGoal": "Own goal!",
    "victory": "You win!",
    "defeat": "Opponent wins!"
  },
  "errors": {
    "invalidMove": "Invalid move",
    "notYourTurn": "It's not your turn",
    "noPieceSelected": "Select a piece first"
  }
}
```

### 10.2 Using Translations

**Access via dot notation:**

```javascript
import i18n from './i18n';

// Simple translation
const title = i18n.t("game.title");

// With interpolation
const turnMsg = i18n.t("game.playerTurn", { player: playerName });

// Pluralization
const movesLeft = i18n.t("game.movesRemaining", { count: 3 });
```

### 10.3 Fallback Strategy

```javascript
// Default to English if translation missing
const translation = i18n.t(key, { 
  fallbackLng: "en",
  defaultValue: key // Show key itself if no translation exists
});
```

---

## 11. Documentation

### 11.1 Inline Documentation

**Document public APIs:**

```javascript
/**
 * Calculate all legal moves for a piece
 * 
 * @param {GameState} state - Current game state
 * @param {string} pieceId - ID of piece to move
 * @returns {Move[]} Array of legal moves
 * 
 * @example
 * const moves = legalMoves(state, "WF");
 * // => [{ type: "walk", f: 4, r: 6 }, ...]
 */
export function legalMoves(state, pieceId) {
  // Implementation
}
```

### 11.2 README Files

**Each major directory should have a README:**

```
src/core/README.md
src/ui/README.md
docs/game-rules.md
docs/architecture.md
```

### 11.3 Architecture Decisions

**Document significant technical choices:**

```markdown
# Architecture Decision Record: State Immutability

## Context
Game state needs to be reproducible for replay system and AI lookahead.

## Decision
Use immutable state updates throughout codebase.

## Consequences
- Positive: Easy state history tracking, replay system trivial
- Positive: AI can explore moves without side effects
- Negative: Slightly higher memory usage
- Negative: More verbose update code

## Alternatives Considered
- Mutable state with deep cloning on demand
- Event sourcing with command pattern
```

---

## 12. Code Review Checklist

Before submitting code, verify:

- [ ] All code and comments are in English
- [ ] User-facing text uses localization keys
- [ ] Comments are concise and meaningful
- [ ] Functions follow single responsibility principle
- [ ] No magic numbers (use named constants)
- [ ] State updates are immutable
- [ ] Input validation is present
- [ ] Error cases are handled gracefully
- [ ] Tests cover new functionality
- [ ] No console.log() in production code
- [ ] Performance is acceptable for critical paths
- [ ] Code passes linter (ESLint/TSLint)
- [ ] Documentation is updated if API changed

---

## 13. Continuous Integration

### 13.1 Pre-commit Hooks

```bash
# .husky/pre-commit
npm run lint
npm run test:unit
npm run type-check
```

### 13.2 CI Pipeline

```yaml
# .github/workflows/ci.yml
- Lint code
- Run unit tests
- Run integration tests
- Check test coverage (fail if < 80%)
- Build production bundle
- Run performance benchmarks
```

---

## 14. Anti-Patterns to Avoid

### ❌ Don't Do This

**1. Turkish in Code:**
```javascript
const tas = { tip: "KAL", konum: { x: 4, y: 2 } };
```

**2. Overly Long Comments:**
```javascript
// This function is called when the user clicks on a square and it will first
// check if there's a piece selected and then it will validate the move...
```

**3. Obvious Comments:**
```javascript
i++; // Increment i
```

**4. Hardcoded Strings:**
```javascript
alert("Gol!");
button.innerHTML = "Yeni Maç";
```

**5. Magic Numbers:**
```javascript
if (ply > 24) { ... }
if (score >= 2) { ... }
```

**6. Deep Nesting:**
```javascript
if (condition1) {
  if (condition2) {
    if (condition3) {
      if (condition4) {
        // Do something
      }
    }
  }
}
```

**7. Mutating State:**
```javascript
state.pieces[0].f = 5;
state.score[0]++;
```

---

## 15. Summary

**Core Principles:**
1. **English-only code and comments**
2. **Localize all user-facing text**
3. **Keep comments concise and meaningful**
4. **Write clear, maintainable code first**
5. **Test thoroughly**
6. **Document public APIs**
7. **Follow immutable state patterns**
8. **Handle errors gracefully**

**Questions to Ask:**
- Is this code readable by non-Turkish developers?
- Will this comment still be relevant in 6 months?
- Is this user-facing text localized?
- Does this function have side effects?
- Is this error handled gracefully?
- Is this performance-critical path optimized?

---

**Last Updated:** 2026-09-01  
**Version:** 1.0  
**Maintained By:** ChessBall Development Team
