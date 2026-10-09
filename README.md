<div align="center">

<h1>B2B Lead Research and Cold Outreach Automation System</h1>

<p><strong>An n8n workflow for AI-assisted lead research and human-approved cold outreach, using Google Sheets as the lead data layer.</strong></p>

<p>
  <img src="https://img.shields.io/badge/n8n-Workflow_Automation-EA4B71?style=flat-square&logo=n8n&logoColor=white" alt="n8n workflow automation" />
  <img src="https://img.shields.io/badge/Google_Gemini-gemini--flash--lite--latest-8E75B2?style=flat-square&logo=googlegemini&logoColor=white" alt="Google Gemini, model gemini-flash-lite-latest" />
  <img src="https://img.shields.io/badge/Google_Sheets-Lead_Data_Layer-34A853?style=flat-square&logo=googlesheets&logoColor=white" alt="Google Sheets as the lead data layer" />
  <img src="https://img.shields.io/badge/Gmail-Send_and_Inbound-EA4335?style=flat-square&logo=gmail&logoColor=white" alt="Gmail for sending and inbound handling" />
  <img src="https://img.shields.io/badge/Telegram-Human_Approval-26A5E4?style=flat-square&logo=telegram&logoColor=white" alt="Telegram for human approval" />
</p>

<p>
  <img src="https://img.shields.io/badge/Human--in--the--loop-Approval_before_send-2F6FEB?style=flat-square" alt="Human approval before send" />
  <img src="https://img.shields.io/badge/Email_validation-Local_format_check-6E7781?style=flat-square" alt="Local email format validation" />
  <img src="https://img.shields.io/badge/Follow--up-Single_3--day_follow--up-6E7781?style=flat-square" alt="Single follow-up after three days" />
</p>

</div>

> This workflow researches a prospect from public web pages, asks Gemini to analyze the collected evidence and draft a personalized email, and stores the draft in Google Sheets. A person then approves or rejects the send in Telegram. Gmail sends only after that approval passes a final safety gate. A separate inbound branch watches unread Gmail messages for replies and opt-out language and updates the matching lead row. The project is complete and has been tested, and the tests executed produced positive results.

---

## Project Overview

The **B2B Lead Research and Cold Outreach Automation System** is a completed n8n automation that turns a row in a Google Sheet into a researched, reviewed, and (once approved) sent outreach email, and then manages the follow-up and inbound-reply lifecycle for that lead.

It connects four services into one controlled pipeline:

- **Google Sheets** holds the lead table and acts as the state store.
- **Google Gemini** performs evidence-guided analysis and drafts the personalized email.
- **Telegram** provides the human approval step with inline buttons and operator alerts.
- **Gmail** sends the initial email and follow-up, and monitors unread inbound messages.

The design principle is that research evidence is collected source by source, the AI is restricted to that evidence, deterministic code normalizes and flags the AI output, and no email leaves the system without a human decision and a final send-safety check.

---

## Project Snapshot

| Item | Details |
|------|---------|
| ⚙️ Automation Platform | n8n workflow (`01 B2B Lead Research & Cold Outreach Automation System`), 66 nodes. The export is saved with `active: false`, so the workflow is activated after credentials and placeholders are configured |
| ✅ Project Status | Completed and tested. The tests executed produced positive results |
| 🗂️ Data Store | Google Sheets, one sheet used as the lead table and the state store, keyed on the `Email` column |
| 🔎 Research Layer | Multi-source public-web research: company homepage, candidate About and Blog pages, a LinkedIn public-page fetch attempt, and homepage HTML technology indicators, with explicit research-status classification |
| 🧠 AI Layer | Google Gemini node configured with `models/gemini-flash-lite-latest`, prompted to analyze only the supplied evidence and return structured JSON |
| 🛡️ Review Control | Confidence gate (threshold 80) that routes every lead to either `Pending Review` or `Needs Review`; both routes require Telegram approval |
| 👤 Human Review | Telegram inline buttons (Approve and Send, Reject) backed by 24-hour, single-use, claim-protected tokens |
| ✉️ Email Channel | Gmail for the initial email and the follow-up |
| 🔁 Follow-up | One follow-up after a 3-day Wait node, sent from a fixed text template, only if the lead is still eligible |
| 📥 Inbound Handling | Gmail unread-message trigger that detects opt-out language and replies, matched to leads by sender email |

---

## Workflow Screenshot

<img width="1280" height="596" alt="01 B2B Lead Research   Cold Outreach Automation System" src="https://github.com/user-attachments/assets/cbf1b1a9-84d9-44fe-a751-ca6da5813efc" />

---

## What This System Does

The workflow has three entry points, each with its own trigger:

1. **Lead intake.** A Google Sheets trigger fires when a row is added. The workflow normalizes the row, checks the email, researches the company, asks Gemini for an analysis and draft, saves the result to the sheet, and sends an approval request to Telegram.
2. **Approval handling.** A Telegram trigger listens for button presses. It resolves the approval token, reloads the lead from the sheet, applies state and safety checks, and either rejects the lead or sends the email through Gmail.
3. **Inbound monitoring.** A Gmail trigger polls unread messages. It extracts the sender, looks for opt-out language, and updates the matching lead row to `Do Not Contact` or `Replied`.

