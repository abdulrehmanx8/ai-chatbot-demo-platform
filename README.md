# AI Chatbot Demo Platform for Local Service Businesses

A system that lets me show local home-service businesses a **working AI chatbot trained on their own website**, before they commit to anything. Enter a business's website, and they get a personalized demo page with a live chat assistant that answers customer questions using only their content.

## How it works

```
Workflow 1: Ingestion
Form (business name + URL)
   -> Scrape website pages
   -> Clean + chunk text
   -> Generate embeddings (Gemini, 3072 dimensions)
   -> Store in Supabase (keyed by lead_id)

Workflow 2: Chatbot
Personalized link (?lead_id=...&name=...)
   -> Chat widget sends message + lead_id -> Webhook
   -> Load the business's content + conversation history (SQL)
   -> Build prompt (JavaScript)
   -> Save user message
   -> Gemini generates the answer
   -> Save bot reply
   -> Respond to the widget
```

## Files

| File | Purpose |
|------|---------|
| `business_scraper.json` | n8n workflow: takes a business website, fetches its pages, extracts and cleans the text, chunks it, creates embeddings and stores them in Supabase under a unique `lead_id`. |
| `chatbot.json` | n8n workflow: webhook-driven chat backend that receives each message, loads the lead's knowledge base and recent conversation history from Postgres, generates a reply with Google Gemini, and saves both sides of the conversation. |
| `funnel.html` | Frontend demo page (HTML/CSS/JS) with the embedded chat widget. |
| `schema.sql` | Supabase table definitions (`documents` and `conversations`). |

## Scraper highlights
- Discovers pages through the site's sitemap, then loops through each page with wait intervals between requests.
- Extracts text from HTML and cleans it with custom JavaScript (Code) nodes.
- Uses full-page text chunking, since small business sites have no shared data schema.
- Each lead gets an isolated knowledge base through `lead_id`.

## Chatbot highlights
- **Persistent conversation memory:** every user message and bot reply is stored in Postgres, and recent history is fetched on each turn so follow-up questions work naturally.
- **Grounded answers:** the prompt is built from the business's own scraped content, so the bot answers about their services rather than guessing.
- **Simple integration:** a single webhook endpoint, so the frontend only needs one POST request per message.

## Frontend

A custom-branded landing page with an embedded chat widget. The business name, greeting and branding adapt to each prospect, so the demo feels built specifically for them.

### Personalization with URL parameters

Each prospect gets their own link. The page reads `lead_id` and `name` from the URL query string:

```
funnel.html?lead_id=abc123&name=Miami%20landscape
```

| Parameter | Used for |
|-----------|----------|
| `lead_id` | Sent with every chat message to the webhook, so the bot only uses that business's knowledge base and chat history |
| `name` | The business name shown in the headline, chat header and greeting (e.g. "Hey Miami landscape...") |

So one HTML file serves every lead.

<img width="1917" height="910" alt="image" src="https://github.com/user-attachments/assets/73ecaf02-d977-42fd-9f0d-88ff8adf9b23" />



## Database

Two tables in Supabase (defined in `schema.sql`):

| Table | Purpose |
|-------|---------|
| `documents` | Scraped website chunks: `content`, `metadata` (jsonb) and `embedding` (`vector(3072)`) |
| `conversations` | Chat history: `session_id`, `lead_id`, `role`, `content` |

## Tech stack

- **n8n** for workflow automation
- **Supabase (PostgreSQL + pgvector)** for knowledge base and chat history
- **Google Gemini** for embeddings (3072 dimensions) and answer generation
- **JavaScript, HTML, CSS** for the frontend and Code nodes

## Setup

### 1. Supabase
1. Create a Supabase project.
2. Open the **SQL Editor**, paste the contents of `schema.sql`, and run it. This enables pgvector and creates the `documents` and `conversations` tables.

### 2. n8n
1. Import `business_scraper.json` and `chatbot.json` (**Workflows -> Import from file**).
2. Add your credentials in n8n: Supabase/Postgres connection and Google Gemini API key.
3. Activate both workflows and copy the **production webhook URL** from the chatbot workflow's Webhook node.

### 3. Frontend
1. Open `funnel.html` in a text editor and find the webhook URL (search for `webhook` or `fetch(`).
2. Replace it with your own chatbot webhook URL from step 2.
3. Host it on Netlify, Vercel or GitHub Pages, or open it locally in a browser.
4. Test it with your own parameters, e.g. `funnel.html?lead_id=YOUR_LEAD_ID&name=Your%20Business`. The `lead_id` must match the one used when the scraper stored that business's content.

> No API keys or webhook URLs are included in this repo. You must add your own.

## Use case

Built for outreach to local home-service businesses: instead of describing a chatbot, I send them one that already knows their services and FAQs.

## Author

**Abdul Rehman**
[LinkedIn](https://www.linkedin.com/in/m-abdul-rehman-b58b42404)
