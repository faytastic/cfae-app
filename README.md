# CFAE Web App
Simple static web app deployed automatically using GitHub Actions on OCI.

This is the source code for the CFAE web app.  
It is a static site (HTML/CSS/JS) hosted on an OCI compute instance and served through Nginx.  
Infrastructure is managed separately using Terraform, while application code is deployed through CI/CD.

The code is stored in GitHub and deployed automatically using a CI/CD pipeline.

---

## 🚀 Deployment Overview

Deployment is fully automated.

When code is pushed to the `main` branch:

1. GitHub Actions runs the deployment workflow.
2. The workflow connects securely to the OCI virtual machine using an SSH deploy key.
3. The VM pulls the latest code from this repository.
4. Updated files are copied into the web directory served by Nginx: `/var/www/cfae`

Normal deployments do **not** require manual SSH.

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

After resolving the issue, deployments should return to automated CI/CD.

---

## 📌 Future Improvements

Planned enhancements:

- basic automated tests before deployment  
- rollback/versioning support  
- separate environments (dev vs prod)  
- monitoring and deployment alerts
