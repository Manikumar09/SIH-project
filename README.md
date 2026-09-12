# SIH-project
creating a "National Digital Platform for Research, Policy Innovation, and Evidence-Based Land Governance" 


# Land Governance Knowledge Platform: Build Prompt

## Context
You are building a national land governance research and policy evidence platform for SIH 2025. The core insight: land policy in India is being made without access to existing research, case studies, and data. Your job is to build a web platform that makes this evidence discoverable and actionable.

**Non-negotiable constraint:** This is a full-stack web dev team with 5-6 people and ~4 months. Do not fake the simulation or build a pretend ML model. Instead, make the citation-grounded search and synthesis the star, and keep the simulation honest and transparent.

---

## Core User Journeys (Build These First)

### Journey 1: Researcher Asks a Question, Gets a Sourced Answer
**Actor:** Academic researcher or policy analyst  
**Entry:** Homepage search bar  
**Interaction:**
```
User types: "What evidence exists on gender-inclusive land titling in India?"
↓
System returns: 
  - A synthesized 1-paragraph answer written by Claude API
  - Each sentence is hyperlinked to the exact document and paragraph it cites
  - Buttons below: "View all 23 matching documents" | "Export as bibliography"
↓
User clicks citation → document viewer opens with that passage highlighted
```

**Why this wins:** Every other retrieval system returns documents. You return answers. And every claim is traceable—no hallucinations, no "probably" claims without evidence.

---

### Journey 2: Official Explores a Dispute Hotspot, Then Models a Proposal
**Actor:** State revenue official or policy officer  
**Entry:** Dashboard tab  
**Interaction:**
```
User navigates to "Dispute Analytics"
↓
Map shows districts color-coded by dispute density (real data from LARR portal)
↓
User clicks Bijnor district (high red) → sees:
  - 847 total disputes in past 3 years
  - Top 3 dispute categories: inheritance, trespass, succession
  - Timeline chart of disputes filed per quarter
↓
User thinks: "What if we fast-tracked inheritance claims?"
↓
Clicks "Run Simulation" → opens sidebar with sliders:
  - Reduce inheritance dispute resolution time: 2 years → 6 months (slider)
  - Expected outcome: 160 fewer disputes per year (based on regression model)
  - Confidence: Medium (model trained on 5 similar states)
↓
User can export this scenario as a PDF for the cabinet brief
```

**Why this works:** You're not claiming to predict the future. You're showing: "if this lever moves, this indicator probably moves like this, because here's the historical pattern in similar places."

---

### Journey 3: Researcher Builds a Literature Map
**Actor:** PhD student or policy researcher  
**Entry:** Collaborative workspace  
**Interaction:**
```
User opens "My Research Workspace"
↓
Saves search result: "women's land rights"
↓
Creates a canvas with three sections:
  - Key documents (pinned papers with 1-line summaries)
  - Key debates (e.g., "titling vs. registration" with opposing viewpoints)
  - Key data sources (links to downloadable CSVs and APIs)
↓
Invites a colleague to view/annotate
↓
Both can highlight passages, add notes, and export the whole workspace as a report
```

**Why this matters:** Researchers don't just want documents; they want to synthesize them. Give them that tool and they'll evangelize you.

---

## Phased Implementation (Weeks 1–18)

### Phase 1: Repository & Auth (Weeks 1–4)
**What to build:**
- PostgreSQL schema for documents (title, author, date, type, text, metadata)
- Admin panel to bulk-upload PDFs (ingest from data.gov.in, NITI Aayog, DILRMP reports)
- Role-based access control:
  - **Public:** can search, read documents, see dashboards
  - **Researcher:** can also save searches, create workspaces, annotate
  - **Official:** can also run simulations, download restricted datasets
  - **Admin:** can manage documents and user roles
- User registration with email verification
- Document viewer (display PDF text, highlight passages)

**Tech:**
- Next.js pages router for auth (NextAuth.js) and file upload
- PostgreSQL for docs + metadata
- S3 or local filesystem for PDF storage
- Tailwind for UI

