# ROI Patch Review

A single, self-contained HTML page for reviewing a computational-pathology model's patch-level
predictions against an expert's judgement — confirm, correct, or reject what the model proposed,
patch by patch, over a grid of image regions of interest (ROIs).

It has no server component, no build step, and no backend of its own. It reads a plain folder of
files and writes its labels back as a file in the same shape, so it works the same way whether
that folder is served over HTTP or opened directly from disk.

## Try it

Download `index.html`, put it next to a review set shaped as below, and open it — either by
running a static file server in that folder (`python3 -m http.server`, or any static host) or, in
Chrome or Edge, by opening the page and clicking **Open local folder…**, which needs no server at
all.

## Folder contract

```
manifest.json      list of ROIs + the model's suggested patch grid for each
tiles/<id>.jpg      one image per ROI, named by its id in manifest.json
labels.json         written by the tool; the reviewer's labels, keyed by ROI id
```

`manifest.json`:

```jsonc
{
  "dataset_id": "my-model-v1",              // stable key, used to keep labels apart in a browser
  "model": "my-model",                      // free text, shown in the header
  "unit": "description of the ROI/patch size",  // free text, shown in the header
  "classes": [                              // the model's own label set, in palette order
    { "code": "A", "name": "Class A", "color": "#e0453a", "key": "1" },
    { "code": "B", "name": "Class B", "color": "#36a457", "key": "2" }
  ],
  "legend": ["any dataset-specific caveats, shown above the built-in legend"],
  "rois": [
    {
      "id": "slide01_r011c010",             // must match tiles/<id>.jpg
      "patient": "patient01",               // optional, used by the patient/case filter
      "slide": "slide01",                   // optional
      "x0": 169984, "y0": 163840,           // optional, shown in the ROI bar
      "n_tumour": 1,                        // optional, any free numeric field, shown in the ROI bar
      "grid": [["A", null, ...], ...]       // one row per patch row, one class code or null per cell
    }
  ]
}
```

`labels.json`, written by the tool, read back on the next open:

```jsonc
{
  "slide01_r011c010": {
    "labels": { "11-6": "A" },              // "row-col" -> class code, only cells you changed
    "accept_rest": false,                   // true = treat every unchanged suggestion as confirmed
    "verdict": "A",                         // whole-ROI call, or null
    "note": "free text",
    "reviewed": true
  }
}
```

## Two ways to run it, same page either way

- **Served over HTTP** (GitHub Pages, any static host, or a local `python3 -m http.server`): the
  page fetches `manifest.json` and the tiles normally. Labels save to the browser's local storage,
  keyed by `dataset_id`; use **Copy / download labels.json** to get a copy out, and drop it back
  into the folder as `labels.json` for the next session or the next reviewer to pick up, or use
  **Import labels.json** to merge one back in.
- **Opened as a local folder** (Chrome or Edge): click **Open local folder…** and pick the dataset
  folder directly, including just by double-clicking `index.html` with no server running. Labels
  write straight back into `labels.json` in that same folder as you work.

Switching between the two mid-session (e.g. starting over HTTP, then clicking "Open local
folder…" for direct writes) carries your in-progress labels over.

## Credits

Built with Claude Code (Anthropic) as an AI coding assistant, directed and reviewed by the author.

## License

MIT — see `LICENSE`.
