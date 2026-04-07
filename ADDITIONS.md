# Plan: Add Confluence Labels 

## Context

pages are published to Confluence (space CAG) but have no labels, making them hard to discover via Confluence search/indexing. We need to add both static labels (common to all pages) and per-resident labels after each page is created or updated.

## Option 1: MCP Server Label Support (Plan for forked repo)

### Current state

The `b1ff/atlassian-dc-mcp` repo already has auto-generated client code for labels:
- `packages/confluence/src/confluence-client/services/ContentLabelsService.ts` — `addLabels()`, `labels()`, `deleteLabel()`, `deleteLabelWithQueryParam()`
- Not wired up to any MCP tool registration

### What to add

Three new files/changes in `packages/confluence/src/`:

1. **`confluence-service.ts`** — add three methods:
   - `getContentLabels(contentId: string)` → calls `ContentLabelsService.labels()`
   - `addContentLabels(contentId: string, labels: Array<{prefix: string, name: string}>)` → calls `ContentLabelsService.addLabels()`
   - `removeContentLabel(contentId: string, label: string)` → calls `ContentLabelsService.deleteLabel()`

2. **`confluenceToolSchemas.ts`** (or wherever schemas live) — add Zod schemas:
   - `getContentLabelsSchema` — `{ contentId: z.string() }`
   - `addContentLabelsSchema` — `{ contentId: z.string(), labels: z.array(z.object({ prefix: z.string().default('global'), name: z.string() })) }`
   - `removeContentLabelSchema` — `{ contentId: z.string(), label: z.string() }`

3. **`index.ts`** — register three new `server.tool()` calls:
   - `confluence_getContentLabels`
   - `confluence_addContentLabels`
   - `confluence_removeContentLabel`

### PR scope

- ~100 lines of new code
- No breaking changes — purely additive
- Follows existing patterns in the codebase (same service → schema → tool registration flow)
