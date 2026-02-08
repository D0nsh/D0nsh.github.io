# Project Master Plan: Don Shaked Algorithmic Consultant LLC Website  
**Version:** 1.1  
**Status:** Execution-Ready  
**Tech Stack:** Hugo (Extended) + GitHub Pages + PaperMod  
**Cost:** $0.00 / Year  
**Security Profile:** High (Static Site, HTTPS, No Database)  

---

## 1. Executive Summary  
Deploy a fast, secure, and professional portfolio site for **Don Shaked Algorithmic Consultant LLC**. The site acts as a digital business card that validates expertise in Computer Vision, Edge AI, and Generative Models for high‑value clients in the US and Israel.  

**Success Criteria:**  
- **Zero Cost:** Hosted on GitHub Pages at https://D0nsh.github.io.  
- **Zero Maintenance:** No servers, no databases, no attack surface.  
- **High Performance:** Static HTML with sub‑100ms time‑to‑first‑byte on modern CDNs.  
- **Professional Aesthetic:** Minimalist, text‑forward PaperMod theme optimized for engineering credibility.  

---

## 2. Environment Setup (Linux)  

### 2.1 Install Hugo Extended  
*Note: Avoid Snap due to version drift. Use the official Debian package.*  

```bash
# 1. Download the specific stable extended version
wget https://github.com/gohugoio/hugo/releases/download/v0.121.2/hugo_extended_0.121.2_linux-amd64.deb

# 2. Install the package
sudo dpkg -i hugo_extended_0.121.2_linux-amd64.deb

# 3. Verify installation (Must say "extended")
hugo version
```

### 2.2 Configure Git  
Ensure your global identity matches your professional persona.  

```bash
git config --global user.name "Don Shaked"
git config --global user.email "your.email@example.com"
```

---

## 3. Repository Architecture  

### 3.1 Create Remote Repository  
- Log into GitHub.  
- Create a new repository named exactly: `D0nsh.github.io`  
- Set visibility to **Public**  
- Initialize with a `README.md`  

### 3.2 Local Initialization  
Run these commands to set up the project structure:  

```bash
# 1. Clone the repo
git clone https://github.com/D0nsh/D0nsh.github.io.git

# 2. Enter the directory
cd [your-username].github.io

# 3. Initialize Hugo (Force required due to existing .git folder)
hugo new site . --force

# 4. Install PaperMod Theme
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

---

## 4. Configuration (`hugo.toml`)  
Create `hugo.toml` in the root directory with this content:  

```toml
baseURL = 'https://D0nsh.github.io/'
languageCode = 'en-us'
title = 'Don Shaked Algorithmic Consultant LLC'
theme = 'PaperMod'

[params]
    defaultTheme = "auto"
    ShowReadingTime = false
    ShowShareButtons = false
    ShowPostNavLinks = false
    ShowBreadCrumbs = false
    ShowCodeCopyButtons = true
    
    [params.homeInfoParams]
        Title = "Algorithmic Consulting & AI Solutions"
        Content = """
        Specializing in high-performance Computer Vision, Edge AI Optimization, and Generative Models.
        
        Based in Medellín. Serving clients globally (US/Israel/EU).
        """
    
    [[params.socialIcons]]
      name = "github"
      url = "https://github.com/D0nsh"
    [[params.socialIcons]]
        name = "email"
        url = "mailto:contact@donshaked.com"

[menu]
    [[menu.main]]
        identifier = "services"
        name = "Services"
        url = "/services/"
        weight = 10
    [[menu.main]]
        identifier = "about"
        name = "About"
        url = "/about/"
        weight = 20
```

---

## 5. Content Strategy  

### 5.1 The "About" Page  
**File:** `content/about.md`  
*Goal: Establish trust and authority.*  

```markdown
---
title: "About the Firm"
layout: "page"
summary: "profile"
---

**Don Shaked Algorithmic Consultant LLC** is a boutique engineering firm bridging the gap between state-of-the-art AI research and production-grade software.

### The Principal
Led by Don Shaked, a Senior Algorithmic Engineer with deep roots in the rigorous Israeli tech sector. Specializes in solving "hard" computer vision problems that off-the-shelf models cannot handle.

### Technical Philosophy
We do not just "run models." We optimize them.  
- **Languages:** Python (Research/Prototyping), Kotlin (Production/Android)  
- **Focus:** Mathematical correctness, memory efficiency, and inference speed  

### Location
Headquartered in the US, with operations based in Medellín, Colombia. Enables seamless time-zone overlap with US clients (EST/CST) while maintaining competitive operational costs.
```

### 5.2 The "Services" Page  
**File:** `content/services.md`  
*Goal: Define offerings clearly.*  

```markdown
---
title: "Technical Services"
layout: "page"
---

### 1. Computer Vision Pipelines
End-to-end development of custom vision systems.  
- **Object Detection & Segmentation:** Custom training using foundation models (Florence-2, SAM) for auto-labeling  
- **Visual Anomaly Detection:** Industrial and security applications  

### 2. Edge AI Optimization
Deploying heavy models on resource-constrained devices without sacrificing accuracy.  
- **Techniques:** Quantization (INT8), Pruning, Knowledge Distillation  
- **Targets:** Mobile (Android/Kotlin), Embedded Linux, IoT Edge  

### 3. Generative AI Research
Consulting on the cutting edge of generative modeling.  
- **Architectures:** Diffusion Models, Normalizing Flows  
- **Applications:** Synthetic data generation, clustering, complex distribution modeling  
```

---

## 6. Deployment Pipeline (GitHub Actions)  

### 6.1 Create Workflow Directory  
```bash
mkdir -p .github/workflows
```

### 6.2 Create Workflow File  
**File:** `.github/workflows/hugo.yaml`  

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.121.2
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb
          sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v4
      - name: Build with Hugo
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

---

## 7. Execution Checklist  
- [ ] **Install Hugo:** Run Linux commands in Section 2  
- [ ] **Clone & Init:** Execute commands in Section 3  
- [ ] **Config:** Create `hugo.toml` with content from Section 4  
- [ ] **Content:** Create `content/about.md` and `content/services.md` (Section 5)  
- [ ] **Workflow:** Create `.github/workflows/hugo.yaml` (Section 6)  
- [ ] **Push to Live:**  
  ```bash
  git add .
  git commit -m "Initial website launch"
  git push origin main
  ```  
- [ ] **Enable Pages:** GitHub Repo Settings → Pages → Source: *GitHub Actions*  

> ✅ **Post-Deployment:** Site auto-updates on every `git push`. Zero manual deployment steps required.  
> 🌐 **Live URL:** https://D0nsh.github.io (Updates within ~2 minutes of push)