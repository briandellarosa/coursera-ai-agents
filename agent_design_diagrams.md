# Agent Design Diagrams

## 1. Architecture (Flowchart)
```mermaid
flowchart TD

  subgraph User[User]
    UI["User Input: 'Write a README for this project.'"]
  end

  subgraph Agent
    Goals["Goals: Gather Information, Terminate with README"]
    Memory["Memory: Conversation + Results"]
    AL["AgentLanguage: AgentFunctionCallingActionLanguage"]
    AR["ActionRegistry: Registered Actions"]
    Env["Environment: Executes Actions"]
    GR["generate_response(): Calls LLM"]
  end

  subgraph Actions
    ListFiles["list_project_files()"]
    ReadFile["read_project_file(name)"]
    Terminate["terminate(message)"]
  end

  subgraph LLM[LLM]
    GPT["GPT-4o via LiteLLM"]
  end

  %% Flow (labels removed for compatibility)
  UI --> Memory
  Goals --> AL
  Memory --> AL
  AR --> AL
  AL --> GR
  GR --> GPT
  GPT --> AL
  AL --> Agent
  Agent --> AR
  Agent --> Env
  Env --> Actions
  Actions --> Env
  Env --> Memory
  Memory --> AL
  Terminate -.-> Agent
```

## 2. Agent Sequence Diagram
```mermaid
sequenceDiagram
  autonumber
  participant U as User
  participant Ag as Agent
  participant Mem as Memory
  participant AL as AgentLanguage
  participant GR as generate_response()
  participant L as LLM (GPT-4o via LiteLLM)
  participant AR as ActionRegistry
  participant Env as Environment
  participant Act as Action

  U->>Ag: Provide task ("Write a README for this project.")
  Ag->>Mem: add_memory(user task)

  Ag->>AL: construct_prompt(goals, memory, actions)
  AL-->>Ag: Prompt (messages, tools)
  Ag->>GR: prompt_llm_for_action(Prompt)
  GR->>L: completion(messages, tools)
  L-->>GR: Response (text or tool call)
  GR-->>Ag: response (string)

  Ag->>AL: parse_response(response)
  AL-->>Ag: {tool, args}
  Ag->>AR: get_action(tool)
  AR-->>Ag: Action

  Ag->>Env: execute_action(Action, args)
  Env->>Act: execute(args)
  Act-->>Env: result or exception
  Env-->>Ag: {tool_executed, result/error, timestamp}

  Ag->>Mem: add_memory(assistant response)
  Ag->>Mem: add_memory(environment result)

  Ag->>Ag: should_terminate(response)?
  alt tool is terminate
    Ag-->>U: Return final README and stop
  else continue
    Ag-->>Ag: Next iteration with updated Memory
  end
```

## 3. Agent Swimlane Diagram
```mermaid
flowchart LR
  %% Lanes as subgraphs
  subgraph Lane_User[User]
    U1[Provide task]
    U2[Receive final README]
  end

  subgraph Lane_Agent[Agent]
    A1[Store task in Memory]
    A2["Construct Prompt (Goals + Memory + Actions)"]
    A3[Parse model response]
    A4[Lookup Action]
    A5[Decide continue or terminate]
  end

  subgraph Lane_LLM[LLM]
    L1[Generate response]
  end

  subgraph Lane_Env[Environment]
    E1[Execute Action]
    E2[Wrap Result]
  end

  subgraph Lane_Actions[Actions]
    Ac1[list_project_files()]
    Ac2["read_project_file(name)"]
    Ac3["terminate(message)"]
  end

  %% Cross-lane flow
  U1 --> A1
  A1 --> A2
  A2 --> L1
  L1 --> A3
  A3 --> A4
  A4 --> E1
  E1 --> Ac1
  E1 --> Ac2
  E1 --> Ac3
  Ac1 --> E2
  Ac2 --> E2
  Ac3 --> E2
  E2 --> A5
  A5 -.-> A2
  A5 -.-> U2
```