No part of the AI path sends an email on its own. The Gmail send nodes are reachable only through the Telegram approval path.

---

## Core Capabilities

| Capability | What the workflow implements |
|------------|------------------------------|
| 📋 Lead ingestion | Google Sheets trigger on `rowAdded`, polling every minute |
| 🧹 Input normalization | Trims fields, lowercases the email, adds `https://` to bare URLs, strips trailing slashes, and validates URLs with the JavaScript `URL` constructor |
| ✅ Required-field check | Rows with no email are logged as `Missing Email` and do not continue |
| 📧 Email format validation | Local regular-expression check with a 254-character length limit, performed inside the workflow with no external service or credential |
| 🔎 Public-page research | Homepage fetch plus candidate About and Blog paths, each with its own availability record |
| 🔗 LinkedIn attempt | Best-effort fetch of the lead's public LinkedIn URL |
| 🧪 Technology detection | Ten homepage HTML patterns labeled `confirmed` or `inferred` |
| 🧠 Structured AI analysis | Gemini prompted to return one JSON object containing a summary, pain point, evidence list, subject options, and email body |
| 🔢 Confidence routing | `data_confidence_score >= 80`, no human-review flag, and a successful parse |
| 👤 Telegram approval | Inline Approve and Reject buttons, with warnings listed in the message |
| 🔐 Token lifecycle | 24-hour expiry, single use, a 2-minute claim for duplicate-click protection, and expired-token cleanup |
| 🚦 Final send gate | Checks that email, subject, and body are present and that the status is not in a protected list |
| 🔁 Delayed follow-up | 3-day wait, lead reload, eligibility check, then a templated Gmail send |
| 📥 Inbound handling | Sender extraction, opt-out phrase detection, lead lookup, and status updates |
| 🔔 Operator notifications | Telegram messages for token problems, lookup failures, already-actioned leads, and unmatched inbound email |

---

## Workflow Architecture

### Entry Points

| Trigger | Node type | Behavior |
|---------|-----------|----------|
| Lead intake | Google Sheets Trigger | Event `rowAdded`, poll every minute, sheet `gid=0` |
| Approval clicks | Telegram Trigger | Listens for `callback_query` updates only |
| Inbound mail | Gmail Trigger | Poll every minute with the filter `is:unread` |

### Phases

| Phase | Purpose | Key nodes and controls |
|-------|---------|------------------------|
| Intake | Receive and normalize a lead row | `Google Sheets Trigger`, `Skip Already Processed Rows`, `Normalize Lead Input` |
| Validation | Establish usable input | `Has Email?`, `Email Validation`, `Normalize Validation Result`, `Email Acceptable?` |
| Research | Collect public evidence | `Homepage Research`, `About Page Research`, `Blog Content Research`, `LinkedIn Research`, `Technographic Research`, `Data Consolidation` |
| Analysis | Produce a structured analysis and draft | `AI Lead Analysis`, `Parse AI Output` |
| Routing | Decide the review label | `Confidence Gate`, `Save Lead - Hyper-Personalized`, `Save Lead - Needs Review` |
| Review | Human approval decision | `Generate Approval Token`, `Telegram Approval Request`, `Telegram Trigger`, `Resolve Approval Token`, `Token Valid?` |
| Outreach | Controlled initial send | `Find Lead for Approval`, `Lead Already Actioned?`, `Approve or Reject?`, `Final Send Safety Gate`, `Send Initial Email` |
| Follow-up | Delayed second message | `Wait for Follow-up`, `Find Lead Before Follow-up`, `Follow-up Eligible?`, `Send Follow-up Email` |
| Inbound | Reply and opt-out processing | `Gmail Trigger`, `Extract Sender Email`, `Detect Opt-Out`, lead lookups, status updates |

---

## End-to-End Workflow Stages

| Stage | Component | Purpose | Control |
|-------|-----------|---------|---------|
| 01 | Intake | Read a new sheet row | Fields `Name`, `Email`, `Company`, `Company_URL`, `LinkedIn_URL` |
| 02 | Normalization | Clean the row | Invalid URLs become empty strings. Missing values are reported, not invented |
| 03 | Email presence | Require an email | Missing email is logged and the row stops |
| 04 | Email format | Check syntax locally | Invalid format is logged and the row stops |
| 05 | Research | Gather public evidence | Each source keeps its own available, url, content, and reason fields |
| 06 | Consolidation | Build one payload | Computes `research_status` |
| 07 | AI analysis | Generate analysis and draft | Prompt restricts the model to the supplied evidence |
| 08 | Output parsing | Parse and normalize | Failure produces a safe, flagged result |
| 09 | Confidence gate | Choose the label | Threshold 80, no flag, successful parse |
| 10 | Persistence | Write the draft to Google Sheets | `appendOrUpdate` matched on `Email` |
| 11 | Approval request | Notify the reviewer | Token generated, Telegram message with buttons |
| 12 | Approval resolution | Validate and claim the token | Expiry, used, and claim checks |
| 13 | Lead reload | Read the current row | Lookup errors are treated as not found |
| 14 | State check | Block already-actioned leads | Telegram notice when blocked |
| 15 | Decision | Approve or reject | Anything other than `approve` takes the reject path |
| 16 | Send gate | Final checks before Gmail | Required fields and protected statuses |
| 17 | Initial send | Send through Gmail | Success is `!error && Boolean(id)` |
| 18 | Follow-up | Wait 3 days and re-check | Reloads the lead before any send |

