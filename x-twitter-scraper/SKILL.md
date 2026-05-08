---
name: x-twitter-scraper
description: Use this skill for X/Twitter API automation with Xquik: tweet search, advanced Twitter search, profile tweets, user tweets, follower export, media download, posting, replies, DMs, webhooks, MCP, and SDK workflows.
---

# X Twitter Scraper

Use this skill when the user wants an AI coding agent to work with X/Twitter data or actions through Xquik.

## When to use

- Search tweets by keyword, hashtag, account, date range, or advanced Twitter search operators
- Get profile tweets, user timelines, liked tweets, media tweets, and engagement metrics
- Export followers, following, verified followers, retweeters, likers, replies, and quote tweets
- Download tweet media and collect URLs for images, videos, and GIFs
- Monitor accounts and deliver HMAC-signed webhook events
- Send tweets, post replies, like, repost, follow, DM, or update profiles after explicit approval
- Build workflows with REST API, MCP tools, or official SDKs

## When not to use

- The task only needs general OSINT research without X/Twitter data
- The user asks to bypass access controls or collect private data
- The task needs platform actions without explicit user approval
- The user has no Xquik API key and only wants a no-auth public web search

## Install

```bash
npx skills add Xquik-dev/x-twitter-scraper --skill x-twitter-scraper
```

## Workflow

1. Clarify the target workflow: search, profile tweets, follower export, media download, monitor, webhook, post, reply, DM, or SDK integration.
2. Confirm authentication expectations. Use `XQUIK_API_KEY` from the runtime environment and never print it.
3. For read workflows, choose the smallest useful endpoint or SDK call and include pagination when collecting lists.
4. For write workflows, show the exact action, account, text, media, and destination before running anything.
5. For webhook workflows, require HTTPS destinations and verify HMAC signatures in examples.
6. Return code, commands, or implementation notes with clear setup and retry guidance.

## Core Examples

### Tweet Search

Use tweet search for keywords, hashtags, accounts, and advanced search operators.

```bash
curl "https://xquik.com/api/v1/x/tweets/search?q=from%3Aexample&limit=10" \
  -H "X-API-Key: $XQUIK_API_KEY"
```

### Profile Tweets

Use profile tweet workflows when the user asks to get tweets from a user, export a timeline, or collect recent posts from a profile.

### Follower Export

Use extraction workflows for follower, following, verified follower, retweeter, liker, reply, quote, and mention exports. Prefer CSV export when the user asks for a spreadsheet-ready result.

### Posting and Replies

Write actions require explicit approval. Before sending, restate the account, tweet text, reply target if any, attached media, and expected action.

## SDKs and Tools

Use the official Xquik SDKs when the project language is known: TypeScript, Python, Ruby, Go, Kotlin, Java, PHP, C#, CLI, and Terraform.

## Safety

- Never print API keys, bearer tokens, cookies, or raw auth material.
- Do not claim access to private data or bypasses.
- Keep user-facing wording accurate: own infrastructure, REST API, MCP tools, SDKs, webhooks, monitoring, and confirmation-gated write actions.
- Ask for confirmation before posting, replying, sending DMs, following, unfollowing, liking, reposting, or updating profiles.
