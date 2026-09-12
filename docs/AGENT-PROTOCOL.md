# AGENT PROTOCOL — sasti agents se kaam, galti zero
**v2 · 12 Sep 2026 · SOURCE OF TRUTH = ye file** (Karan: "har chat mein chahiye")

> Naye repo mein: `docs/AGENT-PROTOCOL.md` copy karo + `CLAUDE.md` mein sirf
> `@docs/AGENT-PROTOCOL.md` likho (poora paste MAT karo). Kahin bhi badlo to
> version line badlo aur baaki repo mein file dobara copy karo. Iska summary
> kisi aur file mein MAT likho — do jagah likha = ek jagah purana.
> Mac pe global chahiye to `~/.claude/CLAUDE.md` mein bhi yahi file.

## 1. Kaun kya karega

| Kaam | Kaun |
|---|---|
| Sochna, product/paisa/security ka faisla, DB migration APPLY, RLS/RPC design, aakhri review, Karan se baat | **Main model (Fable)** — sirf yahi |
| Bada/tricky code (multi-file feature, logic-heavy fix, race/bug ki jaanch), migration SQL ka DRAFT + dry-run script (apply NAHI), REFUTER (risky diff mein galti dhoondna), design-level code review | **`Agent` tool, `model: 'opus'`** |
| Code patch (jab plan saaf ho), sweep/rename (N files ek jaisa patch), build, E2E chalana, log padhna, QA poll, screenshots lena+dekhna, i18n entries, docs notes, commit/push (main branch NAHI) | **`model: 'sonnet'`** |
| Bilkul mechanical **aur READ-ONLY**: grep/count, file list, log tail, do file ka diff, ek command chala ke RAW output laana | **`model: 'haiku'`** — **haiku se koi file likhwani nahi** |

**Tier chunne ka niyam:** sabse sasta jo kaam bina galti kar de. Shak ho to ek
tier upar (galti ki keemat spawn se zyada hai). Opus tab jab sonnet ko soch
lagani pade (kaunsa fix sahi, kahan race hai) — sirf "commands chalao" ho to
sonnet. Likhna hai to kam se kam sonnet; haiku sirf padhta/chalata hai.

Niyam: **main model khud koi build/E2E/screenshot nahi chalata** — subagent
bhejo, nateeja padho. Ek reply mein jitna ho sake **ek hi subagent** mein bundle
karo (har call ka overhead hai). Do independent kaam ho to do agents **ek saath**.
**"Independent" ka matlab:** alag files, alag port, alag test account, aur dono
mein se koi build/E2E/live-DB ko haath na lagaye. Ek bhi cheez share ho rahi hai
to ek ke baad ek — **E2E hamesha akela.**

## 2. Brief ka template (isse kam mein subagent MAT bhejo)

```
GOAL: <ek line — kya banna/verify hona hai>
CONTEXT: is repo ke AGENTS.md ke jo niyam lagte hain (bhasha/tone/design/naming/
         test accounts): <3-6 bullet, copy-paste> ; repo path; branch
FILES: <exact paths jo chhoone hain — inke alawa KUCH mat chhoona>
STEPS: 1) ... 2) ... 3) ...  (exact commands; env file sirf `set -a; . <file>; set +a` se)
DONE MEANS: <kaunse check green> — e.g. typecheck clean + lint 0 fail +
           `node scripts/e2e-<x>.mjs` ki aakhri line "PASS <x>"
RULES: §3 ke 9 HARD RULES (poore paste karo). Andaaza mat lagao; jo samajh na
       aaye, ruk ke REPORT karo. Jo maanga nahi woh mat karo.
REPORT: teen heading, isse alag nahi —
  WHAT CHANGED: exactly kya badla — `git diff --stat` + har file ka 2-line saar
  EVIDENCE: har check ka RAW output ki aakhri 15 line (PASS/FAIL lines poori),
            token/JWT/key/URL-fragment `[redacted]` karke
  NOT DONE: jo nahi hua / doubt / skip / aadha — saaf likho (chhupana nahi)
```

## 3. Subagent ke liye HARD RULES (brief mein hamesha paste)

1. **Saboot ke bina "done" nahi.** Har claim ke saath command ka asli output.
   "Typecheck clean hoga" nahi — chala ke dikhao.
2. **Scope ke bahar ek line bhi nahi.** Bug dikhe to REPORT karo, fix mat karo.
3. **Fail ho to 2 baar se zyada try nahi** — wajah likh ke lauto. Test ko
   "pass" karne ke liye test badalna MANA hai (jab tak brief mein na likha ho).
   **Lautne se pehle:** aadha kiya kaam ya poora revert karo ya REPORT mein
   EXACT likho "ye file/row aadhi hai"; apne chalu kiye server/process aur test
   data ka teardown chalao. Chupchaap kachra chhod ke jaana = agla run jhootha laal.
4. **Data ko haath nahi:** live DB pe **apne haath se** INSERT/UPDATE/DELETE
   (SQL/REST/MCP) NAHI, migration APPLY kabhi nahi, prod account kabhi nahi.
   Repo ki apni E2E/QA script chalana theek hai (woh sirf test accounts pe
   likhti hai aur apna teardown khud karti hai) — **script ke BAHAR ek bhi
   write nahi.** Test accounts sirf woh jo brief mein hain.
