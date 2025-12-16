# n8n Workflow Exports

Purpose:
- Quick restore if I delete something
- Save workflow structure & logic
- Build a personal template library

Rules:
- Export workflows after meaningful changes
- Save exports in a folder named by date (YYYY-MM-DD)
- No secrets or credentials in this repo
- human-in-loop.json
  human-in-loop-v2.json   ← only if needed


**Local Folder:**

n8n-workflows/
├─ README.md
├─ active/
│  ├─ 2025-12-12/
│  │  ├─ human-in-loop.json
│  │  └─ slack-alert-test.json
│  ├─ 2025-12-20/
│  │  └─ youtube-description-generator.json
│
├─ archived/
│  ├─ 2024/
│  └─ 2025/
│
├─ templates/
│  ├─ webhook-starter.json
│  ├─ slack-notifier.json
│  └─ google-sheets-reader.json
│
└─ experiments/
   └─ 2025-12/
      ├─ random-idea-1.json
      └─ random-idea-2.json



### 1️⃣ **Dates live in folders, not filenames**

You only have to remember **one rule**:

> “Today’s exports go in today’s folder.”

You don’t have to rename files beyond:

```
human-in-loop.json
```

Your brain likes this.

------

### 2️⃣ **`active/` vs `archived/` is future-proof**

- `active/` = things you might re-import or show on camera
- `archived/` = old stuff you don’t want cluttering your view

Once a quarter (or never, honestly):

- drag old date folders into `archived/2025/`

No thinking required.

------

### 3️⃣ **`templates/` quietly becomes gold**

Any time you think:

> “I start every workflow the same way…”

Export it once and drop it here.

This becomes:

- teaching material
- reusable building blocks
- product fuel later

You don’t need to decide *now* what’s a template. Just toss things in when it feels right.

------

### 4️⃣ **`experiments/` removes guilt**

This folder gives you permission to:

- try weird ideas
- half-finish workflows
- never look at them again

No pressure to “do it right.”
 This is where creativity goes to make a mess without consequences.