# Funding Opportunities Search Form
 
Fuzzy search middleware connecting Jotform to Airtable for research funding opportunity lookups.
 
**Live**: https://gfad.vercel.app/
**Search Form**: https://www.jotform.com/form/252758211486058
 
## Problem
 
American University's Office of Research maintains a database of funding announcements in Airtable. Researchers need a way to search this database by keyword, discipline, funder, and deadline without direct Airtable access. The search needs to handle partial matches and typos (fuzzy matching), combine multiple filter criteria, and return results through a simple form interface.
 
## Approach
 
A Node.js/Express middleware sits between Jotform (user-facing search form) and Airtable (funding database). The flow:
 
1. User submits search criteria via Jotform (keyword, discipline, funder, deadline)
2. Vercel-hosted middleware queries Airtable, retrieves records
3. Fuse.js performs fuzzy matching against all text fields (threshold: 0.4)
4. Deadline and discipline filters apply on top of fuzzy results
5. Results return as formatted HTML with relevance scores and direct links
```
Jotform  -->  Vercel (Express + Fuse.js)  -->  Airtable
                      |
                      v
               HTML Results Page
```
 
Field whitelisting ensures only approved columns are exposed. Airtable token is scoped to read-only access on a single base.
 
## Results
 
The system handles four filter types independently or in combination: keyword (fuzzy matched across all fields), discipline (Science, Arts and Humanities, Business, Social Sciences, Law, Education), funder name, and deadline (on or after a given date). Each result displays a relevance percentage and links to the full opportunity record. Changes in Airtable are reflected immediately with no redeployment required.
 
## Project Structure
 
```
search-form/
├── server.js          # Express middleware, Fuse.js search, HTML response
├── package.json       # Dependencies
├── vercel.json        # Deployment config
├── .env.example       # Environment variables template
├── .gitignore
├── LICENSE
└── README.md
```
 
## Usage
 
### Live
 
Search directly at https://www.jotform.com/form/252758211486058
 
### Run Locally
 
```bash
git clone https://github.com/SullenSchemer/Search-Form.git
cd Search-Form
npm install
cp .env.example .env
# Edit .env with Airtable credentials (see below)
npm start
```
 
Test at `http://localhost:3000/health` and `http://localhost:3000/search?keyword=research`.
 
### Environment Variables
 
| Variable | Description |
|----------|-------------|
| `AIRTABLE_API_KEY` | Personal Access Token (read-only, starts with `pat`) |
| `AIRTABLE_BASE_ID` | Base ID (starts with `app`) |
| `AIRTABLE_TABLE_NAME` | Exact table name |
| `AIRTABLE_VIEW_NAME` | Optional: restrict to a specific view |
| `ALLOWED_FIELDS` | Optional: comma-separated field whitelist |
 
Get credentials: Airtable token at https://airtable.com/create/tokens (scope: `data.records:read`). Base ID is in the Airtable URL after `airtable.com/`.
 
### API
 
```
GET /health              # Status check
GET /search?keyword=stem # Keyword search
GET /search?keyword=research&discipline=Science&deadline=2026-01-01  # Combined filters
POST /search             # Programmatic access (JSON body: { "query": "...", "limit": 10 })
```
 
## Tech Stack
 
- **Runtime**: Node.js 18+, Express.js
- **Search**: Fuse.js (fuzzy matching)
- **Database**: Airtable
- **Hosting**: Vercel
- **Form**: Jotform
## Limitations
 
- **No caching**: Every search hits Airtable live. High traffic could hit API rate limits.
- **Fuzzy threshold**: Set to 0.4. Too broad for some queries, too narrow for others. No user-adjustable sensitivity.
- **Single base**: Searches one Airtable base. Multiple bases would require separate deployments or base-switching logic.
- **Linked records**: Airtable linked record and lookup fields return objects, not strings. Some fields may display as `[object Object]` if not handled in the response template.
- **No search analytics**: No logging of what users search for or how often.
## License
 
MIT