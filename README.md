# CFAE Web App
Simple static web app deployed automatically using GitHub Actions on OCI, with validation before deployment.

This is the source code for the CFAE web app.  
It is a static site (HTML/CSS/JS) hosted on an OCI compute instance and served through Nginx.  
Infrastructure is managed separately using Terraform, while application code is deployed through CI/CD.

The code is stored in GitHub and deployed automatically using a CI/CD pipeline.

---

## 🚀 Deployment Overview

Deployment is fully automated.

When code is pushed to the `main` branch:

1. GitHub Actions starts the CI/CD workflow.
2. A `check` job runs first and validates that the project structure is safe to deploy 
   (for example, confirming required files exist).
3. If the check passes, the `deploy` job connects securely to the OCI virtual machine.
4. The VM pulls the latest code from this repository.
5. Updated files are copied into the web directory served by Nginx: `/var/www/cfae`

The pipeline runs on a temporary GitHub-hosted Linux runner and does not require any local setup.

If the check fails, deployment stops and nothing is changed on the server.


Normal deployments do **not** require manual SSH.

---

### ✅ CI/CD Pipeline Behavior

- **Pipeline runs automatically** on pushes to `main`
- A **`check` job** runs first and validates the project structure
- The **`deploy` job** runs only if `check` succeeds
- Deployment occurs through GitHub Actions, **without manual SSH**
- If validation fails, **deployment is blocked** and the server is not changed
- This protects the production site from accidental breaking changes and ensures that only valid builds are deployed.


---

## 🔐 Security Notes

- A GitHub **deploy key** allows the VM to pull updates securely.
- Secrets are stored in GitHub Actions and are never committed to the repo.
- SSH access should be used only for troubleshooting.

If needed, a deploy key can be revoked or replaced at any time.

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

    cd ~/cfae-app
    git pull
    sudo cp -r ~/cfae-app/* /var/www/cfae/

Once the issue is fixed, resume using the CI/CD pipeline for all deployments so that validation and safety checks remain in place.


---

## 📌 Future Improvements

Planned enhancements:

- expand automated validation before deployment
- rollback/versioning support  
- separate environments (dev vs prod)  
- monitoring and deployment alerts
