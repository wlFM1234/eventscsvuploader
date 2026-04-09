# Marketo Lead Loader

A serverless CSV → Marketo pipeline using GitHub Pages + GitHub Actions. No server required.

## How it works

```
Sales uploads CSV (GitHub Pages)
        ↓
File committed to repo via GitHub API
        ↓
process.yml triggers automatically
  • Validates emails, deduplicates, checks required fields
  • Prints approval instructions in the Actions log
        ↓
You review in Actions tab
        ↓
Run "Approve & upload to Marketo" workflow with the timestamp
        ↓
Leads pushed to Marketo REST API
Submission archived to submissions/processed/
```

---

## Setup

### 1. Create the repo

Create a **private** GitHub repo and push this code to it.

### 2. Enable GitHub Pages

- Go to **Settings → Pages**
- Source: **Deploy from a branch**
- Branch: `main`, folder: `/` (root)
- Your upload page will be at `https://YOUR_USERNAME.github.io/YOUR_REPO/`

### 3. Create a GitHub Personal Access Token (PAT)

The upload page needs to commit files to your repo directly from the browser.

- Go to **GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens**
- Create a token scoped to **this repo only** with:
  - **Contents**: Read and Write
  - **Actions**: Read and Write (to trigger workflows)
- Copy the token

### 4. Configure the upload page

Open `index.html` and replace the three config values near the top of the `<script>`:

```js
const GITHUB_OWNER = 'your-github-username';
const GITHUB_REPO  = 'your-repo-name';
const GITHUB_TOKEN = 'github_pat_xxxxxxxxxx';
```

> ⚠️ The PAT is visible in the page source. For internal use (sales team only) this is acceptable.
> For a public-facing page, move to a backend proxy or GitHub App auth.

### 5. Add Marketo secrets

Go to **Settings → Secrets and variables → Actions → New repository secret** and add:

| Secret name | Value |
|---|---|
| `MARKETO_BASE_URL` | `https://123-ABC-456.mktorest.com` |
| `MARKETO_CLIENT_ID` | Your Marketo API client ID |
| `MARKETO_CLIENT_SECRET` | Your Marketo API client secret |

To find these in Marketo: **Admin → Integration → LaunchPoint → New Service** (API Only).

### 6. Update the Marketo destination list

In `index.html`, update the `<select id="marketoDest">` options to match your actual Marketo static lists and programs.

---

## Approval workflow

When a CSV is submitted:

1. Go to the **Actions** tab in your repo
2. Open the **"Process lead submission"** run — check the log for row count, issues, and the timestamp
3. Go to **Actions → "Approve & upload to Marketo" → Run workflow**
4. Enter the **timestamp** shown in the process log
5. Optionally tick **dry run** to validate without pushing
6. Click **Run workflow**

---

## Marketo field mapping

The upload page maps these column names automatically:

| Marketo field | Recognised column names |
|---|---|
| `firstName` | First, First Name, fname, forename |
| `lastName` | Last, Last Name, lname, surname |
| `email` | Email, Email Address, e-mail |
| `company` | Company, Organisation, Organization, Account |
| `title` | Title, Job Title, Position, Role |
| `phone` | Phone, Telephone, Mobile |
| `country` | Country, Nation |
| `city` | City, Town |

Sales can also remap columns manually on the upload page before submitting.

---

## Folder structure

```
/
├── index.html                        ← Sales upload page (GitHub Pages)
├── .github/
│   └── workflows/
│       ├── process.yml               ← Triggers on CSV push, validates
│       └── approve.yml               ← Manual trigger, pushes to Marketo
└── submissions/
    ├── 1234567890_leads.csv          ← Cleaned CSV (auto-created)
    ├── 1234567890_meta.json          ← Sender info + destination
    ├── 1234567890_summary.json       ← Validation results
    └── processed/                    ← Archived after upload
```
