---
name: deep-talk
version: 1.0.0
description: Interview the user relentlessly about a plan, design, or specific file until reaching shared understanding, resolving each branch of the decision tree. Use when user wants to stress-test a plan, get grilled on their design, deep talk through a file, or mentions "deep talk". Adapted from mattpocock's grill-me skill.
disable-model-invocation: false
user-invocable: true
argument-hint: "[optional file path or topic]"
tags:
  - product development
---

Interview me relentlessly about every aspect of this plan, design, or file until we reach a shared understanding. Walk down each branch of the decision tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

If `$ARGUMENTS` is provided and it points to a file, read that file first and center the discussion on its contents.

If `$ARGUMENTS` is provided but is not a file path, treat it as the topic to deep talk about.

Ask the questions one at a time.

If a question can be answered by exploring the codebase, explore the codebase instead.

When discussing a file, ground your questions and recommendations in concrete details from that file.
