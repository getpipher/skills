---
name: x-api
description: X/Twitter API v2 reference — auth, accounts, endpoints, and usage patterns
allowed-tools: ["bash", "read", "write", "edit"]
---

# X API v2 Reference

Use this skill whenever interacting with X/Twitter API. All auth details, account mappings, and endpoint patterns are here.

Arguments provided: $ARGUMENTS

## Account & Auth Setup

> **Env file:** all X credentials (`X_BEARER_TOKEN`, `X_CONSUMER_*`, `X_ACCESS_*`, `X_CLIENT_*`) live in a shell env file. Set `X_ENV_FILE` to its path (default `$HOME/.env`); every example below sources it.

**App**: your X app (create at console.x.com)
**Pricing**: Pay-as-you-go credits, shared across all apps. Top up at console.x.com.
**Monthly cap**: 2M post reads (Enterprise for more). Auto-recharge available.
**Duplicate protection**: Same post/user on same day won't double-charge.
**No per-request cost in response headers** — full pricing table (source: developer.x.com/#pricing):

### Read Operations
| Resource | Unit Cost | Billing |
|----------|-----------|---------|
| Posts: Read | $0.005 | Per resource fetched |
| User: Read | $0.010 | Per resource fetched |
| DM Event: Read | $0.010 | Per resource fetched |
| List: Read | $0.005 | Per resource fetched |
| Space: Read | $0.005 | Per resource fetched |
| Community: Read | $0.005 | Per resource fetched |
| Note: Read | $0.005 | Per resource fetched |
| Following/Followers: Read | $0.005 | Per resource fetched |
| Media: Read | $0.005 | Per resource fetched |
| Analytics: Read | $0.010 | Per resource fetched |
| Trend: Read | $0.010 | Per resource fetched |
| Counts: Recent | $0.015 | Per request |
| Counts: All | $0.015 | Per request |

### Write Operations
| Resource | Unit Cost | Billing |
|----------|-----------|---------|
| Content: Create | $0.005 | Posts or media, per request |
| DM Interaction: Create | $0.015 | Per request |
| User Interaction: Create | $0.015 | Likes, retweets, follows, per request |
| List: Create | $0.010 | Per request |
| Bookmark | $0.005 | Per request |
| Media Metadata | $0.005 | Create/delete metadata, per request |

### Manage/Delete Operations
| Resource | Unit Cost | Billing |
|----------|-----------|---------|
| Content: Manage | $0.005 | Delete/hide posts, per request |
| Interaction: Delete | $0.005 | Per request |
| List: Manage | $0.005 | Delete/update lists, per request |
| Privacy: Update | $0.010 | Per request |
| Mute: Delete | $0.005 | Per request |
| Music: Create | $0.005 | Per request |

### Bearer Token (App-Only — Read Public Data)
```bash
# IMPORTANT: the agent sandbox may strip bearer tokens from curl -H arguments.
# Always use Python for X API calls — curl will silently fail (401).
source "${X_ENV_FILE:-$HOME/.env}"
python3 -c "
import urllib.request, json, os
token = os.environ['X_BEARER_TOKEN']
url = 'https://api.x.com/2/YOUR_ENDPOINT'
req = urllib.request.Request(url, headers={'Authorization': f'Bearer {token}'})
resp = urllib.request.urlopen(req)
print(json.dumps(json.loads(resp.read()), indent=2))
"
```
- Env var: `X_BEARER_TOKEN` (single-quoted in .env, contains `/` and `=`)
- Use for: reading public tweets, user lookups, search
- No user context — can't post, like, or reply

### OAuth 1.0a (User Context — @your-handle)
```bash
# For posting, replying, liking, retweeting — requires user context
# Uses: X_CONSUMER_KEY, X_CONSUMER_SECRET, X_ACCESS_TOKEN, X_ACCESS_SECRET
# Python example with requests-oauthlib:
from requests_oauthlib import OAuth1Session
oauth = OAuth1Session(
  os.environ["X_CONSUMER_KEY"],
  client_secret=os.environ["X_CONSUMER_SECRET"],
  resource_owner_key=os.environ["X_ACCESS_TOKEN"],
  resource_owner_secret=os.environ["X_ACCESS_SECRET"]
)
response = oauth.post("https://api.x.com/2/tweets", json={"text": "Hello"})
```

### OAuth 2.0 (User Context — @your-handle)
```bash
# Client ID/Secret for OAuth 2.0 PKCE flow
# Env vars: X_CLIENT_ID, X_CLIENT_SECRET
# Used for: advanced user-context operations
```

## Common Endpoints

### Read Operations (Bearer Token — use Python, not curl)

All read examples use this helper pattern:
```python
source "${X_ENV_FILE:-$HOME/.env}"
python3 -c "
import urllib.request, json, os
token = os.environ['X_BEARER_TOKEN']
url = 'ENDPOINT_URL_HERE'
req = urllib.request.Request(url, headers={'Authorization': f'Bearer {token}'})
resp = urllib.request.urlopen(req)
print(json.dumps(json.loads(resp.read()), indent=2))
"
```

**Endpoints:**
- User lookup: `https://api.x.com/2/users/by/username/{username}`
- User lookup + fields: `...?user.fields=created_at,description,public_metrics,verified`
- User tweets (up to 3,200): `https://api.x.com/2/users/{user_id}/tweets?max_results=10&tweet.fields=created_at,public_metrics,text`
  - Params: `max_results` (5-100), `exclude` (retweets,replies), `start_time`, `end_time`, `since_id`, `until_id`, `pagination_token`
- User mentions: `https://api.x.com/2/users/{user_id}/mentions?max_results=10&tweet.fields=created_at,public_metrics,text`
- Tweet by ID: `https://api.x.com/2/tweets/{id}?tweet.fields=created_at,public_metrics,text,author_id&expansions=author_id&user.fields=username,name`
- Search recent (7 days): `https://api.x.com/2/tweets/search/recent?query={query}&max_results=10&tweet.fields=created_at,public_metrics,text,author_id`
- Search all (full archive): `https://api.x.com/2/tweets/search/all?query={query}&max_results=10`

### Write Operations (OAuth 1.0a — @your-handle)

**Create a post:**
```bash
# Using curl with OAuth 1.0a is complex — use Python for write operations
python3 -c "
import os, json
from requests_oauthlib import OAuth1Session
oauth = OAuth1Session(
  os.environ['X_CONSUMER_KEY'],
  client_secret=os.environ['X_CONSUMER_SECRET'],
  resource_owner_key=os.environ['X_ACCESS_TOKEN'],
  resource_owner_secret=os.environ['X_ACCESS_SECRET']
)
r = oauth.post('https://api.x.com/2/tweets', json={'text': 'YOUR_TEXT'})
print(json.dumps(r.json(), indent=2))
"
```

**Reply to a post:**
```python
payload = {"text": "YOUR_REPLY", "reply": {"in_reply_to_tweet_id": "TWEET_ID"}}
r = oauth.post("https://api.x.com/2/tweets", json=payload)
```

**Like a post:**
```python
# Need authenticated user ID first
r = oauth.post(f"https://api.x.com/2/users/{user_id}/likes", json={"tweet_id": "TWEET_ID"})
```

**Retweet:**
```python
r = oauth.post(f"https://api.x.com/2/users/{user_id}/retweets", json={"tweet_id": "TWEET_ID"})
```

**Delete a post:**
```python
r = oauth.delete(f"https://api.x.com/2/tweets/{tweet_id}")
```

## Error Responses

| Status | Title | Meaning |
|--------|-------|---------|
| 401 | Unauthorized | Token invalid/expired — regenerate at console.x.com |
| 402 | Payment Required | Legacy tier, endpoint not included |
| 403 | Forbidden | Insufficient permissions or suspended app |
| 429 | Too Many Requests | Rate limit hit — check `x-rate-limit-reset` header |
| - | CreditsDepleted | Out of credits — top up at console.x.com |

## Cost Guard — MANDATORY

Before ANY X API call, estimate the cost and follow these rules:

**Auto-approve (no confirmation needed):**
- Single user lookup ($0.01)
- Fetching ≤ 20 posts/resources ($0.10 or less)
- Single write operation (post, reply, like — $0.01-$0.015)

**Require user approval first:**
- Fetching > 20 posts in a single query or across pagination
- Any paginated/looped calls (e.g. "get ALL tweets from user")
- Bulk operations (mass like, mass follow, batch posting)
- Full archive search (`/tweets/search/all`) — more expensive
- Any operation estimated > $0.50

**How to ask:** Before executing, show:
```
⚠ X API Cost Estimate:
  Operation: [what you're about to do]
  Resources: ~[count] × $[unit cost] = $[total]
  Proceed?
```

**Hard rules:**
- NEVER use `max_results=100` without approval — default to 10
- NEVER auto-paginate (follow `next_token`) without approval
- NEVER loop API calls in a script without approval
- When user asks for "all" or "every" tweet — warn about cost first, suggest a reasonable limit

## Execution

Based on the arguments provided, determine the operation type and execute accordingly.

If no arguments: display this reference.
If arguments contain a URL or @username: parse and execute the appropriate read/write operation.
