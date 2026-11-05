# OAuth GuideHub: System Architecture & Technical Documentation

[![Vue 3](https://img.shields.io/badge/Vue.js-3.0-4FC08D?logo=vue.js&logoColor=white)](https://vuejs.org/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-3.0-38B2AC?logo=tailwind-css&logoColor=white)](https://tailwindcss.com/)
[![Zero Dependencies](https://img.shields.io/badge/Dependencies-Zero-brightgreen)](#)

## Executive Technical Summary

OAuth GuideHub is a zero-dependency, client-side Single Page Application (SPA) designed to model and simulate the complex state transitions associated with SaaS OAuth integration research. 

Built entirely within a Single-File Component (SFC) architecture (HTML/JS/CSS combined), the application implements an in-memory reactive data store, a synthetic API gateway with programmable latency, and a simulated multi-agent state machine.

## 🏗 System Architecture

Although deployed as a monolithic client-side file to maximize portability, the system is logically partitioned into four distinct layers:

### 1. Presentation Layer (Vue 3 Template)
*   **Declarative Rendering:** Utilizes Vue 3's directive-based templating (`v-if`, `v-for`, `:class`) for dynamic DOM updates without manual manipulation.
*   **Component-Level State:** Modals (Guide Viewer, Agent Investigator) maintain local UI state decoupled from the global data store.
*   **CSS-in-JS (via Tailwind JIT):** Uses CDN-injected Tailwind configuration to define custom color palettes (e.g., `slate-950`, custom `brand` scales) and keyframe animations (`pulse-glow`) directly in the configuration object.

### 2. State Management (In-Memory Store)
*   The application bypasses Vuex/Pinia in favor of a centralized reactive root instance (`createApp`).
*   **Data Models:** 
    *   `apps`: Core schema containing provider metadata, verification tiers, and stateful step-by-step checklists (`steps[]` array with boolean `done` flags).
    *   **Computed Projections:** The `filteredApps` computed property acts as a memoized database view, applying cross-field text search and categorical indexing in real-time, recalculating only when dependencies mutate.

### 3. Synthetic API Gateway & Network Simulator
To validate UI/UX against backend realities without requiring a live server, the app includes a synthetic network layer (`activeView === 'mock-api'`).

*   **Endpoint Routing:** A mock router intercepts requests to endpoints like `GET /v1/guides` and `POST /v1/investigate`.
*   **Latency Injection:** Implements stochastic delay generation (`Math.floor(Math.random() * 80) + 40` ms) to simulate realistic network jitter and Edge propagation delays.
*   **Response Construction:** Dynamically serializes in-memory objects to JSON payloads with corresponding simulated HTTP status codes (200 OK, 400 Bad Request).
*   **Event Logging:** Maintains an immutable append-only log array (`apiNetworkLogs`) to render a terminal-style network traffic view.

### 4. Multi-Agent Pipeline State Machine
The "Live Agent Investigator" is modeled as a time-based deterministic state machine rather than a simple loading spinner.

*   **State Transitions:** Uses a `setInterval` loop to iterate through a predefined Directed Acyclic Graph (DAG) of agent states (DocScraper -> PolicyAnalyzer -> FlowValidator -> ActionSynthesizer).
*   **Progressive Hydration:** Emits simulated logs and progress percentages asynchronously to the DOM to model the behavior of a chunked streaming response (e.g., Server-Sent Events or WebSockets) from a live LLM pipeline.
*   **Dynamic Object Generation:** Upon completion, the pipeline dynamically constructs a new schema-compliant `app` object and unshifts it into the reactive store, immediately hydrating the UI with the synthesized data.

## ⚙️ Engineering Trade-offs & Constraints

| Decision | Rationale | Trade-off |
| :--- | :--- | :--- |
| **Zero-Build Pipeline** | Mandated by the requirement for instant reviewability. Reviewers can execute the code directly via `file://` protocol. | Loss of module bundling (Webpack/Vite), requiring global variable injection via CDN. |
| **In-Memory Store** | Reduces complexity for a prototype scoped to a 6-8 hour APM assignment. | State is volatile; refreshes destroy custom generated guides and checklist progress. |
| **Simulated Agents** | Direct scraping of behind-login developer portals requires headless browser pools (Puppeteer) and anti-bot circumvention, out of scope for frontend prototype. | Agent outputs for custom inputs are hardcoded heuristic responses rather than true LLM generations. |

## 🚀 Execution & Development

No Node.js environment, package manager, or local server is required. 
