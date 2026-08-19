# Agent Guidelines & Working Rules

1. **User Control & Decisions:**
   - Do NOT assume or add tools, libraries, or configuration rules without asking first.
   - All architectural decisions must be approved by the user before documenting or creating.

2. **Workflow:**
   - Provide terminal commands one step at a time.
   - Do NOT execute or create files autonomously unless explicitly told.
   - Keep all code and configuration as minimal as possible (no boilerplate or extras upfront).

3. **Core Tech Stack:**
   - Backend: Java 17 + Spring Boot + Maven.
   - Real-time: Java 17 + Spring Boot (STOMP WebSocket).
   - Frontend: React + Vite + TypeScript.
   - No external infrastructure (Redis, Docker, databases, common libraries) until explicitly requested.