---
published: true
layout: post
title: AI History Timelining
subtitle: A Overview of the AI-hist tool
tags: [ai]
comments: false
share-img: https://swolfsec.github.io/assets/img/img_2026-06-01-ai-hist-timeline/ai-hist-full-timeline-3.png
---
> By Stephan Wolfert

> **_TL:DR_** AI coding assistants leave artifacts on disk. _ai-hist_ parses some of those artifacts and provides a timeline of chat history including commands, and more. Thanks to https://github.com/xFreed0m/ghosttype for the inspiration. 

## Background And Rant - (Skip if you just want to see the tool)

AI coding assistants like _Claude Code_ and _Cursor_ store their chat histories locally. While the AI era has introduced many new ways of doing things, the underlying principles have not changed. Theres a new layer on top, and these tools keep history files like many other tools forensic analysis has relied on in the past i.e. mysql, bash, zsh, etc. 

Regardless of HOW these AI tools work, our concerns in security come down to many of the same functional issues, unauthorized access, execution, and so on. We may need to look at these differently and there may be new considerations or nuance to understand but as of today an execution event is still an execution event. For example, claude code executes its commands using a wrapper, but its still executing. Maybe the nuance is now we are looking for a commandline embedded in a wrapper rather then strictly looking at a parent child process relationship. 

The context may have changed but much has stayed the same. The hype and panic around AI in security is warranted for some issues but thinking we need to change our entire approach to security overnight is short sighted. If we approach AI tooling with a confidence of "we still understand how technology works fundamentally" and with a healthy bit of optimism of where these tools can improve our lives and workflows we will all be better off. 

See the tool here: [ai-hist](https://github.com/SWolfSec/ai-hist)

## Overview 

I came across [ghosttype](https://github.com/xFreed0m/ghosttype) a tool used for parsing these files to search for credentials. Two of the files it looks for are  _Claude Code_ histories in `~/.claude/` and _Cursor_'s application support database. I thought how useful these can be on an investigation when timelined properly. At the time of creating this, I only had personal access to _Cursor_ and _Claude Code_ but this can be expanded to the other history locations too. The idea is to easily timeline on a live system, a mounted forensic image, or point it directly at a history file retrieved from disk. Using this collected history we can see what was actually happening in the prompt itself to mirror with other gathered telemetry.

When _ai-hist_ runs against these local files it produces a timeline including the user inputs, AI responses, tool invocations, and what the outcome of those invocations was. Mapping these events to other telemetry sources can give us a full and holistic view. Output can be filtered by role or date range and exported as text, JSON, or CSV depending on what you need.

Note, this was all tested on a macbook therefore you may need to specify the files you want to timeline if pulled from a different OS. Mileage may vary.

## Cursor

_Cursor_ stores its interaction history in an application support database rather than flat json files. _ai-hist_ handles format detection automatically so you do not need to specify which parser to use. If this does not end up being true in some live environments, please let me know! 

![cursor](https://swolfsec.github.io/assets/img/img_2026-06-01-ai-hist-timeline/ai-hist-cursor-1.png){: .mx-auto.d-block :}

Above you can see the output of running against a specific _Cursor_ history db with some simulated breach data. 

## Claude Code

_Claude Code_ writes conversation history and tool results to `~/.claude/`. This includes the full prompt history between the user and the model. Such as tool calls made during a session like Bash commands, file reads, file edits, and their outputs. Cross reference this with your other telemetry. 

![claude](https://swolfsec.github.io/assets/img/img_2026-06-01-ai-hist-timeline/ai-hist-claude-2.png){: .mx-auto.d-block :}

With the fake sample data you can see how useful this information would be when reviewing an image or live system. 

## Full Timeline

With both _Cursor_ and _Claude Code_ artifacts parsed, _ai-hist_ can produce a single combined timeline. An attacker or insider may pivot between tools, with the full timeline we can match up both prompt results in one file. 

![full-timeline](https://swolfsec.github.io/assets/img/img_2026-06-01-ai-hist-timeline/ai-hist-full-timeline-3.png){: .mx-auto.d-block :}

The full timeline gives you key details like which user initiated "user" "assistant" etc. But also the tool that was used, in our screenshot you can see cursor activity that then turns into claude code activity. 

## Tool Use

One of the more valuable things _ai-hist_ surfaces is tool invocation history. So you can pull this out selectively and not do a full timeline. When an AI coding assistant runs a Bash command, reads a file, or makes an edit, that action is logged as a tool call. Reconstructing that sequence gives investigators a clear picture of what the assistant actually did on the system rather than just what it said. Note, this is primarily for _Claude Code_.

The `--verbose` flag includes additional metadata for each tool call. For investigations involving a mounted forensic image, the `--root` parameter points _ai-hist_ at the correct base path without requiring the artifacts to be in their default locations.

![tool-use](https://swolfsec.github.io/assets/img/img_2026-06-01-ai-hist-timeline/ai-hist-tool-use-4.png){: .mx-auto.d-block :}

Here is an output which is just using the tool-use argument so if you didn't want all the chat history, easy peasy. 


## End

Anyways, I thought this would be a cool idea to timeline out AI usage as time goes on these are going to be more and more relevant to investigations. Let me know if you end up using this, where it won and potential upgrades! 

## References
- https://github.com/xFreed0m/ghosttype
