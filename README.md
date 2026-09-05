🎨 Cymbal Shops Paint Agent

A multi-agent conversational AI system built with Google's Agent Development Kit (ADK) that helps customers of a fictional retail chain, Cymbal Shops, shop for paint online — from product selection through final pricing.

Built as part of a Google Cloud Skills Boost Challenge Lab. Unlike guided labs, challenge labs give you an architecture diagram, a partially broken codebase, and a set of objectives — with minimal step-by-step instructions. You're expected to independently diagnose bugs, apply the correct design patterns, and ship a working cloud deployment.

📖 Overview

Cymbal Shops is expanding into Europe with a new Paint Department for DIY home renovation projects. This agent walks a customer through the full buying journey conversationally:

Product discovery answers questions about available paint lines using grounded search over official product datasheets
Color selection lets the customer choose a color for each room
Coverage calculation computes how much paint is needed based on room dimensions, doors, windows, and number of coats
Pricing calculates total cost based on the selected product and required coverage

The system remembers context across the whole conversation (selected paint, coverage rate, price) using session state — so a customer can naturally move from "I'd like EcoGreens" to "how much will two coats for my office cost?" without repeating themselves.

🏗️ Architecture

The agent is built as a root agent orchestrating a tree of specialized sub-agents, each responsible for one part of the workflow:

root_agent (paint_agent)
├── AgentTool → search_agent          # Vertex AI Search grounding over paint datasheets
├── set_session_value (tool)          # persists user choices to session state
└── sub_agent → room_planner_agent
        └── sub_agent → coverage_calculator_agent
                └── paint_coverage_calculator (tool)

Key design pattern: Vertex AI's Gemini models don't allow a search tool to be combined with other non-search tools within a single agent. To work around this, search_agent (which uses VertexAiSearchTool) is wrapped in an AgentTool, allowing it to be used as a regular tool alongside set_session_value — instead of being invoked as a sub-agent, which would trigger this restriction.

🧠 Skills Demonstrated
Multi-agent orchestration — designing a root agent that delegates to specialized sub-agents based on conversation stage
The AgentTool pattern — wrapping an agent-with-search-tool so it can coexist with other tools, working around Gemini's tool-mixing restriction
Grounded retrieval-augmented generation (RAG) — using Vertex AI Search / Agent Search to ground responses in real product documentation (PDF datasheets) rather than model hallucination
Session state management — using ADK's ToolContext.state to persist values across multi-turn conversations, and referencing them dynamically in agent instructions via key templating ({KEY?} syntax)
Custom tool development — writing Python functions (set_session_value, paint_coverage_calculator) exposed as agent tools
Cloud debugging — diagnosing and resolving real deployment failures using Cloud Logging traceback analysis (IAM propagation issues, ModuleNotFoundError from incorrect deployment paths, malformed environment variables)
IAM & service account permissioning — granting the correct roles (Agent Platform User, Discovery Engine User) to Google-managed service agents for cross-service access
Production deployment — shipping a working agent to a managed serverless runtime, not just running it locally
☁️ Platforms & Technologies
Category	Tools/Services
Agent framework	Google Agent Development Kit (ADK)
LLM	Gemini 2.5 Flash (via Vertex AI)
Retrieval/Search	Vertex AI Agent Search (Discovery Engine), Layout Parser document parsing
Deployment	Vertex AI Agent Runtime (Reasoning Engine)
Frontend	Chainlit (Python chat UI framework)
Cloud infra	Google Cloud Storage, Cloud Logging, IAM
Language	Python 3.12
🚀 Setup & Run Locally

1. Clone the repo

bash
git clone https://github.com/YOUR_USERNAME/cymbal-shops-paint-agent.git
cd cymbal-shops-paint-agent

2. Install dependencies

bash
export PATH=$PATH:"/home/${USER}/.local/bin"
python3 -m pip install -r requirements.txt
python3 -m pip install chainlit==2.11.1

3. Configure environment variables

Create a .env file in the project root:

env
GOOGLE_GENAI_USE_VERTEXAI=TRUE
GOOGLE_CLOUD_PROJECT=your-gcp-project-id
GOOGLE_CLOUD_LOCATION=us-central1
RESOURCES_BUCKET=your-gcs-bucket-name
MODEL=gemini-2.5-flash
SEARCH_ENGINE_ID=your-vertex-search-engine-id

Copy it into the agent directory too:

bash
cp .env paint_agent/.env

4. Run the agent in the terminal

bash
adk run paint_agent

5. Or run the interactive web UI

bash
adk web --allow_origins "regex:https://.*\.cloudshell\.dev"
☁️ Deploying to Vertex AI Agent Runtime
bash
adk deploy agent_engine --display_name "Paint Agent" paint_agent

Grant the following roles to the Vertex AI Reasoning Engine service agent so it can call Agent Search:

Agent Platform User
Discovery Engine User

Once deployed, connect the Chainlit frontend by updating chainlit_ui/app.py with your deployed agent's resource name:

python
agent = client.agent_engines.get(name='projects/.../locations/.../reasoningEngines/...')

Then run:

bash
cd chainlit_ui
chainlit run app.py
💬 Example Conversation
User: hello
Agent: Hello! I'm here to help you find the right paint for your project.
       Would you like to learn more about the paint products offered by Cymbal Shops?

User: yes
Agent: [lists Project Paint, EcoGreens, SureCoverage, Forever Paint with prices & coverage rates]

User: I'd like to use EcoGreens
Agent: Great choice! How many rooms would you like to paint, and what should we call them?

User: Just one room, my office
Agent: What color would you like for your office?

User: Deep Ocean
Agent: What are the dimensions of the room?

User: 3m by 4m. 3m high. 1 door, 2 windows.
Agent: I'd recommend two coats. Does that work for you?

User: Two coats.
Agent: You'll need coverage of approximately 74 square meters, which comes to
       X liters, or Y buckets, for a total cost of $Z.
📁 Project Structure
adk_challenge_lab/
├── paint_agent/
│   ├── agent.py                  # root agent definition
│   ├── tools.py                  # set_session_value tool
│   ├── callback_logging.py       # model I/O logging hooks
│   └── sub_agents/
│       ├── search_agent/
│       │   └── agent.py          # Vertex AI Search-grounded agent
│       └── room_planner/
│           ├── agent.py          # room/color collection agent
│           └── sub_agents/
│               └── coverage_calculator/
│                   ├── agent.py  # coverage & pricing agent
│                   └── tools.py  # paint_coverage_calculator tool
├── chainlit_ui/
│   └── app.py                    # chat frontend connecting to deployed agent
└── requirements.txt
📄 License

This project was completed for educational purposes as part of a Google Cloud Skills Boost lab. Cymbal Shops is a fictional company used for training scenarios.
