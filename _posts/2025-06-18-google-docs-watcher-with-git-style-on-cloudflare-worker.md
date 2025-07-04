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
var __defProp = Object.defineProperty;
var __name = (target, value) => __defProp(target, "name", { value, configurable: true });

// src/goauth.ts
async function getAccessToken(env) {
  const cached = await env.DOC_CACHE.get("access_token", { type: "json" });
  const now = Math.floor(Date.now() / 1e3);
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
  const toBase64 = /* @__PURE__ */ __name((obj) => btoa(JSON.stringify(obj)).replace(/=/g, "").replace(/\+/g, "-").replace(/\//g, "_"), "toBase64");
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
  const jwt = `${unsigned}.${btoa(String.fromCharCode(...new Uint8Array(signature))).replace(/=/g, "").replace(/\+/g, "-").replace(/\//g, "_")}`;
  const res = await fetch("https://oauth2.googleapis.com/token", {
    method: "POST",
    headers: { "Content-Type": "application/x-www-form-urlencoded" },
    body: `grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer&assertion=${jwt}`
  });
  const { access_token, expires_in } = await res.json();
  await env.DOC_CACHE.put("access_token", JSON.stringify({ token: access_token, exp: now + expires_in }));
  return access_token;
}
__name(getAccessToken, "getAccessToken");
function str2ab(pem) {
  const b64 = pem.replace(/-----[^-]+-----|\n/g, "");
  const binary = atob(b64);
  const bytes = new Uint8Array(binary.length);
  for (let i = 0; i < binary.length; i++) bytes[i] = binary.charCodeAt(i);
  return bytes.buffer;
}
__name(str2ab, "str2ab");

// src/utils.ts
function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}
__name(sleep, "sleep");

// node_modules/diff/libesm/diff/base.js
var Diff = class {
  static {
    __name(this, "Diff");
  }
  diff(oldStr, newStr, options = {}) {
    let callback;
    if (typeof options === "function") {
      callback = options;
      options = {};
    } else if ("callback" in options) {
      callback = options.callback;
    }
    const oldString = this.castInput(oldStr, options);
    const newString = this.castInput(newStr, options);
    const oldTokens = this.removeEmpty(this.tokenize(oldString, options));
    const newTokens = this.removeEmpty(this.tokenize(newString, options));
    return this.diffWithOptionsObj(oldTokens, newTokens, options, callback);
  }
  diffWithOptionsObj(oldTokens, newTokens, options, callback) {
    var _a;
    const done = /* @__PURE__ */ __name((value) => {
      value = this.postProcess(value, options);
      if (callback) {
        setTimeout(function() {
          callback(value);
        }, 0);
        return void 0;
      } else {
        return value;
      }
    }, "done");
    const newLen = newTokens.length, oldLen = oldTokens.length;
    let editLength = 1;
    let maxEditLength = newLen + oldLen;
    if (options.maxEditLength != null) {
      maxEditLength = Math.min(maxEditLength, options.maxEditLength);
    }
    const maxExecutionTime = (_a = options.timeout) !== null && _a !== void 0 ? _a : Infinity;
    const abortAfterTimestamp = Date.now() + maxExecutionTime;
    const bestPath = [{ oldPos: -1, lastComponent: void 0 }];
    let newPos = this.extractCommon(bestPath[0], newTokens, oldTokens, 0, options);
    if (bestPath[0].oldPos + 1 >= oldLen && newPos + 1 >= newLen) {
      return done(this.buildValues(bestPath[0].lastComponent, newTokens, oldTokens));
    }
    let minDiagonalToConsider = -Infinity, maxDiagonalToConsider = Infinity;
    const execEditLength = /* @__PURE__ */ __name(() => {
      for (let diagonalPath = Math.max(minDiagonalToConsider, -editLength); diagonalPath <= Math.min(maxDiagonalToConsider, editLength); diagonalPath += 2) {
        let basePath;
        const removePath = bestPath[diagonalPath - 1], addPath = bestPath[diagonalPath + 1];
        if (removePath) {
          bestPath[diagonalPath - 1] = void 0;
        }
        let canAdd = false;
        if (addPath) {
          const addPathNewPos = addPath.oldPos - diagonalPath;
          canAdd = addPath && 0 <= addPathNewPos && addPathNewPos < newLen;
        }
        const canRemove = removePath && removePath.oldPos + 1 < oldLen;
        if (!canAdd && !canRemove) {
          bestPath[diagonalPath] = void 0;
          continue;
        }
        if (!canRemove || canAdd && removePath.oldPos < addPath.oldPos) {
          basePath = this.addToPath(addPath, true, false, 0, options);
        } else {
          basePath = this.addToPath(removePath, false, true, 1, options);
        }
        newPos = this.extractCommon(basePath, newTokens, oldTokens, diagonalPath, options);
        if (basePath.oldPos + 1 >= oldLen && newPos + 1 >= newLen) {
          return done(this.buildValues(basePath.lastComponent, newTokens, oldTokens)) || true;
        } else {
          bestPath[diagonalPath] = basePath;
          if (basePath.oldPos + 1 >= oldLen) {
            maxDiagonalToConsider = Math.min(maxDiagonalToConsider, diagonalPath - 1);
          }
          if (newPos + 1 >= newLen) {
            minDiagonalToConsider = Math.max(minDiagonalToConsider, diagonalPath + 1);
          }
        }
      }
      editLength++;
    }, "execEditLength");
    if (callback) {
      (/* @__PURE__ */ __name(function exec() {
        setTimeout(function() {
          if (editLength > maxEditLength || Date.now() > abortAfterTimestamp) {
            return callback(void 0);
          }
          if (!execEditLength()) {
            exec();
          }
        }, 0);
      }, "exec"))();
    } else {
      while (editLength <= maxEditLength && Date.now() <= abortAfterTimestamp) {
        const ret = execEditLength();
        if (ret) {
          return ret;
        }
      }
    }
  }
  addToPath(path, added, removed, oldPosInc, options) {
    const last = path.lastComponent;
    if (last && !options.oneChangePerToken && last.added === added && last.removed === removed) {
      return {
        oldPos: path.oldPos + oldPosInc,
        lastComponent: { count: last.count + 1, added, removed, previousComponent: last.previousComponent }
      };
    } else {
      return {
        oldPos: path.oldPos + oldPosInc,
        lastComponent: { count: 1, added, removed, previousComponent: last }
      };
    }
  }
  extractCommon(basePath, newTokens, oldTokens, diagonalPath, options) {
    const newLen = newTokens.length, oldLen = oldTokens.length;
    let oldPos = basePath.oldPos, newPos = oldPos - diagonalPath, commonCount = 0;
    while (newPos + 1 < newLen && oldPos + 1 < oldLen && this.equals(oldTokens[oldPos + 1], newTokens[newPos + 1], options)) {
      newPos++;
      oldPos++;
      commonCount++;
      if (options.oneChangePerToken) {
        basePath.lastComponent = { count: 1, previousComponent: basePath.lastComponent, added: false, removed: false };
      }
    }
    if (commonCount && !options.oneChangePerToken) {
      basePath.lastComponent = { count: commonCount, previousComponent: basePath.lastComponent, added: false, removed: false };
    }
    basePath.oldPos = oldPos;
    return newPos;
  }
  equals(left, right, options) {
    if (options.comparator) {
      return options.comparator(left, right);
    } else {
      return left === right || !!options.ignoreCase && left.toLowerCase() === right.toLowerCase();
    }
  }
  removeEmpty(array) {
    const ret = [];
    for (let i = 0; i < array.length; i++) {
      if (array[i]) {
        ret.push(array[i]);
      }
    }
    return ret;
  }
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  castInput(value, options) {
    return value;
  }
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  tokenize(value, options) {
    return Array.from(value);
  }
  join(chars) {
    return chars.join("");
  }
  postProcess(changeObjects, options) {
    return changeObjects;
  }
  get useLongestToken() {
    return false;
  }
  buildValues(lastComponent, newTokens, oldTokens) {
    const components = [];
    let nextComponent;
    while (lastComponent) {
      components.push(lastComponent);
      nextComponent = lastComponent.previousComponent;
      delete lastComponent.previousComponent;
      lastComponent = nextComponent;
    }
    components.reverse();
    const componentLen = components.length;
    let componentPos = 0, newPos = 0, oldPos = 0;
    for (; componentPos < componentLen; componentPos++) {
      const component = components[componentPos];
      if (!component.removed) {
        if (!component.added && this.useLongestToken) {
          let value = newTokens.slice(newPos, newPos + component.count);
          value = value.map(function(value2, i) {
            const oldValue = oldTokens[oldPos + i];
            return oldValue.length > value2.length ? oldValue : value2;
          });
          component.value = this.join(value);
        } else {
          component.value = this.join(newTokens.slice(newPos, newPos + component.count));
        }
        newPos += component.count;
        if (!component.added) {
          oldPos += component.count;
        }
      } else {
        component.value = this.join(oldTokens.slice(oldPos, oldPos + component.count));
        oldPos += component.count;
      }
    }
    return components;
  }
};

// node_modules/diff/libesm/diff/line.js
var LineDiff = class extends Diff {
  static {
    __name(this, "LineDiff");
  }
  constructor() {
    super(...arguments);
    this.tokenize = tokenize;
  }
  equals(left, right, options) {
    if (options.ignoreWhitespace) {
      if (!options.newlineIsToken || !left.includes("\n")) {
        left = left.trim();
      }
      if (!options.newlineIsToken || !right.includes("\n")) {
        right = right.trim();
      }
    } else if (options.ignoreNewlineAtEof && !options.newlineIsToken) {
      if (left.endsWith("\n")) {
        left = left.slice(0, -1);
      }
      if (right.endsWith("\n")) {
        right = right.slice(0, -1);
      }
    }
    return super.equals(left, right, options);
  }
};
var lineDiff = new LineDiff();
function diffLines(oldStr, newStr, options) {
  return lineDiff.diff(oldStr, newStr, options);
}
__name(diffLines, "diffLines");
function tokenize(value, options) {
  if (options.stripTrailingCr) {
    value = value.replace(/\r\n/g, "\n");
  }
  const retLines = [], linesAndNewlines = value.split(/(\n|\r\n)/);
  if (!linesAndNewlines[linesAndNewlines.length - 1]) {
    linesAndNewlines.pop();
  }
  for (let i = 0; i < linesAndNewlines.length; i++) {
    const line = linesAndNewlines[i];
    if (i % 2 && !options.newlineIsToken) {
      retLines[retLines.length - 1] += line;
    } else {
      retLines.push(line);
    }
  }
  return retLines;
}
__name(tokenize, "tokenize");

// src/index.ts
async function run(env, checkPoint = false) {
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
  const docTitle = data.title;
  const currentText = data.body.content.map((e) => e.paragraph?.elements?.map((el) => el.textRun?.content || "").join("") || "").join("");
  const now = (/* @__PURE__ */ new Date()).toISOString();
  const lastText = await env.DOC_CACHE.get("last");
  const lastNow = await env.DOC_CACHE.get("lastTime");
  const diff = diffLines(lastText, currentText);
  var diffFileContent = `${docTitle}
`;
  var diffContentHtml = `<html><head><title> Diff check for GDOC:${docId} at ${now}</title> <meta content="text/html; charset=utf-8" http-equiv="content-type"> </head><body><div><span>`;
  diffContentHtml += `<p> ----------------------------------------------------- </p>`;
  diffContentHtml += `<p> Document title: <b> ${docTitle} </b> </p>`;
  diffContentHtml += `<p style="color: grey;"> Document ID: <i> ${docId} </i> </p>`;
  diffContentHtml += `<p style="color: yellow;"> <i> Last CheckTime: ${lastNow} </i> </p>`;
  diffContentHtml += `<p style="color: blue;"> <i> Current CheckTime: ${now} </i> </p>`;
  diffContentHtml += `<p> ----------------------------------------------------- </p>`;
  diff.forEach((part) => {
    let style = part.added ? `"color: green;"` : part.removed ? `"color: red";` : `"color: grey;"`;
    let annotate = part.added ? `+ ` : part.removed ? `- ` : `  `;
    let paragraph = part.value.split("\n");
    paragraph.forEach((p) => {
      let text = `<p style=${style}> ${p} </p>
`;
      diffContentHtml += text;
      diffFileContent += `${annotate} ${p}
`;
    });
  });
  diffContentHtml += "</span></div></body></html>";
  const boundary = "----WebKitFormBoundary" + Math.random().toString(16).slice(2);
  const body = `--${boundary}\rContent-Disposition: form-data; name="payload_json"\r\r${JSON.stringify({ content: `\u{1F4C4} Google Doc updated at ${now}` })}\r--${boundary}\rContent-Disposition: form-data; name="file"; filename="update.diff"\rContent-Type: text/plain\r\r${diffFileContent}\r--${boundary}--`;
  if (checkPoint) {
    var response_code = -1;
    var tries = 3;
    while (response_code !== 200 && tries > 0) {
      var response = await fetch(env.DISCORD_WEBHOOK, {
        method: "POST",
        headers: {
          "Content-Type": `multipart/form-data; boundary=${boundary}`
        },
        body
      });
      if (response.status == 429) {
        const timeout = Number(response.headers.get("Retry-After"));
        console.warn(`Rate limit (retry after ${timeout}), retry in ${timeout + 10}`);
        await sleep(timeout + 10);
      }
      response_code = response.status;
      tries--;
      if (tries <= 0) {
        console.error(response_code);
      }
      await env.DOC_CACHE.put("last", currentText);
      await env.DOC_CACHE.put("lastTime", now);
    }
  }
  return diffContentHtml;
}
__name(run, "run");
var index_default = {
  async fetch(request, env, ctx) {
    const result = await run(env);
    const http_headers = {
      "Content-Type": "text/html"
    };
    return new Response(result, {
      status: 200,
      statusText: "Run success",
      headers: new Headers(http_headers)
    });
  },
  async scheduled(event, env, ctx) {
    const result = await run(env, true);
  }
};
export {
  index_default as default
};
//# sourceMappingURL=index.js.map

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
