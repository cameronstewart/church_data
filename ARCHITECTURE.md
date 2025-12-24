# Architecture Review: Church Data Platform

**Review Date**: 2025-12-23
**Status**: Architectural Design Phase
**Proposed Stack**: n8n + Python + Supabase + Frontend

---

## Executive Summary

**Recommendation**: ✅ **n8n + Python + Supabase** is an excellent architecture choice for this project.

**Why This Stack Works**:
- 🎯 **Low-code orchestration** (n8n) for RSS feeds, scheduling, and workflows
- 🐍 **Python modules** for complex NLP, transcription, and Bible analysis
- 🗄️ **Supabase** provides database, auth, storage, and real-time APIs
- ⚡ **Rapid development** with minimal infrastructure management
- 💰 **Cost-effective** for MVP and early scaling

---

## Proposed Architecture

### High-Level System Design

```
┌─────────────────────────────────────────────────────────────────┐
│                         Frontend Layer                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Next.js    │  │  React +     │  │   Mobile     │          │
│  │   Web App    │  │  Tailwind    │  │   (Future)   │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└────────────────────────────┬────────────────────────────────────┘
                             │ REST/GraphQL + Real-time subscriptions
┌────────────────────────────▼────────────────────────────────────┐
│                      Supabase Backend                           │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ PostgreSQL Database + PostGIS (for geo queries)          │  │
│  │ - Churches table (name, location, denomination, feeds)   │  │
│  │ - Sermons table (title, author, audio_url, transcript)   │  │
│  │ - Bible_references table (sermon_id, book, chapter, v)   │  │
│  │ - Themes table (sermon_id, theme, confidence)            │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Storage Buckets (for audio files, transcripts)           │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Auth (user accounts, admin roles)                         │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Edge Functions (serverless Python/Deno for API logic)    │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                             │ Webhooks & API calls
┌────────────────────────────▼────────────────────────────────────┐
│                    n8n Orchestration Layer                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Workflows:                                                │  │
│  │ - RSS Feed Ingestion (scheduled daily)                   │  │
│  │ - Audio Download & Storage                                │  │
│  │ - Trigger Transcription (→ Python module)                │  │
│  │ - Trigger Analysis (→ Python module)                      │  │
│  │ - Data Quality Checks                                     │  │
│  │ - Error Notifications (email/Slack)                      │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                             │ HTTP/REST calls
┌────────────────────────────▼────────────────────────────────────┐
│                   Python Processing Modules                     │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Transcription Service (Whisper/AssemblyAI)               │  │
│  │ - Input: audio_url                                        │  │
│  │ - Output: transcript text + timestamps                    │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Bible Reference Extractor (spaCy + regex)                │  │
│  │ - Input: transcript text                                  │  │
│  │ - Output: list of Bible references with context          │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Theme Classifier (BERT/GPT)                               │  │
│  │ - Input: transcript text                                  │  │
│  │ - Output: themes + confidence scores                      │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Component Deep Dive

### 1. n8n Orchestration ⭐⭐⭐⭐⭐

**Role**: Workflow automation and orchestration

**Why n8n?**
- ✅ **Visual workflow builder** - Easy to design and debug pipelines
- ✅ **300+ integrations** - RSS, HTTP, databases, cloud storage, APIs
- ✅ **Scheduling** - Cron jobs for daily RSS checks
- ✅ **Error handling** - Built-in retry logic and error workflows
- ✅ **Self-hosted or cloud** - Control over data and costs
- ✅ **Python support** - Execute Python code directly or call external services
- ✅ **Webhooks** - Trigger workflows from external events

**n8n Workflows for This Project**:

#### Workflow 1: RSS Feed Ingestion
```
[Cron: Daily 6am]
  ↓
[Read Churches from Supabase]
  ↓
[Loop: For each church]
  ↓
[HTTP: Fetch RSS feed]
  ↓
[Parse RSS (feedparser)]
  ↓
[Check for new sermons]
  ↓
[Insert new sermons to Supabase]
  ↓
[Trigger: Audio Download workflow]
```

#### Workflow 2: Audio Processing Pipeline
```
[Webhook: New sermon added]
  ↓
[HTTP: Download audio file]
  ↓
