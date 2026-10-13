# songs-database

A file-based, Git-backed database of song-title and artist recognitions for screenshots in the Hugging Face dataset [`for1096/songs`](https://huggingface.co/datasets/for1096/songs).

This repository is intentionally a **data repository**, not an application or an MCP server. ChatGPT (or another contributor) reads these instructions, discovers source images, recognizes songs, and records results as JSON files committed to Git.

## Task

Gradually build and improve a searchable collection of song recognitions from the source dataset.

When asked to continue the task:

1. Read this README before making changes.
2. Inspect the live Hugging Face dataset file listing; do not guess filenames or assume the dataset layout has not changed.
3. Match each source image path to its corresponding JSON path in this repository.
4. Read existing records before recognizing or adding anything.
5. Work on a manageable batch, prioritizing images with no record. If no unprocessed images can be reliably identified, report that limitation rather than claiming the dataset is complete.
6. Validate every changed JSON file, commit the changes, and report which files were processed, what changed, and any unresolved uncertainty.

A suggested prompt is:

> @GitHub Continue the task in bombless/songs-database. Read README.md first, inspect the Hugging Face dataset file list, and add or improve a small batch of song-recognition JSON files. Preserve existing records, validate JSON, commit the changes, and summarize the result and remaining work.

## Source dataset and file mapping

- Dataset: [for1096/songs](https://huggingface.co/datasets/for1096/songs)
- Hugging Face Hub tree API: `https://huggingface.co/api/datasets/for1096/songs/tree/main`
- The API supports directory and recursive-listing parameters. Check the current API response and pagination before relying on a file list; follow any returned continuation token or pagination mechanism.
- Preserve each image's path relative to the dataset repository. For an image at `aft/pic01.png`, store its recognition data at `aft/pic01.png.json`.
- Do not copy the image files into this repository. Store only recognition data and project documentation.
- Treat the source image path (including its directory) as the identity key. Different directories may contain files with the same basename.
- Only create records for source images that actually exist. Do not invent filenames or assume that an example path is present in the live dataset.

## Directory structure

Mirror the source dataset's relative directory layout for recognition files:

```text
.
├── README.md
├── aft/
│   ├── pic01.png.json
│   └── pic02.png.json
└── <other source directories>/
    └── <image filename>.<extension>.json
```

The tree above is illustrative; actual directories and image names must be discovered from the live dataset.

## JSON schema

Each image has one UTF-8 JSON file containing a JSON array of recognition candidates. Use an empty array (`[]`) only when the image has been examined and no plausible song candidate can be identified. A missing file means no record has been created yet; it does not by itself prove the image is unprocessed.

Each array item must be a JSON object with these fields:

- `title` (string, required): recognized song title. Prefer the title as displayed in the screenshot and preserve its original script/language. Do not translate it unless translation is needed to distinguish a known alternate title.
- `singer` (string, required): singer or artist when identifiable; use `"unknown"` when not reliably identifiable.
- `confidence` (number, optional): model-estimated confidence between `0` and `1`. This is a subjective estimate, not a calibrated probability. Omit it if a meaningful estimate cannot be made.
- `notes` (string, optional): brief explanation of ambiguity, alternate spelling, or why the candidate needs review.

Example:

```json
[
  {
    "title": "One More Time, One More Chance",
    "singer": "Masayoshi Yamazaki",
    "confidence": 0.92
  }
]
```

If the title is readable but the singer is not, this is valid:

```json
[
  {
    "title": "One More Time, One More Chance",
    "singer": "unknown"
  }
]
```

If there are multiple plausible songs, store multiple candidate objects and explain the ambiguity in `notes` where useful. Do not present an uncertain guess as a verified fact. Never add extra fields with inconsistent meanings across files.

## Recognition and data-quality rules

1. **Use the screenshot as evidence.** Prefer readable on-screen text. Use visual context or external lookup to resolve ambiguity when available, but do not claim external verification unless it was actually performed.
2. **Preserve uncertainty.** If the title itself is uncertain, record only plausible candidates and explain the ambiguity. If no useful candidate can be identified, use `[]` after examining the image.
3. **Do not invent an artist.** Use `"unknown"` unless the artist can be identified with reasonable confidence.
4. **Avoid duplicate entries.** Before adding a candidate, compare it with existing entries in the same file. Do not add an exact duplicate. Normalize only obvious formatting differences; do not merge titles merely because they look similar.
5. **Preserve useful existing data.** Read the current file before editing. Merge new evidence into the existing array instead of replacing it wholesale. Do not remove an earlier candidate merely because a new guess differs.
6. **Keep candidate ordering useful.** Put the best-supported candidate first, followed by alternatives in descending order of support. Ordering is not a guarantee of correctness.
7. **Do not silently upgrade guesses to facts.** If new evidence contradicts an existing record, retain relevant alternatives or explain the conflict in `notes`; only remove a record when there is a clear reason.
8. **Keep files valid and focused.** Use valid JSON, UTF-8, and ordinary JSON double-quoted strings. Do not add Markdown comments or trailing commas inside JSON.
9. **Do not fabricate progress.** Report only files actually inspected or changed. If an image URL cannot be opened, an API call fails, or pagination is incomplete, state the limitation.
10. **Do not modify unrelated files.** This repository stores recognition data and its rules; do not add application code, binaries, downloaded screenshots, credentials, or unrelated datasets.

## Progress and commits

Git history is the audit trail. Use concise commit messages such as:

- `Add recognitions for aft screenshot batch`
- `Improve uncertain song candidates for aft/pic01.png`
- `Correct artist metadata for selected screenshots`

For each work session:

- Prefer a small, reviewable batch over an unbounded sweep.
- Validate changed JSON files before committing.
- Do not overwrite concurrent changes: re-read the latest file if an update fails because the file changed.
- In the final report, summarize the source paths examined, files created or updated, uncertain cases, and a sensible next step.
- Never claim the whole dataset is complete unless the full, current source listing has been traversed and every relevant image has been accounted for.

## Scope and limitations

This repository stores **recognition candidates**, not authoritative ground truth. Records can be incomplete or wrong and may need review.

ChatGPT's ability to complete a batch depends on the active tools and permissions. A GitHub connection does not automatically guarantee arbitrary HTTP access to Hugging Face or access to image contents. If the source listing, image, or GitHub write operation is unavailable, report that limitation instead of pretending the task succeeded.

The repository does not require a database server, build step, or runtime. Its source of truth is the committed JSON files and Git history.