---

## Lead Research Layer

Research runs as a multi-source serial chain. Every source writes its own result record (availability, URL, content, and reason), so each source is handled on its own terms. A source that cannot be retrieved stays marked unavailable with a recorded reason, and the pipeline continues with the evidence that was collected.

| Source | Method | Evidence handling |
|--------|--------|-------------------|
| 🏠 Homepage | HTTP Request node to `Company_URL`, `User-Agent: Mozilla/5.0`, up to 5 redirects, 12-second timeout, `neverError` enabled | Counted as available when a non-empty text body is returned. Tags and boilerplate are stripped and the text is cut to 2,500 characters |
| 🏢 About / Company | Code node trying `/about`, `/about-us`, `/company`, `/who-we-are` in order, 8-second timeout each | The first response with a 2xx status and HTML-like content is used. Text is cut to 2,500 characters. Otherwise the reason is `page_not_found` |
| 📰 Blog / Content | Code node trying `/blog`, `/news`, `/insights`, `/resources`, `/articles` in order | Same rules as About. Otherwise `page_not_found` |
| 🔗 LinkedIn | Single GET to the lead's public `LinkedIn_URL`, 8-second timeout | Available on a 2xx HTML response. Text is cut to 1,500 characters. Otherwise a reason such as `no_linkedin_url_provided`, `fetch_blocked_status_<code>`, or `fetch_failed` is recorded |
| 🧪 Technographics | Regular expressions over the fetched homepage HTML | Detects WordPress, Shopify, Webflow, Squarespace, HubSpot, Google Analytics, Stripe, Cloudflare, React, and Intercom. Cloudflare and React are labeled `inferred`. The rest are `confirmed` |

### Research Status

`Data Consolidation` computes `research_status` from the homepage, About, and Blog results. LinkedIn is recorded as supplementary evidence and does not change the status.

| Value | Condition |
|-------|-----------|
| `full_research` | Homepage, About, and Blog are all available |
| `partial_research` | At least one of those three is available |
| `research_failed` | None available, but a company URL exists |
| `company_url_missing` | None available and no valid company URL |

The research status travels with the lead. It appears in the AI prompt, which states when no evidence was retrieved, it is saved to the `Research Status` column, and it appears as a warning in the Telegram approval message whenever it is anything other than `full_research`.

The workflow relies entirely on public pages. It uses no third-party enrichment provider and no authenticated LinkedIn access, and the LinkedIn step is a public-page fetch attempt only.

---

## Email Validation Layer

The workflow validates email format locally in two code nodes (`Email Validation` and `Normalize Validation Result`). No external service, API key, or credential is involved.

| Check | Behavior |
|-------|----------|
| Empty value | Status `invalid`, reason `empty_email` |
| Length over 254 characters | Status `invalid`, reason `exceeds_max_length` |
| Pattern mismatch | Status `invalid`, reason `format_does_not_match_pattern` |
| Pattern match | Status `valid`, reason `format_valid` |
| Unrecognized result | Normalized to `unknown` with a manual-review flag, never treated as valid |

`Email Acceptable?` lets `valid` and `unknown` results continue to research. `invalid` results are written to the sheet with the status `Invalid Email - Skipped`. The `Email Validation Status` and `Validation Source` (`Local_Format_Check`) columns record the outcome, and an unresolved `unknown` result is surfaced as a warning in the Telegram approval message.

This layer is a syntax and length check. It does not verify that a mailbox exists, that an address is deliverable, or that a domain is non-disposable, and the documentation makes no such claim.

---

## AI Lead Analysis

| Aspect | Implementation |
|--------|----------------|
| Model | `models/gemini-flash-lite-latest`, through the n8n Google Gemini node (configured in the exported workflow) |
| Role in prompt | Enterprise B2B Lead Researcher and Cold Outreach Strategist |
| Input | A prospect block (name, email, company, company URL, research status), an evidence block built only from sources that were available, and the technographic JSON |
| Evidence rule | The prompt states that anything not in the evidence section is unknown, and tells the model not to invent facts, names, companies, or LinkedIn details |
| Facts and inference | The prompt asks the model to separate observed facts from inference and use cautious language for inference |
| Output | One raw JSON object, with no markdown and no commentary |

### Requested Output Fields