[Supabase Storage: Upload audio]
  ↓
[HTTP: Call Python Transcription API]
  ↓
[Wait for transcription (polling or webhook)]
  ↓
[Supabase: Update sermon with transcript]
  ↓
[Trigger: Analysis workflow]
```

#### Workflow 3: Content Analysis
```
[Webhook: Transcript ready]
  ↓
[HTTP: Call Python Bible Extractor API]
  ↓
[HTTP: Call Python Theme Classifier API]
  ↓
[Supabase: Insert bible_references]
  ↓
[Supabase: Insert themes]
  ↓
[Set sermon status: "analyzed"]
  ↓
[Notification: Send completion email]
```

**Alternatives Considered**:
| Tool | Pros | Cons | Verdict |
|------|------|------|---------|
| **n8n** | Visual, self-hosted, Python support | Learning curve | ✅ **RECOMMENDED** |
| **Apache Airflow** | Powerful, Python-native | Complex setup, overkill | ❌ Too heavy |
| **Prefect** | Modern, Python-first | Less visual, fewer integrations | ⚠️ Good alternative |
| **Zapier** | Easy, no-code | Expensive, limited control | ❌ Cost prohibitive |
| **Pure Python** | Full control | Must build everything | ❌ Slow development |

---

### 2. Supabase Backend ⭐⭐⭐⭐⭐

**Role**: Database, storage, authentication, and APIs

**Why Supabase?**
- ✅ **PostgreSQL** - Powerful relational database with full SQL support
- ✅ **PostGIS** - Geographic queries for church locations
- ✅ **Storage** - Built-in file storage for audio files
- ✅ **Auth** - User authentication and row-level security (RLS)
- ✅ **Auto-generated APIs** - Instant REST and GraphQL endpoints
- ✅ **Real-time subscriptions** - Live updates for frontend
- ✅ **Edge Functions** - Serverless functions for custom logic
- ✅ **Free tier** - Generous limits for MVP
- ✅ **Self-hostable** - Can migrate to own infrastructure later

**Database Schema**:

```sql
-- Churches table
CREATE TABLE churches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    denomination TEXT,
    website TEXT,
    rss_feed_url TEXT,
    podcast_url TEXT,
    location GEOGRAPHY(POINT), -- PostGIS for lat/long
    city TEXT,
    state TEXT,
    country TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Sermons table
