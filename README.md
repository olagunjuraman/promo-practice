# Release Promotion Practice (no real infra)

This repo simulates the "build once, promote the same artifact" flow using
GitHub Actions Environments + required reviewers. Nothing deploys anywhere real —
every "deploy" is just an echo statement. The point is to feel the mechanic:
build runs once, then each environment hop is a manual approval click, not a PR.

## Setup (5 min)

1. Create a new empty repo on GitHub (e.g. `promo-practice`), no README/gitignore.
2. Unzip this folder locally, then:
   ```bash
   cd promo-practice
   git init
   git add .
   git commit -m "initial commit"
   git branch -M main
   git remote add origin https://github.com/YOUR_USERNAME/promo-practice.git
   git push -u origin main
   ```
   This first push will trigger the workflow but `int`/`uat`/`production`
   environments don't exist yet, so GitHub will just create them on first use
   with no protection rules (build + deploy-int + promote-uat + promote-prod
   will all run straight through, no gates). That's fine for run #1.

3. Now add the gates so run #2 behaves like a real promotion flow:
   - Go to repo **Settings → Environments**
   - Click **uat** (it now exists because the workflow referenced it)
     - Enable **Required reviewers**, add yourself
   - Click **production**
     - Enable **Required reviewers**, add yourself
   - Leave **int** with no protection rules (it should auto-run)

4. Trigger a new run — edit `app.txt`, commit, push to `main`:
   ```bash
   echo "version: v2" >> app.txt
   git add app.txt
   git commit -m "bump version"
   git push
   ```

5. Go to the **Actions** tab in GitHub, open the running workflow. Watch:
   - `build` runs immediately
   - `deploy-int` runs immediately after (no gate)
   - `promote-uat` shows **"Waiting"** — click into it, hit **"Review deployments"**,
     check `uat`, click **Approve and deploy**
   - `promote-prod` then shows **"Waiting"** — same thing, approve it separately

That pause-and-click is exactly the mechanic we talked about: no PR, no merge,
just a human approving that a specific already-built artifact is allowed into
that environment.

## Things to try after it clicks
- Push again and watch a NEW artifact_tag get generated - notice it's a fresh
  build, but the same job structure (build once, promote same tag through the 3 gates).
- Try rejecting a deployment instead of approving it - see what the Actions tab shows.
- Add a second reviewer requirement and see how approval-from-anyone vs
  approval-from-everyone changes things.
