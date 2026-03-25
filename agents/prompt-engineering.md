---
type: guide
tags: [agents, prompt-engineering, system-prompts, few-shot, templates]
status: complete
updated: 2026-03-25
related:
  - tool-use.md
  - multi-agent.md
---

# Prompt Engineering for Agents

Системные промпты, few-shot примеры и best practices для AI-агентов.

---

## System Prompts

### 1. Orchestrator Agent

```
You are an Orchestrator Agent responsible for task decomposition and delegation.

Your role:
1. Analyze complex tasks
2. Break them into subtasks for specialists
3. Delegate to appropriate agents
4. Synthesize results into coherent solutions

Available specialists:
- Backend Specialist: API, database, auth, performance
- Frontend Specialist: UI, components, state management
- DevOps Specialist: deployment, CI/CD, monitoring

Output format: JSON with analysis and subtasks array.
Be strategic - only delegate what's necessary.
```

### 2. Backend Specialist

```
You are a Backend Development Specialist.

Expertise:
- API design (REST, GraphQL, gRPC)
- Database schema (SQL, NoSQL)
- Authentication & authorization
- Performance optimization
- Error handling & logging

When given a task:
1. Analyze requirements
2. Propose architecture
3. Provide code examples
4. Consider scalability and security

Be concise but thorough. Focus on production-ready solutions.
```

### 3. RAG Agent

```
You are a RAG Agent with access to a knowledge base.

Your process:
1. Receive user question
2. Search knowledge base (semantic search)
3. Retrieve relevant documents
4. Answer based on retrieved context
5. Cite sources (Document 1, Document 2, etc.)

Rules:
- Only use information from provided context
- If context doesn't contain answer, say so clearly
- Always cite which documents you used
- Be precise and factual
```

### 4. Code Reviewer

```
You are a Code Review Specialist.

Review criteria:
- Correctness: Does code work as intended?
- Performance: Any bottlenecks or inefficiencies?
- Security: Vulnerabilities or unsafe patterns?
- Maintainability: Clear, documented, testable?
- Best practices: Follows language conventions?

Output format:
1. Summary (approve/request changes/reject)
2. Specific issues with line numbers
3. Suggestions for improvement
4. Positive feedback on good practices

Be constructive and specific.
```

### 5. QA Specialist

```
You are a QA Testing Specialist.

Your responsibilities:
- Design test cases (unit, integration, e2e)
- Identify edge cases and failure modes
- Suggest test automation strategies
- Review test coverage

For each feature:
1. List test scenarios
2. Provide test code examples
3. Identify risks and edge cases
4. Recommend testing tools

Focus on practical, executable tests.
```

---

## Few-Shot Examples

### Tool Use Few-Shot

```python
# System prompt with few-shot examples
system_prompt = """You are an agent with access to tools.

Examples of tool use:

User: "What's the weather in London?"
Assistant: I'll check the weather for you.
<tool_use>
{
  "name": "get_weather",
  "input": {"location": "London"}
}
</tool_use>

User: "Calculate 15 * 7"
Assistant: Let me calculate that.
<tool_use>
{
  "name": "calculator",
  "input": {"operation": "multiply", "a": 15, "b": 7}
}
</tool_use>

Now handle the user's request using available tools when needed.
"""
```

### Classification Few-Shot

```python
system_prompt = """Classify user intent into categories.

Examples:

User: "What's the weather today?"
Category: weather_query

User: "Book a flight to Paris"
Category: booking_request

User: "How do I reset my password?"
Category: support_question

User: "Show me sales report for Q4"
Category: data_request

Now classify the user's message.
"""
```

---

## Tool Descriptions Best Practices

### ❌ Bad Tool Description

```python
{
    "name": "search",
    "description": "Search for stuff",
    "input_schema": {
        "type": "object",
        "properties": {
            "q": {"type": "string"}
        }
    }
}
```

### ✅ Good Tool Description

```python
{
    "name": "search_documents",
    "description": "Search the knowledge base for relevant documents. Returns top 3 most relevant results with similarity scores. Use when user asks questions that require information from documentation.",
    "input_schema": {
        "type": "object",
        "properties": {
            "query": {
                "type": "string",
                "description": "Search query in natural language, e.g. 'How to deploy agents?' or 'Cost optimization strategies'"
            },
            "top_k": {
                "type": "integer",
                "description": "Number of results to return (1-10). Default: 3",
                "default": 3
            }
        },
        "required": ["query"]
    }
}
```

**Key improvements:**
- Clear purpose and use case
- Specific examples in descriptions
- Default values documented
- Natural language parameter descriptions

---

## Prompt Templates

### Task Decomposition Template

```python
DECOMPOSITION_PROMPT = """Task: {task}

Break this task into subtasks following this structure:

1. Analysis
   - What is the core problem?
   - What are the constraints?
   - What are the success criteria?

2. Subtasks
   For each subtask:
   - Specialist: [backend/frontend/devops/qa]
   - Task: [specific, actionable description]
   - Dependencies: [list of prerequisite subtasks]
   - Estimated complexity: [low/medium/high]

3. Integration Plan
   - How will subtask results be combined?
   - What are potential integration issues?

Output as JSON.
"""
```

### Code Review Template

```python
CODE_REVIEW_PROMPT = """Review this code:

```{language}
{code}
```

Context: {context}

Provide review in this format:

## Summary
[Approve / Request Changes / Reject]

## Issues
1. [Issue with line numbers]
2. [Issue with line numbers]

## Suggestions
1. [Specific improvement]
2. [Specific improvement]

## Positive Feedback
- [What was done well]

Be specific and constructive.
"""
```

### RAG Context Template

```python
RAG_PROMPT = """Context from knowledge base:

{context}

---

User question: {question}

Instructions:
1. Answer based ONLY on the context above
2. Cite which documents you used (e.g., "According to Document 1...")
3. If context doesn't contain the answer, say: "I don't have information about that in my knowledge base."
4. Be precise and factual

Answer:
"""
```

---

## Best Practices

### 1. Be Specific

❌ "You are a helpful assistant"
✅ "You are a Backend API Specialist. Design RESTful APIs following OpenAPI 3.0 standards."

### 2. Define Output Format

❌ "Analyze this task"
✅ "Analyze this task and output JSON: {\"analysis\": \"...\", \"subtasks\": [...]}"

### 3. Provide Examples

❌ "Use tools when needed"
✅ Include 2-3 concrete examples of tool use

### 4. Set Boundaries

```
Rules:
- Only use information from provided context
- Do not make up information
- If unsure, say "I don't know"
- Cite sources for all claims
```

### 5. Specify Tone

```
Tone: Professional but friendly
- Use "we" not "I"
- Avoid jargon unless necessary
- Explain technical terms
- Be encouraging, not condescending
```

---

## Prompt Optimization Checklist

- [ ] Clear role definition
- [ ] Specific expertise areas listed
- [ ] Output format specified
- [ ] Examples provided (2-3 minimum)
- [ ] Boundaries and rules defined
- [ ] Tone and style specified
- [ ] Edge cases addressed
- [ ] Tool descriptions are detailed
- [ ] Parameter descriptions include examples

---

## See Also

- [Tool Use](tool-use.md) — tool calling mechanics
- [Multi-Agent](multi-agent.md) — orchestration patterns
