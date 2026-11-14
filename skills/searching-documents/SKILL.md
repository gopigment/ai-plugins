---
name: searching-documents
description: "Use this skill with tool:search_documents to find and use the files stored in the user's applications (plans, assumptions, contracts, board decks, policies, exports), alongside the blocks and boards searches whenever you look for a business concept or topic: when to search files, and how to read, cite and name what it returns."
metadata:
  base_directory: /skills/searching-documents
  includes:
    - "SKILL.md"
---

# Searching the User's Files

Besides Pigment blocks, the user's applications hold files — plans, assumptions, contracts, board decks, policies, exports — stored under `/shared_uploads/` and shown to users as **Files** in an application's sidebar. They often hold what a metric does not: why a number moved, what an assumption was, what a plan or policy says. `tool:search_documents` finds them by name and by content, across every application the user can read.

## When to Search Files

- **Alongside blocks.** When you search for a business concept or topic, call `tool:search_documents` in parallel with the block searches. Skip it only when the request explicitly names a Pigment block type ("the revenue metric", "the list of accounts"), or the block is already resolved and the question is about its data.
- **First**, when the user refers to a document — a file, deck, presentation, report, contract, plan, policy or export, or "what I uploaded" — or asks about reasoning, assumptions, definitions or decisions rather than figures.
- **Before saying nothing exists**: if the block searches find nothing relevant, a file may hold the answer.
- Files attached to this conversation are listed in `<file_attachments>`; read those directly. `tool:search_documents` covers the stored ones.

## Using What It Returns

- A result's description says what a file is about; it is not the file's content. Read the file with `tool:read_file` on the returned path before quoting it or relying on what it contains.
- When an answer combines Pigment data and a file, say which part comes from which.
- Name a file to the user by its name and the application whose Files it is in, never by its `/shared_uploads/` path. Give file sizes in MB, as the tools report them.
- When a file cannot be read, pass on the reason the tool gives, and what the user can do about it.
