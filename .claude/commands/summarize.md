---
description: Summarize the last N assistant messages in this conversation. Usage: /summarize <number> [optional topic focus]
argument-hint: <number> [topic]
---

Look back through this conversation and identify the last $ARGUMENTS assistant messages.

Parse the arguments like this:
- The first token is a number — that's how many of **your own (assistant) messages** to look back through. If no number is given, default to 5.
- Everything after the number is an optional topic filter. If a topic is given, extract only the content from those messages that is relevant to that topic.

**Rules:**
- Include only content from assistant messages — skip user questions, instructions, and prompts entirely
- If a topic filter was given: focus the summary on that thread, discard unrelated content from those messages
- If no topic filter: synthesize all key information, findings, conclusions, and outputs from the selected messages
- Preserve logical flow and relationships between pieces of information — this is not a bullet dump, it's a synthesis
- Use headers if the content spans clearly distinct topics
- Remove redundancy — if you said the same thing twice across messages, say it once
- Format the output as if it will be shared with a colleague or pasted into another session — clean, self-contained, no conversational filler
- End with a one-line **"Key takeaway:"** that captures the most important thing from the set

Do not explain what you are doing. Just output the summary.