| Field | Purpose |
|-------|---------|
| `status` | Model-reported status string |
| `data_confidence_score` | 0 to 100 research and data confidence. It measures how well the evidence supports personalization and is explicitly not lead quality |
| `strategy_used` | `Hyper-Personalized` or `Value-First B2B Outreach Strategy` |
| `company_summary` | Short company description based on the evidence |
| `target_audience` | Who the company appears to serve |
| `identified_pain_point` | Possible pain point, phrased from the evidence |
| `evidence_used` | List of evidence snippets or sources the model reports using |
| `email_subject_options` | Exactly three options (curiosity, direct value, pain point) |
| `selected_subject` | Must copy one of the three options |
| `email_body` | Three paragraphs: hook, value pitch, soft call to action |
| `requires_human_flag` | Set when the company is unknown, evidence is thin or contradictory, or personalization is uncertain |

### How the AI Pipeline Controls Output

The model is instructed to analyze only the evidence the workflow supplies. The workflow's controls on that output are deterministic and procedural: the evidence-restricted prompt, the parsing and normalization code, the confidence gate, and mandatory human review in Telegram. `evidence_used` is the model's own report of what it relied on, and the workflow does not cross-check statements against external fact sources. The reviewer sees the full draft, the score, and any warnings before deciding.

### AI Output Validation

`Parse AI Output` is a code node that normalizes the model response. It does the following:

| Step | Behavior |
|------|----------|
| JSON extraction | Removes code fences, extracts the outermost `{ ... }` block, then runs `JSON.parse` |
| Subject normalization | Filters empty subject options. If `selected_subject` is not among the options, the first option is used |
| Score normalization | Converts to a number and clamps to 0 through 100. A non-numeric value becomes 0 |
| Strategy label | Recomputed in code from the score (80 or higher is `Hyper-Personalized`), regardless of the model's own label |
| Human-review flag | Set when the model flags it, the body is empty, the subject is empty, or the score is below 80 |
| Failure fallback | On any parse error the node returns `parse_status: failed`, score 0, the Value-First label, empty draft fields, the human-review flag set, and the error message and raw text in the item |

If the Gemini node itself errors, it uses `continueRegularOutput`. The resulting item contains no model text, so the parse step produces the same flagged failure result and the lead is routed to `Needs Review`.

---

## Confidence and Review Logic

The `Confidence Gate` node evaluates three conditions joined with AND:

| Condition | Required value |
|-----------|----------------|
| `data_confidence_score` | 80 or higher |
| `requires_human_flag` | Not `true` |
| `parse_status` | `success` |

| Gate result | Save node | Sheet `Status` |
|-------------|-----------|----------------|
| All conditions met | `Save Lead - Hyper-Personalized` | `Pending Review` |
| Any condition not met | `Save Lead - Needs Review` | `Needs Review` |

Both save nodes continue into the same approval token and Telegram request. The gate sets the review label and determines which warnings the reviewer sees. Authorization to send always comes from the human approval step.

`data_confidence_score` is a research and data confidence signal produced by the model under the prompt's rules. It indicates how well the collected evidence supports personalization. It is distinct from lead quality, conversion likelihood, or fit.

---

## Human Approval Workflow

| Step | Control |
|------|---------|
| 01 | AI analysis and draft are saved to Google Sheets |
| 02 | An approval token is generated and stored in workflow static data |
| 03 | A Telegram message is sent with the draft, key lead fields, and any warnings |
| 04 | The reviewer presses Approve and Send or Reject |
| 05 | The Telegram trigger receives the callback and `Resolve Approval Token` checks the token |
| 06 | A valid token is claimed immediately to block duplicate or concurrent clicks |
| 07 | The lead row is reloaded from Google Sheets by email |
| 08 | A lead already in a protected state is blocked with a Telegram notice |
| 09 | The decision is evaluated. Only the action `approve` proceeds toward a send |
| 10 | The final send safety gate checks required fields and status |
| 11 | Gmail sends the email |
| 12 | The token is marked used after a successful send, and the row is set to `Sent` |

**The Telegram message includes:** prospect name, company, email, confidence score and strategy label, email validation status, research status, the subject, the full email body, and a warnings block.

**Warnings generated by the workflow:**

| Warning | Shown when |
|---------|------------|
| Email validation unresolved | The validation result needs manual review |
| Research status | Research status is anything other than `full_research` |
| AI flagged | `requires_human_flag` is true |
| Parse failure | AI output could not be parsed |

### Approval Token Details

| Property | Implementation |
|----------|----------------|
| Storage | Workflow static data (`$getWorkflowStaticData('global')`) under `approvalTokens` |
| Format | 12-character-limited base-36 string generated from a lightweight non-cryptographic hash of email, timestamp, and `Math.random()` |
| Lifetime | 24 hours. Expired tokens are removed each time a new token is generated |
| Record fields | `email`, `createdAt`, `expiresAt`, `used`, `action`, `claimed`, `claimedAt` |
| Single use | `used` is set to true only when the requested action completes |
| Duplicate-click protection | A valid token is claimed as soon as it is resolved. A second callback inside the claim window is treated as invalid |
| Claim window | 2 minutes. A stale claim can be re-claimed by a new click |
| Reject path | The token is marked used, then the row is set to `Rejected` |
| Approve path | The token is marked used only after Gmail reports a successful send |
| Blocked or failed sends | The token is not marked used, so a later click after the claim window expires can retry |

