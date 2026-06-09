# Portable shopping-dive

The platform-neutral version of the shopping dive — same methodology as the Claude Code plugin (`../`), with
all Claude/MCP plumbing removed. It assumes only that **the agent can see the page you're shopping** (read a
live tab, or be shown the page content/screenshots). The funnel model, pattern library, journal schema,
report format, root-cause tags, and privacy rule are identical to the Claude version.

## Two artifacts here

| Artifact | For | What it is |
|---|---|---|
| **Foldered skill** (`SKILL.md` + `references/`) | Copilot Cowork, any agent that loads folders | A standard Agent Skill folder. Drop into your work OneDrive **Claude** folder for Cowork. |
| **`gemini-dive-skill.md`** | Gemini in Chrome | The four docs flattened into one self-contained prompt for Gemini's single-prompt Skill box. |

Both say the same thing — keep them in sync when you edit the methodology.

## Use in Copilot Cowork (Frontier)

Cowork supports custom **foldered skills** placed in the Claude folder in your work OneDrive, and (you
confirmed) its skills can observe live external sites — so a real ride-along works there.

1. Copy the whole foldered skill — `SKILL.md` **and** the `references/` folder — into your work OneDrive
   Claude skills folder (the location your other custom skills live in).
2. Let Cowork pick it up (its skill-create / refresh flow).
3. Start a dive: tell Cowork to run the shopping dive on a storefront, then shop. It writes the journal as a
   file (Cowork can create files) and produces the report at the end.

> Because Cowork is Claude-based, the foldered skill behaves like the native plugin minus the
> Claude-in-Chrome MCP — it uses Cowork's own browsing to observe pages.

## Use in Gemini in Chrome (Skills)

A Gemini Skill is one saved prompt, so use the flattened file:

1. Open `gemini-dive-skill.md`, copy its entire contents.
2. In the Gemini-in-Chrome sidebar, paste and run once on a storefront to confirm behavior.
3. From chat history, **Save as Skill** → name it `dive`, pick an emoji. (Manage at `chrome://skills/browse`.)
4. **Run it:** while shopping, type `/dive`; when prompted, **share the tab(s)** you're on (current + up to
   10). Re-invoke on each new page as you move through the funnel.
5. **Output:** Gemini can't write files — the journal lives in the conversation. Say "end dive" to get the
   full journal + report as markdown to copy and save.

## Use in any other browser agent

Paste `SKILL.md` (plus the references, or the flattened file) as a system/custom instruction, then shop and
let it observe each page.

## Sync note

This folder and the Claude plugin (`../skills/shopping-dive/`) intentionally share the same funnel model,
pattern names, journal fields, report sections, root-cause tags, and privacy rule. When you change one,
change the others so findings read the same everywhere.
