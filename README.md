# TalkPat — legal pages

Three static pages, no build step. Fill in 11 placeholders, push to GitHub Pages, done.

```
index.html      privacy.html      terms.html      style.css
```

---

## 1. Fill in the placeholders

All in `[[DOUBLE_BRACKETS]]`, rendered highlighted yellow so nothing ships half-finished. List them with:

```bash
grep -on '\[\[[^]]*\]\]' *.html
```

### Straight swaps — do these with sed

```bash
sed -i \
  -e 's|\[\[LEGAL_NAME\]\]|Your Name|g' \
  -e 's|\[\[ADDRESS\]\]|ul. Example 1, 00-001 Warszawa|g' \
  -e 's|\[\[EMAIL\]\]|hello@talkpat.app|g' \
  -e 's|\[\[DATE\]\]|30 July 2026|g' \
  -e 's|\[\[COUNTRY\]\]|Poland|g' \
  -e 's|\[\[AUTHORITY\]\]|the Urząd Ochrony Danych Osobowych (uodo.gov.pl)|g' \
  *.html
```

On macOS use `sed -i ''`. **Check `COUNTRY` and `AUTHORITY` against where your entity is actually established**, not where you happen to live — the supervisory authority and governing law both follow the establishment. I've defaulted to Poland; if you're registered elsewhere, that's the swap to change.

`ADDRESS` is required: GDPR needs an identifiable controller, and most EU e-commerce rules expect a postal address. A registered address or a post box is fine.

### Four sentences you need to write yourself

These depend on how the app actually works, so guessing them would put a false statement in a published policy.

| Placeholder | Where | What to write |
|---|---|---|
| `[[STORAGE]]` | privacy opening, §02 | Where reflections live. `on your device only`, or `on your device, and in your iCloud backup if you have iCloud Backup switched on` if they're in the backup set |
| `[[DELETE_PATH]]` | privacy §08, §09 | Exact tap path to wipe everything, e.g. `Settings → Your data → Delete everything` |
| `[[EXPORT_PATH]]` | privacy §09 | The export path. If you have no export feature, see item 1 below |
| `[[ID_PATH]]` | privacy §09 | Where the RevenueCat app user ID is visible, e.g. `Settings → About → tap the version number` |
| `[[ZDR_NOTE]]` | privacy §05 | One sentence, only if you have Zero Data Retention: `We have Zero Data Retention enabled, so your messages are not logged or stored by OpenAI at all.` **If you don't have ZDR, delete the placeholder entirely** — leave the 30-day statement standing alone |

When you're done, delete the `.fill` rule from `style.css` so nothing renders highlighted.

---

## 2. Push to GitHub Pages

```bash
cd talkpat-legal
git init && git add . && git commit -m "Legal pages"
gh repo create talkpat-legal --public --source=. --push
```

Then **Settings → Pages → deploy from branch `main`, folder `/ (root)`**. Two minutes later:

- `https://<user>.github.io/talkpat-legal/`
- `https://<user>.github.io/talkpat-legal/privacy.html`
- `https://<user>.github.io/talkpat-legal/terms.html`

If you own the domain, add a `CNAME` file containing `legal.talkpat.app` and point a CNAME record at `<user>.github.io`. Better in App Store Connect, and it survives a host migration.

The URLs must load with no login wall, no redirect chain, no consent banner. That's why these are plain HTML rather than a Notion page — Notion and Google Docs links are the most common cause of rejection at this step.

**Where the URLs go:** App Store Connect → App Privacy → Privacy Policy URL, and App Information → License Agreement. Play Console → App content → Privacy policy, plus the store listing field. Plus a link in the app's own Settings, which Apple requires next to the price for auto-renewing subscriptions.

---

## 3. One technical thing to check today

**Set `store: false` on your API calls.** If you're on the Responses API, OpenAI stores response data server-side for 30 days by default as "application state" — that's separate from the abuse-monitoring window, and it's on unless you turn it off. Privacy §02 tells users you keep no server-side copy of their reflections, which stays true either way since it's OpenAI's storage not yours, but leaving `store: true` on means a conversation history is sitting in OpenAI's dashboard for a month for no benefit to you. Turn it off.

Related: without ZDR, prompt caching is applied to eligible requests. Since TalkPat re-sends an accumulating context block on every turn, you're a textbook caching case — good for your latency and bill, worth knowing about for your own records.

---

## 4. What the pivot bought you

Taking the child out of the equation removed most of the legal exposure these documents were previously carrying, and the new versions are written to claim that ground rather than just quietly stop mentioning children.

**Privacy §04 is now a feature, not a mitigation.** It states affirmatively that nothing is collected from the child, nothing comes back from them, nothing is assessed, and nothing profiles anyone. That's the clause a worried parent looks for, and it's also your answer to a store reviewer.

**Special-category data is now self-disclosure.** A parent writing "I snap because of my own anxiety" is volunteering health data about themselves — Art. 9 still applies, but consent from the person the data is about is clean, whereas the old version had you processing a child's health data on a guardian's say-so. Much better footing.

**The DPIA question has probably gone away.** The previous design — systematic accumulating records about identifiable minors — sat close to the pattern that triggers a mandatory assessment. An adult-only skills tool with no child subject, no profiling, and no automated decisions is a different risk class. Probably no DPIA. Still worth ten minutes of a lawyer's time to confirm rather than assuming, because if one is required, no privacy policy substitutes for it.

**The scoped assistant is now doing legal work.** Terms §02 lists what the model is designed to refuse, then §09 warns that scope limits can't be guaranteed and §07 makes jailbreaking a breach. That combination is much stronger than a generic "not professional advice" line — you're describing a real guardrail and being honest about its failure mode instead of claiming perfection.

**Worth adding in-app:** a way for users to report a reply that strayed outside scope. §02 invites them to tell you, and having somewhere to actually do that turns the promise into a process — which is exactly what you'd want to point at if anyone ever questioned it.

---

These are careful documents written for your architecture and your positioning, and the OpenAI retention and training statements in §05 match OpenAI's published enterprise terms as of today. But they aren't legal advice and I'm not a lawyer — for anything you'd hate to be wrong about, an hour with someone who does EU consumer and data protection work is money well spent.
