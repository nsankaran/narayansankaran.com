# Building narayansankaran.com with an al-folio Personal Site + Minimal Mistakes Lab Page

**Goal:** `narayansankaran.com` runs on the **al-folio** Jekyll theme, and `narayansankaran.com/sankaran-lab` runs on the **Minimal Mistakes** Jekyll theme, both served from one repo, one GitHub Pages deployment, one custom domain. A "Lab" tab in your al-folio nav links to the lab section.

This works by building the two Jekyll sites **independently**, then combining their output folders before deploying — rather than trying to merge the themes themselves.

---

## Step 0: Prerequisites

- GitHub account
- Git and Ruby/Jekyll installed locally (recommended, for previewing before you push) — see https://jekyllrb.com/docs/installation/
- A domain registrar account if you haven't bought `narayansankaran.com` yet

---

## Step 1: Create the repository and folder structure

### 1a. Create the repo on GitHub

1. Go to https://github.com and log in.
2. Click the **"+"** icon in the top-right corner → **"New repository"**.
3. Fill in the form:
   - **Repository name:** `narayansankaran.com` (name is just for your own clarity — it doesn't need to match the domain, since we're using a custom domain either way)
   - **Description:** optional, e.g. "Personal site + lab site"
   - **Visibility:** Public (GitHub Pages requires a public repo, unless you're on a paid plan that supports private Pages sites)
   - **Do NOT** check "Add a README," "Add .gitignore," or "Choose a license" — leave the repo completely empty. This avoids merge conflicts when you push the al-folio template contents in Step 2.
4. Click **"Create repository"**. GitHub will show you a page with setup instructions and a URL like `https://github.com/YOUR-USERNAME/narayansankaran.com.git` — copy that.

### 1b. Clone it locally

Open a terminal, navigate to wherever you keep your projects, and run:

```bash
git clone https://github.com/YOUR-USERNAME/narayansankaran.com.git
cd narayansankaran.com
```

(Replace `YOUR-USERNAME` with your actual GitHub username.) This creates a `narayansankaran.com` folder on your machine, connected to the empty GitHub repo, and moves you into it. Since the repo is empty, `ls -la` at this point should show only a `.git` folder.

### 1c. Set up the folder structure

You're aiming for this layout by the end of Steps 2–4:

```
narayansankaran.com/
├── _config.yml          <- al-folio config (lives at repo root)
├── (all other al-folio files/folders)
├── lab/
│   ├── _config.yml      <- Minimal Mistakes config
│   ├── Gemfile
│   └── (all other Minimal Mistakes files/folders)
└── .github/
    └── workflows/
        └── deploy.yml
```

Right now, just create the empty scaffolding so you have somewhere to put things in the next steps:

```bash
mkdir lab
mkdir -p .github/workflows
```

At this point your repo folder contains only `lab/` and `.github/workflows/`, both empty — that's expected. You'll populate:
- The **repo root** with al-folio's files in Step 2 (this is where al-folio's own template contents get copied in — directly into `narayansankaran.com/`, not into a further subfolder)
- **`lab/`** with the Minimal Mistakes starter files in Step 3
- **`.github/workflows/deploy.yml`** with the build workflow in Step 4

A quick sanity check before moving on — confirm you're set up correctly:
```bash
git remote -v
```
This should print your GitHub repo URL twice (for fetch and push). If it doesn't, double check you cloned the right repo in step 1b.

---

## Step 2: Set up the al-folio site at the repo root

### 2a. Pull in the al-folio template

