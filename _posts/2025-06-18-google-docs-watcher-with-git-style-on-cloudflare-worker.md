---
title: Google Docs Watcher with Git-Style Diff via Cloudflare Worker
date: 2025-06-18 17:15:00 +0700
categories: [Blogging, Tutorial]
tags: [cloudflare worker, googgle docs, javascript]
author: j12t
toc: true
comments: true
math: true
mermaid: false
pin: false
---

Monitor changes to a shared Google Doc (even with **read-only access**) and post Git-style diffs to a Discord channel—powered entirely by Cloudflare Workers and Google Docs API.

## ✨ Features

- 🕒 Runs every 12 hours (cron trigger)
- 📑 Reads content from Google Docs API
- 🧠 Calculates Git-like line-by-line diffs
- 📢 Sends alerts to Discord via webhook
- ⚡ Fully serverless and free (Cloudflare + Google free tiers)

---

## 🛠️ Prerequisites

### Google Cloud

- Create a project at [Google Cloud Console](https://console.cloud.google.com/)
- Enable **Google Docs API**
- Create a **Service Account** with a JSON key
- Share your Google Doc with the service account email

### Cloudflare

- A Cloudflare account with access to **Workers & Pages**

---

## 🧑‍💻 Cloudflare Worker Setup

### 1. Create a New Worker

In your [Cloudflare dashboard](https://dash.cloudflare.com/):

- Go to **Workers & Pages**
- Create a new **Worker**
- Use the code below (see **`worker.js`**)

### 2. Add KV Storage

In your Worker’s **Settings** → scroll to **KV Namespaces**:

- Create a new namespace
- Bind it with the variable name: `DOC_CACHE`

### 3. Set Environment Variables

Add these under **Settings → Environment Variables**:

| Variable              | Type   | Description                           |
|-----------------------|--------|---------------------------------------|
| `DOC_ID`              | Text   | The Google Doc ID from its URL        |
| `DISCORD_WEBHOOK`     | Secret | Discord webhook URL                   |
| `GCP_SERVICE_ACCOUNT` | Secret | Paste your full service account JSON  |

### 4. Add a Cron Trigger

Go to the **Triggers** tab and add this:

```txt
0 */12 * * *   (every 12 hours)
```

---

## 🧾 Worker Code (`worker.js`)

Paste this entire script into the Cloudflare Worker editor:

```js
// Main logic
async function run(env) {
  const token = await getAccessToken(env);
  const docId = env.DOC_ID;
  const webhook = env.DISCORD_WEBHOOK;
  
  const res = await fetch(`https://docs.googleapis.com/v1/documents/${docId}`, {
    headers: { Authorization: `Bearer ${token}` }
  });
  if (!res.ok) {
    console.error("Failed to fetch doc:", await res.text());
    return "ERROR";
  }
  const data = await res.json();
  const text = data.body.content
    .map(e => e.paragraph?.elements?.map(el => el.textRun?.content || "").join("") || "")
    .join("");
  
  const now = new Date().toISOString();
  const last = await env.DOC_CACHE.get("last");
  const diff = getGitDiff(last || "", text);
  if (diff && diff.length > 0) {  
    await fetch(webhook, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ content: `📄 Google Doc updated at ${now}\n\`\`\`diff\n${diff.slice(0, 1800)}\n...\`\`\`` }) // Discord limit: 2000 chars
    });
    await env.DOC_CACHE.put("last", text);
    return `${now} | OK, something changed!\n${diff}`;
  } else {
    return `${now} | OK, nothing changed.`
    }
}

// Get normal diff
function getDiff(oldText, newText) {
  const oldLines = oldText.split("\n");
  const newLines = newText.split("\n");
  const changes = [];

  for (let i = 0; i < newLines.length; i++) {
    if (oldLines[i] !== newLines[i]) {
      changes.push(`🔸 Line ${i + 1} changed:\n- ${oldLines[i] || ""}\n+ ${newLines[i]}`);
    }
  }

  return changes.length ? changes.join("\n\n") : null;
}

// Git-like diff
function getGitDiff(oldText, newText) {
  const oldLines = oldText.split("\n");
  const newLines = newText.split("\n");
  const diff = [];

  const max = Math.max(oldLines.length, newLines.length);
  for (let i = 0; i < max; i++) {
    const oldLine = oldLines[i] || "";
    const newLine = newLines[i] || "";

    if (oldLine === newLine) {
      continue;
    } else {
      if (oldLine) diff.push(`- ${oldLine}`);
      if (newLine) diff.push(`+ ${newLine}`);
    }
  }

  return diff.join("\n");
}

// Get OAuth access token 
async function getAccessToken(env) {
  const cached = await env.DOC_CACHE.get("access_token", { type: "json" });
  const now = Math.floor(Date.now() / 1000);
  if (cached && cached.exp > now + 60) return cached.token;

  const key = JSON.parse(env.GCP_SERVICE_ACCOUNT);
  const iat = now;
  const exp = now + 3600;

  const header = {
    alg: "RS256",
    typ: "JWT"
  };

  const payload = {
    iss: key.client_email,
    scope: "https://www.googleapis.com/auth/documents.readonly",
    aud: "https://oauth2.googleapis.com/token",
    iat,
    exp
  };

  const enc = new TextEncoder();
  const toBase64 = obj => btoa(JSON.stringify(obj)).replace(/=/g, "").replace(/\+/g, "-").replace(/\//g, "_");
  const unsigned = `${toBase64(header)}.${toBase64(payload)}`;

  const keyData = str2ab(key.private_key);
  const cryptoKey = await crypto.subtle.importKey(
    "pkcs8",
    keyData,
    { name: "RSASSA-PKCS1-v1_5", hash: "SHA-256" },
    false,
    ["sign"]
  );

  const signature = await crypto.subtle.sign("RSASSA-PKCS1-v1_5", cryptoKey, enc.encode(unsigned));
  const jwt = `${unsigned}.${btoa(String.fromCharCode(...new Uint8Array(signature)))
    .replace(/=/g, "").replace(/\+/g, "-").replace(/\//g, "_")}`;

  const res = await fetch("https://oauth2.googleapis.com/token", {
    method: "POST",
    headers: { "Content-Type": "application/x-www-form-urlencoded" },
    body: `grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer&assertion=${jwt}`
  });

  const { access_token, expires_in } = await res.json();
  await env.DOC_CACHE.put("access_token", JSON.stringify({ token: access_token, exp: now + expires_in }));
  return access_token;
}

function str2ab(pem) {
  const b64 = pem.replace(/-----[^-]+-----|\n/g, "");
  const binary = atob(b64);
  const bytes = new Uint8Array(binary.length);
  for (let i = 0; i < binary.length; i++) bytes[i] = binary.charCodeAt(i);
  return bytes.buffer;
}

// Export functions
export default {
  // Run on demand
  async fetch(request, env, ctx) {
    const result = await run(env);
    return new Response("✅ Ran Worker logic: " + result);
  },
  // Run on schedule
  async scheduled(event, env, ctx) {
    await run(env);
  }
}

```

---

## ✅ Done

Within 12 hours, if your doc changes, your Discord channel will light up with a message like:

```diff
📄 Google Doc updated at 2025-06-18T17:45:00+07:00
- Old content removed
+ New content added
```

🎉 Congrats—you're now monitoring Google Docs like a Git-savvy ninja.

---

## 📎 License

MIT.
