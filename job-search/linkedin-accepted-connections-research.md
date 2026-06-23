---
name: linkedin-accepted-connections-research
description: Use when user asks to check, research, or analyze LinkedIn connection accepts from Gmail (last 60 days or similar). Forces deep verification of current title + company for every person beyond email headlines, plus application status and open PM roles. Always output complete vetted table.
---

# LinkedIn accepted connections research skill

**Version:** 1.4  
**Last updated:** 2026-06-23  
**Changelog highlights:**
- v1.4: Changed required table format so "Accepted date" is the first column (removed "(desc)" from header while keeping descending sort order)
- v1.3: Strengthened Open PM Roles Phase — broadened keywords for senior/leadership titles + context-aware deeper research whenever there is a recent connection or known application history at the company (applies to any connection, not just recruiters)
- v1.2: Explicitly require official job title from Experience section (not LinkedIn headline/tagline) in the "Role" column + strengthened `browse_page` instructions
- v1.2: Enhanced Application Check Phase with mandatory name + company paired searches to catch prior application/interview history
- v1.1: Mandatory `browse_page` on LinkedIn profiles for verbatim current headline verification (primary source enforcement)
- v1.1: Improved Gmail extraction robustness + URL discovery fallback when email body/URL missing
- v1.1: Added lightweight job-change / timeline awareness check post-acceptance
- v1.1: Executive summary block before the required table for faster job-search actionability
- v1.1: Workflow efficiency note: batching, parallel calls, and company-level deduplication

**Core Rule:** Never stop at the short headline in the LinkedIn acceptance email. Always perform deep, multi-source verification of every person's current actual title and company using web searches and profile data. Be obsessive about accuracy. No laziness. No "N/A (individual)" unless truly verified as having no public company affiliation.

## Mandatory Process (Follow Exactly Every Time)

**Workflow Optimization Note:** Complete the full Gmail Extraction phase first and compile a deduplicated list of people and candidate companies. Then batch independent `web_search` / `browse_page` calls in parallel where possible. Perform company-level application status searches and open-PM-roles searches **once per unique company** and map results back to individuals. This reduces redundant tool calls significantly.

1. **Gmail Extraction Phase**
   - Use `gmail_search` with query: `from:invitations@linkedin.com (accepted OR acceptance) (invitation OR "connection request" OR connect) after:YYYY/MM/DD` (calculate 60 days back from current date, e.g. after:2026/04/24 for a June 23 run). Retrieve up to 50 results.
   - For every thread, use `gmail_get_message` on the message_id to extract:
     - Exact person name (from subject or body)
     - Clean LinkedIn vanity URL (from href in HTML if present, remove tracking params, convert /comm/in/ to /in/)
     - Accepted date
     - Raw headline from the email body (if body_text or body_html is non-empty and contains it)
   - **Robustness for missing email content or URL:** If gmail_get_message returns empty body_text and body_html, or if no clean vanity URL is present in the email, do NOT use placeholders or skip.
     - Parse the exact full name from the email subject line (pattern: "Jamie accepted your invitation..." or equivalent) and the from: field.
     - As a fallback, immediately run a targeted `web_search` for the exact name + `LinkedIn` (plus any location/company hints from context or mutuals) to discover the most likely profile URL.
     - Note the limitation explicitly in Research Notes (e.g., "LinkedIn URL not in email body; discovered via web_search fallback + browse_page verification").
     - Treat the Deep Company Research Phase (especially `browse_page` on the profile) as the primary/mandatory source for title and company.
   - Never output vague notes like "(Verified via profile; likely Director/ops-focused...)" or "(Role at Company verified via profile)" — always state the specific verified title verbatim.

2. **Deep Company Research Phase (Non-Negotiable)**
   - For **every single person**, run targeted web_search queries such as:
     - `"Full Name" LinkedIn current role OR company OR "Managing Partner" OR "Director" OR CEO OR founder`
     - `"Full Name" "at CompanyName"` 
     - If name is common/ambiguous (e.g., Jamie Frame, multiple hits): Immediately run 2-3 disambiguating queries e.g. `"Full Name" "Ramsey Solutions" OR "Director of Acquisition Strategy"`, `"Full Name" LinkedIn (Franklin OR TN OR Tennessee)`, or name + recent activity keywords. Cross-reference with any email context (location hints, mutuals).
   - **Mandatory LinkedIn Profile Verification (Primary Source):** If a clean LinkedIn vanity URL is available (from email or the fallback web_search above), **always** call `browse_page` on `https://www.linkedin.com/in/vanityurl` with these exact instructions:  
     "Extract the person's CURRENT official job title from the top/current entry in the Experience section (the formal position title, e.g. 'Director of Acquisition Strategy & Recruiting Operations' — NOT the headline or summary tagline at the very top of the profile). Also extract the current company name, location, and the first 1–2 sentences of the About section if present. If the profile requires login or shows limited info, note that explicitly. Do not interpret or summarize — quote directly from the page content."
   - Cross-reference multiple results. Prioritize recent LinkedIn profile data, company pages, and posts. The `browse_page` result (when successful) is authoritative for the current title/company.
   - **Job Change / Timeline Awareness:** After confirming current title/company, run one quick targeted search for evidence of recent moves: `"Full Name" (joined OR started OR promoted OR "new role at" OR "moved to" OR left) since:2026-01-01`. Note in Research Notes if the current role differs from what they likely held around the acceptance date.
   - If the person is at a known company (Google, Amazon, Apollo, Rebrandly, WorkOS, PostHog, Devolution Capital, etc.), confirm it explicitly.
   - Only mark as "Individual professional (verified)" after confirming no prominent current company affiliation exists in public sources.

