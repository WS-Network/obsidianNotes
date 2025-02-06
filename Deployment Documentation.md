## Overview

We deployed a **new frontend** and **backend** to Heroku, updated our **Cloudflare DNS** records to point to the new frontend domain, and modified our **`.env`** file in the frontend project to reference the new backend URL provided by Heroku.

---

## Steps

1. **Deploy New Heroku Backend**
    
    1. Created a new Heroku app (for the backend).
    2. Deployed the backend code (e.g., via GitHub auto-deploy or manual push).
    3. Obtained the new Heroku URL (e.g., `https://my-new-backend.herokuapp.com`).
2. **Update Frontend `.env` File**
    
    4. In the frontend repository, located the `.env` file.
    5. Found the environment variable pointing to the backend (e.g., `REACT_APP_API_URL` or similar).
    6. Replaced the old backend URL with the **new Heroku-provided URL** (e.g., `https://my-new-backend.herokuapp.com`).
    7. Saved and committed the change to the frontend repo.
3. **Update DNS Settings in Cloudflare**
    
    1. Logged into Cloudflare and opened the DNS management for our domain.
    2. **Edited the CNAME record** for the frontend (e.g., `www.mydomain.com`) to point to the new Heroku domain (either `something.herokudns.com` or a direct Heroku `app.herokuapp.com` address).
        - Ensured it is set to **Proxied** (orange cloud) to utilize Cloudflare’s SSL/caching.
    3. Confirmed all necessary email (MX/TXT) records and other services remained unchanged.
4. **Redeploy or Rebuild Frontend**
    
    4. If using a CI/CD or Heroku pipeline for the frontend, triggered a new build/deploy.
    5. If building locally, ran the build process (e.g., `npm run build`) and then pushed to Heroku or the hosting platform for the frontend.
5. **Verification and Testing**
    
    1. Visited the new frontend domain (e.g., `https://www.mydomain.com`) to confirm the site loads.
    2. Tested the **backend endpoints** from the frontend to ensure requests go to the updated `.env`-specified URL.
    3. Checked the **Cloudflare SSL** (look for the lock icon in the browser) to confirm secure HTTPS.
    4. Verified that any environment-specific functionality (e.g., APIs, forms) works correctly with the new backend.