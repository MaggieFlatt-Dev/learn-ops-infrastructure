# System Map AI Prompts

## 1. Describe the System

I want you to put together an ASCII diagram of the system architecture of this codebase. I want all services, ports, and frameworks labeled and the connections between them labeled. Every service should get its own labeled box, and every dependency between services gets an arrow showing direction, labeled with the type of connection (HTTP, database query, pub/sub event, etc.)



## 2. Convert to a Mermaid Diagram

Take the ASCII diagram you just made and convert it to a mermaid file, placing it in the system-map-ai.md file replacing the ASCII diagram. Use graph LR layout with each service as a node. Label each connection showing direction and connection type (HTTP, DB query, pub/sub, etc). Then place this prompt in the system-map-ai-prompts.md under #2.