3. **Application Check Phase (Per-Company, Non-Negotiable)**
   - After identifying companies from the Deep Research Phase, for **each company**, perform dedicated gmail_search queries using the company name combined with application/interview keywords, e.g.:
     - `CompanyName (application OR applied OR interview OR recruiter OR "video interview" OR "thank you for interviewing" OR "we have decided" OR "move forward with other candidates" OR rejection OR status OR "not moving forward" OR "Executive Director of Product")`
   - **Mandatory name + company paired search:** In addition to the above, always run at least one targeted search combining the specific person's name with the company (e.g., `"Jamie Frame" (Ramsey OR "Ramsey Solutions") (application OR applied OR interview OR "not moving forward" OR rejection OR "other candidates")`). This catches prior application/interview history that company-level searches may miss.
   - Also search using any recruiter name identified in the thread (e.g., the LinkedIn acceptor if they are internal).
   - Broaden beyond just LinkedIn "your application was sent" notifications — include inbound recruiter outreach, interview scheduling/confirmation, and decision/follow-up emails.
   - If found, extract: exact role applied for/interviewed for, key dates (outreach, interview, decision), and final status/outcome (e.g., "Interviewed [date]; Decided not to move forward [date and reason if stated]").
   - Review the full thread for context and accurate status.
   - When the connection is at a company where the user has known prior engagement, perform extra targeted name-based searches and explicitly note any history in Research Notes.

4. **Open PM Roles Phase**
   - For every company that has a real employer, run web_search for current open Product Management roles. Use a broad set of keywords to catch both IC and leadership roles:
     - `CompanyName ("Product Manager" OR "Head of Product" OR "VP Product" OR "Director of Product" OR "Executive Director of Product" OR "Chief Product Officer" OR "Principal Product Manager" OR "Staff Product Manager" OR "Product Leadership") (jobs OR hiring OR opening OR role OR "we're hiring")`
   - **Context-aware deeper research (applies to any connection):** If there is a recent LinkedIn connection at the company **or** known prior application/interview history, perform additional targeted research for open roles at that specific company. This includes searching for roles mentioned in the email thread (if any) and checking the company’s careers page more proactively.
   - Check company careers page if a link appears in results.
   - Record any open roles + approximate post date if available. If none, state "None found". Be specific about level and focus area when possible (e.g., "Executive Director of Product – EveryDollar").

5. **Output Rules**
   - **Always prepend this executive summary block** before the table (customize counts and highlights from your research):
     ```
     ## LinkedIn Connection Accept Research — Last ~60 days (as of YYYY-MM-DD)
     - Total accepts researched: X
     - Unique companies/employers: Y
     - Companies with open PM roles right now: Z (list standout ones, e.g. CompanyX — Head of Product)
     - Connections at companies with prior application/interview history: W
     - Highest-potential warm outreach targets: [brief note or "see flagged items in Research Notes"]
     ```
   - Always output in the exact markdown table format requested by the user:
     `Accepted date | Name | Role | Company | LinkedIn profile link | Applied? | Applied date (if applied) | Application status/result | Open product role(s) | Open product role(s) date posted`
   - Sort the table by Accepted date in descending order (most recent first).
   - **"Role" column rule (strict):** The Role must be the person's official current or most recent job title from the Experience section (e.g., "Director of Acquisition Strategy & Recruiting Operations at Ramsey Solutions"). Do **not** use the LinkedIn headline, profile summary, or tagline as a substitute. If the person is truly fractional/independent with no formal title, state it clearly (e.g., "Founder & Fractional CPO (independent)").
   - Use proper markdown hyperlink syntax for the LinkedIn profile link column: `[Full Name](https://www.linkedin.com/in/vanityurl)`
   - Every row must be fully populated with verified information. No placeholders.
   - At the end, include a "Research Notes" section. For every non-obvious verification (common names, missing email content, etc.), list the specific search queries used and/or the `browse_page` result that provided the definitive title/company. Also include a dedicated note for any detected or user-known prior application/interview history with that company/connection. Structure as a bulleted list or clear per-person entries.

## Anti-Laziness Safeguards
- If you feel tempted to use the email headline as the final company, stop and do another search.
- If results are ambiguous for a person (common names, multiple profiles), run 2-3 different targeted/disambiguating search queries before deciding. NEVER default to "likely" or parenthetical guesses.
- Prefer primary sources (LinkedIn profile snippets, company announcements, exact headline quotes) over secondary.
- When in doubt about a company or title, explicitly state the verification method used (e.g., "Exact headline from LinkedIn profile snippet via web_search 'Jamie Frame Director of Acquisition Strategy'").
- **No placeholders ever:** Every row in the output table must contain a specific, verified job title (e.g., "Director of Acquisition Strategy & Recruiting Operations at Ramsey Solutions"). The "Role" column must use the official title from the Experience section — never the LinkedIn headline or tagline. If email headline is unavailable, the web_search / browse_page results become the authoritative source — extract and use the exact title verbatim from LinkedIn snippets or equivalent. Update the table only after full verification; never leave vague notes like "(Verified via profile; likely...)" or "(Role at ...)" or similar. Always list the precise queries and sources in Research Notes for edge cases.

This skill exists because previous attempts repeatedly stopped at surface-level data. Use it to deliver complete, correct results on the first try.
