# Deploying to Render.com

## 1. Push this folder to a GitHub repo
```bash
cd deploy
git init
git add .
git commit -m "Prep for cloud deploy"
git branch -M main
git remote add origin https://github.com/<your-username>/call-analyzer.git
git push -u origin main
```

## 2. Create the Render service
1. Go to https://render.com → New → Web Service
2. Connect your GitHub repo
3. Settings:
   - **Runtime**: Python 3
   - **Build Command**: `pip install -r requirements.txt`
   - **Start Command**: (leave blank — Render reads the `Procfile` automatically)
4. Environment variables (Settings → Environment):
   - `WHISPER_MODEL` = `tiny` — **important**, see note below
   - `PYTHON_VERSION` = `3.11.9`

## 3. Two things that WILL bite you if skipped

### RAM — use the `tiny` model, and pick a paid plan
Render's free tier gives 512 MB RAM. The `small` model (the app's local
default) alone needs ~1 GB+ once loaded, so it will crash on free tier.
Set `WHISPER_MODEL=tiny` (fine for testing) and move to at least the
**Starter plan (512MB→2GB)** or higher for real usage. Accuracy trade-off:
tiny < base < small, but tiny still works reasonably for clear audio.

### Disk is NOT persistent by default
Render's filesystem resets on every redeploy/restart. That means every
recording in `data/uploads`, every result JSON, and every archived call
**disappears** whenever the service restarts (idles out after 15 min on
free tier, or you push a new deploy).

To fix this, add a **Render Disk** (Settings → Disks → Add Disk):
- Mount path: `/opt/render/project/src/data`
- This is a paid add-on (starts small, cheap) but is the only way results
  survive restarts on Render.

If you don't need cross-restart persistence for the MVP stage, skip this
for now and just know that results are temporary until you add it.

## 4. Test it
Once deployed, Render gives you a URL like:
```
https://call-analyzer-xxxx.onrender.com
```
Open it in a browser — you should see the same UI as localhost. Upload a
test recording and confirm it transcribes + classifies correctly. **Note:**
first request after an idle period can take 30-60s (free/starter tier
"cold start" — Render spins the container back up).

Next step: point the mobile app wrapper at this URL.