The token mechanism is designed as an operational control against replays, expired links, and double clicks. The callback handler validates the token itself and does not verify which Telegram user or chat pressed the button, so access to the approval chat is the practical access boundary. The token generator is not cryptographic, and the documentation makes no cryptographic-strength claim.

### Outbound Email Safety

The `Final Send Safety Gate` requires all of the following before Gmail is called:

| Requirement | Detail |
|-------------|--------|
| `Email` | Not empty |
| `Subject` | Not empty |
| `Email Body` | Not empty |
| `Status` | None of `Sent`, `Replied`, `Do Not Contact`, `Opted Out`, `Rejected`, `Follow-up Sent` |

If the gate blocks, the row is updated to `Needs Review` with a `Last Error` message and no email is sent.

The implemented outbound safeguards work together:

- Every send requires a human decision in Telegram.
- The lead is reloaded from Google Sheets so the decision is made against current state.
- Protected statuses block both the initial send and the follow-up.
- Inbound opt-out language moves a lead to `Do Not Contact`, which blocks later sends.

These are send-state and consent-handling controls inside the workflow. The workflow does not automatically insert an unsubscribe footer or a sender identification block, and it does not claim to guarantee compliance with email regulations. Operators remain responsible for message content and applicable legal requirements in their jurisdiction.

---

## Outreach and Follow-up

| Item | Behavior |
|------|----------|
| ✉️ Initial email | Gmail node sends to the row's `Email` with the stored `Subject` and `Email Body` |
| Success test | `!$json.error && Boolean($json.id)` |
| On success | Token marked used, then the row is set to `Sent` with `Sent At` |
| On failure | Row set to `Send Failed` with the error message in `Last Error` |
| ⏳ Wait | Wait node, 3 days |
| Lead reload | Row fetched again from Google Sheets by email |
| Eligibility | Lead found, and `Status` is none of `Replied`, `Do Not Contact`, `Opted Out`, `Rejected`, `Follow-up Sent` |
| 🔁 Follow-up email | Gmail node using a fixed text template. Subject is `Re: Quick thought regarding <Company or "your company">` |
| Follow-up success | Row set to `Follow-up Sent` with `Follow-up Sent At` |
| Follow-up failure | Row set to `Send Failed` with the error message |
| Not eligible | The false output ends the run without a send |

The workflow sends one follow-up per lead. The follow-up text is a documented design choice: a consistent, reviewable template defined in the `Send Follow-up Email` node rather than a generated message. It is sent as a new Gmail message, and no thread reference is configured. The delayed eligibility check means a reply, opt-out, or rejection recorded during the three-day wait prevents the follow-up.

---

## Inbound Reply and Opt-Out Handling

The inbound branch runs alongside the outreach workflow and updates the same lead rows.

| Step | Behavior |
|------|----------|
| 1 | Gmail trigger polls every minute with the filter `is:unread` |
| 2 | `Extract Sender Email` reads the address from `from.value`, then falls back to `from.text`, then to `headers.from`. It never substitutes a default address |
| 3 | If no sender can be determined, a Telegram message is sent with the email subject |
| 4 | `Detect Opt-Out` tests the message snippet and text, case-insensitively, for opt-out phrases |
| 5a | Opt-out language found: the sheet is searched by sender email |
| 5b | No opt-out language: the sheet is searched by sender email as a reply candidate |
| 6a | Opt-out lead found: `Status` set to `Do Not Contact` |
| 6b | Reply lead found: `Status` set to `Replied` and `Reply Detected At` recorded |
| 7 | No matching row: a Telegram notification is sent for the unmatched opt-out or unmatched reply |

**Opt-out phrases matched:** `unsubscribe`, `stop emailing me`, `remove me` (with optional `from this list`), `opt out`, `opt-out`, `optout`, `do not contact me`, `do not email me`, and `take me off this list` or `take me off your list`.

Matching is performed on the sender's email address against the `Email` column. Opt-out detection uses the configured phrase list. Lookup nodes treat Google Sheets errors as not found, so an API failure is never mistaken for a matched lead, and unmatched senders are reported to the operator in Telegram without any sheet change. `Update Status - Replied` writes the reply status for a matched sender without a status guard. The branch does not perform email-thread matching or free-form natural-language intent analysis.

---

## Reliability and Safety Controls