CREATE TABLE sermons (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    church_id UUID REFERENCES churches(id),
    title TEXT NOT NULL,
    author TEXT,
    summary TEXT,
    sermon_date DATE,
    audio_url TEXT,
    audio_file_path TEXT, -- Supabase Storage path
    duration_seconds INTEGER,
    transcript TEXT,
    transcript_status TEXT CHECK (transcript_status IN ('pending', 'processing', 'completed', 'failed')),
    analysis_status TEXT CHECK (analysis_status IN ('pending', 'processing', 'completed', 'failed')),
    guid TEXT UNIQUE,
    published_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Bible references table
CREATE TABLE bible_references (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    sermon_id UUID REFERENCES sermons(id) ON DELETE CASCADE,
    book TEXT NOT NULL, -- e.g., "Genesis", "Matthew"
    chapter INTEGER NOT NULL,
    verse_start INTEGER,
    verse_end INTEGER,
    reference_text TEXT, -- e.g., "John 3:16-17"
    context TEXT, -- Surrounding text from sermon
    confidence NUMERIC(3,2), -- 0.00 to 1.00
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create index for fast lookups
CREATE INDEX idx_bible_ref_book_chapter ON bible_references(book, chapter);
CREATE INDEX idx_bible_ref_sermon ON bible_references(sermon_id);

-- Themes table
CREATE TABLE themes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    sermon_id UUID REFERENCES sermons(id) ON DELETE CASCADE,
    theme TEXT NOT NULL, -- e.g., "grace", "salvation", "prayer"
    confidence NUMERIC(3,2), -- 0.00 to 1.00
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Users table (for future admin/contributors)
CREATE TABLE users (
    id UUID PRIMARY KEY REFERENCES auth.users(id),
    email TEXT,
    role TEXT CHECK (role IN ('viewer', 'contributor', 'admin')),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create full-text search indices
CREATE INDEX idx_sermons_fulltext ON sermons USING GIN(to_tsvector('english', title || ' ' || COALESCE(summary, '') || ' ' || COALESCE(transcript, '')));
CREATE INDEX idx_churches_fulltext ON churches USING GIN(to_tsvector('english', name || ' ' || COALESCE(denomination, '')));
```

**Supabase Storage Buckets**:
- `sermon-audio` - Audio files (mp3, m4a)
- `transcripts` - Transcript files (txt, json)
- `exports` - User-generated exports

**Row-Level Security (RLS)**:
```sql
-- Allow public read access to churches and sermons
ALTER TABLE churches ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Public read access" ON churches FOR SELECT USING (true);

ALTER TABLE sermons ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Public read access" ON sermons FOR SELECT USING (analysis_status = 'completed');

-- Only admins can insert/update
CREATE POLICY "Admin write access" ON churches FOR ALL USING (
    auth.jwt() ->> 'role' = 'admin'
);
```

**Alternatives Considered**:
| Tool | Pros | Cons | Verdict |
|------|------|------|---------|
| **Supabase** | All-in-one, fast dev, generous free tier | Newer platform | ✅ **RECOMMENDED** |
| **Firebase** | Mature, Google-backed | NoSQL, vendor lock-in | ⚠️ Good but less flexible |
| **AWS (RDS+S3+Lambda)** | Powerful, scalable | Complex, expensive | ❌ Overkill for MVP |
| **Self-hosted Postgres** | Full control | Must manage infrastructure | ❌ Too much ops work |
| **MongoDB Atlas** | NoSQL flexibility | Complex queries harder | ❌ Relational better fit |

---

### 3. Python Processing Modules 🐍

**Role**: Heavy lifting for transcription, NLP, and analysis

**Deployment Options**:

#### Option A: Supabase Edge Functions (Recommended for MVP)
```python
# supabase/functions/transcribe-sermon/index.py
import whisper
from supabase import create_client

def handler(req):
    """Transcribe sermon audio using Whisper."""
    sermon_id = req.json()['sermon_id']
    audio_url = req.json()['audio_url']

    # Download audio
    # Run Whisper
    # Save transcript to Supabase

    return {'status': 'completed'}
```

**Pros**: Integrated, serverless, auto-scaling
**Cons**: Cold starts, execution time limits (2-10 min)

#### Option B: Standalone FastAPI Service (Recommended for Production)
```python
# main.py
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

@app.post("/transcribe")
async def transcribe_sermon(sermon_id: str, audio_url: str):
    """Transcribe sermon and save to Supabase."""
    # Long-running transcription
    return {"status": "processing"}

@app.post("/extract-bible-references")
async def extract_references(sermon_id: str, transcript: str):
    """Extract Bible references from transcript."""
    # NLP processing
    return {"references": [...]}

@app.post("/classify-themes")
async def classify_themes(sermon_id: str, transcript: str):
    """Classify sermon themes."""
    # ML classification
    return {"themes": [...]}
```

**Deployment**: Fly.io, Railway, or DigitalOcean App Platform
**Pros**: No time limits, persistent models in memory
**Cons**: Must manage service

**Python Module Structure**:
```
python-services/
├── transcription_service/
│   ├── main.py
│   ├── models/
│   │   └── whisper_model.py
│   └── requirements.txt
├── analysis_service/
│   ├── main.py
│   ├── extractors/
│   │   ├── bible_reference.py
│   │   └── theme_classifier.py
│   └── requirements.txt
└── shared/
    ├── supabase_client.py
    └── utils.py
```

---

### 4. Frontend Options 🎨

**Recommended**: **Next.js 14+ with App Router**

**Why Next.js + Supabase?**
- ✅ **Server-side rendering** - SEO for church search
- ✅ **Supabase Auth** - Built-in integration
- ✅ **Real-time updates** - Subscribe to new sermons
- ✅ **API routes** - Backend logic when needed
- ✅ **Tailwind CSS** - Rapid UI development
- ✅ **Vercel deployment** - Free hosting, global CDN

**Key Pages/Features**:

```
Frontend Structure:
├── app/
│   ├── page.tsx                    # Homepage (search, featured sermons)
│   ├── churches/
│   │   ├── page.tsx                # Church directory (map + list)
│   │   └── [id]/page.tsx           # Church detail + sermons
│   ├── sermons/
│   │   ├── page.tsx                # Sermon search/filter
│   │   └── [id]/page.tsx           # Sermon detail (audio + transcript)
│   ├── bible/
│   │   └── [book]/[chapter]/page.tsx  # "What's preached on Romans 8?"
│   ├── themes/
│   │   └── [theme]/page.tsx        # Sermons by theme
│   ├── visualizations/
│   │   ├── map.tsx                 # Geographic heat map
│   │   ├── trends.tsx              # Theme trends over time
│   │   └── coverage.tsx            # Bible book coverage
│   └── admin/
│       ├── churches.tsx            # Add/edit churches
│       └── dashboard.tsx           # Processing stats
├── components/
│   ├── SearchBar.tsx
│   ├── ChurchCard.tsx
│   ├── SermonPlayer.tsx
│   ├── BibleReferenceList.tsx
│   ├── ThemeCloud.tsx
│   └── Map.tsx (Mapbox/Leaflet)
└── lib/
    └── supabase.ts                 # Supabase client
```

**UI/UX Mockup Concepts**:

1. **Homepage**: Search bar + map of churches + recent sermons
2. **Church Page**: Details + sermon list + "What they preach about" (theme cloud)
3. **Sermon Page**: Audio player + transcript (with timestamps) + Bible references
4. **Bible Explorer**: "Show me all sermons on John 3:16" → list with context
5. **Visualization Dashboard**: Charts and maps

**Frontend Stack**:
```json
{
  "dependencies": {
    "next": "^14.0.0",
    "@supabase/supabase-js": "^2.x",
    "@supabase/auth-helpers-nextjs": "^0.x",
    "react": "^18.x",
    "tailwindcss": "^3.x",
    "shadcn/ui": "latest",  // Beautiful components
    "mapbox-gl": "^3.x",    // Maps
    "recharts": "^2.x",     // Charts
    "lucide-react": "latest" // Icons
  }
}
```

**Alternatives**:
| Framework | Pros | Cons | Verdict |
|-----------|------|------|---------|
| **Next.js** | SSR, best Supabase support | React only | ✅ **RECOMMENDED** |
| **SvelteKit** | Faster, simpler | Smaller ecosystem | ⚠️ Good alternative |
| **Astro** | Ultra-fast, any framework | Less dynamic | ❌ Too static |
| **React SPA** | Simple | No SSR (bad SEO) | ❌ Need SEO |

---

## Data Flow Example

**End-to-End: RSS Feed → Visualization**

```
1. n8n Cron Job (6am daily)
   ↓
2. n8n fetches RSS feeds from Supabase churches table
   ↓
3. n8n parses new sermons → Inserts to Supabase sermons table
   ↓
4. Supabase trigger → Webhook to n8n "audio_download" workflow
   ↓
5. n8n downloads audio → Uploads to Supabase Storage
   ↓
6. n8n calls Python FastAPI: POST /transcribe
   ↓
7. Python runs Whisper → Saves transcript to Supabase
   ↓
8. Supabase trigger → Webhook to n8n "analysis" workflow
   ↓
9. n8n calls Python FastAPI: POST /extract-bible-references
   ↓
10. Python runs spaCy + regex → Saves to bible_references table
    ↓
11. n8n calls Python FastAPI: POST /classify-themes
    ↓
12. Python runs BERT → Saves to themes table
    ↓
13. Frontend queries Supabase → Displays in real-time
    ↓
14. User searches "John 3:16" → Instant results
```

---

## Cost Analysis (MVP Phase)

### Free Tier Coverage:
- **Supabase Free**: 500 MB database, 1 GB storage, 2 GB bandwidth/month
- **n8n Cloud Free**: 20 workflows, 5k executions/month (or self-host for free)
- **Vercel Free**: Unlimited deployments, 100 GB bandwidth/month
- **Python Service**: Fly.io free tier (3 shared CPU VMs) or Railway $5/month

### Estimated Monthly Costs (100 churches, 1000 sermons/month):
```
Supabase Pro:       $25/month (more storage + bandwidth)
n8n Cloud:          $20/month (or $0 if self-hosted on VPS)
Transcription:      $240/month (AssemblyAI: $0.40/hr × 10 hrs/week × 4 weeks)
                    or $0 (self-hosted Whisper on GPU instance)
Python Hosting:     $10/month (Railway/Fly.io)
Frontend (Vercel):  $0 (free tier sufficient)
─────────────────────────────────────
Total:              $295/month (with cloud transcription)
                    or $55/month (with self-hosted Whisper)
```

**Cost Optimization**:
- Start with Whisper small model (free, self-hosted)
- Use Supabase free tier initially (sufficient for MVP)
- Self-host n8n on cheap VPS ($5/month DigitalOcean)
- Scale up only when needed

---

## Security & Privacy

1. **Data Privacy**:
   - Only ingest public sermons
   - Store minimal PII
   - GDPR-compliant data handling

2. **Supabase RLS**:
   - Public read for completed sermons
   - Admin-only write access
   - Rate limiting on APIs

3. **API Security**:
   - Python services require API keys
   - n8n workflows use webhook secrets
   - Supabase enforces JWT auth

4. **Audio Storage**:
   - Signed URLs with expiration
   - CDN caching for public sermons

---

## Scalability Plan

### Phase 1: MVP (100 churches, 10k sermons)
- Supabase free tier
- Single Python service instance
- n8n (5k workflow executions/month)

### Phase 2: Growth (1000 churches, 100k sermons)
- Supabase Pro ($25/month)
- Scale Python services horizontally
- CDN for audio files (Cloudflare)
- Caching layer (Redis)

### Phase 3: Scale (10k+ churches, 1M+ sermons)
- Dedicated PostgreSQL (RDS/managed)
- Kubernetes for Python services
- Distributed transcription queue
- Multi-region deployment

---

## Alternative Architecture (If NOT Using n8n)

**Pure Python Stack**:
```
FastAPI (main app)
  ├── Celery (task queue for async processing)
  ├── Redis (task broker + cache)
  ├── PostgreSQL (Supabase or self-hosted)
  └── React/Next.js frontend
```

**Pros**: Full control, Python-native
**Cons**: More code to write, must handle scheduling/retries/monitoring

**Verdict**: n8n is better for this project because:
- Visual workflows easier to maintain
- Less code to write and debug
- Built-in error handling and retries
- Non-developers can modify workflows

---

## Final Recommendation

### ✅ Recommended Architecture:

```
Frontend:      Next.js 14 + Tailwind + shadcn/ui
Backend:       Supabase (Postgres + Storage + Auth + APIs)
Orchestration: n8n (self-hosted or cloud)
Processing:    Python FastAPI services (transcription, analysis)
Deployment:    Vercel (frontend), Fly.io (Python), DigitalOcean (n8n)
```

### Why This Stack?
- ⚡ **Fast MVP development** (weeks, not months)
- 💰 **Low initial cost** ($0-50/month for MVP)
- 📈 **Scales to 1M+ sermons** with architecture adjustments
- 🛠️ **Easy maintenance** with visual workflows
- 🔧 **Flexible** - Can swap components as needed
- 🚀 **Modern stack** with strong community support

### Implementation Timeline:
- **Week 1-2**: Supabase setup + database schema
- **Week 3-4**: n8n RSS ingestion workflows
- **Week 5-6**: Python transcription service (Whisper)
- **Week 7-8**: Python analysis services (Bible refs + themes)
- **Week 9-12**: Frontend (Next.js) development
- **Week 13-14**: Testing + deployment
- **Week 15-16**: Beta launch + user feedback

**Total MVP Timeline**: ~4 months with 1-2 developers

---

## Questions & Concerns?

1. **Is n8n stable enough?** Yes, mature product (5+ years), used by enterprises
2. **Can we migrate later?** Yes, all components are modular and replaceable
3. **What about vendor lock-in?** Supabase is open-source, can self-host
4. **Performance concerns?** This stack handles millions of records efficiently
5. **Alternative to n8n?** Prefect or pure Python Celery (more code, less visual)

**Next Steps**: See USER_REQUIREMENTS.md for detailed user stories and acceptance criteria.
