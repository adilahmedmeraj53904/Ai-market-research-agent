# AI Market Research Agent

An n8n workflow that turns a simple form into a professional local market research report — delivered to your Gmail as a clean, mobile-friendly HTML email.

Fill in a research topic, and the workflow finds real businesses on Google Maps, checks each one again through SerpApi, analyses customer reviews with AI, and emails you a structured report with strengths, weaknesses, opportunities, and recommendations.

## What it does

- Collects a research request from a form: **Research Topic, Category, Country, Language**
- Searches Google Maps through **SerpApi** for matching local businesses
- Keeps the **top 4 unique businesses** — no more, no duplicates
- Verifies each business one by one with a second SerpApi check
- Samples up to **5 customer reviews per business**
- Analyses everything with an **AI Agent (Groq — `openai/gpt-oss-120b`)**
- Formats the report as responsive HTML and sends it by **Gmail**

## How it works

```
Form submission
  → Edit Fields
  → SerpApi Google Maps search
  → Rearrange data (Code — clean, dedupe, limit to 4)
  → Loop Over Items
      → HTTP Request (SerpApi review data)
      → Looping Structure (Code — validate & classify)
      → back to Loop Over Items
  → Aggregate
  → Code in JavaScript (prepare data for AI)
  → AI Agent + Groq Chat Model
  → Markdown (Markdown to HTML)
  → Professional Email Formatter (Code — responsive HTML email)
  → Send a message (Gmail)
```

### 1. Input — Form Trigger

The user submits:

| Field | Required |
| --- | --- |
| Research Topic | Yes |
| Category | Yes |
| What is your Country | Yes |
| Language? | No |

### 2. Search — SerpApi Google Maps

The topic, category, and country from the form are sent to SerpApi Google Maps search. The first result set is cleaned in the **Rearrange data** Code node: whitespace and formatting are normalised, duplicates are removed by `place_id` / `data_id` / name + address, and only the first **4 businesses** are kept.

Each business keeps its name, type, address, GPS coordinates, phone, website, Google rating, review count, `place_id`, `data_id`, reviews link, description, price, and operating hours.

### 3. Verify — Loop, HTTP Request, Validation

Each of the 4 businesses is checked again, one at a time:

- **Loop Over Items** sends one business per iteration.
- **HTTP Request** calls SerpApi again using that business's `reviews_link`, with SerpApi API key authentication.
- **Looping Structure** (Code) scores the match between the original listing and the new SerpApi result:
  - Name similarity — weight 65%
  - Address similarity — weight 25%
  - Category similarity — weight 10%
  - An exact `place_id` match scores 1.0

Classification:

| Result | Rule |
| --- | --- |
| `valid` | Exact Place ID match, or name similarity ≥ 0.75 and total score ≥ 0.60 |
| `needs_review` | Total score ≥ 0.35 |
| `invalid` | Below 0.35, or no matching business found |

Unverified listings stay marked `unverified` in the final report — they are never presented as confirmed facts.

### 4. Prepare — Aggregate + Code in JavaScript

**Aggregate** collects all validated businesses back into one item. The **Code in JavaScript** node then builds the AI input:

- Up to 4 businesses, up to 5 reviews per business (review text cut to 300 characters)
- Statistics: businesses found, reviews sampled, average Google rating
- Clear analysis instructions: analyse each business separately, compare strengths and weaknesses, find customer satisfaction patterns and common complaints, evaluate competitive positioning, and give actionable recommendations
- Clear limits: do not invent missing information, distinguish facts from interpretations, treat unverified listings as unverified

### 5. Analyse — AI Agent (Groq)

The AI Agent receives the prepared JSON and writes a professional report of approximately **700–1000 words** in Markdown, with these sections:

1. Executive Summary
2. Research Scope and Data Quality
3. Market and Industry Overview
4. Competitor Landscape
5. Customer Review Analysis
6. Business Strengths
7. Business Weaknesses
8. Market Opportunities
9. Potential Risks
10. SWOT Analysis
11. Strategic Recommendations
12. Final Conclusion
13. Research Limitations

Model: **Groq — `openai/gpt-oss-120b`**

### 6. Deliver — Markdown, Email Formatter, Gmail

- **Markdown** converts the AI report from Markdown to HTML.
- **Professional Email Formatter** builds a responsive HTML email (max-width 700px, works on mobile and laptop) with a header, research overview (topic, industry, country, date in Asia/Dhaka), the full report, and a methodology footer.
- **Send a message** (Gmail) sends it with the subject:

```
Market Research Report | <Research Topic> | <Country>
```

## Nodes

14 nodes in total:

| Node | Type | Purpose |
| --- | --- | --- |
| On form submission | Form Trigger | Collect research request |
| Edit Fields | Set | Pass topic, category, country forward |
| Google_maps search | SerpApi | Google Maps business search |
| Rearrange data | Code | Clean, dedupe, limit to 4 businesses |
| Loop Over Items | Split In Batches | Process one business at a time |
| HTTP Request | HTTP Request | Second SerpApi check via reviews link |
| Looping Sturcter | Code | Validate and classify each business |
| Aggregate | Aggregate | Collect validated results |
| Code in JavaScript | Code | Prepare clean JSON for the AI Agent |
| AI Agent | AI Agent | Write the market research report |
| Groq Chat Model | Groq Chat Model | `openai/gpt-oss-120b` |
| Markdown | Markdown | Markdown to HTML |
| Professional Email Formatter | Code | Responsive HTML email layout |
| Send a message | Gmail | Send the finished report |

## Setup

### Requirements

- n8n (self-hosted or cloud)
- SerpApi account + API key
- Groq API key
- Gmail account connected with OAuth2

### Credentials to connect in n8n

| Credential | Used by |
| --- | --- |
| SerpApi account | Google_maps search |
| Query Auth account (SerpApi API key) | HTTP Request |
| Groq account | Groq Chat Model |
| Gmail account | Send a message |

### Steps

1. Import `ai-market-research-agent-workflow.json` into n8n.
2. Connect the four credentials above.
3. In the **Send a message** (Gmail) node, set the recipient to your own email address.
4. Activate the workflow and open the production Form Trigger URL.
5. Submit a topic, category, and country — the report arrives by Gmail.

## Example use cases

- A new café, salon, clinic, or shop checking local competitors before opening
- A freelancer researching a client's market before a sales call
- A small business comparing ratings, reviews, and positioning in its area

## Limitations

- Maximum **4 businesses** and **5 reviews per business** per report — this keeps the report focused and the AI input reliable, but it is a sample, not a full market census.
- Google Maps ratings, review counts, and business details can change at any time.
- Sampled reviews may not represent all customers.
- The AI report is an interpretation of the supplied data. Verify important details independently before making a business decision.
- The form collects a **Language** field, but the current workflow does not pass it to the search or the AI Agent yet.

## Files

```
ai-market-research-agent/
├── README.md
└── ai-market-research-agent-workflow.json
```

## Status

Built — workflow file included in this repo. Import it, connect your own credentials, and run a test form submission before using it for client work.

## Disclaimer

This workflow is for research and educational use. It analyses publicly available Google Maps business information and AI-assisted summaries. It does not guarantee business results, and its findings should be reviewed before any important business decision.

---

Built with n8n, SerpApi, Groq, and Gmail by **Adil Ahmed**.