| Control | Where it lives |
|---------|----------------|
| 🛡️ No invented inputs | `Normalize Lead Input` reports missing fields instead of guessing |
| 🧱 Evidence-restricted prompt | `AI Lead Analysis` prompt rules |
| 🧯 Safe parse fallback | `Parse AI Output` failure result |
| 👤 Human approval before send | Telegram buttons gate every Gmail send |
| ⏱️ Token expiry | 24-hour lifetime |
| 🔒 Single use and claim | `Resolve Approval Token` and both token-marking nodes |
| 🔍 State re-check | Lead is reloaded before the send and before the follow-up |
| 🚦 Final send gate | Required fields and protected statuses |
| 🔔 Operator alerts | Telegram messages for token, lookup, state, and inbound exceptions |
| 📝 Failure-state updates | `Send Failed`, `Needs Review` with `Last Error` recorded on the lead row |
| 🔌 Continue on error | 31 nodes use `continueRegularOutput` and one uses `continueErrorOutput`, so a failing step produces a defined fallback path |

The error-handling model is fallback-and-notify. A failing step continues along a defined output, the lead row records the resulting state, and Telegram alerts the operator where a decision or attention is needed. Automatic retry settings and a separate n8n error workflow are not part of the current configuration. Recovery is operator-driven: a failed initial send leaves its token unused so a later click can retry, and a lead in `Send Failed` or `Needs Review` stays visible in the sheet.

---

## Lead Status Lifecycle

| Status | Written by | Role |
|--------|-----------|------|
| `Missing Email` | `Log Missing Email` | Row had no email. Research is skipped |
| `Invalid Email - Skipped` | `Log Invalid Email` | Email failed the local format check |
| `Pending Review` | `Save Lead - Hyper-Personalized` | Passed the confidence gate. Awaiting Telegram approval |
| `Needs Review` | `Save Lead - Needs Review`, `Log Safety Gate Block` | Below threshold, flagged, parse failed, or send blocked by the final gate |
| `Rejected` | `Update Status - Rejected` | Reviewer pressed Reject (or any action other than approve) |
| `Sent` | `Update Status - Sent` | Gmail reported a successful initial send |
| `Send Failed` | `Update Status - Send Failed`, `Update Status - Follow-up Failed` | Initial or follow-up Gmail send did not succeed |
| `Follow-up Sent` | `Update Status - Follow-up Sent` | The single follow-up was sent |
| `Do Not Contact` | `Update Status - Do Not Contact` | Opt-out language detected from a matching sender |
| `Replied` | `Update Status - Replied` | Inbound email from a matching sender without opt-out language |

### Reserved and Protective Status Values

Two values are recognized by workflow logic without being written by a dedicated status node:

| Value | Role |
|-------|------|
| `Opted Out` | Recognized as a protected state in the final send gate, the lead-already-actioned check, and follow-up eligibility. It acts as a protective alias for opt-out handling, so a row marked this way is never sent to. The workflow's own opt-out path writes `Do Not Contact` |
| `Disposable Email - Skipped` | A branch in the `Log Invalid Email` expression that applies if a validation result ever carries a `disposable` status. The local format check produces only `valid`, `invalid`, and `unknown`, so current runs log `Invalid Email - Skipped` |

---

## Integrations

<p>
  <img src="https://img.shields.io/badge/n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white" alt="n8n" />
  <img src="https://img.shields.io/badge/Google_Sheets-34A853?style=flat-square&logo=googlesheets&logoColor=white" alt="Google Sheets" />
  <img src="https://img.shields.io/badge/Google_Gemini-8E75B2?style=flat-square&logo=googlegemini&logoColor=white" alt="Google Gemini" />
  <img src="https://img.shields.io/badge/Gmail-EA4335?style=flat-square&logo=gmail&logoColor=white" alt="Gmail" />
  <img src="https://img.shields.io/badge/Telegram-26A5E4?style=flat-square&logo=telegram&logoColor=white" alt="Telegram" />
  <img src="https://img.shields.io/badge/JavaScript-Code_Nodes-F7DF1E?style=flat-square&logo=javascript&logoColor=black" alt="JavaScript code nodes" />
</p>

The system is built on four connected services, each with a distinct responsibility:

| Service | Responsibility |
|---------|----------------|
| Google Sheets | Lead table and state store: intake trigger, draft storage, status updates, and lead lookups |
| Google Gemini | Evidence-guided lead analysis and personalized email drafting |
| Telegram | Human approval requests with inline buttons, callback handling, and operator alerts |
| Gmail | Initial email, follow-up, and the unread-message trigger for inbound handling |

| Integration | Used for | n8n credential type |
|-------------|----------|---------------------|
| Google Sheets (trigger) | Detect new lead rows | `googleSheetsTriggerOAuth2Api` |
| Google Sheets (nodes) | Save drafts, update statuses, look up leads | `googleSheetsOAuth2Api` |
| Google Gemini | Lead analysis and draft generation | `googlePalmApi` |
| Telegram | Approval requests, button callbacks, operator alerts | `telegramApi` |
| Gmail | Initial email, follow-up, and the inbound unread trigger | `gmailOAuth2` |
| HTTP and JavaScript | Public page fetches, parsing, token logic, normalization | Not applicable (one HTTP Request node and Code nodes) |

