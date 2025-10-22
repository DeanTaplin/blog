---
description: Generate a curated weekly blog post from web articles with AI-powered summaries, key insights, and provocative titles
tags: [project]
---

You are tasked with generating a weekly blog digest from a list of URLs. Follow these steps carefully:

## Step 1: Parse Input URLs

First, determine how the user is providing URLs and extract them:

**Input methods to support:**
1. `--from-file <path>`: Read URLs from a file (one per line)
2. `--from-clipboard`: Read URLs from clipboard (newline or space-separated)
3. Direct arguments: URLs provided as space-separated arguments
4. stdin: If no arguments provided, read from stdin

**Parse the URLs:**
- If user provided `--from-file <path>`, read the file and extract URLs (one per line, skip empty lines and lines starting with #)
- If user provided `--from-clipboard`, read from clipboard and split by newlines or spaces
- If URLs are provided directly as arguments (starting with http:// or https://), use those
- If no arguments, read from stdin

Store all URLs in a list and validate they start with http:// or https://. Remove duplicates.

**If fewer than 1 URL is found, show an error with usage examples and stop.**

## Step 2: Fetch Article Content

For each URL, use the WebFetch tool to retrieve the article content. Use this prompt:

```
Extract the full article content from this page, including:
- The article title
- The main featured image URL (hero image, og:image meta tag, or primary visual)
- The main body text (all paragraphs)
- Any key technical details, data, or research findings
- Important lists, bullet points, or structured information

Provide the content in a structured format with clear sections.
```

**Error Handling:**
- If a URL fetch fails (timeout, 404, paywall, etc.), note the failure and skip it
- Continue processing remaining URLs
- Track which URLs succeeded and which failed

**If all URLs fail, show an error listing the failures and stop.**

## Step 3: Analyze Each Article

For each successfully fetched article, analyze it to generate:

### 3a. Provocative Title
Generate a provocative, humorous, or thought-provoking title that:
- Captures attention while being informative
- May use patterns like "X: What Could Possibly Go Wrong?", "When Your X Becomes Y", or "Company Finally Does X (And Here's Why That Matters)"
- Sometimes questions or is skeptical
- Is not clickbait but has personality

Examples:
- "AI Wrote My Code and All I Got Was This Lousy Bug Report"
- "Oops, I Put 3,000 Flood Victims' Data Into ChatGPT: A Cautionary Tale"
- "Google Finally Lets AI Coding Agents Take Off the Blindfold (What Could Possibly Go Wrong?)"

### 3b. Detailed Summary (4-8 paragraphs)
Write a conversational, opinionated summary following this structure:

1. **Hook/Context**: Start with a provocative question, observation, or framing that sets up why this matters
2. **What is this?**: Explain the core concept, announcement, or research
3. **Technical Details**: Dive into how it works, what the risks are, or what the data shows
   - Use embedded bullet-point lists for multiple technical points, risks, or vulnerabilities
4. **Why it matters**: Explain implications, challenges, or broader context
5. **Editorial commentary**: Personal observations, skepticism, or practical takes
6. **Bottom Line**: A synthesizing paragraph that starts with "**Bottom Line**:" and connects everything with a key insight, often using metaphors or analogies

**Voice and tone:**
- Conversational and opinionated, like explaining tech news to a knowledgeable friend over coffee
- Skeptical of hype but appreciative of real innovation
- Punchy and engaging with humor and sarcasm
- Technical but accessible
- Use first-person observations where appropriate

**Use embedded lists** for technical details, risks, or key points:
```markdown
The research identified three critical vulnerabilities:

- **Remote Code Execution**: Developers using exec() on LLM output without sandboxing
- **Permission Leakage**: RAG systems that don't properly enforce per-user access controls
- **Data Exfiltration**: Markdown rendering that can send data to attacker servers

Each of these represents...
```

### 3c. Key Takeaways (3-7 items)
Extract specific, actionable takeaways, risks, or lessons learned. Not generic observations.

Label the section based on the article type:
- "**Key Takeaways:**" for general articles
- "**Key Risks:**" for security or vulnerability articles
- "**Lessons Learned:**" for case studies or post-mortems

### 3d. Featured Image (Optional)
Extract and include a featured image **only for select articles** where visuals add meaningful context:

**When to include images:**
- Product announcements with official product images
- Architecture diagrams or technical visualizations
- Research papers with key charts or graphs
- Before/after comparisons that illustrate the point
- UI/UX changes showing visual differences

**When NOT to include images:**
- Generic stock photos or placeholder images
- Author headshots or profile pictures
- Text-heavy screenshots that don't add clarity
- Decorative images without informational value

**How to extract:**
- Look for hero images, og:image meta tags, or prominent article visuals
- Prefer images hosted on official domains (avoid third-party CDNs when possible)
- Ensure image URLs are direct links (not requiring authentication)

**Format:**
```markdown
![Descriptive alt text](image-url "Optional caption")
*Image credit/attribution if needed*
```

**Placement:** After the Key Takeaways section, before the horizontal rule separator

## Step 4: Synthesize Introduction

After analyzing all articles, write an engaging 2-3 paragraph introduction that:
- Identifies patterns, themes, or contradictions across the articles
- Questions assumptions or highlights tensions in the industry
- Uses conversational tone with personality
- Sets up why this week's reading matters
- Is not afraid to editorialize

## Step 5: Generate Suggested Further Reading

Based on the themes identified across all articles, suggest 3-5 topics for further reading:
- Each should be a **Topic/Theme Title** followed by a brief explanation of why it's relevant
- Should connect to multiple articles if possible
- Should identify adjacent or deeper areas worth exploring

## Step 6: Format the Output

Create a markdown file with this structure:

```markdown
# Weekly Digest - Week of [YYYY-MM-DD]

## Introduction

[2-3 engaging paragraphs from Step 4]

---

## Articles

### [Provocative Title from 3a]

**Original**: [Original article title]

**Source**: [URL]

**Date Read**: [YYYY-MM-DD]

[4-8 paragraph summary from 3b, including embedded lists and ending with Bottom Line paragraph]

**[Key Takeaways/Key Risks/Lessons Learned]:**

- [Takeaway 1]
- [Takeaway 2]
- [Takeaway 3]
- [Additional takeaways as needed]

![Optional featured image alt text](image-url "Optional caption")
*Optional image attribution*

---

[Repeat for each article]

---

## Suggested Further Reading

Based on the themes in this week's digest, you might find these topics interesting:

1. **[Topic 1]**: [Why it's relevant]
2. **[Topic 2]**: [Why it's relevant]
3. **[Topic 3]**: [Why it's relevant]
4. **[Topic 4]**: [Optional]
5. **[Topic 5]**: [Optional]

---

*Generated on [YYYY-MM-DD HH:MM:SS] with Claude Code weekly-blog command*
```

## Step 7: Save the Output

Save the markdown content to `YYYY-week-##.md` in the `weekly/` directory, where:
- `YYYY` is the 4-digit year
- `##` is the 2-digit ISO week number (01-53)

For example: `weekly/2025-week-41.md`

To calculate the ISO week number, use the date from "Week of [YYYY-MM-DD]" in the blog title and determine its ISO week number.

This ensures each blog post is preserved without overwriting previous versions.

## Step 8: Report Results

After saving, provide a summary:
- Number of articles processed successfully
- List any URLs that failed to fetch
- Path to the generated file
- Brief preview of the introduction

---

**Important Notes:**
- Use today's date for "Week of" and "Date Read" fields
- Ensure all markdown formatting is clean and compatible with standard readers
- The writing should sound human-written with opinions, not like a generic AI summarizer
- Embedded bullet lists should be used within prose for technical details, not as standalone sections
- Every article summary must end with a "**Bottom Line:**" paragraph
- Use the WebFetch tool for all URL fetching operations
- Featured images are optional and should only be included when they add meaningful visual context
