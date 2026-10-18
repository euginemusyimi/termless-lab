# Lab Task Debrief Template

> Fill this out immediately after finishing a task or lab session — while it's fresh, not days later. Takes 10–15 minutes. This is the raw material; the Lab Concept Map Index is what you build *from* these over time using the `#tags` in Section 8.

---

## 0. Task identifier

**Date:**
**Task/lab name:** (short, specific — e.g. "OPNsense Kea DHCP port-67 conflict" not "networking stuff")
**Related project:** (e.g. MSP Homelab / Module 4a / etc.)
**Time spent:**

---

## 1. What was I trying to do?

One or two sentences. The goal, not the process.

---

## 2. What did I actually do?

Step-by-step, concrete. Commands, tools, config screens touched. This is your build-log entry — write it like future-you, six months from now with no memory of this, needs to reconstruct exactly what happened.

```
[commands / config steps here]
```

---

## 3. What broke — on purpose or not?

**Broke on purpose? (Y/N):**

**What was the failure signature?** (exact error messages, symptoms, what a monitoring tool or log would have shown)

**How did I diagnose it?** (the actual troubleshooting path — dead ends included; dead ends are often the most useful part to remember)

**Root cause:**

**Fix:**

---

## 4. What would I do differently next time?

Faster diagnostic path, a check I should have run first, an assumption I shouldn't have made, a step I'd automate.

---

## 5. Real-world context

Find and link **one** real breach report, postmortem, CVE, vendor advisory, or documented incident where this same category of issue caused real damage or downtime. A sentence or two on how it connects.

**Link:**
**Connection:**

---

## 6. Explain it to a junior colleague

Write 3–5 sentences explaining this concept/fix as if onboarding someone with no context. If you can't do this cleanly, you don't understand it as well as you think — that's useful information, not a failure.

---

## 7. Variation test (do this on a later pass, not required same-day)

Change one variable from the original task (different topology, injected fault, different constraint) and note what happened. This is what separates "I did this once" from "I understand this."

**Variable changed:**
**Result:**

---

## 8. Tags

List every concept this task touched — these are what feed the Concept Map Index. Be generous; a task usually touches more concepts than it feels like at first glance.

`#tag1` `#tag2` `#tag3`

---

## After filling this out

1. Copy the tags from Section 8.
2. Open the Lab Concept Map Index.
3. For each tag, add/update a row in that concept's table:
   - Task/Lab name (from Section 0)
   - Tool used
   - What angle did this task add (pull from Section 2–4)
   - Broke on purpose? Y/N + result (from Section 3)
4. If any tag doesn't have a concept section yet, create one using the index's block template.
5. Every 5–10 debriefs, run the Coverage Check in the index.
