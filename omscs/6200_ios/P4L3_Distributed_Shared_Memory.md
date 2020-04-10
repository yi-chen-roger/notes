# P4L2: Distributed Shared Memory
## 1. Preview
Distributed Shared Memory
- DSM
- Distributed state management and design alternatives
- consistency model

## 2. Visual Metaphor
"Managing distributed shared memory is like managing tools/parts across all workspaces in a toy shop"

Toy Shop
- must decide placement
  - place resources close to relevant workers
- must decide migration
  - move resources as soon as possible to relevant works
- must decide sharing rule
  - how long can resources be kept? when are they ready? how to store?

Distributed Memory
- must decide placement
  - place memory (pages) close to relevant processes
- must decide migration
  - when to copy memory(pages) from remote to local
- must decide sharing rule
  - how long can resources be kept? when are they ready? how to store?
  - ensure memory operations are properly order

## 3. Reviewing DFS
Clients
- send requests to files service

Caching
- improve performance (seen by clients) and scalability (supported by servers)

Servers
- own and manage state (files)
- provide service (file access)

## 4. Peer Distributed Applications
Each node
- "owns" state
- provides service
=> all nodes are "peers"

Examples: bit data analytics, web searches, content sharing, or distributed shared memory(DSM)