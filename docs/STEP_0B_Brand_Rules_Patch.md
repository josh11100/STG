# STEP 0B PATCH — LOAD SHARED BRAND RULES FROM GITHUB
# Insert into all 8 Project 022 agent routine prompts
# July 13, 2026 · bumps each agent one minor version
#
# WHERE IT GOES, PER AGENT (insert immediately AFTER the existing
# Step 0 / Step 0A URGENT-bypass check, BEFORE any content work):
#
#   RADAR_Agent_Routine_PROMPT    v1.0 → v1.1
#   WREN_Agent_Routine_PROMPT     v1.0 → v1.1
#   STORY_Agent_Routine_PROMPT    v1.1 → v1.2
#   TAILWIND_Agent_Routine_PROMPT v1.1 → v1.2
#   ANCHOR_Agent_Routine_PROMPT   v1.3 → v1.4
#   SARA_Agent_Routine_PROMPT     v1.0 → v1.1
#   EZRA_Agent_Routine_PROMPT     v1.1 → v1.2
#   SPARK_Agent_Routine_PROMPT    v1.0 → v1.1
#
# AFTER inserting the block, DELETE the now-duplicated inline brand
# rules from each prompt (product-moment table, banned words list,
# sign-off rules, "never prefix Skip The Groomer") and replace them
# with one line: "Brand rules: see fetched BRAND_RULES.md (Step 0B)."
# Keep agent-specific mechanics (URGENT formats, approval chains,
# connector steps) inline — only the shared brand rules move out.

───────────────────────────────────────────────────────────────

STEP 0B — LOAD SHARED BRAND RULES (runs every time, ~2 seconds)

Fetch the current brand rules from GitHub. The repo is public — no
token needed for reads:

    import requests
    url = ("https://raw.githubusercontent.com/josh11100/STG/"
           "main/docs/BRAND_RULES.md")
    resp = requests.get(url, timeout=15)
    if resp.status_code == 200:
        brand_rules = resp.text
    else:
        brand_rules = None

Treat the fetched contents as BINDING for everything produced this
run: product moments, banned words, sign-offs, voice rules, locked
numbers. Where the fetched file and this prompt state the same rule
differently, the FETCHED FILE wins — it is newer by definition.
Agent-specific mechanics in this prompt (schedules, URGENT formats,
approval routing, connector steps) are NOT brand rules and are never
overridden by the fetch.

FAILURE HANDLING — a fetch failure never blocks the run:
  - If status != 200 or the request times out: proceed using the
    brand rules summary built into this prompt, and add one line to
    the close-out post:
      "⚠️ BRAND_RULES.md fetch failed (HTTP <code>) — ran on
       built-in rules."
  - Do not retry in a loop. One attempt, then move on.
  - Never fabricate rules or skip the banned-words check because the
    fetch failed.

SANITY CHECK — before trusting the fetch:
  - The file must start with "# STG BRAND RULES".
  - It must contain a "BANNED WORDS" section.
  If either is missing, treat it as a failed fetch (the file may be
  mid-edit or corrupted) and fall back to built-in rules with the
  same close-out warning.

───────────────────────────────────────────────────────────────

# ROLLOUT ORDER (suggested):
#   1. Commit docs/BRAND_RULES.md to main
#   2. Patch ONE agent first (Story — it runs daily and its output is
#      easiest to eyeball). Watch one full run.
#   3. Confirm close-out shows no fetch warning and output respects
#      the rules. Then patch the remaining 7.
#   4. Log the change in #stg-decisions-log:
#      "All 022 agents now source brand rules from
#       docs/BRAND_RULES.md — one commit updates all eight."