The workflow uses no email verification vendor, enrichment vendor, CRM, or external database.

---

## Data Structure and Google Sheets Model

Google Sheets is the lead table and the state store. The `Email` column is the match key for every write and lookup: save nodes use `appendOrUpdate` on `Email`, and the approval, follow-up, and inbound branches locate leads by email. The workflow reads current row state before it sends, so each action is evaluated against the latest `Status`. Each email should therefore appear once in the sheet.

| Field | Direction | Purpose |
|-------|-----------|---------|
| `Name` | Input | Prospect name |
| `Email` | Input and key | Prospect email, lowercased on intake |
| `Company` | Input | Company name |
| `Company_URL` | Input | Company website used for research |
| `LinkedIn_URL` | Input | Optional LinkedIn page for the fetch attempt |
| `Email Validation Status` | Workflow | `valid`, `invalid`, or `unknown` |
| `Validation Source` | Workflow | `Local_Format_Check` |
| `Research Status` | Workflow | `skipped_no_email`, `full_research`, `partial_research`, `research_failed`, or `company_url_missing` |
| `Status` | Workflow | Lifecycle state (see the status table) |
| `Strategy` | Workflow | `Hyper-Personalized` or `Value-First B2B Outreach Strategy` |
| `Subject` | Workflow | Selected email subject |
| `Email Body` | Workflow | Draft email text |
| `Confidence Score` | Workflow | Research and data confidence signal, 0 to 100 |
| `Sent At` | Workflow | Timestamp of the initial send |
| `Follow-up Sent At` | Workflow | Timestamp of the follow-up |
| `Reply Detected At` | Workflow | Timestamp of reply detection |
| `Last Error` | Workflow | Latest send or gate error message |

Rows with no email are logged under a placeholder value of the form `missing-email-row-<row number or timestamp>` in the `Email` column.

### Intake Row Handling

`Skip Already Processed Rows` is the intake checkpoint that follows the Google Sheets trigger. Its condition is constant-true, so it passes every triggered row into `Normalize Lead Input`. Because the trigger fires on `rowAdded`, rows are picked up when they are added, and the workflow's own writes use `appendOrUpdate` keyed on `Email`. Operators who want stricter reprocessing control can tighten this node's condition, for example against the `Status` column.

---

## Configuration and Required Credentials

Configuration is part of deployment and integration setup. The exported JSON is a portable definition with placeholders for the values that belong to your own environment.

| Setting | Where | Notes |
|---------|-------|-------|
| `REPLACE_WITH_YOUR_GOOGLE_SHEET_ID` | Google Sheets Trigger and every Google Sheets node | Setup placeholder. Select your own spreadsheet and sheet |
| `REPLACE_WITH_YOUR_TELEGRAM_CHAT_ID` | All seven Telegram send nodes | Setup placeholder for the reviewer chat |
| Google Sheets OAuth credential | Sheets nodes | Created in n8n |
| Google Sheets Trigger credential | Sheets trigger | Created in n8n |
| Gemini credential | `AI Lead Analysis` | Created in n8n |
| Telegram credential | Telegram nodes and trigger | Bot token managed in n8n |
| Gmail OAuth credential | Gmail trigger and both Gmail send nodes | Created in n8n |
| Sheet name cache | Sheets nodes | Nodes carry a cached display name from the original setup. Reselect the document and sheet in your instance |
| Follow-up text | `Send Follow-up Email` | Fixed subject and body template. Adjust it to your voice before use |

The exported JSON contains no credential IDs. Credential references are name-only and are reconnected in your n8n instance during import.

---

## Setup and Import Instructions

1. **Prepare the sheet.** Create a Google Sheet with the headers listed in the data model. The input columns are `Name`, `Email`, `Company`, `Company_URL`, and `LinkedIn_URL`. Add the workflow-managed columns so the update nodes can write to them.
2. **Import the workflow.** In n8n, import the workflow JSON file.
3. **Connect credentials.** Attach credentials for Google Sheets (node and trigger), Gemini, Telegram, and Gmail.
4. **Select the spreadsheet.** Update the Google Sheets Trigger and every Google Sheets node to your document and sheet.
5. **Set the Telegram destination.** Replace `REPLACE_WITH_YOUR_TELEGRAM_CHAT_ID` in all Telegram send nodes.
6. **Review the Gemini node.** Confirm the model selection and your account access to it.
7. **Review email content.** Check the follow-up template in `Send Follow-up Email` and the prompt in `AI Lead Analysis`.
8. **Validate with controlled data.** Use sheet rows you control and an inbox you own before using real prospects (see the validation checklist below).
9. **Activate last.** Activate the workflow after credentials and placeholders are verified. n8n documents that workflow static data (used for approval tokens) is saved for production executions and not for manual test runs, so exercise token behavior on the activated workflow.

---

## Failure Handling by Scenario