Clone al-folio into a temporary folder (not directly into your project — this keeps its `.git` history separate so it doesn't conflict with yours):

```bash
git clone https://github.com/alshedivat/al-folio.git /tmp/al-folio-template
```

Now copy its contents into your repo root, **excluding** the `.git` folder (you want al-folio's files, not its git history):

```bash
rsync -av --exclude='.git' /tmp/al-folio-template/ ./
```

You're still inside `narayansankaran.com/` at this point — `./` refers to your repo root. `rsync` will copy everything over, merging with the `lab/` and `.github/workflows/` folders you already created.

Clean up the temp clone once the copy is done:
```bash
rm -rf /tmp/al-folio-template
```

*(No `rsync` on your system? On Mac it's preinstalled; on most Linux distros too. On Windows/WSL you can install it with `sudo apt install rsync`, or substitute `cp -r /tmp/al-folio-template/* /tmp/al-folio-template/.[!.]* ./` as a rougher equivalent.)*

### 2b. Remove al-folio's own deploy workflow

al-folio ships with its own `.github/workflows/deploy.yml` for deploying *itself* as a standalone site. Since we're writing a custom combined-build workflow in Step 4, delete al-folio's version now so the two don't conflict:

```bash
rm -f .github/workflows/deploy.yml
```

(Leave the `.github/workflows/` folder itself in place — you'll add your own `deploy.yml` there in Step 4.) It's also worth a quick look through `.github/workflows/` for any other al-folio CI files (e.g. a `ci.yml` for automated checks) and removing ones you don't need.

### 2c. Install dependencies and do a first pass on `_config.yml`

```bash
bundle install
```

If this fails on a Ruby version mismatch, al-folio's README will specify the Ruby version it expects — install that version (e.g. via `rbenv` or `rvm`) and retry.

Open the root `_config.yml` and fill in the basics for now (you'll set `url` properly in Step 7 once the domain is ready — a placeholder like `https://narayansankaran.com` is fine to put in already):
- `first_name`, `last_name`
- `email`
- `description`
- `avatar` (path to your headshot, added under `assets/img/`)

Leave deeper customization (CV, publications, projects) for after you've confirmed the site builds — don't block on perfecting content yet.

### 2d. Test it locally

```bash
bundle exec jekyll serve
```

Visit `http://localhost:4000`. You should see the al-folio homepage. Stop the server with `Ctrl+C` once confirmed.

**Use Claude here** to help you draft your bio, research statement, and publication entries in al-folio's expected YAML/Markdown formats — paste in al-folio's `_data/cv.yml` or `_bibliography/papers.bib` structure and ask Claude to fill it in from your CV.

---

## Step 3: Set up the Minimal Mistakes lab site in `/lab`

### 3a. Pull in the Minimal Mistakes starter

Same pattern as Step 2 — clone into a temp folder, then copy across, this time into `lab/`:

```bash
git clone https://github.com/mmistakes/mm-github-pages-starter.git /tmp/mm-starter
rsync -av --exclude='.git' /tmp/mm-starter/ ./lab/
rm -rf /tmp/mm-starter
```

Your `lab/` folder should now contain `_config.yml`, `Gemfile`, `index.md`, `_data/navigation.yml`, and the rest of the starter's structure.

### 3b. Remove the starter's own deploy workflow

The Minimal Mistakes starter also ships a `.github/workflows/` folder of its own (for deploying itself standalone). Since `rsync` just copied it in, remove it — you don't want two competing workflow files:

```bash
rm -rf lab/.github
```

Your single combined workflow at the repo root (`.github/workflows/deploy.yml`, written in Step 4) handles building *both* sites, so `lab/` doesn't need its own.

### 3c. Install dependencies

```bash
cd lab
bundle install
```

If you hit a Ruby/gem version conflict with the al-folio install from Step 2c, that's expected and fine — each site has its own `Gemfile` and gets its own `bundle install`, run from its own folder. They don't need to share a Ruby environment locally (the GitHub Actions workflow handles this cleanly too, since each build step `cd`s into the right folder).

### 3d. Set the baseurl and basic config

While still in `lab/`, open `_config.yml` and set:
```yaml
url: "https://narayansankaran.com"
baseurl: "/sankaran-lab"
```
This tells Jekyll that every internal link on the lab site should be prefixed with `/sankaran-lab`, so links resolve correctly once nested under your main domain. Fill in `title`, `description`, and the rest of the basics too.

### 3e. Test it locally

From inside `lab/`:
```bash
bundle exec jekyll serve --baseurl ""
```
(The `--baseurl ""` override lets you preview at `http://localhost:4000` directly without needing the `/sankaran-lab` prefix locally. Omit the flag if you specifically want to test the prefix behavior.)

Confirm the Minimal Mistakes homepage loads, then stop the server with `Ctrl+C`.

### 3f. Fill in the rest of the lab site content

Complete `lab/_config.yml`, `lab/_data/navigation.yml`, and content pages (Research, People, Publications, Code of Conduct) exactly as described in the earlier guide — same process, just nested in this subfolder. Use Claude the same way: paste each file and describe your lab to get drafted content.

At this point, commit your progress before moving on:
```bash
cd ..
git add .
git commit -m "Add al-folio personal site and Minimal Mistakes lab site"
git push origin main
```

---

## Step 4: Write the GitHub Actions workflow to build and combine both sites

Create `.github/workflows/deploy.yml`:

```yaml
name: Build and Deploy Combined Site

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      # --- Build al-folio (personal site) ---
      - name: Set up Ruby (personal site)
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.1'
          bundler-cache: true

      - name: Build personal site
        run: |
          bundle install
          bundle exec jekyll build --destination _site

      # --- Build Minimal Mistakes (lab site) ---
      - name: Set up Ruby (lab site)
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.1'
          bundler-cache: true
          working-directory: lab

      - name: Build lab site
        working-directory: lab
        run: |
          bundle install
          bundle exec jekyll build --destination ../_site/sankaran-lab

      # --- Deploy combined output ---
      - name: Upload combined site
        uses: actions/upload-pages-artifact@v3
        with:
          path: _site

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

What this does: builds al-folio into `_site/`, builds the lab site straight into `_site/sankaran-lab/`, then uploads and deploys the combined folder as one GitHub Pages site.

---

## Step 5: Point GitHub Pages at the Actions workflow

1. In your repo: **Settings → Pages**
2. Under "Build and deployment," set **Source** to **"GitHub Actions"** (not "Deploy from a branch" — that setting is for the old single-Jekyll-build method and won't run our custom workflow).
3. Push your changes:
   ```bash
   git add .
   git commit -m "Set up combined al-folio + lab site build"
   git push origin main
   ```
4. Check the **Actions** tab in your repo to watch the build run. Once it succeeds, your site is live at the GitHub-provided URL — check it before moving to the custom domain.

---

## Step 6: Add the "Lab" tab to your al-folio navigation

Find where al-folio defines its nav (typically `_config.yml` has a `nav:` / menu section, or `_pages/` contains nav-linked pages — check al-folio's docs, since this varies slightly by version). Add an entry linking to `/sankaran-lab/`.

**Use Claude here**: paste your al-folio nav config and ask it to add a "Lab" entry pointing to `/sankaran-lab/`.

---

## Step 7: Set up the custom domain

1. **Buy `narayansankaran.com`** through a registrar if you haven't (roughly $10–15/year for `.com`; Cloudflare Registrar sells at cost).
2. In your repo root, add a file named `CNAME` (no extension) containing:
   ```
   narayansankaran.com
   ```
   (Only one `CNAME` file, at the repo root — not inside `lab/`. It applies to the whole combined site.)
3. At your domain registrar, set DNS records:
   - Four **A records** for the apex domain pointing to:
     ```
     185.199.108.153
     185.199.109.153
     185.199.110.153
     185.199.111.153
     ```
   - A **CNAME record** for `www` pointing to `your-username.github.io`
4. In **Settings → Pages**, enter `narayansankaran.com` as the custom domain, wait for DNS to verify, then check **"Enforce HTTPS"**.
5. Update `url:` in your **root** `_config.yml` to `https://narayansankaran.com`, and confirm `lab/_config.yml` still has `url: "https://narayansankaran.com"` + `baseurl: "/sankaran-lab"`.

DNS propagation can take anywhere from minutes to ~24 hours.

---

## Step 8: Verify everything

Once DNS resolves:
- `https://narayansankaran.com` → al-folio personal site
- `https://narayansankaran.com/sankaran-lab/` → Minimal Mistakes lab site
- Click the "Lab" tab from your homepage nav and confirm it lands correctly
- Click around the lab site's internal links (Research, People, Publications) to confirm the `/sankaran-lab` prefix is applied consistently

---

## Ongoing maintenance

- **Personal site edits** (CV, publications, bio) happen in the repo root, following al-folio's normal workflow.
- **Lab site edits** (new lab members, new papers, research description updates) happen inside `lab/`, following the Minimal Mistakes structure from the earlier guide.
- Every push to `main` re-triggers the GitHub Actions workflow and rebuilds both sites automatically — no manual rebuild step needed.
- For ongoing content help, keep bringing specific files to Claude (e.g. "here's my `lab/_data/publications.yml`, help me add this new paper") rather than re-explaining the whole setup each time.

---

## Where Claude Code fits in

This setup — two nested Jekyll projects, a custom GitHub Actions workflow, `baseurl` configuration — is exactly the kind of thing worth handing to **Claude Code** directly rather than copy-pasting snippets by hand:

- Point it at your cloned repo
- Ask it to scaffold the folder structure in Step 1, pull in both themes, and write the Actions workflow in Step 4
- Ask it to fix any build errors that come up when you first run the workflow (mismatched Ruby/gem versions between the two sites are the most common issue)
- Ask it to wire up the nav link in Step 6 once it can see al-folio's actual file structure
