# AI/Cloud/Semiconductor Ecosystem Knowledge Graph

## What this is
An interactive map of the global AI infrastructure ecosystem — from raw
materials and fab equipment through chip design, manufacturing, servers,
cloud providers, and the AI model companies that consume it all. The
product should feel like a blend of Apple Vision Pro UI, Linear, Stripe,
Arc Browser, Bloomberg Terminal, Figma, and Notion: premium, elegant,
futuristic, and data-dense — an "AI infrastructure atlas," not a network
diagram.

## Current state (V2 — district-flow atlas)
`knowledge_graph.html` is a single-file HTML/CSS/JS prototype: 64
companies, 104 typed relationships, organized into 13 left-to-right
"districts" that mirror the physical AI supply chain (materials →
equipment → design software → fabrication → chip design → memory →
networking → server OEMs → ODMs → power & cooling → data centers →
cloud → AI companies). No backend, no persistence — same constraint as
V1, data is baked into the page.

**V1 → V2 change:** V1 was a D3.js force-directed graph (organic node
network, physics simulation, drag-to-rearrange). It proved the concept
but was hard to read at a glance — relationships created visual clutter
and the supply-chain flow wasn't obvious. V2 replaces the force layout
with fixed glassmorphism "district" panels in ecosystem order. Company
relationships are hidden by default and only drawn (as animated SVG
curves) on hover/click, so the page reads instantly rather than requiring
untangling. A "Trace the AI supply chain" guided tour auto-highlights
ASML → TSMC → NVIDIA → Supermicro → Microsoft → OpenAI to make the
flow legible within seconds.

## Target architecture (what Claude Code should build toward)
- **Frontend:** React + React Flow for the graph canvas eventually, but
  the current district-panel layout does not require a node-graph engine
  — a CSS grid/flex layout (as built) may remain the better fit even
  after a React migration. Re-evaluate before assuming React Flow is
  still the right call for this visual style.
- **Data layer:** Move the hardcoded `COMPANIES`/`EDGES` arrays into a
  real data store (structured JSON/SQLite is fine, don't over-engineer a
  backend on day one) so data isn't baked into the frontend bundle.
- **Detail panel:** The right sidebar (description, relationships,
  primary customers, notes, ecosystem position indicator) is built to
  the target shape already — extend it, don't replace it.

## Data model
Each company node has:
- Company name (`id`), district, subcategory, "what they make"
- Description, revenue driver, primary customers
- AI exposure (Very High / High / Medium)
- Size tier (Mega-cap / Large-cap / Mid-cap / Private — real dollar
  figures are a future feature, not V2)
- Headquarters
- TransPak relevance (Yes / No / TBD) — custom internal flag, not a
  public data point
- Notes (freeform internal notes, currently mostly empty — the sidebar
  renders a placeholder when empty)
- `domain` — used to fetch a company logo via Clearbit's logo API at
  render time (`https://logo.clearbit.com/{domain}`), with a graceful
  fallback to a colored initials badge if the logo fails to load or the
  page has no network access.

Relationships are first-class typed edges, not just lines:
Manufactures for, Supplies, Sells to, Competes with, Partners with,
Invests in, Owns. The sidebar groups a company's edges into Upstream
suppliers / Downstream customers / Partners / Competitors based on edge
type + direction.

Example: `ASML —Supplies→ TSMC`, `TSMC —Manufactures for→ NVIDIA`,
`NVIDIA —Sells to→ Microsoft`.

## Ecosystem districts (left → right flow order)
1. Materials & Chemicals
2. Semiconductor Equipment
3. Design Software
4. Chip Manufacturing (fabrication/foundry)
5. Chip Designers
6. Memory
7. Networking & Interconnect
8. Server OEMs
9. ODMs / Contract Manufacturers
10. Power & Cooling
11. Data Centers
12. Cloud Providers / Hyperscalers
13. AI Companies

Note: "Cloud Providers" and "Hyperscalers" were merged into one district
(same five companies — Microsoft/Amazon/Google/Oracle/Meta — play both
roles). "Power & Cooling" was added as its own district (not in the
original ask) because the source spreadsheet has 9 companies in that
space (Vistra, Constellation Energy, NextEra Energy, GE Vernova, Vertiv,
Schneider Electric, CoolIT Systems, Boyd, LiquidStack) with no natural
home in the requested 13-stage flow otherwise.

## Design requirements (V2)
- Left-to-right horizontal scroll through districts, not a force graph
- Each district is a glassmorphism panel with its own accent color,
  gradient background, header, and company count
- Company relationships hidden by default; revealed via animated SVG
  curves on hover (subtle) and click/select (strong, with sidebar open
  and unrelated companies faded)
- Guided "Trace the AI supply chain" tour as the flagship "understand in
  3 seconds" feature
- Dark mode (`#050816` deep background) as the primary aesthetic; light
  mode as a secondary toggle
- Space Grotesk for headings, Plus Jakarta Sans for body text (Geist was
  requested but isn't reliably available on Google Fonts; Plus Jakarta
  Sans was the next suggested alternative)
- Company logos via Clearbit's logo API with an initials-badge fallback
  (works fully offline, upgrades visually when network is available)

## Roadmap (build toward this, don't build it all now)
The sidebar already has a "Coming soon" section stubbing these out:
- Live financial data (real market cap, not size tiers), stock price
- Earnings dates, company news feeds, SEC filings
- AI-generated company summaries
- Relationship discovery (auto-suggesting new edges)
- Custom "TransPak lens" — already live as a dimming filter toggle in V2
- Search/filter — already live in V2 (company name + subcategory)
- Saved views

## Explicit scope guardrail
Don't try to build the full Bloomberg-terminal vision in one pass. Build
incrementally: the district-flow layout and 64-company dataset are
solid for V2 — layer in real data persistence next, then the roadmap
features above one at a time.

## Source data
`Industry_Ecosystem_Tracker.xlsx` in this folder is the source of truth
for company facts (category, subcategory, customers, revenue driver, AI
exposure, TransPak flag) — treat it as authoritative if it conflicts
with anything hardcoded in the HTML. It covers 60 of the 64 companies in
the current build; the other 4 (Synopsys, Cadence, Cerebras Systems,
Groq) came from the V1 prototype and were kept because the requested
"Design Software" and additional "Chip Designers" coverage would
otherwise be thin or empty.
