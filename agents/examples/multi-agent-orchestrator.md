---
type: guide
tags: [agents, examples, multi-agent, orchestration, delegation, python, advanced]
status: complete
updated: 2026-03-25
related:
  - ../multi-agent.md
  - ../tool-use.md
  - ../planning.md
---

# Multi-Agent Orchestrator

Система из оркестратора и специализированных агентов. Оркестратор декомпозирует задачу, делегирует подзадачи специалистам, собирает и синтезирует результаты.

---

## Overview

Этот пример показывает:
- Как декомпозировать сложную задачу на подзадачи
- Как делегировать работу специализированным агентам
- Как собрать результаты и синтезировать финальный ответ

**Сложность:** ⭐⭐⭐ Продвинутый  
**Строк кода:** ~250  
**Агенты:** Orchestrator, Backend Specialist, Frontend Specialist

---

## Architecture

```
User Task
    ↓
Orchestrator (Claude Sonnet 4)
    ├─→ Backend Specialist (Claude Sonnet 4)
    │       ↓
    │   Backend Solution
    │
    ├─→ Frontend Specialist (Claude Sonnet 4)
    │       ↓
    │   Frontend Solution
    │
    └─→ Synthesis
            ↓
        Final Answer
```

---

## Code

```python
#!/usr/bin/env python3
"""
Multi-Agent Orchestrator — система из оркестратора и специалистов.
Демонстрирует декомпозицию задач и делегирование между агентами.
"""

import os
import json
from typing import List, Dict, Any
from anthropic import Anthropic
from dotenv import load_dotenv

load_dotenv()

client = Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))

# ============================================================================
# SPECIALIST AGENTS
# ============================================================================

class BackendSpecialist:
    """Специалист по backend разработке"""
    
    def __init__(self):
        self.system_prompt = """You are a Backend Development Specialist.

Your expertise:
- API design (REST, GraphQL, gRPC)
- Database schema design (SQL, NoSQL)
- Authentication & authorization
- Performance optimization
- Error handling & logging

When given a task:
1. Analyze backend requirements
2. Propose architecture and tech stack
3. Provide code examples
4. Consider scalability and security

Be concise but thorough. Focus on practical solutions."""

    def solve(self, task: str) -> str:
        """Решить backend задачу"""
        print(f"\n🔧 Backend Specialist working on: {task[:80]}...")
        
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2048,
            system=self.system_prompt,
            messages=[{"role": "user", "content": task}]
        )
        
        solution = response.content[0].text
        print(f"✅ Backend solution ready ({len(solution)} chars)\n")
        
        return solution


class FrontendSpecialist:
    """Специалист по frontend разработке"""
    
    def __init__(self):
        self.system_prompt = """You are a Frontend Development Specialist.

Your expertise:
- UI/UX design patterns
- React, Vue, Svelte
- State management (Redux, Zustand, Pinia)
- Responsive design & accessibility
- Performance optimization (lazy loading, code splitting)

When given a task:
1. Analyze UI/UX requirements
2. Propose component structure
3. Provide code examples
4. Consider accessibility and performance

Be concise but thorough. Focus on modern best practices."""

    def solve(self, task: str) -> str:
        """Решить frontend задачу"""
        print(f"\n🎨 Frontend Specialist working on: {task[:80]}...")
        
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2048,
            system=self.system_prompt,
            messages=[{"role": "user", "content": task}]
        )
        
        solution = response.content[0].text
        print(f"✅ Frontend solution ready ({len(solution)} chars)\n")
        
        return solution


# ============================================================================
# ORCHESTRATOR
# ============================================================================

class Orchestrator:
    """Оркестратор — декомпозирует задачи и делегирует специалистам"""
    
    def __init__(self):
        self.backend_specialist = BackendSpecialist()
        self.frontend_specialist = FrontendSpecialist()
        
        self.system_prompt = """You are an Orchestrator Agent.

Your role:
1. Analyze complex tasks
2. Break them into subtasks for specialists:
   - Backend Specialist (API, database, auth, etc.)
   - Frontend Specialist (UI, components, state, etc.)
3. Delegate subtasks
4. Synthesize results into a coherent solution

When given a task, output JSON:
{
  "analysis": "Brief task analysis",
  "subtasks": [
    {"specialist": "backend", "task": "Specific backend task"},
    {"specialist": "frontend", "task": "Specific frontend task"}
  ]
}

Be strategic. Only delegate what's necessary."""

    def decompose(self, user_task: str) -> Dict[str, Any]:
        """Декомпозировать задачу на подзадачи"""
        print(f"\n🎯 Orchestrator analyzing task...\n")
        
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1024,
            system=self.system_prompt,
            messages=[{"role": "user", "content": user_task}]
        )
        
        # Парсим JSON из ответа
        text = response.content[0].text
        
        # Извлекаем JSON (может быть в markdown блоке)
        if "```json" in text:
            json_str = text.split("```json")[1].split("```")[0].strip()
        elif "```" in text:
            json_str = text.split("```")[1].split("```")[0].strip()
        else:
            json_str = text.strip()
        
        plan = json.loads(json_str)
        
        print(f"📋 Analysis: {plan['analysis']}")
        print(f"📝 Subtasks: {len(plan['subtasks'])}\n")
        
        return plan

    def delegate(self, subtasks: List[Dict[str, str]]) -> Dict[str, str]:
        """Делегировать подзадачи специалистам"""
        results = {}
        
        for subtask in subtasks:
            specialist = subtask["specialist"]
            task = subtask["task"]
            
            if specialist == "backend":
                results["backend"] = self.backend_specialist.solve(task)
            elif specialist == "frontend":
                results["frontend"] = self.frontend_specialist.solve(task)
            else:
                print(f"⚠️  Unknown specialist: {specialist}")
        
        return results

    def synthesize(self, user_task: str, analysis: str, results: Dict[str, str]) -> str:
        """Синтезировать финальный ответ из результатов специалистов"""
        print(f"\n🔄 Orchestrator synthesizing final answer...\n")
        
        synthesis_prompt = f"""Original task: {user_task}

Analysis: {analysis}

Specialist results:

{chr(10).join([f"**{k.upper()}:**{chr(10)}{v}{chr(10)}" for k, v in results.items()])}

---

Synthesize a coherent final answer that:
1. Integrates all specialist solutions
2. Provides a clear implementation roadmap
3. Highlights key decisions and tradeoffs
4. Includes next steps

Be concise but complete."""

        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2048,
            messages=[{"role": "user", "content": synthesis_prompt}]
        )
        
        final_answer = response.content[0].text
        
        return final_answer

    def run(self, user_task: str) -> str:
        """Полный цикл: декомпозиция → делегирование → синтез"""
        print("=" * 70)
        print("MULTI-AGENT ORCHESTRATOR")
        print("=" * 70)
        print(f"\n🧑 User Task: {user_task}\n")
        
        # 1. Декомпозиция
        plan = self.decompose(user_task)
        
        # 2. Делегирование
        results = self.delegate(plan["subtasks"])
        
        # 3. Синтез
        final_answer = self.synthesize(user_task, plan["analysis"], results)
        
        print("=" * 70)
        print("FINAL ANSWER")
        print("=" * 70)
        print(f"\n{final_answer}\n")
        
        return final_answer


