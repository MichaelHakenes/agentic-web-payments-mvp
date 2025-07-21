# Setup Guide - Agentic Web Payments MVP

> Step-by-step guide to replicate the working system

## Prerequisites
- Docker Desktop installed and running
- OpenAI API key ([get here](https://platform.openai.com/api-keys))
- Basic terminal knowledge
- ngrok account (for public access)

## Step 1: Start n8n with Docker

```bash
# Start n8n container
docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n

Access: http://localhost:5678
Create: Admin account when prompted

## Step 2: Configure OpenAI Integration

1. **In n8n:** Go to Credentials → Add Credential → OpenAI
2. **API Key:** Enter your OpenAI API key
3. **Test:** Connection should show "Connected"

## Step 3: Import Workflow

1. **Create Nodes:**
   - Webhook (trigger)
   - HTTP Request (OpenAI Moderation)
   - IF (condition node)  
   - HTTP Request (Solana)
   - Edit Fields (logging)

2. **Configure Each Node:**
   - **Webhook:** Path = `/youtube-comment`
   - **OpenAI:** POST to `https://api.openai.com/v1/moderations`
   - **IF:** Condition = `{{$json.results[0].flagged}} == true`
   - **Solana:** POST to `https://api.devnet.solana.com` with `getHealth`

## Step 4: Test Locally

```bash
curl -X POST http://localhost:5678/webhook-test/youtube-comment \
  -d '{"comment": {"id": "test", "text": "test hate speech"}}'

Expected: Workflow executes through all nodes

## Step 5: Public Access (Optional)
# Install ngrok
brew install ngrok  # macOS
# or download from ngrok.com

# Start tunnel
ngrok http 5678

# Test external access
curl -X POST https://YOUR-NGROK-URL.ngrok-free.app/webhook/youtube-comment \
  -d '{"comment": {"text": "test content"}}'

## Troubleshooting

**Problem:** n8n won't start
**Solution:** Check if port 5678 is free: `lsof -i :5678`

**Problem:** OpenAI 401 error  
**Solution:** Verify API key has sufficient credits

**Problem:** Webhook not found
**Solution:** Activate workflow (toggle in top-right)

## What You Should See

**Successful execution produces:**
```json
{
  "action": "hate_comment_processed",
  "ai_flagged": "true", 
  "solana_connection": "ok",
  "timestamp": "2025-07-21T17:26:30.078Z",
  "mvp_status": "agentic_payment_infrastructure_complete"
}
