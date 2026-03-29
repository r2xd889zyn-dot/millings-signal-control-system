# Millings Signal-Control System

## Overview

The Millings Signal-Control System is an AI infrastructure layer that controls what a system is allowed to recognize as valid input ("reality") before execution occurs.

Unlike traditional architectures that validate after processing, this system enforces a pre-execution Signal Gate that prevents invalid inputs from propagating through identity, permissions, and execution layers.

---

## Core ArchitectureInput
↓
Signal Gate (Reality Validation)
↓
IAM (Identity)
↓
Permissions (Authority)
↓
Execution
↓
Audit (Traceability)
↓
Memory (Drift + Continuity)---

## Key Components

### 1. Signal Gate
- Validates input before processing
- Computes signal score and drift
- Blocks invalid or low-confidence inputs

### 2. IAM (Identity Layer)
- Verifies identity
- Issues session tokens

### 3. Permission Engine
- Role-based access control (RBAC)
- Scope enforcement

### 4. Audit Layer
- Append-only logging
- Tracks system behavior and failures

### 5. Memory + Drift Engine
- Embeddings-based similarity tracking
- Detects drift and anomaly patterns

---

## Why This Matters

Most AI systems optimize decisions.

This system controls:
- what is considered valid input
- whether execution should occur at all

---

## Installation

### Backend

```bash
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload
⸻

Frontendcd frontend
npm install
npm run devDocker Deploymentdocker-compose up --buildAPI EndpointsEndpoint
Description
POST /login
Full pipeline execution
POST /signal
Signal validation only
GET /metrics
Drift + system stats
Environment Variables

Create .env:Deployment

Render
	•	Connect GitHub repo
	•	Use Docker runtime
	•	Deploy automatically

AWS
	•	ECS (containers)
	•	RDS (Postgres)
	•	ALB (routing)

⸻

Vision

“We don’t control what AI does.
We control what it is allowed to believe before it acts.”---

# 🐳 docker-compose.yml

```yaml
version: "3.8"

services:
  backend:
    build: ./backend
    ports:
      - "10000:10000"
    env_file:
      - .env
    depends_on:
      - db

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: signal_db
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"backend/app/main.py (FINAL ENTRY)from fastapi import FastAPI
from signal_gate import signal_gate
from iam import verify_identity, issue_passport
from permissions import get_permissions
from audit import log_event

app = FastAPI()

@app.post("/login")
def login(payload: dict):
    user_input = payload["content"]

    signal = signal_gate(user_input)

    if not signal["valid"]:
        log_event("UNKNOWN", "BLOCKED_AT_SIGNAL", "FAIL")
        return {"status": "REALITY_REJECTED", **signal}

    identity = verify_identity(user_input)

    if identity["status"] != "SUCCESS":
        return {"status": "CRACK_DETECTED"}

    passport = issue_passport(identity)
    scopes = get_permissions(identity["uid"])

    return {
        "status": "ROAD_OPEN",
        "token": passport["passport"],
        "access": scopes,
        "signal_score": signal["score"],
        "drift": signal["drift"]
    }🎯 FRONTEND ENTRY (frontend/pages/index.js)import { useState } from "react";

export default function Home() {
  const [input, setInput] = useState("");
  const [data, setData] = useState(null);

  const run = async () => {
    const res = await fetch("http://localhost:10000/login", {
      method: "POST",
      headers: {"Content-Type": "application/json"},
      body: JSON.stringify({content: input})
    });

    const result = await res.json();
    setData(result);
  };

  return (
    <div className="p-6">
      <h1>Signal Control Dashboard</h1>

      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
      />

      <button onClick={run}>Run</button>

      {data && <pre>{JSON.stringify(data, null, 2)}</pre>}
    </div>
  );
}