# ============================================================================
# EXAMPLES
# ============================================================================

if __name__ == "__main__":
    orchestrator = Orchestrator()
    
    # Пример 1: Простое веб-приложение
    print("\n" + "=" * 70)
    print("EXAMPLE 1: Build a Todo App")
    print("=" * 70 + "\n")
    
    orchestrator.run(
        "Build a todo list web application with user authentication. "
        "Users should be able to create, edit, delete, and mark todos as complete. "
        "Include priority levels and due dates."
    )
    
    print("\n\n")
    
    # Пример 2: Dashboard с real-time данными
    print("=" * 70)
    print("EXAMPLE 2: Real-time Analytics Dashboard")
    print("=" * 70 + "\n")
    
    orchestrator.run(
        "Create a real-time analytics dashboard that displays user activity metrics. "
        "Include charts for daily active users, session duration, and feature usage. "
        "Data should update every 30 seconds without page refresh."
    )
```

---

## How It Works

### 1. Task Decomposition

Оркестратор анализирует задачу и разбивает на подзадачи:

```json
{
  "analysis": "Todo app requires backend API + frontend UI",
  "subtasks": [
    {
      "specialist": "backend",
      "task": "Design REST API for todo CRUD operations with auth"
    },
    {
      "specialist": "frontend",
      "task": "Build React UI with todo list, forms, and auth flow"
    }
  ]
}
```

### 2. Delegation

Каждая подзадача отправляется соответствующему специалисту:

```python
results = {
    "backend": backend_specialist.solve(backend_task),
    "frontend": frontend_specialist.solve(frontend_task)
}
```

### 3. Synthesis

Оркестратор собирает результаты и создаёт финальный ответ:

```python
final_answer = orchestrator.synthesize(
    user_task=original_task,
    analysis=plan["analysis"],
    results={"backend": "...", "frontend": "..."}
)
```

---

## Running

```bash
# 1. Установите зависимости
pip install anthropic python-dotenv

# 2. Создайте .env
echo "ANTHROPIC_API_KEY=sk-ant-..." > .env

# 3. Запустите
python multi-agent-orchestrator.py
```

**Ожидаемый вывод:**
```
======================================================================
MULTI-AGENT ORCHESTRATOR
======================================================================

