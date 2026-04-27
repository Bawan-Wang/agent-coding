Instruction & Context Documentation

Solo Project 

```
project_template/
│
├── AGENTS.md             # On startup: confirm permissions, tone, and available tools
├── PLAN.md               # Global blueprint and overall project progress
├── README.md             # Project overview (for human developers)
├── requirements.txt      # Python dependency management
│
├── .docs/                # Core documentation folder (also usable as .spec)
│   ├── api.md            # RESTful API or internal Function interface definitions
│   ├── architecture.md   # Before writing code: confirm folder structure and module dependencies
│   ├── context.md        # Current dev focus, known bugs, or environment constraints
│   ├── rules.md          # Coding style, naming conventions, forbidden libraries
│   ├── skill.md          # Defines Functions or external tools available to AI
│   │
│   ├── exec-plans/       # Stores all active or completed detailed execution plans
│   │   ├── 001-setup-env.md
│   │   ├── 002-vlm-model-bridge.md      
│   │   ├── 003-taiwan-server-racing-fix.md  
│   │   └── done/         # Archived completed execution plans
│   │
│   └── product-specs/    # Before a task: confirm feature logic to avoid wrong implementation
│       ├── index.md      # Index of all product specs
│       └── new-user-onboarding.md        # Spec: new user onboarding flow
│
├── src/                  # Source code
│
└── tests/                # Test scripts
```