| Situation | Workflow behavior |
|-----------|-------------------|
| Missing email | Row logged as `Missing Email` with `Research Status` of `skipped_no_email`. Run ends |
| Invalid email format | Row logged as `Invalid Email - Skipped`. Run ends |
| Unresolved validation result | Treated as `unknown`, continues, and shows a warning in Telegram |
| Research source unavailable | Source marked unavailable with a reason. Remaining sources and the AI step still run |
| No research at all | Status `research_failed` or `company_url_missing`. The prompt states that no evidence was retrieved |
| AI parsing failure | Failure result with empty draft and review flag. Row saved as `Needs Review` |
| Gemini node error | Item continues without model text and falls into the parsing failure result |
| Low confidence or flagged output | Saved as `Needs Review`. Still requires Telegram approval |
| Invalid, unknown, or expired token | Telegram notice that no email was sent |
| Used token or duplicate click | Telegram notice. No second send |
| Sheets lookup error or lead not found | Treated as not found. Telegram alert says the click was not sent |
| Lead already actioned | Telegram notice naming the current status. No send |
| Safety gate block | Row set to `Needs Review` with a `Last Error` message. No send |
| Initial Gmail send failure | Row set to `Send Failed` with the error text. Token stays unused |
| Follow-up not eligible | Run ends with no send and no notification |
| Follow-up Gmail failure | Row set to `Send Failed` with the error text |
| Unidentified inbound sender | Telegram message with the subject |
| Unmatched opt-out or reply | Telegram notification with the sender address |

---

## Testing and Validation

### Completed Testing

The project is complete and has been tested. The tests executed produced positive results across the workflow's research, AI drafting, approval, sending, follow-up, and inbound-handling paths. This README does not publish test counts, coverage figures, performance numbers, or environment details.

### Validation Checklist for Another Deployment

When you deploy the workflow in your own environment, the scenarios below, derived from the workflow's branches, give you a repeatable way to confirm that your credentials, sheet, and configuration behave as designed.

| Scenario | Expected behavior |
|----------|-------------------|
| Valid lead with reachable site | Research completes, draft saved, Telegram request sent |
| Missing email | `Missing Email` row, no research |
| Invalid email format | `Invalid Email - Skipped`, no research |
| Partial research | `partial_research`, warning in Telegram |
| Missing company URL | `company_url_missing`, prompt notes no evidence |
| AI parsing failure | `Needs Review`, empty draft, parse warning |
| Low confidence lead | `Needs Review` label, approval still required |
| Approval accepted | Safety gate passes, Gmail sends, status `Sent` |
| Approval rejected | Status `Rejected`, no send |
| Expired token | Telegram notice, no send |
| Duplicate token click | Second click blocked by claim or used state |
| Initial email failure | `Send Failed` with `Last Error` |
| Eligible follow-up | After 3 days, one templated email, `Follow-up Sent` |
| Follow-up suppressed | Reply, opt-out, or rejection before day 3 prevents the send |
| Reply detected | Matching sender sets `Replied` and `Reply Detected At` |
| Opt-out detected | Matching sender sets `Do Not Contact` |
| Unmatched sender | Telegram notification, no sheet change |

---

## Repository Structure

| Path | Purpose |
|------|---------|
| `README.md` | Project documentation |
| `B2B_Lead_Research___Cold_Outreach_Automation_System.json` | Exported n8n workflow |
| `screenshots/workflow-overview.png` | Optional local copy of the workflow canvas screenshot. The image at the top of this README is loaded from its hosted URL |

---

## Security and Secret Handling

- Configure all credentials through n8n credential management. The exported JSON holds no credential IDs, tokens, or keys.
- Do not commit Google OAuth secrets, Gemini keys, Telegram bot tokens, or Gmail credentials.
- Replace `REPLACE_WITH_YOUR_GOOGLE_SHEET_ID` and `REPLACE_WITH_YOUR_TELEGRAM_CHAT_ID` locally in n8n, and avoid committing real values.
- Keep the Telegram approval chat private to authorized reviewers. Approval callbacks are validated by token, so the chat's membership is the access boundary for who can approve a send.
- Treat the sheet as sensitive. It holds prospect names, emails, and draft messages.
- Review any re-exported workflow JSON for credential identifiers and private URLs before publishing it.
- Review message content and local email regulations before using the workflow with real prospects. The workflow provides send-state and opt-out handling controls, and it does not add unsubscribe footers or guarantee legal compliance.

---

## Conclusion

This project is a complete, tested lead research and outreach pipeline built in n8n. Research evidence is collected source by source with explicit research-status classification, and Gemini is restricted to that evidence. A parsing and normalization step flags the model output, and a confidence gate sets the review label while keeping research confidence distinct from lead quality. Every send requires a Telegram approval backed by a 24-hour, single-use, claim-protected token, a lead-state re-check, and a final safety gate. Follow-up and inbound handling re-check lead state before acting, and opt-outs are recorded in the same sheet that drives sending. Fallback paths, status updates, and operator notifications give the system a coherent reliability model across Google Sheets, Gemini, Gmail, and Telegram.