🧑 User Task: Build a todo list web application...

🎯 Orchestrator analyzing task...

📋 Analysis: Todo app requires backend API + frontend UI
📝 Subtasks: 2

🔧 Backend Specialist working on: Design REST API for todo CRUD...
✅ Backend solution ready (1523 chars)

🎨 Frontend Specialist working on: Build React UI with todo list...
✅ Frontend solution ready (1847 chars)

🔄 Orchestrator synthesizing final answer...

======================================================================
FINAL ANSWER
======================================================================

[Integrated solution with backend + frontend + implementation roadmap]
```

---

## Cost Estimation

**Per task with 2 specialists:**
- Orchestrator decompose: ~500 input tokens = $0.0015
- Backend specialist: ~2000 tokens = $0.006
- Frontend specialist: ~2000 tokens = $0.006
- Orchestrator synthesis: ~3000 input + 1000 output = $0.024

**Total per task:** ~$0.037 (3.7 cents)

**100 tasks:** ~$3.70

---

## Extending

### 1. Добавление нового специалиста

```python
class DevOpsSpecialist:
    """Специалист по DevOps и инфраструктуре"""
    
    def __init__(self):
        self.system_prompt = """You are a DevOps Specialist.

Your expertise:
- Docker & Kubernetes
- CI/CD pipelines (GitHub Actions, GitLab CI)
- Cloud infrastructure (AWS, GCP, Azure)
- Monitoring & logging (Prometheus, Grafana, ELK)
- Security & compliance

Provide practical, production-ready solutions."""

    def solve(self, task: str) -> str:
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2048,
            system=self.system_prompt,
            messages=[{"role": "user", "content": task}]
        )
        return response.content[0].text

# Добавить в Orchestrator
self.devops_specialist = DevOpsSpecialist()

# Обновить system_prompt оркестратора
# - DevOps Specialist (Docker, CI/CD, cloud, monitoring)
```

### 2. Параллельное выполнение

```python
import asyncio
from anthropic import AsyncAnthropic

async_client = AsyncAnthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))

async def delegate_parallel(self, subtasks: List[Dict[str, str]]) -> Dict[str, str]:
    """Параллельное делегирование для ускорения"""
    tasks = []
    
    for subtask in subtasks:
        specialist = subtask["specialist"]
        task = subtask["task"]
        
        if specialist == "backend":
            tasks.append(("backend", self.backend_specialist.solve_async(task)))
        elif specialist == "frontend":
            tasks.append(("frontend", self.frontend_specialist.solve_async(task)))
    
    results = {}
    for name, coro in tasks:
        results[name] = await coro
    
    return results
```

### 3. Iterative refinement

```python
def refine(self, solution: str, feedback: str) -> str:
    """Итеративное улучшение решения"""
    refine_prompt = f"""Original solution:
{solution}

Feedback:
{feedback}

Refine the solution based on feedback. Keep what works, improve what doesn't."""

    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=2048,
        messages=[{"role": "user", "content": refine_prompt}]
    )
    
    return response.content[0].text
```

---

## Best Practices

1. **Clear specialist roles** — каждый специалист должен иметь чёткую зону ответственности
2. **Structured communication** — используйте JSON для передачи задач между агентами
3. **Context isolation** — специалисты не должны видеть результаты друг друга (только оркестратор)
4. **Synthesis is key** — оркестратор должен интегрировать результаты, не просто конкатенировать
5. **Error handling** — добавьте retry логику для failed delegations

---

## Patterns

### Pattern 1: Sequential Delegation
```
Task → Backend → Frontend → DevOps → Synthesis
```
Используйте когда задачи зависят друг от друга.

### Pattern 2: Parallel Delegation
```
Task → [Backend, Frontend, DevOps] → Synthesis
```
Используйте когда задачи независимы.

### Pattern 3: Hierarchical Delegation
```
Task → Orchestrator
         ├→ Backend Orchestrator
         │    ├→ API Specialist
         │    └→ Database Specialist
         └→ Frontend Orchestrator
              ├→ UI Specialist
              └→ State Specialist
```
Используйте для очень сложных задач.

---

## Troubleshooting

**JSON parsing error:**
- Добавьте более строгий промпт: "Output ONLY valid JSON, no markdown"
- Используйте regex для извлечения JSON из текста

**Specialists going off-topic:**
- Уточните system prompts
- Добавьте примеры (few-shot) в промпты

**Synthesis is just concatenation:**
- Улучшите synthesis prompt: "Integrate, don't concatenate"
- Попросите оркестратор выделить ключевые решения и tradeoffs

---

## See Also

- [Multi-Agent Systems](../multi-agent.md) — теория multi-agent архитектур
- [Planning](../planning.md) — стратегии планирования для агентов
- [Tool Use](../tool-use.md) — как добавить инструменты специалистам