**Acceptance criteria:**
- Upload 50+ real land governance documents (DILRMP handbooks, case studies, legal codes)
- Log in as three roles, verify access differences
- Search by title/author/date works
- Read a full document with highlights

---

### Phase 2: Hybrid Search (Weeks 4–6)
**What to build:**
- BM25 keyword search over document text (use PostgreSQL full-text search or Elasticsearch)
- Vector embeddings for semantic search:
  - Break documents into 500-word chunks
  - Call Claude API to get embeddings for each chunk
  - Store embeddings in `pgvector` column in PostgreSQL
  - On search, embed the user query and find nearest-neighbor chunks (cosine distance)
- Search results page shows both BM25 (exact keyword) and semantic (meaning-based) matches, ranked by relevance

**Tech:**
- pgvector extension for PostgreSQL (store + query embeddings)
- Claude API for embeddings (or Anthropic's embedding model if available)
- Frontend: search bar that queries both methods, displays ranked results

**Acceptance criteria:**
- User searches "women inherit ancestral land" → gets documents on inheritance law, gender rights, and succession, even if those exact words don't appear in the title
- Search is fast (<2 sec for 10k documents)
- Results are sortable by relevance, date, document type

---

### Phase 3: Citation-Grounded Synthesis (Weeks 6–8)
**What to build:**
- RAG (Retrieval-Augmented Generation) pipeline:
  1. User enters a natural-language question
  2. Retrieve top 5–10 relevant document chunks (from Phase 2 semantic search)
  3. Pass those chunks + the question to Claude API with this system prompt:

```
You are a policy research assistant analyzing land governance documents.

Given the user's question and the document excerpts below, write a clear 1-paragraph answer.

CRITICAL: For every claim you make, you MUST cite the exact document and excerpt.
Use this format: [Document: "Title", Page X, Paragraph Y: "quote"]

Do NOT make claims without evidence from the documents.
If the documents don't answer the question fully, say so explicitly.

User Question: {question}

Documents:
{documents_with_citations}

Answer:
```

  4. Parse the Claude response and convert citations into clickable links
  5. Display the answer with hyperlinked citations; user can click any citation to jump to that document

- Optional: show the search query used to fetch documents ("Showing results for: gender land inheritance India")

**Tech:**
- Next.js API route that calls Claude API
- Frontend: question input, spinner while Claude responds, formatted answer display with citation links
- Backend: store each synthesis query + response in DB for analytics

**Acceptance criteria:**
- User asks "What is the status of digital land records in rural India?" → gets a sourced paragraph with 3–5 citations to specific documents
- Every claim is linked; click any link and the document opens with that passage highlighted
- System explicitly says "I don't have enough information" if the corpus doesn't cover the question

---

### Phase 4: GIS & Dashboards (Weeks 8–11)
**What to build:**

**Part A: Dispute Hotspot Map**
- Fetch real data from LARR portal (land acquisition, disputes by district)
- GeoJSON of India districts with dispute counts
- Leaflet.js map colored by dispute density (choropleth)
- Click a district → show a detail panel with:
  - Total disputes (last 3 years)
  - Dispute types breakdown (pie chart)
  - Timeline (line chart, disputes per quarter)
  - Top 3 documents mentioning this district's land issues
  
**Part B: Land-Use Layer** (optional, only if time permits)
- Fetch open land-use data from Bhuvan (ISRO) or state geospatial portal
- Overlay as a separate map layer (toggle on/off)
- Show forest, agriculture, urban, water categories

**Part C: Analytics Dashboard**
- Cards showing:
  - Total documents in corpus
  - Total disputes (indexed)
  - States with active digitization (from DILRMP)
  - Most-cited research areas (tag cloud)
- Charts:
  - Documents added per month (line chart)
  - Document type distribution (bar chart)
  - Top 10 most-downloaded documents (bar chart)

**Tech:**
- Leaflet.js + Leaflet Draw for maps
- Recharts or Chart.js for dashboards
- PostgreSQL with PostGIS for spatial queries
- Fetch LARR data via API (larr.dolr.gov.in) or CSV export

**Acceptance criteria:**
- Map renders India districts and shows dispute counts
- Click a district, see detailed statistics
- Dashboard shows real numbers (not fake data)
- Filters work (e.g., "disputes in last 1 year" or "only inheritance disputes")

---

### Phase 5: Policy Simulation (Weeks 11–14)
**What to build:**

**Simulation Engine (Backend):**
- Define 3–4 policy levers you can model:
  1. **Dispute resolution speed:** Reduce average time to close a dispute (e.g., 2 years → 6 months)
  2. **Digitization rate:** Increase % of districts with fully digitized records
  3. **Women's titling:** % of titles issued to women (increase from 20% to 50%)
  4. **Grievance redressal:** Reduce time from filing to first hearing

- For each lever, define a regression model:
  ```
  Projected disputes = Base disputes × (1 - lever_impact × lever_change)
  
  Example:
  If dispute resolution speed improves by 40% (2yr → 14mo):
    lever_impact = 0.6 (60% of disputes are sensitive to speed)
    lever_change = 0.4
    new_disputes = 1000 × (1 - 0.6 × 0.4) = 760 disputes
  ```
  
  These coefficients come from:
  - Academic papers in your corpus
  - DILRMP impact assessments
  - State-level case studies
  - When you run the simulation, cite which paper justified each coefficient

- Output: a JSON object with projected indicators + confidence intervals

**Frontend:**
- Sidebar with sliders for each lever
- In real-time, show projected outcomes:
  ```
  Current: 1,200 disputes/year
  Projected: 920 disputes/year (↓ 23%)
  Based on: 5 similar states, avg. effect size 0.58
  ```
- Button to "Compare scenarios" (save multiple slider combos and show them side-by-side)
- Export as PDF with the assumptions cited

**Key principle:** The simulation is not predicting the future. It's showing: "Historical data from similar contexts suggests this change correlates with this outcome, but your actual results will depend on implementation quality and local factors."

**Tech:**
- Next.js API route that runs the simulation logic (no ML, just formulas)
- Store simulation runs in DB (for analytics: which levers do officials care about?)
- Frontend: React slider component (react-slider or similar)

**Acceptance criteria:**
- User adjusts one slider → output updates immediately
- Simulation output includes the source papers that justify the coefficients
- User can export scenario as PDF with assumptions listed
- Run the same scenario twice, get the same result (deterministic, no randomness)

---

### Phase 6: Collaborative Workspace (Weeks 14–16)
**What to build:**
- User workspace (dashboard):
  - Saved searches (bookmarks)
  - Collections (pinned documents, organized by topic)
  - Shared workspaces (invite collaborators via email)
  
- Collection features:
  - Drag-and-drop to organize documents
  - Add 1-line summary for each doc
  - Pin the most important ones
  - Add tags ("key finding", "case study", "counterargument")
  - Generate workspace summary as a report (markdown + citations)

- Collaboration:
  - Invite others with role (read-only, edit, admin)
  - See who's currently viewing
  - Basic real-time sync (if two people both editing, use last-write-wins; no fancy CRDT needed for an MVP)

**Tech:**
- Next.js pages for workspace UI
- PostgreSQL for workspaces + membership + document collections
- WebSocket for real-time notifications (optional; can use polling)

**Acceptance criteria:**
- Researcher 1 creates a workspace, invites Researcher 2
- Researcher 2 sees it and can add documents
- Both can export the collection as a markdown report
- Workspace persists (reload page, it's still there)

---

### Phase 7: Public-Facing & Polish (Weeks 16–18)
**What to build:**
- Landing page:
  - Hero: "Access India's land governance research in one place"
  - Demo video (30 sec: search → get cited answer → run simulation)
  - Feature overview
  - Call-to-action: "Search the corpus" or "Sign up"
  
- Innovation portal (if time):
  - List of active grants and pilot projects (seeded with real data from NITI Aayog / state govt sites)
  - Filter by state, topic, funding amount
  - Link to each project's external page

- Hardening:
  - SQL injection protection (use parameterized queries everywhere)
  - Rate-limiting on search and API endpoints
  - Delete old sessions / auth tokens
  - Backup database
  - HTTPS only

- Demo script (for the 10-minute presentation):
  1. Show homepage (5 sec)
  2. Search for "female land ownership" → get synthesized answer with citations (30 sec)
  3. Click a citation → document viewer pops up (10 sec)
  4. Close and navigate to dashboard (5 sec)
  5. Show dispute hotspot map (15 sec)
  6. Run a simulation (reduce dispute resolution time) → show outcomes (20 sec)
  7. Export scenario as PDF (5 sec)
  8. Show a saved workspace (10 sec)
  
  **Total: 100 seconds, 1 complete story arc**

---

## Narrow Scope Rules (What to Cut First)

If you're running out of time:

1. **Cut:** Innovation portal (grants listing)  
   **Keep:** Repository + search + synthesis + GIS + simulation
   
2. **Cut:** Real-time collaborative editing  
   **Keep:** Workspaces and saved collections (async)
   
3. **Cut:** PostGIS land-use overlays  
   **Keep:** Just the dispute hotspot map
   
4. **Cut:** Multiple simulation scenarios with side-by-side comparison  
   **Keep:** Single scenario runner with one set of sliders
   
5. **Cut:** Advanced analytics (PDF trend analysis, ML-driven insights)  
   **Keep:** Basic dashboards (card + chart)

---

## Data Sources to Seed the Corpus

**Easy wins (all public, downloadable):**
- DILRMP handbooks: https://dolr.gov.in/
- NITI Aayog land governance reports: https://niti.gov.in/
- World Bank LGAF studies on India: https://www.worldbank.org/
- Census of India land statistics: https://censusindia.gov.in/
- LARR portal data: https://larr.dolr.gov.in/ (disputes, acquisitions by district)
- Bhuvan geospatial data: https://bhuvan.nrsc.gov.in/ (land use, satellite imagery)
- State revenue department websites (UP, Karnataka, Telangana all publish land circulars + case studies)
- Academic papers from SSRN or ResearchGate tagged "land governance India"

**Target: 100+ documents in the corpus by Week 3. Real content >> polished UI.**

---

## Tech Stack Summary

| Layer | Tech | Why |
|-------|------|-----|
| Frontend | Next.js + React | SSR, API routes, fast dev |
| Auth | NextAuth.js | Simple RBAC, integrates with Next |
| Database | PostgreSQL + pgvector | ACID, GIS-capable (PostGIS), vector search native |
| Search | pgvector + Claude embeddings | Semantic search without external DB |
| Maps | Leaflet.js | Lightweight, easy choropleth maps |
| Charts | Recharts | React-friendly, clean UI |
| AI | Claude API | Embeddings + synthesis, citation-grounded |
| Hosting | Vercel (frontend) + Railway/Render (backend) | Free tier sufficient for demo |

---

## Definition of Done: Winning Demo

The judges see:

1. **Real corpus:** 100+ actual land governance documents, not placeholders
2. **Working search:** Type a policy question, get a synthesized answer citing specific passages
3. **Working map:** Click a district, see real dispute data with a chart
4. **Working simulation:** Slide a lever, see projected indicator change with justification
5. **Working auth:** Log in as different roles, see different features
6. **Honest scope:** The platform does 5 things well, not 12 things poorly

You walk in, demo this flow in 90 seconds, and say: "This is a proof-of-concept. In production, states could plug their own land-records APIs into this, and policymakers could model reforms before enacting them. Right now, land policy is made blind. This gives them eyes."

---

## Common Pitfalls (Avoid These)

❌ **Don't:** Build a chatbot that makes up citations  
✅ **Do:** Only cite passages that actually appear in the documents

❌ **Don't:** Fake the simulation with a black-box ML model  
✅ **Do:** Use simple, transparent formulas and cite the research behind each coefficient

❌ **Don't:** Try to build all 12 capabilities  
✅ **Do:** Pick repository + search + GIS + simulation, execute perfectly

❌ **Don't:** Spend weeks on design before you have working code  
✅ **Do:** Use Tailwind, ship fast, iterate on real user feedback

❌ **Don't:** Leave the corpus empty or with dummy data  
✅ **Do:** Spend 1 week downloading and cleaning 100+ real documents

❌ **Don't:** Make claims the judges can't verify in a live demo  
✅ **Do:** Show working code, real data, and clear assumptions

---

## Success Metrics (For Judging)

**Technical execution (40%):**
- Does the search work? Does it return sensible results?
- Is the synthesis grounded in actual document passages?
- Does the map render and update?
- Does the simulation output make sense?

**Problem framing (30%):**
- Did you identify a real pain point? (Policy made without evidence)
- Is your solution actually responsive to that pain point?
- Do you acknowledge what you're NOT doing? (e.g., you're not replacing econometricians; you're surfacing existing evidence)

**Demo clarity (20%):**
- Can a non-technical judge understand what the tool does in 90 seconds?
- Does the demo flow like a real researcher or official using it?
- Can you explain the assumptions live?

**Finish (10%):**
- Does it crash?
- Are there obvious bugs?
- Does the UI look intentional (even if simple)?

---

## Deployment Checklist (Week 18)

- [ ] Database backed up
- [ ] Environment variables for API keys not committed to Git
- [ ] HTTPS enabled
- [ ] Rate limits on search/API endpoints
- [ ] Auth cookies are secure (httpOnly, sameSite)
- [ ] Demo script rehearsed 5+ times
- [ ] PDF export working
- [ ] Mobile responsive (at least tablet-friendly)
- [ ] 100+ documents seeded in corpus
- [ ] Simulation model documented (which papers justify each coefficient?)
- [ ] Slide deck prepared (problem → solution → impact)

---

## Pitch Deck Outline (For Finale)

**Slide 1: Problem**  
"Land policy in India affects 50M+ families and ₹1T+ in assets, yet policymakers make decisions without access to existing research and evidence."

**Slide 2: Solution**  
"A national platform that aggregates land governance research, makes it discoverable via semantic search, and lets officials model the impact of proposed reforms before enacting them."

**Slide 3: Core Demo**  
(Record a 60-sec screen capture of the demo flow)

**Slide 4: How It Works**  
- Semantic search finds relevant documents
- LLM synthesis with citation grounding
- Policy simulation with transparent assumptions

**Slide 5: Impact**  
"If adopted by 5 states' revenue departments, this could reduce land disputes by 15–20% and accelerate digitization adoption."

**Slide 6: Feasibility**  
"We've built a working MVP in [timeframe]. Scaling requires API integration with state land-records systems and regular model recalibration."

**Slide 7: Team**  
(Photos + roles)

---

## Example Simulation Coefficients

When you model "faster dispute resolution," here's how you might set it up:

```json
{
  "lever_id": "dispute_resolution_speed",
  "name": "Reduce average dispute resolution time",
  "unit": "months",
  "default_value": 24,
  "range": [6, 48],
  "impact_model": {
    "formula": "base_disputes * (1 - elasticity * change_pct)",
    "elasticity": 0.55,
    "source_papers": [
      "Meinzen-Dick et al. (2019) - Land conflict and speed of adjudication",
      "DILRMP impact assessment, Ministry of Rural Development (2021)"
    ],
    "confidence": "Medium (based on 5 state case studies)"
  },
  "example": {
    "base_disputes": 1200,
    "current_resolution_time": 24,
    "proposed_resolution_time": 12,
    "change_pct": -0.5,
    "projected_disputes": 1200 * (1 - 0.55 * 0.5) = 1200 * 0.725 = 870,
    "interpretation": "If we halve resolution time, we'd expect a 27% drop in disputes filed, but actual impact depends on whether people believe the system will actually be faster."
  }
}
```

---

## Final Note

**This is a policy research and evidence platform, not a crystal ball.** The value isn't predicting the future perfectly; it's surfacing existing research and modeling transparent assumptions. If you stay honest about what the data actually says and what it doesn't, you'll build something that policymakers will actually use.

Good luck. Ship it.