5. **Git:** sirf bataayi gayi branch pe commit; `main` (ya jo bhi live/default
   branch ho) pe push KABHI nahi; force-push KABHI nahi; commit se pehle
   `git status` — anjaan file stage nahi. **Kabhi bhi** `reset --hard` /
   `checkout -- .` / `clean` / `stash` / `rebase` / `branch -f` nahi —
   uncommitted kaam kisi aur ka bhi ho sakta hai. Repo ganda lage to REPORT
   karo, saaf mat karo.
6. **Server/process:** `pkill` nahi (apna shell mar jaata hai); naya server
   `setsid nohup ... &` se; E2E ek waqt pe EK.
7. **Secrets:** koi token/key/password na padho na chhaapo. **`.env*` kabhi
   `cat`/`grep`/print mat karo** — chahiye to sirf `set -a; . <file>; set +a`
   (values screen pe nahi aate). Kaunsi env file safe hai woh brief mein likhi
   hogi; na ho to ruk ke poochho. Output paste karne se pehle `eyJ…`,
   `sb_secret…`, `Bearer …`, URL `#access_token` → `[redacted]`; poori line
   secret ho to line chhod do aur "1 line redacted" likho. **Redact karna
   "raw" todna nahi hai — secret chhapna ye rule todna hai.**
8. **Repo-specific niyam** (bhasha/tone/design/naming/test accounts) us repo ke
   `AGENTS.md`/`CLAUDE.md` se brief ke CONTEXT mein aate hain — is file mein
   app-specific kuch nahi. Brief mein na ho to andaaza mat lagao, poochho.
9. **Report ka format fixed:** WHAT CHANGED / EVIDENCE / NOT DONE — teen heading,
   isse alag nahi. Kahani nahi, output.

## 4. Main model ka REVIEW (subagent ke baad, har baar — skip nahi)

1. `git diff` khud padho (sirf `--stat` nahi) — scope se bahar kuch badla? revert.
2. Evidence mein asli PASS lines hain? Count brief ke "DONE MEANS" se match?
   Ek check **independently dobara chalwao** — **FRESH sasta agent (haiku)**
   se: sirf command chalaye, RAW output laye, kuch badle nahi. Worker wale
   agent se dobara mat poochho. Main model sirf output padhta hai — subagent
   ke apne output pe andha bharosa nahi.
3. "NOT DONE" section khaali hai to shak karo — poochho "kya skip hua?".
4. Risky kaam (paisa, RLS, booking logic, auth) pe **doosra subagent (fresh
   context, `model: 'opus'`) sirf REFUTE karne ke liye**: "is diff mein galti
   dhoondo, default maano ki galti HAI; har claim ke saath file:line aur
   reproduce ka tareeka". Uske nateeje ke baad hi merge. Jo refute na ho
   paaye woh Karan ko batao, chhupao mat.
5. Karan ko sirf tab "ho gaya" bolo jab (1)-(4) ho chuke. Warna "aadha hua,
   ye baaki" — sach.

## 5. Kab subagent NAHI

- Ek-do line ka edit jo main model 10 second mein kar de (spawn overhead zyada).
- Faisla jo product/paisa/security badalta ho.
- Jab brief mein "DONE MEANS" likha hi na ja sake — pehle plan saaf karo.
- Jab §6 ki repo-lines bhari hi na ho — pehle woh bharo.

## 6. Pattern examples

**⚠️ Neeche ke commands SIRF fitness-booking (HattaKatta) ke hain.** Naye repo
mein pehle ye chaar line apne repo ke hisaab se bhar do (yahin, is file mein),
tab examples padho — bina bhare subagent bhejna hi mat:
```
BUILD = npm run build:cf            (fitness-booking)
SERVE = setsid nohup node scripts/serve.mjs dist 8788 &
E2E   = cd scripts && node e2e-<x>.mjs
ENV   = .env.production (sirf EXPO_PUBLIC_* values) — source karo, cat nahi
```
- **Build + E2E:** ek sonnet agent: cache saaf (`rm -rf /tmp/metro-* .expo
  node_modules/.cache`) → BUILD → SERVE → `set -a; . $ENV; set +a` → E2E →
  RAW PASS/FAIL lines lautao (redact ke saath).
- **Sweep (N files ek jaisa patch):** sonnet: regex + syntax-check har file +
  `git diff --stat` + 2 files ka diff sample. (Haiku nahi — likhna hai.)
- **Screenshot review:** sonnet screenshot le (Playwright), main model dekhe
  (design ka faisla main ka).
- **Risky diff:** worker (sonnet ya opus) → refuter (opus, fresh) → main faisla.
- **Naya feature (poora chakkar):** main design + brief → opus code likhe →
  sonnet build + E2E + screenshot → opus refute → haiku ek check dobara →
  main diff padhe → tab Karan ko "ho gaya".
- **Do independent kaam:** do agents EK message mein saath bhejo (parallel) —
  §1 wali "independent" ki paribhasha poori ho tabhi; warna ek ke baad ek.
