# AI Cloud Engineering Log

A free, hands-on, day-by-day path from **software engineer** to **cloud and AI infrastructure engineer**: AWS → Terraform → Docker → Kubernetes → distributed systems → GPUs → LLM serving → enterprise RAG.

I'm a senior full-stack engineer (Rails, React, Node, PostgreSQL) learning this in public. Every day folder holds the lesson summary, the lab I did and the design docs I wrote, so **you can follow the same path, copy the structure and compare your answers.**

> ⭐ If this helps you, star the repo, fork it, and keep your own log alongside it.

## Who this is for

- Backend or full-stack developers who deploy apps but want to understand the infrastructure underneath
- Engineers preparing for **AWS SAA-C03**, **Terraform Associate** or **CKA**
- Anyone aiming for platform, MLOps or AI-infrastructure roles

**Prerequisites:** comfortable with a terminal, Git, and building a basic web app. No prior cloud experience is needed.

## How to use this repo

1. Start at [Phase 1](phase-1/) and work through the days in order. Each day takes about **75–90 minutes**.
2. In each day folder, read `README.md` first. It covers what you'll learn, the lab steps, cost and cleanup, and self-check questions.
3. **Do the lab yourself before reading my design docs.** Then compare.
4. Fork the repo or create your own `ai-cloud-engineering-log` and commit your own evidence each day.
5. At the end of each phase, build the phase project.

## Roadmap

| Phase | Focus | Duration | Certification | Status |
|---|---|---|---|---|
| [1](phase-1/) | AWS Cloud Architecture | 6–8 weeks | AWS SAA-C03 | ⏳ In progress |
| 2 | Terraform & Infrastructure as Code | 3–4 weeks | Terraform Associate | ⬜ |
| 3 | Linux, Networking & Docker | 3–4 weeks | – | ⬜ |
| 4 | Kubernetes & Platform Engineering | 6–8 weeks | CKA | ⬜ |
| 5 | Python Automation & Distributed Systems | 4–5 weeks | – | ⬜ |
| 6 | AI, ML & GPU Foundations | 4–5 weeks | NVIDIA (optional) | ⬜ |
| 7 | LLM Serving & GPU Infrastructure | 5–6 weeks | NVIDIA (optional) | ⬜ |
| 8 | Enterprise AI / RAG Architecture | 4–5 weeks | – | ⬜ |

Status key: ✅ done · ⏳ in progress · ⬜ not started (I update this as I go)

## Phase projects

| Phase | Project you'll build |
|---|---|
| 1 | Production-ready AWS web app (ALB → EC2 → RDS) |
| 2 | Reusable Terraform modules with remote state |
| 3 | Containerized multi-service app (App + PostgreSQL + Redis) |
| 4 | Kubernetes application platform with autoscaling and monitoring |
| 5 | Reliable API and background-worker system |
| 6 | LLM benchmarking tool (latency, tokens/s, memory, cost) |
| 7 | GPU-powered LLM hosting platform (auth, quotas, routing) |
| 8 | Enterprise multi-tenant RAG platform |

## ⚠️ Cost and safety

- Many labs are **design only and free**. Labs that create resources say so clearly and include **cleanup steps**.
- Use a separate AWS account, enable **MFA on the root user**, and set up a **billing alarm** before your first paid lab.
- NAT gateways, load balancers, databases, public IPv4 addresses and GPUs bill by the hour, even when idle. **Delete resources after every lab.**
- Never commit access keys, `.tfvars` files with secrets, or `terraform.tfstate`.
- You're responsible for your own cloud bill. Always check current pricing and free-tier terms on the official AWS pages.

## Repo structure

```
ai-cloud-engineering-log/
├── README.md              ← you are here
└── phase-N/
    ├── README.md          ← phase goals, day list, project
    ├── day-NN/
    │   ├── README.md      ← lesson summary, lab steps, self-check
    │   └── *.md / *.mmd   ← design docs, diagrams, configs
    └── project/           ← phase project
```

## Contributing

Found a mistake or an outdated link? Open an issue or a PR. Corrections are very welcome, since AWS changes often.

## Disclaimer

These are personal learning notes, not official training material. They aren't affiliated with or endorsed by AWS, HashiCorp, CNCF or NVIDIA. Always check details against the official documentation linked in each day.

## License

<!-- Choose one before publishing, e.g. MIT for code, CC BY 4.0 for docs -->
See [LICENSE](LICENSE).