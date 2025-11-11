# Static Files Troubleshooting for Render

If static files (CSS, images, JS) are not loading on Render, follow these steps:

## 1. **Check Build Logs on Render**
   - Go to your Render dashboard → your service
   - Click "Logs" and look for the build phase
   - Search for "Collecting static files" or any errors
   - The build should show: `177 static files copied to...`

## 2. **Verify Your Settings**
   In `dallas/settings.py`, you should have:
   ```python
   STATIC_URL = '/static/'
   STATIC_ROOT = BASE_DIR / 'staticfiles'
   STATICFILES_STORAGE = 'whitenoise.storage.CompressedStaticFilesStorage'
   ```
   And WhiteNoise middleware should be in MIDDLEWARE:
   ```python
   MIDDLEWARE = [
       'django.middleware.security.SecurityMiddleware',
       'whitenoise.middleware.WhiteNoiseMiddleware',
       ...
   ]
   ```

## 3. **Check Your Build Command**
   Your Render service build command should be:
   ```bash
   bash ./build.sh
   ```
   (Or manually: `python manage.py collectstatic --noinput --clear`)

## 4. **Verify Start Command**
   Your Render service start command should be:
   ```bash
   gunicorn -b 0.0.0.0:10000 dallas.wsgi
   ```
   (Or use the Procfile: `web: gunicorn -b 0.0.0.0:10000 dallas.wsgi`)

## 5. **Environment Variables**
   Make sure these are set in Render:
   - `DEBUG=False` (static files won't work correctly if DEBUG=True in some cases)
   - `ALLOWED_HOSTS` includes your Render domain (e.g., `dallas-owo3.onrender.com`)

## 6. **Common Issues & Fixes**

   **Issue:** Static files 404 errors (404 Not Found)
   - **Cause:** collectstatic didn't run during build
   - **Fix:** Check build logs; re-run build if needed

   **Issue:** CSS not loading but images load
   - **Cause:** Tailwind CDN might be blocked or MIME type issue
   - **Fix:** Check browser DevTools (F12) → Network tab for failed requests

   **Issue:** Images showing broken links but CSS loads
   - **Cause:** Image paths might be incorrect in templates
   - **Fix:** Verify all image src use `{% static 'images/...' %}` tags

   **Issue:** Everything 500 errors
   - **Cause:** WhiteNoise or settings error
   - **Fix:** Check Render runtime logs for detailed error; ensure whitenoise is installed

## 7. **Quick Local Test**
   Run this locally to verify collectstatic works:
   ```bash
   python manage.py collectstatic --noinput --clear
   ```
   
   Should output:
   ```
   177 static files copied to '...staticfiles'.
   ```

## 8. **Debug Mode (if needed)**
   To debug on Render (temporarily):
   1. Set `DEBUG=True` in Render environment
   2. Deploy and check if you see more error details
   3. **Remember to set `DEBUG=False` for production!**

## 9. **Force Render Rebuild**
   In Render dashboard:
   1. Go to your service
   2. Click "Manual Deploy" → "Deploy Latest Commit"
   3. Wait for build to complete
   4. Check build logs for collectstatic output

## 10. **Browser Cache**
   Clear your browser cache:
   - **Chrome:** Ctrl + Shift + Delete
   - **Firefox:** Ctrl + Shift + Delete
   - Hard refresh: **Ctrl + F5** (Windows/Linux) or **Cmd + Shift + R** (Mac)

---

If static files still don't load after all these steps, check:
- Render service runtime logs (not just build logs)
- Any permission errors on the `staticfiles/` directory
- Whether your Render plan allows disk write access
