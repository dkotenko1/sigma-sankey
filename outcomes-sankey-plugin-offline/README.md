# Sankey Plugin — Terminating / Originating Flows (Sigma Computing)

A single-file Sigma Computing plugin that renders a Sankey / flow diagram where bars can
**terminate early** and **begin late** instead of routing every flow into a "null" node.
Built on ECharts. No build step, no server-side code — it's one static HTML file.

## What's in this folder
- `index.html` — the entire plugin (this is all you host).

## 1. Host it (pick any static HTTPS host)
The plugin is a static file. Host `index.html` at an HTTPS URL:
- **Netlify / Vercel / Cloudflare Pages** — drag-and-drop the folder, or connect a repo.
- **GitHub Pages** — commit `index.html`, enable Pages.
- **AWS S3 + CloudFront**, Azure Blob + CDN, or any internal static web server.

Sigma requires the plugin be served over **HTTPS**. The URL you host it at (e.g.
`https://your-host.example.com/`) is what you register in step 2.

### Air-gapped / CDN-restricted environments (read if your org blocks external CDNs)
By default `index.html` loads three libraries at runtime from public CDNs:
- `https://unpkg.com/react@18.3.1/umd/react.production.min.js` (the Sigma SDK's UMD build requires React on the page)
- `https://unpkg.com/@sigmacomputing/plugin@1.2.0/dist/umd/sigmacomputing-plugin.umd.js`
- `https://cdn.jsdelivr.net/npm/echarts@5.5.0/dist/echarts.min.js`

If your environment blocks those, use the **self-contained bundle** instead (the
`-offline` folder): it ships all three files locally with the `<script>` tags already
repointed, so it has zero external runtime dependencies. (Or download the three files
yourself, place them next to `index.html`, and change the three `<script src="...">`
tags to the local filenames.)

## 2. Register it in your Sigma org (one-time, admin)
**UI:** Administration → Account → **Plugins** → *Add plugin* → paste your hosted HTTPS URL → name it.

**API (optional):**
```
POST /v2/plugins
{ "name": "Intervention Sankey", "url": "https://your-host.example.com/", "type": "element" }
```

## 3. Use it in a workbook
1. Add a **Plugin** element to a page and select this plugin.
2. In the plugin's editor panel, set **Source element** to a table/data element that has one
   row per flow (or raw rows — the plugin aggregates duplicates automatically).
3. Bind three columns:
   - **Source stage** — the "from" node
   - **Target stage** — the "to" node
   - **Flow value** — a numeric measure (member/patient/record count, $, etc.)

To make a flow **terminate** (end at a stage) or **originate** (enter mid-flow), put a null /
blank / `null` value in the Target (terminate) or Source (originate) column of that row. See
**Null / missing handling** below.

## Configuration options (editor panel)
| Option | Values | Notes |
|---|---|---|
| **Null / missing handling** | Terminate & originate · Show as node · Drop rows | *Terminate & originate* (default) is the feature: null target → the bar ends; null source → the bar begins late. *Show as node* renders a literal "(null)" node. *Drop rows* removes them. |
| **Values treated as null** | comma list (default `null, n/a, none, blank`) | Which cell values count as null, in addition to true blanks. |
| **Orientation** | Horizontal · Vertical | |
| **Node alignment / width / spacing** | | Layout controls. Nodes are also drag-to-reposition. |
| **Link color** | Gradient · Source · Target | |
| **Link curve / opacity** | | |
| **Show node labels / Label detail** | Name · Name+value · Name+% · Value only | |
| **Number format / Value prefix** | Auto (K/M) · Full · Thousands · `$` etc. | |
| **Highlight on hover / Tooltips / Legend** | on/off | |
| **Color palette** | Outcomes · Teal · Vibrant · Colorblind-safe | |
| **Title / Subtitle** | text | |

## Robustness notes
- **Cycles are handled.** A Sankey must be a directed acyclic graph; cyclic or self-referencing
  links are detected and dropped (with an on-screen note) instead of crashing the chart.
- **Raw rows are fine.** Duplicate source→target pairs are summed automatically.
- **Resizes cleanly** inside Sigma's panel (redraws on every resize).
- Non-numeric / negative / zero flow values and unbound columns are handled gracefully.

## License / support
Provided as-is for use in Sigma Computing workbooks.
