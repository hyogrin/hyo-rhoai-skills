# hyo-rhoai-skills

Production-ready AI skills for Red Hat OpenShift AI (RHOAI) model deployment and operations.

These skills solve real-world problems encountered during model serving on OpenShift AI — particularly the common issue of HuggingFace model downloads failing mid-stream during KServe storage-initializer execution.

## Installation

### Using Lola (recommended)

```bash
# Add as a module
lola mod add https://github.com/hyogrin/hyo-rhoai-skills.git

# Install to Cursor (current project)
lola install hyo-rhoai-skills -a cursor

# Install globally (all projects)
lola install hyo-rhoai-skills -a cursor --scope user
```

### Using .lola-req (declarative)

Add to your project's `.lola-req` file:

```
https://github.com/hyogrin/hyo-rhoai-skills.git@main
```

Then run:

```bash
lola sync
```

### Manual Installation

```bash
# Copy skill files into your project
cp -r skills/hf-model-deploy /path/to/your-project/.cursor/skills/
```

## Available Skills

| Skill | Description |
|-------|-------------|
| **[hf-model-deploy](skills/hf-model-deploy/SKILL.md)** | Stable, failure-resistant model deployment for OpenShift AI |

### /hf-model-deploy

Solves the #1 pain point in RHOAI model serving: **unreliable model weight downloads**.

**Strategy Priority:**

1. **OCI ModelCar** — Models as container images (zero download at startup)
2. **S3/MinIO Pre-Cache** — Download once, serve many (enterprise-grade)
3. **PVC Pre-Cache** — Local volume caching (single-pod)
4. **Direct hf://** — AVOID for models >10GB (hangs, no resume)

**What you get:**
- OCI ModelCar catalog discovery (quay.io API search)
- S3/MinIO full setup with HF→S3 transfer Job
- PVC-based download with `hf_transfer` (fast, resumable)
- GPU VRAM vs model size guide
- Tool calling configuration for all major model families
- Troubleshooting table for common deployment failures

## Complementary Skills

This module is designed to work alongside the [rh-ai-engineer](https://github.com/RHEcosystemAppEng/agentic-collections) module from Red Hat Ecosystem Engineering:

```bash
# Install both for full coverage
lola install -f rh-ai-engineer -a cursor  # Deployment workflow
lola install hyo-rhoai-skills -a cursor   # Model acquisition
```

| Concern | rh-ai-engineer | hyo-rhoai-skills |
|---------|---------------|-----------------|
| Model acquisition (getting weights) | ❌ Assumes model is available | ✅ Full download strategies |
| Deployment workflow (InferenceService) | ✅ Full workflow | Basic manifests |
| Runtime selection (vLLM/NIM/Caikit) | ✅ Interactive | vLLM focused |
| Troubleshooting download failures | ❌ | ✅ XET, timeout, OOM fixes |
| GPU sizing guide | Partial (references) | ✅ Complete table |

## Prerequisites

- OpenShift cluster with RHOAI installed
- `oc` CLI authenticated
- GPU nodes available (NVIDIA)
- [Lola](https://github.com/RedHatProductSecurity/lola) for installation (optional)

## Tested Environments

| Platform | Version | GPU |
|----------|---------|-----|
| OpenShift | 4.16+ | L4, L40S, A100 |
| RHOAI | 2.16+ / 3.0+ | NVIDIA |
| vLLM | 0.6.x+ (Red Hat image) | CUDA 12+ |

## Project Structure

```
hyo-rhoai-skills/
├── README.md                              # This file (English)
├── README_ko.md                           # Korean version
├── skills/
│   └── hf-model-deploy/
│       ├── SKILL.md                       # Main skill definition
│       └── ansible-role-example.yml       # Ansible role reference
└── .gitignore
```

## Contributing

1. Fork this repository
2. Create a skill under `skills/<skill-name>/SKILL.md`
3. Follow the [Lola skill format](https://lobstertrap.org/lola/concepts/skills-and-modules/)
4. Test with `lola mod add ./ && lola install hyo-rhoai-skills -a cursor`
5. Submit a PR

## License

Apache License 2.0
