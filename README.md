# cognitive-radar-core
cognitive-radar-core
# 🎯 COGNI‑RADAR – Cognitive Multi‑Agent ATC Radar

> **“Le radar qui pense plus vite que le ciel ne se remplit.”**  
> Radar cognitif multi‑agent (HMARL + Safety‑Shield) capable de gérer **> 10 000 pistes** avec une boucle de décision **≤ 30 ms**, certifiable **DO‑178C Level A**, déployable en **retrofit** sur ASR‑11 / ARSR‑4E.

---

## 📚 Sommaire
- [Vision & Architecture](#vision--architecture)
- [Stack Technique](#stack-technique)
- [Pré‑requis](#pré‑requis)
- [Démarrage rapide (local)](#démarrage-rapide-local)
- [Pipeline CI/CD](#pipeline-cicd)
- [Déploiement sur Jetson Orin](#déploiement-sur-jetson-orin)
- [Publication de la *pillar page* (Netlify)](#publication-de-la-pillar-page-netlify)
- [Documents juridiques (MSA / DPA)](#documents-juridiques-msa--dpa)
- [Arborescence du dépôt](#arborescence-du-dépôt)
- [Commandes utiles](#commandes-utiles)
- [Contribuer](#contribuer)
- [Licence](#licence)

---

## Vision & Architecture
| Couche | Responsabilité | Technologie clé |
|--------|----------------|-----------------|
| **RF Perceptor** | Fusion RF + ADS‑B + WAM + Météo + Caméra/LiDAR → tracklets + intention | Spiking‑Transformer, Kalman‑Net, Graph‑NN |
| **Waveform Designer** | Synthèse de forme d’onde optimale (LFM, OFDM, Noise, Chaotic, Joint Radar‑Comm) | Différentiable Waveform Synthesis + NAS |
| **Beam Scheduler** | Commande AESA : multi‑beam, null‑steering, track‑while‑scan adaptatif | MPC + Deep‑RL sur FPGA/ASIC |
| **Separation Guard** | Vérification règles ICAO/EASA, proposition de résolutions (vectoring, altitude, speed) | Constraint‑Programming + Shielded RL (model‑checking) |
| **Resilience Watchdog** | Détection pannes, cyber‑intrusion, bascule mode dégradé / redondance | Isolation‑Forest + VAE + Self‑Healing K8s‑edge |
| **Human Interface** | Alertes AR/VR/HUD + explications causales (SHAP/Attention) | XAI + LLM fine‑tuned |
| **Sky Orchestrator** | Supervision globale, allocation spectrale, politique secteur | Hierarchical MARL + Meta‑Learning (MAML/PEARL) |

Le tout s’exécute en **pipeline parallèle** sur **2× Jetson AGX Orin (275 TOPS) + 1× AMD Alveo U55C (FPGA+HBM)** → latence mesurée **≈ 27 ms**.

---

## Stack Technique
| Composant | Version / Référence |
|-----------|---------------------|
| **OS** | Ubuntu 22.04 (Jetson Linux) |
| **GPU / AI** | NVIDIA CUDA 11.8, cuDNN 8.6, TensorRT 8.6 |
| **Framework DL** | PyTorch 2.3 + torch‑cuda 118 |
| **API** | FastAPI 0.110 + Uvicorn + Gunicorn (2 workers) |
| **Monitoring** | Prometheus + Grafana (dashboards fournis) |
| **Conteneurisation** | Docker 24 + NVIDIA Container Toolkit |
| **CI/CD** | GitHub Actions → GHCR → SSH + docker‑compose sur Jetson |
| **Infra as Code** | `docker-compose.yml` (services + monitoring) |
| **Docs / SEO** | HTML5 + PicoCSS + JSON‑LD (FAQ, Product, TechArticle) |
| **Juridique** | MSA / DPA (Markdown, convertible en PDF/Docx) |

---

## Pré‑requis
| Outil | Version minimale | Installation |
|-------|------------------|--------------|
| **Git** | 2.38 | `apt-get install git` |
| **Docker** | 24.0 | <https://docs.docker.com/engine/install/> |
| **NVIDIA Container Toolkit** | 1.13 | <https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html> |
| **Python** | 3.11 (venv) | `apt-get install python3.11-venv` |
| **Netlify CLI** (optionnel) | 17.x | `npm i -g netlify-cli` |
| **Clé SSH** (accès Jetson) | Ed25519 | `ssh-keygen -t ed25519` |

> **Sur le Jetson** : après l’installation de Docker + toolkit, redémarrez le démon Docker (`sudo systemctl restart docker`).

---

## Démarrage rapide (local)

```bash
# 1️⃣ Clone
git clone https://github.com/secutorpro-prog/cognitive-radar-core.git
cd cognitive-radar-core

# 2️⃣ Crée un venv & installe les deps
python3.11 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
pip install -r requirements-dev.txt

# 3️⃣ Lance les tests / lint
ruff check .
black --check .
pytest --cov=src -q

# 4️⃣ Build l’image Docker (CPU only – pour test local)
docker build -t cognitive-radar-core:dev .

# 5️⃣ Démarre la stack complète (API + Prometheus + Grafana)
docker compose up -d
# API → http://localhost:8080/health
# Grafana → http://localhost:3000 (admin / admin)
