# L3 Engineer — Complete Troubleshooting Guide

> **Purpose**: End-to-end troubleshooting reference for every service, flow, and failure mode in Emergent.
> **Audience**: L3 support engineers with no assumed prior knowledge.
> **How to use**: Find the flow or symptom you're debugging, follow the RCA steps, use the commands listed.

---

## TABLE OF CONTENTS

1. [How to Read This Guide](#1-how-to-read-this-guide)
2. [Toolbox — Every Tool You Need](#2-toolbox--every-tool-you-need)
3. [Layer 0 — Frontend (E1ectron + Browser)](#3-layer-0--frontend-e1ectron--browser)
4. [Layer 1 — Gateway](#4-layer-1--gateway)
5. [Layer 2 — app-service](#5-layer-2--app-service)
6. [Layer 3 — agent-service](#6-layer-3--agent-service)
7. [Layer 4 — cortex (Temporal Workflows)](#7-layer-4--cortex-temporal-workflows)
8. [Layer 5 — llm-proxy](#8-layer-5--llm-proxy)
9. [Layer 6 — deployer](#9-layer-6--deployer)
10. [Layer 7 — envcore + Pods](#10-layer-7--envcore--pods)
11. [Databases — PostgreSQL, Redis, BigTable](#11-databases--postgresql-redis-bigtable)
12. [Networking & DNS](#12-networking--dns)
13. [Cloudflare & Custom Domains](#13-cloudflare--custom-domains)
14. [Payments & Billing](#14-payments--billing)
15. [Mobile Builds (iOS/Android)](#15-mobile-builds-iosandroid)
16. [Security Issues](#16-security-issues)
17. [Observability — How to Find Anything](#17-observability--how-to-find-anything)
18. [User Code Bugs — How to Debug App-Level Issues](#18-user-code-bugs--how-to-debug-app-level-issues)
19. [RCA Templates](#19-rca-templates)
20. [Master Flowcharts](#20-master-flowcharts)

---

## 1. HOW TO READ THIS GUIDE

Each section follows this structure:

```
WHAT THIS LAYER DOES
    → Brief reminder of the service's job

ISSUES AT THIS LAYER
    → Every failure mode, A-Z

FOR EACH ISSUE:
    Symptoms        → What the user sees / what you see in logs
    Root Causes     → Why this happens
    Blast Radius    → Which other services break because of this
    RCA Steps       → Exact steps to find the root cause, in order
    Commands        → Copy-paste commands
    Fix             → How to resolve it
    Escalate if     → When to hand off to engineering
```

**Golden rule of L3**: Always confirm the symptom before forming a hypothesis. Never guess.

---

## 2. TOOLBOX — EVERY TOOL YOU NEED

### 2.1 Browser DevTools (F12) — For Frontend Issues

```
Open: F12 or Right-click → Inspect

Tabs you'll use:
  Network tab   → See every HTTP request, response, status code, headers, body
  Console tab   → JavaScript errors, log messages
  Application   → LocalStorage, Cookies (where JWT tokens are stored)
  Elements      → Inspect DOM if UI looks broken

Key workflow:
  1. F12 → Network tab
  2. Reproduce the issue
  3. Look for red requests (4xx/5xx)
  4. Click the failing request
  5. Check: Headers tab → what was sent
             Response tab → what came back
             Preview tab → formatted response body
```

### 2.2 Redash — Query Prod Databases

```
URL: https://redash.internal-apps.emergentagent.com

Data Sources:
  DS 10  → emergent prod (jobs, users, credits, environments, heartbeats)
  DS 5   → deployer prod (apps, deployments, pipeline_runs, domains, secrets)

Important: First query returns status=1 (pending). Click Run again to get results.
```

### 2.3 Loki — Log Aggregation

```bash
# Script (preferred):
./.claude/skills/loki-logs/scripts/loki-query.sh <service> [filter] [time_range] [env] [limit]

# Examples:
./.claude/skills/loki-logs/scripts/loki-query.sh app-service "error" 1h
./.claude/skills/loki-logs/scripts/loki-query.sh deployer "job_id_here" 6h
./.claude/skills/loki-logs/scripts/loki-job.sh <job_id>    # Search across all services

# Manual curl (prod):
ORG_ID="emergent-default-app-cluster|emergent-default-cluster|emergent-default-cluster-1|emergent-default-cluster-2|emergent-default-cluster-3|emergent-default-cluster-4|emergent-default-cluster-5|emergent-default-cluster-6|emergent-default-cluster-7|emergent-default-cluster-8|emergent-default-cluster-9|emergent-default-cluster-internal|env-cluster|env-worker-cluster|env-worker-cluster-1|env-worker-cluster-10|env-worker-cluster-11|env-worker-cluster-12|env-worker-cluster-2|env-worker-cluster-3|env-worker-cluster-4|env-worker-cluster-5|env-worker-cluster-6|env-worker-cluster-7|env-worker-cluster-8|env-worker-cluster-9|env-worker-cluster-internal"

curl -G -s \
  -H "X-Scope-OrgID: $ORG_ID" \
  "http://loki.internal-apps.emergentagent.com/loki/api/v1/query_range" \
  --data-urlencode 'query={app="app-service"}' \
  --data-urlencode "start=$(date -u -v-1H +%s)000000000" \
  --data-urlencode "end=$(date -u +%s)000000000" | jq '.data.result[].values[][1]' -r

# Retention: 24-48 hours only
```

### 2.4 gcloud — Cloud Run & Build Logs

```bash
# Quick script:
python3 ./.claude/skills/gcloud-logs/scripts/gcloud_logs.py <service> [options]
python3 ./.claude/skills/gcloud-logs/scripts/gcloud_logs.py cortex-agentsdk -j <job_id> -f 6h

# Set project:
gcloud config set project emergent-default       # prod
gcloud config set project emergent-client-1      # dev/staging/eph

# List Cloud Run services:
gcloud run services list --region=us-central1

# Stream logs for a service:
gcloud run services logs tail <service-name> --region=us-central1 --project=emergent-default

# Get build logs:
gcloud builds list --region=us-central1 --limit=10
gcloud builds log <build-id> --region=us-central1 --stream
```

### 2.5 kubectl — Kubernetes

```bash
# Namespaces:
#   emergent-core       → core services (app-service, deployer, cortex)
#   emergent-agents-env → agent pods (one per job)
#   customers-app       → user-deployed apps

# Get pods:
kubectl get pods -n emergent-core
kubectl get pods -n emergent-agents-env | grep <job_id>
kubectl get pods -n customers-app | grep <app_name>

# Describe a pod (events, resource limits, crash reason):
kubectl describe pod <pod-name> -n <namespace>

# Get logs:
kubectl logs <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --previous    # logs from crashed container
kubectl logs <pod-name> -n <namespace> -f            # follow/tail

# Exec into a pod:
kubectl exec -it <pod-name> -n <namespace> -- /bin/bash

# Get all contexts (clusters):
kubectl config get-contexts

# Switch context:
kubectl config use-context gke_emergent-default_us-central1_target-1

# Get events (shows OOMKilled, eviction, etc):
kubectl get events -n <namespace> --sort-by='.lastTimestamp' | tail -30

# Get resource usage:
kubectl top pods -n <namespace>
kubectl top nodes

# Check persistent volumes:
kubectl get pvc -n emergent-agents-env | grep <job_id>
kubectl describe pvc <pvc-name> -n emergent-agents-env
```

### 2.6 Deployer Internal API

```bash
BASE="http://deployer.internal-apps.emergentagent.com/api/v1"
USER_ID="<user_id>"

# App details:
curl -s "$BASE/apps/<app_name>" -H "X-User-ID: $USER_ID" | jq .

# Latest deployment:
curl -s "$BASE/deploy/<app_name>/latest" -H "X-User-ID: $USER_ID" | jq .

# App logs:
curl -s "$BASE/apps/logs?app=<app_name>" -H "X-User-ID: $USER_ID" | jq .

# Domain status:
curl -s "$BASE/domains" -H "X-User-ID: $USER_ID" | jq .

# Verify domain:
curl -s -X POST "$BASE/domains/<domain>/verify" -H "X-User-ID: $USER_ID" | jq .

# Pod status:
curl -s "$BASE/pods/<job_id>" -H "X-User-ID: $USER_ID" | jq .

# Secrets (filter for DB/Mongo):
curl -s "$BASE/secrets/<app_name>" -H "X-User-ID: $USER_ID" | \
  python3 -c "import sys,json; [print(f'{s[\"key\"]}={s[\"value\"]}') for s in json.load(sys.stdin).get('secrets',[]) if 'MONGO' in s.get('key','') or 'DB' in s.get('key','')]"
```

### 2.7 MCP Deployer Tools (in Claude Code)

```
mcp__deployer__fetch_app_details(identifier="<app_name>")
mcp__deployer__fetch_app_logs(identifier="<app_name>")
mcp__deployer__describe_app_pod(identifier="<app_name>")
mcp__deployer__fetch_recent_deployments(identifier="<app_name>")
mcp__deployer__fetch_job_logs(job_id="<job_id>")
mcp__deployer__fetch_pipeline_logs(run_id="<run_id>")
mcp__deployer__fetch_domain_status(domain="<domain>")
mcp__deployer__run_mongo_query(app="<app>", operation="find", collection="<col>", query={})
mcp__deployer__get_deployment_manifest(identifier="<app>", filename="entrypoint.sh")
mcp__deployer__update_deployment_manifest(identifier="<app>", filename="entrypoint.sh", content="...")
mcp__deployer__fetch_app_secrets(identifier="<app>")
```

### 2.8 Redis

```bash
# Exec into redis pod:
kubectl exec -it redis-0 -n emergent-core -- redis-cli

# Useful commands:
KEYS auth_token:*                          # List cached auth tokens
GET auth_token:<token>                     # Check token cache
KEYS env_operations_lock:*                 # Check job locks
TTL auth_token:<token>                     # Check cache expiry
DEL auth_token:<token>                     # Force token cache flush
KEYS replay:in_queue:*                     # Check replay queue
LLEN replay_queue                          # Queue depth
```

### 2.9 PostgreSQL (Direct)

```bash
# Connect (local/staging):
psql "postgresql://postgres:mysecretpassword@postgres:5432/emergent"

# Connect (eph):
psql "postgresql://postgres:pYOjidM5JFUMJZp@10.0.2.3:6544/postgres-<eph-name>"

# Common queries (see each section for specific queries)
\dt                         # List tables
\d jobs                     # Describe table
\x                          # Toggle expanded output
\q                          # Quit
```

---

## 3. LAYER 0 — FRONTEND (E1ECTRON + BROWSER)

### What this layer does
E1ectron is the React/TypeScript UI. It renders the chat interface, sends API requests to the backend with JWT auth, and displays responses.

---

### Issue 0-A: Page loads but nothing happens when user clicks Send

**Symptoms**
- User clicks Send, spinner appears, then disappears — no response
- No error shown in the UI

**Root Causes**
1. API request silently failing (network error)
2. JWT token expired — 401 returned but not shown to user
3. Wrong API base URL in frontend config
4. CORS error blocking the request

**RCA Steps**
```
1. F12 → Network tab
2. Reproduce (click Send)
3. Find the POST request to /jobs/v0/submit-queue/
4. Check status code:
   - 401 → auth token problem (see Issue 0-B)
   - 403 → permissions issue
   - 0 or CORS error → see Issue 0-D
   - 5xx → backend issue (see Layer 2)
   - No request at all → JavaScript error (check Console tab)

5. F12 → Console tab
   - Look for red errors
   - TypeError usually means frontend code crashed before sending
```

**Commands**
```javascript
// In browser console — check if token exists:
localStorage.getItem('supabase.auth.token')
// or
JSON.parse(localStorage.getItem('sb-<project>-auth-token'))?.access_token
```

---

### Issue 0-B: User gets logged out unexpectedly / 401 errors

**Symptoms**
- User is redirected to login page
- Network tab shows 401 on API calls

**Root Causes**
1. Supabase session expired (default TTL varies)
2. JWT_SECRET changed on backend (all tokens invalid)
3. Token refresh failed (network error during refresh)

**RCA Steps**
```
1. F12 → Application tab → Local Storage
2. Find the auth token entry (key: sb-*-auth-token)
3. Check expires_at field — is it in the past?
4. F12 → Network tab → look for POST to /auth/token (refresh attempt)
   - If refresh returns 401 → refresh token also expired
   - If refresh returns 500 → backend auth service down
5. Check app-service logs for JWT validation errors:
```
```bash
./.claude/skills/loki-logs/scripts/loki-query.sh app-service "JWT" 1h
```

**Fix**
- User just needs to log in again (expired token — normal)
- If widespread: check if JWT_SECRET changed in app-service env vars

---

### Issue 0-C: UI shows wrong data / stale data

**Symptoms**
- Job shows old status
- Credits don't update after payment

**Root Causes**
- Frontend cache (RTK Query caches responses)
- User not refreshing after completing action

**RCA Steps**
```
1. F12 → Network tab → check if request is going out at all
   (RTK Query may serve from cache without network request)
2. Hard reload: Cmd+Shift+R (Mac) / Ctrl+Shift+R (Windows)
3. Check request timestamp — is it recent?
```

---

### Issue 0-D: CORS error

**Symptoms**
- Console shows: "Access to fetch at 'https://api.emergent.test' from origin has been blocked by CORS policy"
- Network tab shows OPTIONS preflight returning 4xx

**Root Causes**
1. Gateway CORS config missing for this origin
2. Accessing wrong API URL (local vs prod mismatch)
3. Frontend env var VITE_API_BASE_URL pointing to wrong URL

**RCA Steps**
```
1. F12 → Console → copy the full CORS error
2. Check what URL the frontend is calling:
   F12 → Application → Local Storage → look for API_URL
   or check Network tab → failing request URL
3. Compare to VITE_API_BASE_URL in E1ectron build config
4. Check gateway CORS headers in response:
   F12 → Network → preflight OPTIONS request → Response Headers
   Should see: Access-Control-Allow-Origin: *
```

**Fix**
```bash
# Check gateway CORS config:
kubectl get configmap gateway-config -n emergent-core -o yaml | grep -A5 cors
```

---

### Issue 0-E: White screen / app crashes on load

**Symptoms**
- Blank page after login
- Console shows JavaScript error on startup

**RCA Steps**
```
1. F12 → Console → find the first red error
2. Common causes:
   - "Cannot read properties of undefined" → API returned unexpected shape
   - "ChunkLoadError" → static asset failed to load (CDN issue)
   - Network requests failing for static .js files → check CDN/hosting
3. Check E1ectron deployment:
```
```bash
# Check if frontend build is healthy:
curl -I https://app.emergent.test
# Should return: HTTP/2 200
```

---

### Issue 0-F: How to trace a frontend bug to exact backend code

```
This is the most important frontend skill.

STEP 1: F12 → Network tab → find the failing request
         Note: URL path, method, status code

STEP 2: Click the request → Headers tab
         Note the exact URL path: e.g., POST /api/v0/jobs/submit-queue/

STEP 3: Map URL to backend file:
         /api/v0/jobs/*         → emergent/app/apis/v0/job_api.py
         /api/v0/payments/*     → emergent/app/apis/payments_api.py
         /api/v1/deploy/*       → deployer/internal/httpapi/deploy.go
         /api/v1/domains/*      → deployer/internal/httpapi/domains.go
         /auth/*                → emergent/app/apis/custom_auth.py
         /api/v0/android/*      → emergent/app/apis/android_api.py
         /api/v0/ios/*          → emergent/app/apis/ios_api.py

STEP 4: Click request → Response tab
         Read the error message — it usually says exactly what failed

STEP 5: F12 → Network tab → click request → Timing tab
         See where time was spent (waiting = backend slow)

STEP 6: Match error message to backend code:
         Search for the exact error string in the backend:
```
```bash
grep -r "error message here" emergent/app/
grep -r "error message here" deployer/
```

---

## 4. LAYER 1 — GATEWAY

### What this layer does
The gateway is the reverse proxy/load balancer that receives all external HTTPS traffic and routes it to the correct internal service. In production it's a Kubernetes Gateway API resource managed by GKE.

---

### Issue 1-A: 502 Bad Gateway for all requests

**Symptoms**
- All API calls return 502
- Widespread — affects every user

**Root Causes**
1. Target service (app-service) is down or crashing
2. Gateway misconfiguration after deployment
3. Network policy blocking gateway → service traffic

**Blast Radius**: All users, all features

**RCA Steps**
```
1. First: is the gateway itself up?
   curl -I https://api.emergent.test
   - 502 = gateway up, backend down
   - Connection refused = gateway itself is down

2. Check which service is returning 502:
   The gateway just forwards — the 502 comes FROM the backend service.
   Check app-service:
```
```bash
kubectl get pods -n emergent-core | grep app-service
kubectl describe pod <app-service-pod> -n emergent-core
```
```
3. If app-service pods are CrashLoopBackOff → see Layer 2 issues
4. If app-service pods are Running → check service health:
   kubectl exec -it <app-service-pod> -n emergent-core -- curl localhost:8003/status
```

---

### Issue 1-B: 503 Service Unavailable

**Symptoms**
- Intermittent 503 (not constant)
- Usually during traffic spikes

**Root Causes**
1. All app-service pods busy / at max connections
2. Autoscaling lagging behind traffic
3. One pod is bad, others healthy (partial 503)

**RCA Steps**
```bash
# Check pod count and resource usage:
kubectl get pods -n emergent-core | grep app-service
kubectl top pods -n emergent-core | grep app-service

# Check HPA (Horizontal Pod Autoscaler):
kubectl get hpa -n emergent-core
kubectl describe hpa app-service -n emergent-core
```

---

### Issue 1-C: 404 for valid endpoint

**Symptoms**
- User reports endpoint not working
- Network tab shows 404

**Root Causes**
1. Route not configured in gateway routes.yaml
2. Path mismatch (trailing slash, different version)
3. New endpoint deployed but gateway not updated

**RCA Steps**
```bash
# Check current routes:
cat gateway/resources/routes.yaml
# or
kubectl get httproute -n emergent-core -o yaml | grep -A5 "path:"

# Test directly (bypassing gateway):
kubectl exec -it <any-pod> -n emergent-core -- curl http://app-service:8003/<path>
```

---

### Issue 1-D: Requests timing out at gateway

**Symptoms**
- Requests hang for 30-60s then timeout
- No response, not a 5xx

**Root Causes**
1. Backend service is hanging (not responding)
2. Gateway timeout config too short for long-running operations

**RCA Steps**
```bash
# Check gateway timeout settings:
kubectl get gateway -n emergent-core -o yaml | grep -i timeout

# Test backend directly:
kubectl exec -it <any-pod> -n emergent-core -- \
  curl -m 5 http://app-service:8003/status
```

---

## 5. LAYER 2 — APP-SERVICE

### What this layer does
app-service (Python FastAPI, port 8003) is the core business logic service. It owns user accounts, jobs, credits, payments, and orchestrates requests to agent-service.

---

### Issue 2-A: Job submission returns 400 Bad Request

**Symptoms**
- User gets error when trying to start a job
- Network tab: POST /jobs/v0/submit-queue/ → 400

**Root Causes**
1. Invalid payload (missing required field)
2. Rate limit exceeded
3. Feature flag check failing
4. Invite check failing (user not invited)

**RCA Steps**
```
1. F12 → Network → click the failing request → Response tab
   Read the exact error message field

2. Common 400 reasons:
   "rate limit exceeded"     → User hitting too many requests
   "insufficient credits"    → Credits check (see Issue 2-D)
   "not authorized"          → Invite/permission check

3. Check app-service logs for the exact error:
```
```bash
./.claude/skills/loki-logs/scripts/loki-query.sh app-service "400" 30m
```

---

### Issue 2-B: Job stuck in PENDING forever

**Symptoms**
- Job status never changes from PENDING
- User waits but nothing happens

**Root Causes**
1. agent-service is down — app-service queued it but nobody picked it up
2. Redis Pub/Sub broken — message published but not consumed
3. No warm pods available for the job
4. Deadlock on job lock in Redis

**Blast Radius**: All async jobs fail silently

**RCA Steps**
```
1. Check job in Redash (DS 10):
```
```sql
SELECT id, status, state, created_at, updated_at, payload->>'is_cloud' as is_cloud
FROM jobs
WHERE id = '<job_id>';
```
```
   status=PENDING + state=ENV_INITIATED = never reached agent-service

2. Check agent-service health:
```
```bash
kubectl get pods -n emergent-core | grep agent-service
kubectl logs deployment/agent-service -n emergent-core --tail=50
```
```
3. Check Redis connectivity:
```
```bash
kubectl exec -it redis-0 -n emergent-core -- redis-cli PING
# Should return: PONG

# Check if job message is in queue:
kubectl exec -it redis-0 -n emergent-core -- redis-cli LLEN job_queue
```
```
4. Check for pod availability:
```
```bash
kubectl get pods -n emergent-agents-env | grep -c Running
# If 0 warm pods → no capacity
```

---

### Issue 2-C: Job stuck in IN_PROGRESS forever

**Symptoms**
- Job never completes
- User sees spinner indefinitely

**Root Causes**
1. Agent-service crashed mid-execution
2. LLM request hanging (llm-proxy timeout)
3. Tool execution infinite loop
4. Cortex workflow stuck (see Layer 4)
5. Job hit 21-minute timeout but status not updated

**RCA Steps**
```
1. Check job state timeline:
```
```sql
SELECT state, created_at, metadata
FROM job_audits
WHERE job_id = '<job_id>'
ORDER BY created_at ASC;
```
```
2. Check heartbeats (for cortex jobs):
```
```sql
SELECT job_id, status, created_at
FROM heartbeats
WHERE job_id = '<job_id>'
ORDER BY created_at DESC
LIMIT 10;
```
```
   Last heartbeat > 5 minutes ago = job is frozen

3. Find which pod was running the job:
```
```sql
SELECT pod_name, pod_ip, status, last_accessed_at
FROM pod_and_slug_infos
WHERE slug = '<job_id>';
```
```bash
kubectl logs <pod_name> -n emergent-agents-env --tail=100
```
```
4. Check LLM proxy for hanging requests:
```
```bash
./.claude/skills/loki-logs/scripts/loki-query.sh llm-proxy-service "<job_id>" 2h
```

---

### Issue 2-D: "Insufficient credits" error

**Symptoms**
- User gets credit error when starting job
- User claims they have credits

**Root Causes**
1. Credits genuinely at 0 (check all 3 pools)
2. Subscription credits expired but ecu credits exist (order of deduction)
3. Cost estimate too high for available balance
4. Bug in credit calculation

**RCA Steps**
```sql
-- Check all credit pools (DS 10):
SELECT user_id, ecu, monthly_credits, daily_credits,
       monthly_credits_refresh_date, daily_credits_last_updated
FROM credits
WHERE user_id = '<user_id>';

-- Check recent transactions:
SELECT type, reference_type, ecu, created_at
FROM transactions
WHERE user_id = '<user_id>'
ORDER BY created_at DESC
LIMIT 20;

-- Check subscription status:
SELECT plan_id, status, start_date, end_date
FROM user_subscriptions
WHERE user_id = '<user_id>'
ORDER BY created_at DESC;
```

**Fix options**
```
1. If monthly_credits = 0 but subscription active → subscription refresh bug
   → Escalate to engineering

2. If ecu = 0 but user paid → check payment webhook (see Layer 14)

3. If all pools = 0 → user genuinely has no credits
   → Explain credit system, suggest top-up

4. If subscription expired → user needs to resubscribe
```

---

### Issue 2-E: app-service pod CrashLoopBackOff

**Symptoms**
- All API calls failing
- kubectl shows app-service in CrashLoopBackOff

**Root Causes**
1. Database connection string wrong (PostgreSQL down)
2. Missing required environment variable
3. Python import error (bad deployment)
4. OOMKilled (out of memory)

**RCA Steps**
```bash
# Check why it's crashing:
kubectl logs deployment/app-service -n emergent-core --previous

# Check resource limits:
kubectl describe pod <app-service-pod> -n emergent-core | grep -A10 Limits

# Check if DB is reachable:
kubectl exec -it <any-other-pod> -n emergent-core -- \
  psql "postgresql://postgres:mysecretpassword@postgres:5432/emergent" -c "SELECT 1;"

# Check events:
kubectl get events -n emergent-core --sort-by='.lastTimestamp' | grep app-service
```

**Escalate if**: OOMKilled — engineering needs to increase memory limits

---

### Issue 2-F: Slow API responses (> 5 seconds)

**Symptoms**
- API calls take very long
- User sees slow loading

**Root Causes**
1. Database query N+1 problem (missing index)
2. External service slow (Supabase, Stripe)
3. Redis cache miss causing repeated DB queries
4. High CPU on pods

**RCA Steps**
```bash
# Check pod CPU/memory:
kubectl top pods -n emergent-core

# Check slow queries in PostgreSQL:
# Run in psql:
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

# Check app-service response times in Loki:
./.claude/skills/loki-logs/scripts/loki-query.sh app-service "duration" 30m | grep -E "[0-9]+ms"
```

---

## 6. LAYER 3 — AGENT-SERVICE

### What this layer does
agent-service (Python FastAPI, port 8009) executes the AI agent logic — it receives jobs, calls the LLM, executes tools, and manages Kubernetes pods for cloud jobs.

---

### Issue 3-A: Agent never starts executing after job queued

**Symptoms**
- Job in IN_PROGRESS but no activity
- No trajectories created

**Root Causes**
1. Pub/Sub message not consumed
2. No warm pods in the pool
3. EnvTaskProcessor crashed before pod assignment

**RCA Steps**
```bash
# Check agent-service logs:
./.claude/skills/loki-logs/scripts/loki-query.sh emergent-agents "<job_id>" 1h

# Check pod pool:
kubectl get pods -n emergent-agents-env | grep -c Running
kubectl get pods -n emergent-agents-env | grep Pending

# Check if pod was assigned:
```
```sql
SELECT pod_name, pod_ip, slug, status, created_at
FROM pod_and_slug_infos
WHERE slug = '<job_id>';
```

---

### Issue 3-B: Agent exits after 1 LLM call

**Symptoms**
- Job completes instantly
- Response is minimal / incomplete
- Only 1 trajectory entry

**Root Causes**
1. Agent chose `finish` tool immediately (task too simple or prompt mismatch)
2. LLM returned stop_reason=end_turn after first message
3. Max iterations = 1 (wrong config)
4. Context limit hit immediately (huge message history)

**RCA Steps**
```sql
-- Check trajectory entries:
SELECT id, type, content, created_at
FROM trajectories
WHERE job_id = '<job_id>'
ORDER BY created_at ASC;

-- Check agent config used:
SELECT metadata
FROM job_metadata
WHERE job_id = '<job_id>';
```
```bash
# Check LLM response for this job:
./.claude/skills/loki-logs/scripts/loki-query.sh cortex-agentsdk "<job_id>" 2h | grep -i "stop_reason"
```

---

### Issue 3-C: Agent running in wrong mode (LocalApp vs Cloud)

**Symptoms**
- Long task times out at 21 minutes
- User expects persistent pod but gets local execution

**Root Causes**
- `is_cloud` flag not set in job payload
- Experiment config routing to wrong processor

**RCA Steps**
```sql
-- Check job payload:
SELECT payload->>'is_cloud' as is_cloud, payload->>'processor_type' as processor
FROM jobs
WHERE id = '<job_id>';
```

---

### Issue 3-D: Tool execution failing

**Symptoms**
- Agent keeps retrying same tool
- Specific tool error in trajectories

**Root Causes**
1. Tool not installed in pod image
2. Permission denied inside pod
3. MCP server not reachable
4. Tool argument schema mismatch

**RCA Steps**
```bash
# Check pod logs for tool errors:
kubectl logs <pod-name> -n emergent-agents-env | grep -i "tool.*error\|error.*tool"

# Check MCP discovery:
```
```sql
SELECT metadata->>'mcp_tools' as mcp_config
FROM job_metadata
WHERE job_id = '<job_id>';
```
```bash
# Test envcore reachability:
kubectl exec -it <agent-pod> -n emergent-agents-env -- \
  curl http://envcore-service:8080/healthz
```

---

### Issue 3-E: Agent consuming too many credits

**Symptoms**
- User reports unexpected high credit usage
- Job cost much higher than expected

**Root Causes**
1. Agent in infinite tool loop
2. Large context window (many messages)
3. Expensive model selected (Opus vs Haiku cost difference)
4. Many sub-jobs spawned

**RCA Steps**
```sql
-- Check credit transactions for this job:
SELECT type, ecu, created_at, reference_id
FROM transactions
WHERE reference_id = '<job_id>'
ORDER BY created_at ASC;

-- Check LLM calls (token usage):
SELECT input_tokens, output_tokens, model, created_at
FROM llm_logs_partitioned
WHERE job_id = '<job_id>'
ORDER BY created_at ASC;
-- (use llm-proxy Redash DS)

-- Count iterations:
SELECT COUNT(*) as iterations
FROM trajectories
WHERE job_id = '<job_id>' AND type = 'llm_call';
```

---

## 7. LAYER 4 — CORTEX (TEMPORAL WORKFLOWS)

### What this layer does
Cortex is a Go service using Temporal for durable, fault-tolerant agent workflow execution. It runs Neo (setup) and Elite (agent loop) workflows.

---

### Issue 4-A: Temporal workflow stuck / not progressing

**Symptoms**
- Job in IN_PROGRESS but no heartbeat updates
- Temporal UI shows workflow running but no recent activity

**Root Causes**
1. Activity timeout — activity hanging waiting for LLM/tool response
2. Worker crashed — no worker to pick up tasks
3. Deadlock in workflow logic
4. External dependency (envcore, app-service) not responding

**RCA Steps**
```bash
# Use temporal-debugging skill:
# Check workflow status:
temporal workflow describe --workflow-id "neo-scratch-<job_id>-*" \
  --address temporal.internal-apps.emergentagent.com:7233

# List pending activities:
temporal workflow list --query 'WorkflowType="NeoWorkflow" AND ExecutionStatus="Running"'

# Check cortex worker logs:
./.claude/skills/loki-logs/scripts/loki-query.sh cortex-agentsdk "<job_id>" 2h

# Check heartbeats in app-service:
```
```sql
SELECT status, created_at FROM heartbeats
WHERE job_id = '<job_id>'
ORDER BY created_at DESC LIMIT 5;
```

---

### Issue 4-B: Workflow fails with non-retryable error

**Symptoms**
- Job immediately fails
- Temporal shows workflow terminated with error

**Root Causes**
1. Activity returned non-retryable error (e.g., 4xx from app-service)
2. Credit check failed in pre-loop hook
3. Agent config YAML not found for agent_id
4. Session creation failed

**RCA Steps**
```bash
# Get workflow history (shows exact failure point):
temporal workflow show --workflow-id "<workflow_id>" \
  --address temporal.internal-apps.emergentagent.com:7233

# Check cortex logs at time of failure:
./.claude/skills/loki-logs/scripts/loki-query.sh cortex-agentsdk "error\|FAIL" 1h

# Check agent config exists:
ls cortex/resources/agents/<agent_id>.yaml
```

---

### Issue 4-C: Context limit hit — fork button grayed out

**Symptoms**
- User can't fork the job
- UI shows "context limit reached"

**Root Causes**
- Session message history exceeds model's context window
- Squashing failed or not triggered

**RCA Steps**
```bash
# Check context squash records:
```
```sql
SELECT * FROM context_squashes
WHERE session_id = (
  SELECT session_id FROM jobs WHERE id = '<job_id>'
);
```
```bash
# Check cortex logs for squash activity:
./.claude/skills/loki-logs/scripts/loki-query.sh cortex-agentsdk "squash\|context" 2h | grep "<job_id>"
```

**Fix**: Engineering needs to trigger manual squash or increase context window config.

---

### Issue 4-D: Wrong model or agent ran

**Symptoms**
- User selected Claude Opus but Haiku ran
- Wrong system prompt / agent behavior

**Root Causes**
1. APS (Agent Prompt Selection) experiment overriding model
2. Job payload missing model_name field
3. Agent config YAML has wrong default model

**RCA Steps**
```sql
-- Check what model was requested vs used:
SELECT payload->>'model_name' as requested_model,
       metadata->>'resolved_model' as resolved_model,
       metadata->>'agent_id' as agent_id
FROM jobs j
LEFT JOIN job_metadata jm ON jm.job_id = j.id
WHERE j.id = '<job_id>';
```
```bash
# Check experiment config:
./.claude/skills/loki-logs/scripts/loki-query.sh cortex-agentsdk "experiment\|model_name" 1h | grep "<job_id>"
```

---

## 8. LAYER 5 — LLM-PROXY

### What this layer does
llm-proxy (Go, port 3000) routes LLM requests from cortex/agent-service to Anthropic, OpenAI, Vertex, Bedrock, Gemini, or Groq based on the model name.

---

### Issue 5-A: LLM calls failing / returning 5xx

**Symptoms**
- Jobs fail during LLM calls
- Error mentions "LLM" or "model" in failure message

**Root Causes**
1. Anthropic/OpenAI API outage (upstream down)
2. API key invalid or quota exhausted
3. Model name not recognized by routing logic
4. Request too large (context too long)
5. Rate limit from provider

**RCA Steps**
```bash
# Check llm-proxy logs:
./.claude/skills/loki-logs/scripts/loki-query.sh llm-proxy-service "error\|5[0-9][0-9]" 30m

# Check provider status pages:
# Anthropic: status.anthropic.com
# OpenAI:    status.openai.com

# Check which provider is failing:
./.claude/skills/loki-logs/scripts/loki-query.sh llm-proxy-service "target_url" 30m
```

```sql
-- Check LLM logs for error patterns:
SELECT model, status_code, error_message, created_at
FROM llm_logs_partitioned
WHERE created_at > NOW() - INTERVAL '1 hour'
  AND status_code >= 400
ORDER BY created_at DESC
LIMIT 20;
```

---

### Issue 5-B: Very slow LLM responses

**Symptoms**
- Jobs take much longer than usual
- LLM call appears to be the bottleneck

**Root Causes**
1. Provider experiencing high latency (upstream slow)
2. Very long prompt (large context)
3. Streaming disabled — waiting for full response
4. Network latency to provider region

**RCA Steps**
```sql
-- Check average LLM call duration:
SELECT model,
       AVG(response_time_ms) as avg_ms,
       MAX(response_time_ms) as max_ms,
       COUNT(*) as calls
FROM llm_logs_partitioned
WHERE created_at > NOW() - INTERVAL '1 hour'
GROUP BY model
ORDER BY avg_ms DESC;
```

---

### Issue 5-C: Model routing to wrong provider

**Symptoms**
- Specific model returns unexpected errors
- Wrong provider being called

**RCA Steps**
```bash
# Check routing logic in code:
grep -A5 "getTargetURL" llm-proxy-service/pkg/gllm/client.go

# Check logs for routing decision:
./.claude/skills/loki-logs/scripts/loki-query.sh llm-proxy-service "RPURL\|target" 30m | grep "<model_name>"
```

---

## 9. LAYER 6 — DEPLOYER

### What this layer does
The deployer (Go, port 8080) manages the entire lifecycle of user apps: Docker builds, Kubernetes deployment, MongoDB provisioning, SSL/domain management, and secrets.

---

### Issue 6-A: Deployment stuck at BUILD step

**Symptoms**
- Pipeline run shows `build` step `in_progress` for > 20 minutes
- No Docker image produced

**Root Causes**
1. Kaniko pod stuck on Alpine apk fetch (common)
2. Source code zip upload to GCS failed
3. Dockerfile has syntax error
4. Build ran out of disk space / memory
5. runc process hung in OrbStack/Docker

**RCA Steps**
```bash
# Check pipeline run status:
```
```sql
-- Deployer DB (DS 5):
SELECT id, name, status, logs, retries, created_at
FROM pipeline_run_steps
WHERE run_id = '<run_id>'
ORDER BY created_at;
```
```bash
# Get pipeline logs:
mcp__deployer__fetch_pipeline_logs(run_id="<run_id>")
# or:
curl "$BASE/pipeline-runs/<run_id>/logs" -H "X-User-ID: $USER_ID"

# Check Kaniko pod:
kubectl get pods -n emergent-core | grep kaniko
kubectl logs <kaniko-pod> -n emergent-core

# If stuck on Alpine apk (OrbStack):
pkill -9 runc
# Then retrigger the build

# Check Cloud Build logs (alternative builder):
gcloud builds list --region=us-central1 --filter="tags=<app_name>" --limit=5
gcloud builds log <build_id> --region=us-central1
```

---

### Issue 6-B: Deployment fails at HEALTH CHECK step

**Symptoms**
- Build succeeded, deploy succeeded, but health check fails
- App pod starts but health check endpoint returns error

**Root Causes**
1. App crashes on startup (entrypoint error, missing env var)
2. Port mismatch (app listening on 8000, health check hitting 8080)
3. Entrypoint uses missing tools (wget, curl, ss) — see Issue 6-C
4. App starts slowly (health check too early)
5. MongoDB connection failing on startup

**RCA Steps**
```bash
# Check app pod logs:
mcp__deployer__describe_app_pod(identifier="<app_name>")
# or:
kubectl get pods -n customers-app | grep <app_name>
kubectl logs <pod-name> -n customers-app --previous

# Check what port the app is listening on:
kubectl exec -it <pod-name> -n customers-app -- netstat -tlnp
# or:
kubectl exec -it <pod-name> -n customers-app -- ss -tlnp

# Check entrypoint.sh:
mcp__deployer__get_deployment_manifest(identifier="<app_name>", filename="entrypoint.sh")
```

---

### Issue 6-C: CrashLoopBackOff due to wget/curl in entrypoint

**Symptoms**
- Every deploy fails at health_check
- Pod logs show: `"Backend process running but not listening on port 8001 after 120s"`
- Preview URL works fine

**This is a VERY common issue. The entrypoint.sh uses shell tools not present in prod images.**

**Root Cause**
```bash
# Broken patterns in entrypoint.sh (any of these):
wget -q --spider http://127.0.0.1:8001/health
curl -s -o /dev/null http://127.0.0.1:8001/health
ss -tlnp | grep ":8001"
nc -z 127.0.0.1 8001
grep -q ":1F41 " /proc/net/tcp
```

**RCA Steps**
```bash
# 1. Read entrypoint.sh:
mcp__deployer__get_deployment_manifest(identifier="<app_name>", filename="entrypoint.sh")

# 2. Look for any of the broken patterns above
# 3. Check pod logs to confirm:
kubectl logs <pod-name> -n customers-app --previous | tail -30
```

**Fix**
```bash
# Replace wget/curl/ss/nc with python3 (always available):
# OLD:
if wget -q --spider http://127.0.0.1:8001/api/health 2>/dev/null; then

# NEW:
if python3 -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8001/api/health')" 2>/dev/null; then
```
```bash
# Update manifest and redeploy:
mcp__deployer__update_deployment_manifest(
  identifier="<app_name>",
  filename="entrypoint.sh",
  content="<full corrected content>"
)
# Then trigger redeployment via deployer API
```

---

### Issue 6-D: App deployed but showing 520 error

**Symptoms**
- App URL returns 520 (Cloudflare origin error)
- App was recently deployed

**Root Causes**
1. App pod crashing on startup (see 6-B, 6-C)
2. Cloudflare can't reach origin (load balancer issue)
3. App slow to start, Cloudflare timeout hit
4. SSL certificate issue between Cloudflare and origin

**RCA Steps**
```bash
# Check if pod is running:
mcp__deployer__describe_app_pod(identifier="<app_name>")

# Check pod restart count:
kubectl get pods -n customers-app | grep <app_name>
# RESTARTS column — if > 0, pod is crashing

# Check app logs:
mcp__deployer__fetch_app_logs(identifier="<app_name>")

# Try hitting origin directly (bypassing Cloudflare):
curl -H "Host: <app_name>.apps.emergentagent.com" http://<load_balancer_ip>/health
```

---

### Issue 6-E: Production database empty after first deployment

**Symptoms**
- User's app deployed successfully
- But all data from preview/dev is missing
- MongoDB database is empty

**Root Causes (3 scenarios)**
1. **Silent migration skip**: MongoDB migration step found no data to migrate
2. **Wrong database detected**: Migration used wrong source DB name
3. **Migration failed silently**: mongorestore ran but failed without pipeline failure

**RCA Steps**
```bash
# 1. Check pipeline run for mongo step:
```
```sql
SELECT name, status, logs
FROM pipeline_run_steps
WHERE run_id = '<run_id>' AND name = 'mongodb';
```
```bash
# 2. Check preview pod — does data exist there?
kubectl get pods -n emergent-agents-env | grep <job_id>
kubectl exec -it <pod-name> -n emergent-agents-env -- \
  mongosh --eval "db.adminCommand('listDatabases')"

# 3. Check what Atlas DB the app is connected to:
mcp__deployer__fetch_app_secrets(identifier="<app_name>")
# Look for MONGO_URL value

# 4. Manually migrate data (if data exists in preview pod):
# Inside preview pod:
mongodump --host 127.0.0.1 --port 27017 --db <source_db> --out /tmp/dump

mongorestore \
  --uri "<MONGO_URL from secrets>" \
  --nsFrom "<source_db>.*" \
  --nsTo "<target_db>.*" \
  /tmp/dump

# 5. Verify after restore:
mcp__deployer__run_mongo_query(
  app="<app_name>",
  operation="aggregate",
  collection="<collection>",
  query=[{"$count": "total"}]
)
```

---

### Issue 6-F: Blue-Green deployment failed mid-switch

**Symptoms**
- Deployment shows partial failure
- App might be serving from old or new deployment randomly
- Traffic switch step shows error

**Root Causes**
1. SwitchTraffic DB transaction failed (DB connectivity)
2. New app failed health check but old app was already marked for deletion
3. Cleanup step ran before confirming new app is healthy

**CRITICAL**: This can leave the app in a broken state.

**RCA Steps**
```sql
-- Check deployment records (DS 5):
SELECT id, deployment_name, status, created_at
FROM deployments
WHERE app_name = '<app_name>'
ORDER BY created_at DESC;

-- Should have exactly 1 active deployment
-- If 2 active → switch didn't complete atomically
-- If 0 active → both were deleted (very bad)
```
```bash
# Check which K8s deployment is actually serving traffic:
kubectl get deployments -n customers-app | grep <app_name>

# Check Ingress:
kubectl get ingress -n customers-app | grep <app_name>
kubectl describe ingress <ingress-name> -n customers-app
```

**Escalate immediately if**: 0 active deployments in DB but app still exists in K8s. Engineering needs to manually reconcile.

---

### Issue 6-G: Rollback not working

**Symptoms**
- User triggers rollback
- App still showing broken version

**Root Causes**
1. Previous image tag not found in GCR (tag deleted or never pushed)
2. Rollback deployed but health check failing for old version too
3. Database schema change incompatible with old version

**RCA Steps**
```bash
# Check available image tags in GCR:
gcloud container images list-tags gcr.io/emergent-default/<app_name> --limit=10

# Check rollback pipeline run:
```
```sql
SELECT pr.id, pr.type, pr.status, prs.name, prs.status as step_status, prs.logs
FROM pipeline_runs pr
JOIN pipeline_run_steps prs ON prs.run_id = pr.id
WHERE pr.app = '<app_name>' AND pr.type = 'rollback'
ORDER BY pr.created_at DESC, prs.created_at ASC;
```

---

## 10. LAYER 7 — ENVCORE + PODS

### What this layer does
envcore (Go, port 8080) is the bridge that lets agent-service run commands inside Kubernetes pods. The pods themselves run the agent's code in isolated environments.

---

### Issue 7-A: Pod not found / slug not in database

**Symptoms**
- envcore returns 404 for command execution
- agent-service logs: "pod not found for env_key"

**Root Causes**
1. Pod was never registered in pod_and_slug_infos (registration race condition)
2. Pod was cleaned up but job resumed expecting it
3. Slug format mismatch (job_id vs slug)

**RCA Steps**
```sql
-- Check pod registry:
SELECT pod_name, pod_ip, slug, status, created_at, last_accessed_at
FROM pod_and_slug_infos
WHERE slug = '<job_id>'
OR pod_name LIKE '%<job_id_prefix>%';
```
```bash
# Check if pod actually exists in K8s:
kubectl get pods -n emergent-agents-env | grep <job_id>

# Check envcore logs:
./.claude/skills/loki-logs/scripts/loki-query.sh envcore-service "<job_id>" 1h
```

---

### Issue 7-B: Pod in CrashLoopBackOff

**Symptoms**
- kubectl shows agent pod restarting repeatedly
- Job fails with pod crash error

**Root Causes**
1. OOMKilled — pod using more memory than limit
2. Process crashing on startup (dependency not installed)
3. Bad entrypoint command
4. Volume mount failing

**RCA Steps**
```bash
# Get crash reason:
kubectl describe pod <pod-name> -n emergent-agents-env | grep -A5 "Last State:"
# Look for: Reason: OOMKilled / Error / Completed

# Check exit code:
kubectl describe pod <pod-name> -n emergent-agents-env | grep "Exit Code"
# Exit code 137 = OOMKilled
# Exit code 1   = Application error
# Exit code 2   = Startup/config error

# Get logs from crashed container:
kubectl logs <pod-name> -n emergent-agents-env --previous

# Check memory usage before crash:
kubectl top pod <pod-name> -n emergent-agents-env
```

**Fix for OOMKilled**
```bash
# Escalate to engineering — memory limits need to be increased in the pod spec
# Or user's code is leaking memory (check app logs for memory growth pattern)
```

---

### Issue 7-C: Pod stuck in Pending (not starting)

**Symptoms**
- kubectl shows pod as Pending
- No node assigned

**Root Causes**
1. No node has sufficient resources (CPU/memory)
2. NodeSelector not matching any node
3. PVC not binding (storage class issue)
4. Image pull failing (GCR auth issue)

**RCA Steps**
```bash
# Check why pod is pending:
kubectl describe pod <pod-name> -n emergent-agents-env | grep -A20 Events

# Common messages:
# "Insufficient memory" → scale up nodes
# "no nodes available" → cluster capacity issue
# "FailedScheduling" → node selector/taint issue
# "ErrImagePull" → image pull issue

# Check node capacity:
kubectl describe nodes | grep -A10 "Allocated resources"

# Check if image exists:
gcloud container images describe gcr.io/emergent-default/<image>:<tag>
```

---

### Issue 7-D: PVC stuck in Terminating

**Symptoms**
- kubectl shows PVC with status Terminating for hours
- Cannot delete job resources

**Root Causes**
- Kubernetes finalizer preventing deletion (protection finalizer)
- Pod still holding reference to volume

**RCA Steps**
```bash
# Check PVC status:
kubectl get pvc -n emergent-agents-env | grep <job_id>
kubectl describe pvc <pvc-name> -n emergent-agents-env | grep -A5 Finalizers

# Check if any pod is still using it:
kubectl get pods -n emergent-agents-env -o json | \
  jq '.items[].spec.volumes[].persistentVolumeClaim.claimName' | grep <pvc-name>
```

**Fix**
```bash
# Remove finalizer (force delete):
kubectl patch pvc <pvc-name> -n emergent-agents-env \
  -p '{"metadata":{"finalizers":null}}'

# Wait for deletion:
kubectl get pvc -n emergent-agents-env | grep <pvc-name>
# Should disappear
```

---

### Issue 7-E: Commands timing out inside pod

**Symptoms**
- Tool execution times out
- envcore logs show: "command timeout after Xs"

**Root Causes**
1. Command genuinely taking too long (slow npm install, large file operation)
2. Pod under memory pressure (swap thrashing)
3. Network call inside pod timing out

**RCA Steps**
```bash
# Exec into pod and check:
kubectl exec -it <pod-name> -n emergent-agents-env -- /bin/bash

# Inside pod:
ps aux              # What processes are running?
top                 # CPU/memory usage?
df -h               # Disk full?
free -h             # Memory free?
netstat -tlnp       # What ports are listening?
```

---

### Issue 7-F: Volume snapshot failed — data lost on resume

**Symptoms**
- User resumes job but files are gone
- Pod starts fresh instead of restoring previous state

**Root Causes**
1. Snapshot never created (job ended too fast)
2. Snapshot creation failed silently
3. Restore failed — snapshot found but corrupt

**RCA Steps**
```bash
# Check volume snapshots for this job:
kubectl get volumesnapshot -n emergent-agents-env | grep <job_id>
kubectl describe volumesnapshot <snapshot-name> -n emergent-agents-env
# Check: readyToUse: true

# Check PVC for this job:
kubectl get pvc -n emergent-agents-env | grep <job_id>
```
```sql
-- Check DB for snapshot record:
SELECT * FROM volume_snapshots WHERE job_id = '<job_id>';
SELECT * FROM gcp_disk_mappings WHERE job_id = '<job_id>';
```

**Escalate if**: Snapshot readyToUse=false and data is critical.

---

## 11. DATABASES — POSTGRESQL, REDIS, BIGTABLE

### PostgreSQL Issues

#### Issue DB-A: Database connection exhausted

**Symptoms**
- Error: "too many connections" in app-service logs
- All DB queries failing

**Root Causes**
1. Connection pool misconfigured (too small)
2. Long-running queries holding connections
3. Spike in traffic

**RCA Steps**
```sql
-- Check active connections:
SELECT count(*) FROM pg_stat_activity;

-- Check connections by state:
SELECT state, count(*)
FROM pg_stat_activity
GROUP BY state;

-- Find long-running queries:
SELECT pid, query, state, now() - query_start as duration
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY duration DESC;

-- Kill a specific connection if needed:
SELECT pg_terminate_backend(<pid>);
```

---

#### Issue DB-B: Slow queries / high DB CPU

**RCA Steps**
```sql
-- Top slow queries:
SELECT query, mean_exec_time, calls, total_exec_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Check missing indexes:
SELECT relname AS table, seq_scan, idx_scan,
       seq_scan - idx_scan AS diff
FROM pg_stat_user_tables
WHERE seq_scan > idx_scan
ORDER BY diff DESC;
```

---

#### Issue DB-C: Data inconsistency (job status wrong)

**RCA Steps**
```sql
-- Check full job lifecycle:
SELECT j.id, j.status, j.state, j.created_at, j.updated_at,
       e.status as env_status, e.pod_lifecycle_status
FROM jobs j
LEFT JOIN environments e ON e.entity_id = j.id
WHERE j.id = '<job_id>';

-- Check audit trail:
SELECT state, metadata, created_at
FROM job_audits
WHERE job_id = '<job_id>'
ORDER BY created_at;

-- Check heartbeats:
SELECT status, created_at
FROM heartbeats
WHERE job_id = '<job_id>'
ORDER BY created_at DESC LIMIT 10;
```

---

### Redis Issues

#### Issue DB-D: Redis down — auth broken for all users

**Symptoms**
- All requests return 401 or 500
- Redis-related errors in app-service logs

**Root Causes**
- Redis pod crashed
- Network partition between app-service and Redis

**RCA Steps**
```bash
# Check Redis pod:
kubectl get pods -n emergent-core | grep redis
kubectl logs redis-0 -n emergent-core --tail=50

# Test connectivity from app-service:
kubectl exec -it <app-service-pod> -n emergent-core -- \
  redis-cli -h redis -p 6379 PING

# Check Redis memory usage:
kubectl exec -it redis-0 -n emergent-core -- redis-cli INFO memory | grep used_memory_human
```

**Impact if Redis is down**:
- Auth token cache misses → every request hits JWT decode (slow but works)
- Job queue might fail → new jobs not accepted
- Locks broken → possible duplicate processing

---

#### Issue DB-E: Stale lock blocking job execution

**Symptoms**
- Job operation hangs waiting for lock
- Logs show: "waiting for lock: env_operations_lock:<job_id>"

**RCA Steps**
```bash
# Check for stale locks:
kubectl exec -it redis-0 -n emergent-core -- redis-cli KEYS "env_operations_lock:*"

# Get lock details:
kubectl exec -it redis-0 -n emergent-core -- redis-cli GET "env_operations_lock:<job_id>"

# Force unlock if lock is stale (confirm job is not actually running first!):
kubectl exec -it redis-0 -n emergent-core -- redis-cli DEL "env_operations_lock:<job_id>"
```

**WARNING**: Only delete a lock after confirming the job is not actively running. Deleting an active lock can cause duplicate execution.

---

### BigTable Issues (Production Only)

#### Issue DB-F: Session messages not loading

**Symptoms**
- Agent can't load previous conversation history
- Error: "bigtable key not found"

**Root Causes**
1. BigTable key reference in PostgreSQL doesn't match actual BigTable entry
2. BigTable row expired (TTL)
3. BigTable service auth issue

**RCA Steps**
```sql
-- Check message refs in PostgreSQL:
SELECT id, content_ref, role, created_at
FROM messages
WHERE session_id = '<session_id>'
ORDER BY created_at ASC;
-- content_ref should be a BigTable key like "sessionID#messageID#llm_message"
-- If content_ref contains JSON directly → BigTable is NOT being used (ok for dev)
```

**Escalate to engineering** — BigTable issues require GCP access to diagnose fully.

---

## 12. NETWORKING & DNS

### Issue NET-A: Custom domain DNS not resolving

**Symptoms**
- User added custom domain
- Browser shows DNS error

**Root Causes**
1. User hasn't set up DNS A-record yet
2. DNS propagation not complete (can take up to 48 hours)
3. Wrong IP in A-record
4. CNAME vs A-record mismatch

**RCA Steps**
```bash
# Check DNS resolution:
dig <domain> A +short                    # What IP does domain resolve to?
nslookup <domain> 8.8.8.8               # Check via Google DNS

# Check what IP they should be pointing to:
curl "$BASE/domains/expected-ip" -H "X-User-ID: $USER_ID"
# or check deployer's load balancer IP:
kubectl get svc -n emergent-core | grep LoadBalancer

# Check domain record in Deployer DB (DS 5):
SELECT domain_name, verified, provider, created_at
FROM domains
WHERE domain_name = '<domain>';
```

**Fix**
```
Tell user:
1. Go to your DNS provider (GoDaddy/Namecheap/Cloudflare)
2. Add an A record: <domain> → <load_balancer_ip>
3. TTL: 300 seconds (or lowest available)
4. Wait up to 15 minutes for propagation
5. Come back and click "Verify Domain"
```

---

### Issue NET-B: SSL certificate not provisioning

**Symptoms**
- Domain resolves but shows "Certificate not valid" or browser SSL warning

**Root Causes**
1. DNS not pointing to correct IP (cert validation fails)
2. Caddy/Cloudflare can't reach domain for HTTP validation
3. Rate limit on Let's Encrypt (too many cert requests)

**RCA Steps**
```bash
# Check SSL status for domain (DS 5):
```
```sql
SELECT domain_name, verified, cf_ssl_status, cf_custom_hostname_id, provider
FROM domains
WHERE domain_name = '<domain>';
```
```bash
# For Cloudflare-managed SSL, check CF API:
curl -s "https://api.cloudflare.com/client/v4/zones/<zone_id>/custom_hostnames/<hostname_id>" \
  -H "Authorization: Bearer <CF_TOKEN>" | jq '.result.ssl.status'
# Should be: "active"
# If: "pending_validation" → waiting for DNS / HTTP validation

# Try re-verifying:
curl -X POST "$BASE/domains/<domain>/verify" -H "X-User-ID: $USER_ID"
```

---

### Issue NET-C: Internal service-to-service connectivity broken

**Symptoms**
- One service can't reach another
- Error: "connection refused" or "no route to host" in service logs

**Root Causes**
1. K8s Service DNS not resolving
2. NetworkPolicy blocking traffic
3. Service on wrong port
4. Target service down

**RCA Steps**
```bash
# Test DNS resolution from inside cluster:
kubectl exec -it <any-pod> -n emergent-core -- \
  nslookup app-service.emergent-core.svc.cluster.local

# Test connectivity:
kubectl exec -it <any-pod> -n emergent-core -- \
  curl -m 5 http://app-service:8003/status

# Check NetworkPolicies:
kubectl get networkpolicy -n emergent-core

# Check service endpoints:
kubectl get endpoints app-service -n emergent-core
# Should show pod IPs — if empty, no pods matched the selector
```

---

## 13. CLOUDFLARE & CUSTOM DOMAINS

### Issue CF-A: Cloudflare 1010 — Server-to-Server Blocked

**Symptoms**
- User's deployed app API returns 1010 error
- External services (webhooks, integrations) can't call the app's API
- curl from user's app to their own backend returns Cloudflare HTML block page

**Root Cause**
Cloudflare Bot Management is blocking server-to-server API calls because they don't come from browsers.

**Diagnosis**
```bash
# Confirm it's CF blocking, not the app:
curl -v -H "x-api-key: test" https://<app>.emergent.host/api/<endpoint>

# CF block indicators:
# - Response body is HTML
# - Content-Type: text/html
# - Status: 403 or 503
# - Body contains "Cloudflare" or error code 1010

# App-level error (not CF):
# - Response body is JSON: {"detail": "Invalid token"}
# - Content-Type: application/json
```
```sql
-- Verify app itself is healthy (DS 5):
SELECT name, status, is_suspended FROM apps WHERE name = '<app_name>';

-- Check deployment:
SELECT deployment_name, status FROM deployments
WHERE app_name = '<app_name>' AND status = 'active';
```

**Fix** (Requires Cloudflare Admin access — escalate to engineering):
```
Create WAF skip rule on emergent.host zone:
  Scope: *.emergent.host/api/*
  Condition: Request header "x-twin-api-key" exists
  Action: Skip Bot Management check
```

**Validation after fix**:
```bash
# Should get 403 JSON (reaches app, correct behavior):
curl -v -H "x-twin-api-key: invalid" https://<app>.emergent.host/api/<endpoint>
# Response: {"detail": "Invalid machine API token"}

# Should get 200 with real token:
curl -v -H "x-twin-api-key: <real_token>" https://<app>.emergent.host/api/<endpoint>
```

---

### Issue CF-B: Cloudflare 520/522/524 errors

**Code meanings:**
```
520 → Origin returned unexpected response (app crashing)
522 → Connection timed out (app unreachable)
524 → Timeout waiting for response (app too slow to respond)
526 → Invalid SSL certificate at origin
```

**RCA for 520** (most common — app crash)
```bash
# Check if pod is up:
mcp__deployer__describe_app_pod(identifier="<app_name>")

# Check pod logs:
mcp__deployer__fetch_app_logs(identifier="<app_name>")

# Try hitting origin directly:
# Get load balancer IP first
kubectl get svc -n customers-app | grep <app_name>
curl -H "Host: <app_name>.apps.emergentagent.com" http://<lb_ip>/health
```

**RCA for 524** (timeout — app too slow)
```bash
# Check if startup is slow:
kubectl logs <pod-name> -n customers-app | head -50
# Look for: "Server started in Xs"
# If > 30s startup → Cloudflare times out before app is ready

# Solution: Add startup delay to health check in entrypoint.sh
sleep 10
```

---

### Issue CF-C: Orphaned Cloudflare Custom Hostname

**Symptoms**
- User deleted app but custom hostname in Cloudflare still exists
- New app with same domain can't create custom hostname (conflict)

**RCA Steps**
```sql
-- Check for orphaned hostname (DS 5):
SELECT d.domain_name, d.cf_custom_hostname_id, d.cf_ssl_status,
       da.app_id, a.name as app_name
FROM domains d
LEFT JOIN domain_apps da ON da.domain_id = d.id
LEFT JOIN apps a ON a.id = da.app_id
WHERE d.domain_name = '<domain>';
-- If app_name is NULL → orphaned record
```
```bash
# Check Cloudflare directly:
curl -s "https://api.cloudflare.com/client/v4/zones/<zone>/custom_hostnames?hostname=<domain>" \
  -H "Authorization: Bearer <CF_TOKEN>" | jq '.result[].id'

# Delete orphaned hostname from Cloudflare:
curl -X DELETE "https://api.cloudflare.com/client/v4/zones/<zone>/custom_hostnames/<hostname_id>" \
  -H "Authorization: Bearer <CF_TOKEN>"

# Clean up deployer DB:
DELETE FROM domain_apps WHERE domain_id = (SELECT id FROM domains WHERE domain_name = '<domain>');
DELETE FROM domains WHERE domain_name = '<domain>';
```

---

### Issue CF-D: Stale Cloudflare Cache Serving Old Content

**Symptoms**
- App was redeployed but browser still shows old version
- `curl` shows old content too (not just browser cache)

**RCA Steps**
```bash
# Check CF cache status:
curl -I https://<app>.apps.emergentagent.com
# Look for: CF-Cache-Status: HIT (cached) vs MISS (fresh)
# x-cache-age: how old the cache is

# Force purge (deployer does this automatically on deploy):
curl -X POST "https://api.cloudflare.com/client/v4/zones/<zone>/purge_cache" \
  -H "Authorization: Bearer <CF_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"tags":["<app_name>"]}'
```

---

## 14. PAYMENTS & BILLING

### Issue PAY-A: User paid but credits not added

**Symptoms**
- User shows payment confirmation (email from Stripe)
- But credit balance unchanged in app

**Root Causes**
1. Webhook not received (Stripe failed to deliver)
2. Webhook signature verification failed (dropped silently)
3. Duplicate webhook — second ignored due to idempotency lock
4. Payment succeeded on Stripe but transaction status not updated

**RCA Steps**
```sql
-- Find the payment transaction (DS 10):
SELECT id, user_id, session_id, amount, currency, status, gateway, created_at
FROM payment_transactions
WHERE user_id = '<user_id>'
ORDER BY created_at DESC
LIMIT 10;

-- Check if transaction exists:
-- If status = PENDING → webhook not processed
-- If status = SUCCEEDED → credits should exist, check credit table
-- If no row → Stripe created session but we didn't save it

-- Check credits:
SELECT ecu, monthly_credits, daily_credits
FROM credits
WHERE user_id = '<user_id>';

-- Check transactions for this payment:
SELECT type, ecu, reference_type, reference_id, created_at
FROM transactions
WHERE reference_id = '<payment_id>'
ORDER BY created_at;
```
```bash
# Check Stripe dashboard:
# Go to Stripe Dashboard → Webhooks → find the event
# Check: was it delivered? Did it fail?

# Check webhook logs:
./.claude/skills/loki-logs/scripts/loki-query.sh app-service "webhook\|stripe" 2h | grep -i "payment_id"
```

**Fix**
```bash
# If webhook genuinely not received — trigger manual credit addition (support endpoint):
curl -X POST "https://api.emergent.test/support/credit/v1" \
  -H "Authorization: Bearer <ADMIN_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"user_id": "<user_id>", "amount": <amount>, "reason": "Payment received, webhook failed - PaymentTx: <id>"}'
```

---

### Issue PAY-B: Subscription active but monthly credits not refreshing

**Symptoms**
- User has active subscription
- monthly_credits shows 0
- Can't use subscription benefits

**RCA Steps**
```sql
-- Check subscription status (DS 10):
SELECT plan_id, status, start_date, end_date, created_at
FROM user_subscriptions
WHERE user_id = '<user_id>'
ORDER BY created_at DESC;

-- Check credit refresh date:
SELECT monthly_credits, monthly_credits_refresh_date, daily_credits
FROM credits
WHERE user_id = '<user_id>';

-- Is refresh date in the past? It should have refreshed already.
-- Check if cron ran:
```
```bash
./.claude/skills/loki-logs/scripts/loki-query.sh app-service "monthly_refresh\|credit.*refresh" 24h
```

**Fix**
```bash
# Manual monthly credit refresh (support endpoint):
curl -X POST "https://api.emergent.test/support/refresh-credits/v1" \
  -H "Authorization: Bearer <ADMIN_TOKEN>" \
  -d '{"user_id": "<user_id>"}'
```

---

### Issue PAY-C: Razorpay payment not reflected

**Symptoms**
- Indian user paid via Razorpay
- No credit added

**RCA Steps**
```sql
-- Check Razorpay transaction (DS 10):
SELECT id, user_id, session_id, amount, status, gateway, created_at
FROM payment_transactions
WHERE user_id = '<user_id>' AND gateway = 'RAZORPAY'
ORDER BY created_at DESC LIMIT 5;
```
```bash
# Check Razorpay webhook logs:
./.claude/skills/loki-logs/scripts/loki-query.sh app-service "razorpay" 2h

# Razorpay webhook verification uses HMAC:
# If signature verification fails → event dropped silently
# Check Razorpay dashboard → Webhooks → Event logs
```

---

### Issue PAY-D: Credits deducted but job failed

**Symptoms**
- User's credits were taken
- But job errored out before completing

**Root Cause Logic**
Credits are deducted BEFORE execution. If job fails, a refund/reversal should happen automatically. If it doesn't → bug.

**RCA Steps**
```sql
-- Check if credit was deducted:
SELECT type, ecu, reference_id, created_at
FROM transactions
WHERE reference_id = '<job_id>'
ORDER BY created_at;

-- If DEBIT exists but no CREDIT refund → refund didn't happen
-- Check job failure reason:
SELECT status, state, error_message
FROM jobs
WHERE id = '<job_id>';
```

**Fix**: Issue credit manually via support endpoint with reference to the failed job.

---

## 15. MOBILE BUILDS (iOS/ANDROID)

### Issue MOB-A: iOS build fails at EAS step

**Symptoms**
- Cloud Build succeeds for Kaniko step
- EAS step fails with signing/certificate error

**Root Causes**
1. Apple Developer session expired (2FA needed)
2. Provisioning profile expired
3. Apple ID password changed
4. EAS token invalid

**RCA Steps**
```bash
# Check EAS build logs:
gcloud builds log <build_id> --region=us-central1 | grep -A5 "EAS\|error\|Error"

# Check Apple session:
# If logs show "Session expired" → user needs to re-authenticate

# Trigger re-authentication:
# User must call: POST /api/v0/ios/apple/login { apple_id, password }
# Then: POST /api/v0/ios/apple/2fa { code } if 2FA required
```

---

### Issue MOB-B: Android APK fails to install on device

**Symptoms**
- Build succeeded
- APK downloaded but device rejects install

**Root Causes**
1. APK signed with debug keystore (not release)
2. App bundle (AAB) downloaded instead of APK
3. Device has older version with different signing key

**RCA Steps**
```bash
# Verify the build type requested:
```
```sql
SELECT payload->>'build_type' as build_type
FROM jobs WHERE id = '<job_id>';
```
```bash
# Check if APK vs AAB:
# APK = direct install
# AAB = Play Store only

# Check signing:
# apksigner verify --print-certs app.apk
```

---

## 16. SECURITY ISSUES

### Issue SEC-A: Unauthorized access to another user's job

**Symptoms**
- User can access job_id belonging to another user

**RCA Steps**
```bash
# Verify authorization check exists in code:
grep -n "user_id\|created_by\|auth" emergent/app/apis/v0/job_api.py | head -20

# Check the job's owner:
```
```sql
SELECT id, created_by, organization_id
FROM jobs
WHERE id = '<job_id>';
```
```
# If user_id != created_by AND no org membership → authorization bypass bug
# ESCALATE IMMEDIATELY to security team
```

---

### Issue SEC-B: API key or secret exposed in logs

**Symptoms**
- Secret value visible in pod logs or Loki

**Immediate Actions**
```
1. Note which secret was exposed and where
2. Rotate the secret immediately (Deployer secrets management)
3. Redeploy the app with new secret
4. If API key → revoke in the provider dashboard (Stripe, etc.)
5. Check log retention → purge affected log entries if possible
6. Escalate to security team
```

---

### Issue SEC-C: User's app accessible without authentication

**Symptoms**
- Deployed app endpoints return data without auth token

**RCA**
```
This is user code, not Emergent infrastructure.
The user's app code doesn't implement authentication.
Direct them to add auth middleware in their app.
```

---

### Issue SEC-D: JWT secret rotation needed

**Symptoms**
- Suspected token compromise
- Need to invalidate all existing sessions

**Steps**
```bash
# 1. Generate new JWT secret
openssl rand -hex 32

# 2. Update app-service env var:
kubectl set env deployment/app-service JWT_SECRET=<new_secret> -n emergent-core

# 3. Flush Redis token cache (all users will need to log in again):
kubectl exec -it redis-0 -n emergent-core -- redis-cli FLUSHDB
# WARNING: This logs out ALL users immediately

# 4. Monitor for auth errors in logs:
./.claude/skills/loki-logs/scripts/loki-query.sh app-service "401\|invalid token" 30m
```

---

## 17. OBSERVABILITY — HOW TO FIND ANYTHING

### The Investigation Hierarchy

```
Level 1 — User Report
    ↓ Get: job_id, user_id, app_name, timestamp, error message

Level 2 — Redash (Database State)
    ↓ What happened in the DB? Job status, credits, deployments

Level 3 — Loki (Recent Logs)
    ↓ What did the services log? Error messages, stack traces
    Retention: 24-48 hours

Level 4 — gcloud (Cloud Run Logs)
    ↓ For agent-service, cortex (Cloud Run services)
    Retention: longer than Loki

Level 5 — kubectl (Live/Recent K8s State)
    ↓ Pod status, events, resource usage

Level 6 — Deployer API / MCP Tools
    ↓ App status, pod status, pipeline runs, manifests
```

---

### Finding a Job's Full Lifecycle

```bash
# 1. Start with job ID — get everything:
JOB_ID="<your_job_id>"

# Database state:
# Redash DS 10:
SELECT j.id, j.status, j.state, j.created_at,
       j.payload->>'model_name' as model,
       j.payload->>'is_cloud' as is_cloud,
       e.status as env_status, e.pod_lifecycle_status
FROM jobs j
LEFT JOIN environments e ON e.entity_id = j.id
WHERE j.id = '$JOB_ID';

# Job audit trail:
SELECT state, created_at FROM job_audits
WHERE job_id = '$JOB_ID' ORDER BY created_at;

# Credit deductions:
SELECT type, ecu, created_at FROM transactions
WHERE reference_id = '$JOB_ID';

# Heartbeats:
SELECT status, created_at FROM heartbeats
WHERE job_id = '$JOB_ID' ORDER BY created_at DESC LIMIT 5;

# 2. Logs:
./.claude/skills/loki-logs/scripts/loki-job.sh $JOB_ID 6h

# 3. Pod that ran the job:
```
```sql
SELECT pod_name, pod_ip, status FROM pod_and_slug_infos WHERE slug = '$JOB_ID';
```
```bash
kubectl logs <pod_name> -n emergent-agents-env | grep -i "error\|fail\|exception"
```

---

### Finding a Deployment's Full Lifecycle

```bash
APP_NAME="<your_app_name>"
USER_ID="<user_id>"

# Database state (Redash DS 5):
SELECT a.name, a.status, a.template, a.mongo_cluster, a.is_suspended,
       d.deployment_name, d.status as deploy_status
FROM apps a
LEFT JOIN deployments d ON d.app_name = a.name AND d.status = 'active'
WHERE a.name = '$APP_NAME';

# Pipeline run history:
SELECT id, type, status, created_at FROM pipeline_runs
WHERE app = '$APP_NAME' ORDER BY created_at DESC LIMIT 5;

# Step details for latest run:
SELECT name, status, retries, logs FROM pipeline_run_steps
WHERE run_id = '<latest_run_id>' ORDER BY created_at;

# Logs:
mcp__deployer__fetch_pipeline_logs(run_id="<run_id>")
mcp__deployer__fetch_app_logs(identifier="$APP_NAME")

# Pod state:
mcp__deployer__describe_app_pod(identifier="$APP_NAME")
```

---

### Useful Log Patterns

```bash
# All errors in last hour across all services:
./.claude/skills/loki-logs/scripts/loki-errors.sh "" 1h

# Find when a user's job was processed:
./.claude/skills/loki-logs/scripts/loki-job.sh <job_id> 24h

# Check payment webhook processing:
./.claude/skills/loki-logs/scripts/loki-query.sh app-service "webhook\|stripe\|razorpay" 2h

# Check domain SSL provisioning:
./.claude/skills/loki-logs/scripts/loki-query.sh deployer "ssl\|domain\|cloudflare" 6h

# Check LLM call failures:
./.claude/skills/loki-logs/scripts/loki-query.sh llm-proxy-service "error\|5[0-9][0-9]" 1h
```

---

## 18. USER CODE BUGS — HOW TO DEBUG APP-LEVEL ISSUES

This section covers issues that are in the user's deployed app code, not Emergent infrastructure.

### How to Tell If It's User Code vs Platform Issue

```
User code issue:
  - App starts successfully (pod Running, health check passed)
  - Error occurs when hitting specific app endpoints
  - Error message is from the app's own code (FastAPI traceback, React error)
  - Other apps work fine
  - 5xx from the app itself (not Cloudflare 5xx codes)

Platform issue:
  - All apps affected
  - Pod won't start (CrashLoopBackOff before app code runs)
  - Infrastructure errors (K8s scheduling, DB connectivity)
  - Build fails before code runs
```

---

### Debugging User's Python Backend (FastAPI/Flask)

```bash
# 1. Get live app logs:
mcp__deployer__fetch_app_logs(identifier="<app_name>")
# or:
kubectl logs <pod-name> -n customers-app -f

# 2. Look for Python tracebacks:
# Pattern: "Traceback (most recent call last):"
# Then: "File '/app/...' line X in function_name"
# Then: "ErrorType: error message"

# 3. Common Python issues:
# ImportError → dependency not in requirements.txt
# KeyError → accessing dict key that doesn't exist
# AttributeError → calling method on None
# ConnectionRefusedError → can't connect to DB/Redis inside pod

# 4. Exec into pod and test manually:
kubectl exec -it <pod-name> -n customers-app -- /bin/bash
# Inside pod:
python3 -c "import requests; r=requests.get('http://localhost:8001/api/test'); print(r.status_code, r.text)"

# 5. Check env vars are set correctly:
kubectl exec -it <pod-name> -n customers-app -- env | grep -E "DB|MONGO|API|SECRET"
```

---

### Debugging User's Node.js/Next.js Backend

```bash
# Get logs:
kubectl logs <pod-name> -n customers-app | grep -E "Error|WARN|error"

# Common issues:
# "Cannot find module" → npm install failed or package missing
# "ECONNREFUSED" → can't connect to MongoDB/Redis
# "process.env.X is undefined" → env var not injected
# Syntax error → code bug in JS/TS

# Check if Next.js built correctly:
kubectl exec -it <pod-name> -n customers-app -- ls -la /app/.next/
# Should have: standalone/, static/, BUILD_ID
# If missing → build failed silently
```

---

### Debugging MongoDB Connection Issues in User's App

```bash
# 1. Get the MONGO_URL from secrets:
mcp__deployer__fetch_app_secrets(identifier="<app_name>")

# 2. Test connection from inside the pod:
kubectl exec -it <pod-name> -n customers-app -- \
  mongosh "<MONGO_URL>" --eval "db.adminCommand('ping')"
# Should return: { ok: 1 }

# 3. Common MongoDB errors:
# "MongoServerSelectionError: connection timed out"
#   → URL wrong, network issue, Atlas cluster down
# "AuthenticationFailed"
#   → Wrong username/password in MONGO_URL
# "Database not found"
#   → DB name in URL different from actual DB name

# 4. Check actual DB name vs connected DB:
kubectl exec -it <pod-name> -n customers-app -- \
  mongosh "<MONGO_URL>" --eval "db.getName()"
```

---

### Debugging User's Frontend (React/Next.js)

```bash
# For Kubernetes-deployed frontend:
# 1. Check if nginx is serving files:
kubectl exec -it <pod-name> -n customers-app -- nginx -t
kubectl exec -it <pod-name> -n customers-app -- ls /app/dist/

# 2. Check nginx config:
mcp__deployer__get_deployment_manifest(identifier="<app_name>", filename="nginx.conf")

# 3. For API routing issues (frontend can't reach backend):
# Check nginx proxy_pass config — is it pointing to correct backend URL?
# Check VITE_API_BASE_URL or REACT_APP_API_URL was set at build time

# For Cloudflare-deployed frontend (R2):
# Check Cloudflare Worker routing:
# Does /api/* route to backend or to R2?
```

---

### How to Fix User Code Without Giving Them Root Access

```
As L3, you can:
1. Read their manifests (entrypoint.sh, Dockerfile, nginx.conf)
   → mcp__deployer__get_deployment_manifest()

2. Edit their manifests:
   → mcp__deployer__update_deployment_manifest()

3. Exec into their pod (read-only investigation):
   → kubectl exec -it <pod> -n customers-app -- /bin/bash

4. Check their secrets:
   → mcp__deployer__fetch_app_secrets()

5. Trigger redeployment:
   → Deployer API: POST /api/v1/deploy

You CANNOT:
- Access their Git repository
- Modify their application source code directly
- Access their production database without MongoDB URI
```

---

## 19. RCA TEMPLATES

### Template A: Job Failure RCA

```
INCIDENT: Job Failure

User: <name> (<user_id>)
Job ID: <job_id>
Time: <when the job ran>
Impact: <what the user couldn't do>

TIMELINE:
  <timestamp> Job submitted
  <timestamp> Job reached state: <state>
  <timestamp> Error occurred: <what happened>
  <timestamp> Job marked as <status>

ROOT CAUSE:
  <one clear sentence: what broke and why>

EVIDENCE:
  1. DB: <what the jobs table showed>
  2. Logs: <exact log line that shows the error>
  3. Pod: <pod status at time of failure>

CONTRIBUTING FACTORS:
  - <anything that made this more likely>

FIX APPLIED:
  <what was done to resolve for this user>

PREVENTION:
  <what should be done to prevent recurrence — engineering task or user education>

ESCALATION:
  [ ] Resolved by L3
  [ ] Needs engineering: <reason>
```

---

### Template B: Deployment Failure RCA

```
INCIDENT: Deployment Failure

User: <name> (<user_id>)
App: <app_name>
Pipeline Run ID: <run_id>
Failed Step: <build|mongodb|secrets|deploy|health_check>
Time: <when>

FAILING STEP LOGS:
  <paste relevant log lines>

ROOT CAUSE:
  <e.g., "entrypoint.sh uses wget which is not available in production image">

FIX APPLIED:
  <e.g., "Updated entrypoint.sh to use python3 urllib for health check">
  <e.g., "Triggered redeployment after fix">

RESULT:
  Deployment <succeeded/still failing>
  App URL: <url>
```

---

### Template C: Payment Issue RCA

```
INCIDENT: Payment Not Reflected

User: <name> (<user_id>)
Payment Method: <Stripe/Razorpay/RevenueCat>
Amount: <amount> <currency>
Payment Date: <date>
Payment Confirmation: <provided by user?>

INVESTIGATION:
  Payment Transaction found: <yes/no>
    - ID: <id>
    - Status: <PENDING/SUCCEEDED/FAILED>
  Webhook received: <yes/no — check logs>
  Credits before: <amount>
  Credits after: <amount>

ROOT CAUSE:
  <e.g., "Stripe webhook delivered but signature verification failed">
  <e.g., "Webhook not received — Stripe retry queue exhausted">

FIX:
  <e.g., "Manually added X ECU via support endpoint. Ref: PaymentTx <id>">

VERIFICATION:
  Credits now: <amount>
  User confirmed: <yes/no>
```

---

## 20. MASTER FLOWCHARTS

### Flowchart 1: Diagnosing Any User-Reported Issue

```
User reports a problem
          │
          ▼
   Get: job_id / app_name / user_id / timestamp
          │
          ▼
    What kind of issue?
          │
    ┌─────┼──────────────────────────────────┐
    │     │                                  │
    ▼     ▼                                  ▼
 Job    Deploy                          Payment/Credits
 Issue  Issue                           Issue
    │     │                                  │
    ▼     ▼                                  ▼
Section 5 Section 9                    Section 14
(Layers   (Deployer)                   (Payments)
2,3,4)
```

---

### Flowchart 2: Job Not Working

```
User: "My job didn't work"
            │
            ▼
    Check job status in Redash (DS 10)
            │
       ┌────┴────────────┬────────────────┐
       ▼                 ▼                ▼
   PENDING           IN_PROGRESS      FAILED/ERROR
       │                 │                │
       ▼                 ▼                ▼
 Never reached     Still running?    Check error_message
 agent-service                       in jobs table
       │            │       │             │
       ▼            ▼       ▼             ▼
 Check agent-    Yes: Is  No: Job    Match error to
 service health  it past  frozen     KB article →
 Check Redis     timeout? │           troubleshoot
 Check pod pool  │        ▼
                 ▼     Last heartbeat?
               User    > 5min ago → frozen
               must    Check cortex/Temporal
               wait    for stuck workflow
```

---

### Flowchart 3: Deployment Failing

```
User: "My app won't deploy"
            │
            ▼
  Get pipeline run ID from Redash (DS 5)
  pipeline_runs WHERE app = '<app>'
            │
            ▼
  Check pipeline_run_steps for failed step
            │
    ┌───────┼──────────┬────────────┬────────────┐
    ▼       ▼          ▼            ▼            ▼
  BUILD   MONGODB   SECRETS      DEPLOY      HEALTH_CHECK
    │       │          │            │            │
    ▼       ▼          ▼            ▼            ▼
 Check   Check       Check       Check        Check
 Cloud   Atlas       K8s secrets  K8s deploy  pod logs
 Build   migration   creation     events      for crash
 logs    logs                                reason
    │       │          │            │            │
    ▼       ▼          ▼            ▼            ▼
 Kaniko  Data       Env var      Image        entrypoint.sh
 stuck?  missing?   wrong?       pull fail?   uses wget?
```

---

### Flowchart 4: Custom Domain Not Working

```
User: "My domain example.com isn't working"
            │
            ▼
  Check domains table (DS 5)
  SELECT * FROM domains WHERE domain_name = 'example.com'
            │
       ┌────┴──────────────────────┐
       ▼                           ▼
  verified = false            verified = true
       │                           │
       ▼                      ┌────┴────────────┐
  Check DNS:                  ▼                 ▼
  dig example.com A      SSL active?        App behind
       │               cf_ssl_status        domain up?
    ┌──┴──────┐              │                  │
    ▼         ▼         ┌────┴──────┐      ┌───┴────┐
  DNS      DNS OK       ▼           ▼      ▼        ▼
  wrong    but not    active:     pending:  Yes:    No:
           verified   Check app   Re-run   Check   Check
           yet        is up       verify   CF WAF   pod logs
```

---

### Flowchart 5: Payment Not Working

```
User: "I paid but have no credits"
            │
            ▼
  Check payment_transactions (DS 10)
  WHERE user_id = '<id>'
            │
       ┌────┴──────────────────────┐
       ▼                           ▼
  No record found            Record found
       │                           │
       ▼                      ┌────┴────────────┐
  Check Stripe dashboard      ▼                 ▼
  Was payment captured?   PENDING           SUCCEEDED
       │                     │                  │
    ┌──┴──────┐               ▼                 ▼
    ▼         ▼          Webhook not       Check credits
  Not      Captured      received          table for this
  captured: → Manual     → Check           user
  User's    credit add   webhook logs      │
  bank                                ┌────┴──────┐
  issue                               ▼            ▼
                                  Credits      Credits
                                  updated      NOT updated
                                      │            │
                                      ▼            ▼
                                  User just   Bug: manual
                                  didn't      add credits
                                  refresh     + escalate
```

---

### Flowchart 6: Pod Issues

```
Agent pod issue reported
            │
            ▼
  kubectl get pods -n emergent-agents-env | grep <job_id>
            │
    ┌───────┼──────────┬────────────┐
    ▼       ▼          ▼            ▼
  Not     Pending   Running      CrashLoop/
  Found             but broken   OOMKilled
    │       │          │            │
    ▼       ▼          ▼            ▼
  Check   No node   Exec in and  kubectl logs
  slug    resources  debug       --previous
  table   → Check    commands     │
          node       failing      ▼
          capacity                Exit code?
                                  │
                                ┌─┴──────┐
                                ▼        ▼
                              137:     1/2:
                            OOMKill   App
                            → Escalate error
                            eng for   → Check
                            limits    app logs
```

---

*Last updated: 2026-03-30*
*Maintained by: L3 Engineering Team*
*Add new patterns: `.claude/skills/debugging-knowledgebase/CONTRIBUTING.md`*
