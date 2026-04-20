# دماغ (Demagh) — Egyptian Arabic Wordle

A full-stack Arabic word puzzle game inspired by Wordle, built with a focus on **RTL language processing**, **low-latency gameplay**, and **production-grade backend architecture**.

---

## 🎮 What is Demagh?

Demagh (دماغ - Arabic for "brain") is a daily word puzzle game designed for Egyptian Arabic speakers. Players get 6 attempts to guess a 5-letter Arabic word, with real-time feedback on correct letters and positions. The core engineering challenge: Arabic text is non-trivial — characters change shape based on position, Unicode normalization is required, and RTL rendering demands a purpose-built UI architecture.

---

##  Architecture

```
Client (React)
     │
     ▼
API Gateway / Load Balancer
     │
     ▼
Stateless Node.js Services
     │
     ├──▶ Redis (Cache Layer)
     │       └── Word of the day (O(1) retrieval)
     │       └── Validation cache
     │
     └──▶ PostgreSQL (Source of Truth)
             └── Word dictionary
             └── Game sessions
             └── User metrics
```

- **Read-heavy workload** optimized via Redis caching
- **Write operations** isolated to user metrics only
- **Stateless API layer** — no session dependency, supports horizontal scaling behind a load balancer

---

##  Tech Stack

| Layer | Technology | Why |
|---|---|---|
| Frontend | React + Tailwind CSS | RTL-aware component architecture, dynamic Arabic keyboard |
| Backend | Node.js + Express | Stateless, horizontally scalable API layer |
| Database | PostgreSQL | Normalized schema, indexed queries for fast word lookup |
| Cache | Redis | Cached daily word + validation data, reduced DB load |
| Deployment | Vercel (frontend) + AWS/Render (backend) | Cloud-native, scalable deployment |

---

##  Core Engineering Highlights

### Custom Arabic Text-Matching Engine
Arabic presents unique NLP challenges that required a purpose-built solution:
- **Unicode normalization** — handles character variants (e.g., أ / إ / ا treated correctly)
- **Position-aware matching** — correct letter, wrong position vs. correct position
- **Duplicate letter handling** — non-trivial edge cases matched to Wordle's official rules
- **RTL-safe string processing** — avoids common left-to-right library assumptions

### Low-Latency Gameplay
- Daily word cached in Redis on schedule → avoided repeated PostgreSQL hits
- Near **O(1) validation per request** under normal load
- Deterministic, idempotent scoring algorithm — same input always returns same output

### Stateless Backend Design
- No server-side session storage
- Any node can handle any request → clean horizontal scaling
- Designed to sit behind a load balancer without sticky sessions

### Database Design
- Normalized schema: `words`, `game_sessions`, `user_metrics` tables
- Indexed word lookup and validation queries
- Write batching strategy for user metrics to reduce DB pressure under load

---

##  API Design

### Game Engine Service
```
POST /api/guess
Body: { guess: "كلمة", session_id: "..." }
Returns: { result: ["correct", "present", "absent", "absent", "correct"] }
```
Stateless validation — no session stored server-side.

### Word Service
```
GET /api/word/today
Returns: cached daily word metadata (not the word itself — revealed only on win/loss)
```
Daily word selected via cron job, precomputed and cached in Redis for O(1) retrieval.

### User Metrics Service
```
POST /api/metrics
Body: { session_id, guesses, won, time_taken }
```
Tracks streaks, win rates, and guess distributions. Writes batched to reduce DB load.

---

##  Project Structure

```
demagh/
├── client/
│   ├── components/
│   │   ├── Board.jsx          # Game board with RTL rendering
│   │   ├── ArabicKeyboard.jsx # Custom Arabic keyboard layout
│   │   └── Tile.jsx           # Letter tile with state animations
│   └── utils/
│       └── rtlHelpers.js      # RTL string utilities
├── server/
│   ├── routes/
│   │   ├── game.js            # Guess validation endpoint
│   │   ├── word.js            # Daily word service
│   │   └── metrics.js         # User stats tracking
│   ├── services/
│   │   ├── arabicMatcher.js   # Core text-matching engine
│   │   └── cacheService.js    # Redis abstraction layer
│   └── db/
│       └── schema.sql         # PostgreSQL schema
└── README.md
```

---

##  Running Locally

```bash
# Clone the repo
git clone https://github.com/ahmed-manialawy/demagh.git
cd demagh

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
# Fill in: DATABASE_URL, REDIS_URL

# Run database migrations
npm run db:migrate

# Start development server
npm run dev
```

---

##  Why This Project Matters

Arabic is spoken by 400M+ people yet remains severely underrepresented in consumer tech and word games. Demagh was built to be linguistically correct, not just a font swap — handling the full complexity of Arabic script in an interactive, real-time context.

---

##  Author

**Ahmed El-Manialawy** — Senior Data Engineer  
[LinkedIn](https://linkedin.com/in/el-manialawy-ms-mba)
