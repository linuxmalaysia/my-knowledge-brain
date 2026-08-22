# my-knowledge-brain Docs Repo

This repository (`linuxmalaysia/my-knowledge-brain`) is the **Mintlify docs deployment source** for [harisfazillah.mintlify.app](https://harisfazillah.mintlify.app).

> ⚠️ **Do not edit files in this repo directly.** This repo is auto-synced from the source repo. Manual changes here will be overwritten on the next sync.

## Where to edit docs

Edit docs in the source repo: [**linuxmalaysia/deep-state-of-mind-for-my-ai**](https://github.com/linuxmalaysia/deep-state-of-mind-for-my-ai) under the `docs-source/` folder.

On every push to `main` that touches `docs-source/`, a GitHub Actions workflow copies the contents into this repo and pushes to `main` here. Mintlify then rebuilds automatically.

```text
linuxmalaysia/deep-state-of-mind-for-my-ai (source repo)
  └── docs-source/                    ← edit here
       ├── *.mdx
       └── docs.json
  └── .github/workflows/sync-docs.yml
  └── scripts/sync_docs.py

     ↓ (on push to main, paths: docs-source/**)

linuxmalaysia/my-knowledge-brain (this repo, auto-synced)
  └── *.mdx
  └── docs.json

     ↓ (Mintlify GitHub app watches main)

https://harisfazillah.mintlify.app
```

## Setup checklist

### ✅ Done

- Mintlify deployment `harisfazillah` created
- Git source connected: `linuxmalaysia/my-knowledge-brain` on `main`
- Live site: [https://harisfazillah.mintlify.app](https://harisfazillah.mintlify.app)

### ⬜ To do (in source repo `linuxmalaysia/deep-state-of-mind-for-my-ai`)

1. **Create `docs-source/` folder** and copy the current contents of this repo into it (all `.mdx` files, `docs.json`, `logo/`, `favicon.svg`, images, etc.) as the starting point.
2. **Add `scripts/sync_docs.py`:**
   ```python
   import os
   import shutil
   import subprocess
   from pathlib import Path
   SOURCE_DIR = Path("docs-source")
   DOCS_REPO = "linuxmalaysia/my-knowledge-brain"
   BRANCH = "main"
   TOKEN = os.environ["DOCS_REPO_TOKEN"]
   def run(cmd, cwd=None):
       subprocess.run(cmd, cwd=cwd, check=True, shell=isinstance(cmd, str))
   def main():
       tmp = Path("/tmp/docs-repo")
       if tmp.exists():
           shutil.rmtree(tmp)
       # Clone docs repo
       url = f"https://x-access-token:{TOKEN}@github.com/{DOCS_REPO}.git"
       run(["git", "clone", "--branch", BRANCH, url, str(tmp)])
       # Wipe old docs content (keep .git)
       for item in tmp.iterdir():
           if item.name == ".git":
               continue
           if item.is_dir():
               shutil.rmtree(item)
           else:
               item.unlink()
       # Copy new content
       shutil.copytree(SOURCE_DIR, tmp, dirs_exist_ok=True)
       # Commit and push
       run(["git", "config", "user.email", "bot@example.com"], cwd=tmp)
       run(["git", "config", "user.name", "Docs Sync Bot"], cwd=tmp)
       run(["git", "add", "-A"], cwd=tmp)
       result = subprocess.run(["git", "diff", "--staged", "--quiet"], cwd=tmp)
       if result.returncode == 0:
           print("No changes")
           return
       run(["git", "commit", "-m", "Sync docs from source repo"], cwd=tmp)
       run(["git", "push", "origin", BRANCH], cwd=tmp)
       print("Synced")SOURCE_DIR = Path("docs-source")
   DOCS_REPO = "linuxmalaysia/my-knowledge-brain"
   BRANCH = "main"
   TOKEN = os.environ["DOCS_REPO_TOKEN"]
   def run(cmd, cwd=None):
       subprocess.run(cmd, cwd=cwd, check=True, shell=isinstance(cmd, str))
   def main():
       tmp = Path("/tmp/docs-repo")
       if tmp.exists():
           shutil.rmtree(tmp)
       # Clone docs repo
       url = f"https://x-access-token:{TOKEN}@github.com/{DOCS_REPO}.git"
       run(["git", "clone", "--branch", BRANCH, url, str(tmp)])
       # Wipe old docs content (keep .git)
       for item in tmp.iterdir():
           if item.name == ".git":
               continue
           if item.is_dir():
               shutil.rmtree(item)
           else:
               item.unlink()
       # Copy new content
       shutil.copytree(SOURCE_DIR, tmp, dirs_exist_ok=True)
       # Commit and push
       run(["git", "config", "user.email", "bot@example.com"], cwd=tmp)
       run(["git", "config", "user.name", "Docs Sync Bot"], cwd=tmp)
       run(["git", "add", "-A"], cwd=tmp)
       result = subprocess.run(["git", "diff", "--staged", "--quiet"], cwd=tmp)
       if result.returncode == 0:
           print("No changes")
           return
       run(["git", "commit", "-m", "Sync docs from source repo"], cwd=tmp)
       run(["git", "push", "origin", BRANCH], cwd=tmp)
       print("Synced")
   if __name__ == "__main__":
       main()
   ```
3. **Add `.github/workflows/sync-docs.yml`:**
   ```yaml
   name: Sync Docs
   on:
     push:
       branches: [main]
       paths:
         - "docs-source/**"
   jobs:
     sync:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - uses: actions/setup-python@v5
           with:
             python-version: "3.11"
         - name: Sync docs
           env:
             DOCS_REPO_TOKEN: ${{ secrets.DOCS_REPO_TOKEN }}
           run: python scripts/sync_docs.py
   ```
4. **Create a fine-grained Personal Access Token:**
   - Go to [https://github.com/settings/tokens](https://github.com/settings/tokens) → **Fine-grained tokens** → **Generate new token**
   - Repository access: `linuxmalaysia/my-knowledge-brain`
   - Permissions: **Contents: Read and write**
   - Copy the token
5. **Add token as a secret in the source repo:**
   - `linuxmalaysia/deep-state-of-mind-for-my-ai` → **Settings → Secrets and variables → Actions → New repository secret**
   - Name: `DOCS_REPO_TOKEN`
   - Value: paste token
6. **Test the pipeline:**
   - Make a small edit to a file under `docs-source/` in the source repo
   - Commit and push to `main`
   - Watch the workflow run in the **Actions** tab
   - Confirm this repo receives the sync commit
   - Confirm [https://harisfazillah.mintlify.app](https://harisfazillah.mintlify.app) rebuilds

## Rules

- ❌ Do NOT edit files in this repo (`my-knowledge-brain`) directly
- ❌ Do NOT edit in the Mintlify web editor (changes will be overwritten)
- ✅ Only edit `docs-source/` in `linuxmalaysia/deep-state-of-mind-for-my-ai`
- ✅ Push to `main` → auto-sync → Mintlify rebuild

## Local preview (in source repo)

```bash
cd docs-source
npm i -g mint
mint dev
```

Preview at [http://localhost:3000](http://localhost:3000).

## Resources

- [Mintlify documentation](https://mintlify.com/docs)
- [Live site](https://harisfazillah.mintlify.app)
- [Source repo (edit docs here)](https://github.com/linuxmalaysia/deep-state-of-mind-for-my-ai)