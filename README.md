# TestExcel

## Yandex Disk Tree Scanner

This repository includes a GitHub Actions workflow that automatically scans a public Yandex Disk share and generates a CSV file with the complete folder structure.

### Setup

1. **Add Repository Secret**
   
   Go to your repository settings:
   - Navigate to `Settings` → `Secrets and variables` → `Actions`
   - Click `New repository secret`
   - Name: `PUBLIC_KEY`
   - Value: Your Yandex Disk public share URL (e.g., `https://disk.yandex.ru/d/tvV3m-x6GxFmhA`)
   - Click `Add secret`

2. **Run the Workflow**
   
   The workflow runs automatically every 6 hours, or you can trigger it manually:
   - Go to the `Actions` tab in your repository
   - Select `Scan Yandex Disk Tree` workflow
   - Click `Run workflow` button
   - Select the branch and click `Run workflow`

3. **Download the Artifact**
   
   After the workflow completes:
   - Go to the workflow run page
   - Scroll down to the `Artifacts` section
   - Download `ydisk-tree` (contains `tree.csv`)

### CSV Format

The generated `tree.csv` file contains:
- `path` - Full path of the item
- `type` - Either `dir` or `file`
- `name` - Item name
- `size` - File size in bytes (0 for directories)
- `modified` - Last modification timestamp

### Optional Configuration

You can set the `MAX_DEPTH` environment variable in the workflow to limit recursion depth (default: unlimited).

### Local Testing

To run the scanner locally:

```bash
export PUBLIC_KEY="https://disk.yandex.ru/d/tvV3m-x6GxFmhA"
node ydisk_tree.js
```

This will generate `tree.csv` in the current directory.
