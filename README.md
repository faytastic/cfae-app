# CFAE Web App
Live site: https://cfae.ftlgapps.com

Front-end web app deployed automatically to OCI using GitHub Actions, with validation and post-deploy checks.

A backend API validates and processes form submissions and stores them in Oracle Autonomous Database.
The frontend communicates only with the backend API and never connects directly to the database.

This is the source code for the CFAE web app.  
It is a static site (HTML/CSS/JS) hosted on an OCI compute instance and served through Nginx.  
Infrastructure is managed separately using Terraform, while application code is deployed through CI/CD.

The code is stored in GitHub and deployed automatically using a CI/CD pipeline.

---

## 🚀 Deployment Overview

Deployment is fully automated.

This pipeline prevents broken or incomplete code from being deployed, and confirms the live site is healthy after deployment.

When code is pushed to the `main` branch:

1. GitHub Actions starts the CI/CD workflow.
2. A `check` job runs first and validates that the project structure is safe to deploy (for example, confirming required files exist).
3. If the check passes, the `deploy` job connects securely to the OCI virtual machine.
4. The VM pulls the latest code from this repository.
5. Updated files are copied into the web directory served by Nginx: `/var/www/cfae`
6. After deployment, a smoke test checks the live HTTPS site and confirms the expected version is being served.
   
The pipeline runs on a temporary GitHub-hosted Linux runner and does not require any local setup.

If the check fails, deployment stops and nothing is changed on the server.

Normal deployments do **not** require manual SSH.

Manual SSH should only be used for troubleshooting.

---

### ✅ CI/CD Pipeline Behavior

- **Pipeline runs automatically** on pushes to `main`
- A **`check` job** runs first and validates the project structure
- The **`deploy` job** runs only if `check` succeeds
- Deployment occurs through GitHub Actions, **without manual SSH**
- A smoke test confirms the live site is serving the new version before the deployment is considered successful.
- If validation fails, **deployment is blocked** and the server is not changed
- This protects the production site from accidental breaking changes and ensures that only valid builds are deployed.


---

## 🔢 Versioning

This project uses a VERSION file together with GitHub Releases to track and manage deployments.

Releases are tagged in GitHub using semantic versioning:

- Versions are created intentionally (not for every commit)
- Releases act as restore points
- Rollback always uses a previous release

The version is also displayed at the bottom of the live webpage.

- The current version is stored in a file called `VERSION`
- The CI/CD pipeline reads this value and replaces `BUILD_VERSION` in `index.html`
- Whatever is in the `VERSION` file becomes the live version on the site

Example: `1.0.0`

### When to update the version

Update the `VERSION` file only when you intend to make a new release:

- `1.0.1` small bug fix  
- `1.1.0` new feature  
- `2.0.0` major change  

Small commits that don’t change behavior don’t require a version bump.

---

## 🔐 Security Notes

- A GitHub **deploy key** allows the VM to pull updates securely.
- Secrets are stored in GitHub Actions and are never committed to the repo.
- SSH access should be used only for troubleshooting.

If needed, a deploy key can be revoked or replaced at any time.

---
## 🔐 HTTPS / TLS Configuration

The CFAE site uses full end-to-end HTTPS:

- Browser → Cloudflare: encrypted  
- Cloudflare → OCI VM (Nginx): encrypted using Let’s Encrypt

Cloudflare SSL mode: **Full (Strict)**

Certificates on the VM are managed by Certbot and stored at:

`/etc/letsencrypt/live/cfae.ftlgapps.com/`

Certificates renew automatically using:

`certbot renew`

Nginx is configured with a hostname so the certificate attaches properly:

`server_name cfae.ftlgapps.com;`

> HTTPS was implemented early so future backend features (forms, APIs, auth) are already protected.

---

## 🛠 Technologies Used

- Oracle Cloud Infrastructure (Compute + Networking)
- Nginx web server
- GitHub Actions (CI/CD)
- Terraform for infrastructure (managed in a separate repo)

---

## 📂 Key Paths on the VM

- Web root (served by Nginx): `/var/www/cfae`  
- Repository checkout: `/home/opc/cfae-app`  

Nginx only serves files from the web root.

---

## 🔄 Manual Recovery (only if CI/CD fails)

If the pipeline fails and a fix is needed temporarily, you can deploy manually from the VM:

```bash
cd ~/cfae-app
git pull
sudo cp -r ~/cfae-app/* /var/www/cfae/
```

Once the issue is fixed, resume using the CI/CD pipeline so that validation and safety checks remain in place.

---

## 🔁 Rollback

If a deployment introduces an issue, you can roll back to a previously working version:

1. Open **Actions** in GitHub  
2. Select a **previous successful workflow run**  
3. Click **Re-run all jobs**

This redeploys the exact release state (code, files, and config) that was live at that time. 

Releases provide predictable rollback points.


> Rollback is for stabilization. Once the issue is fixed, redeploy the latest version through the normal CI/CD pipeline.

---


## 📌 Planned Enhancements (Next)

- Enhanced backend validation and rate limiting
- Admin authentication for submissions view
- Email notifications on submission

  
## 🌱 Future / Advanced Enhancements (Nice to Have)

These are not required right now, but represent best-practice maturity goals:

- Additional automated validation before deployment  
- Deployment monitoring and alerts  
- Richer rollback visibility and deployment history  
- Centralized logging and error dashboards


