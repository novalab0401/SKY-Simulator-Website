# SKY Simulator Lab — Tutorial Website

This is the standalone learning guide for `swarm-aircombat`, a fighter-air-combat
simulator used to practice rule-based control, Behavior Trees, and
reinforcement learning. It is intentionally kept outside the simulator's
Python package and source tree. The goal is to help learners inspect and modify
decision-making in simulation, not to claim real-world combat fidelity or
research-grade validation.

## Local layout

Place the two repositories next to one another:

```text
src/
├── swarm-aircombat/       # simulator source code
└── swarm-aircombat-site/  # this tutorial website
```

The tutorials refer to files and commands in `swarm-aircombat`. Follow the
simulator's install and execution commands from that repository's root. The
recommended path focuses on a first fighter engagement, rule-based pilots and
BT, then PPO training and result interpretation. N:M allocation and external
assignment plugins are optional extension lessons.
The source repository currently has no configured public Git remote, so pages
refer to source paths by their repository-relative names rather than linking to
an invented hosting URL.

## Run and build the site

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
mkdocs serve
```

Open `http://127.0.0.1:8000`. Build a static site and check all configured
navigation pages with:

```bash
mkdocs build --strict
```

The output is written to the ignored `site/` directory. Once this website
repository has a hosting remote and GitHub Pages is enabled, publish with
`mkdocs gh-deploy`.
