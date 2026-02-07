# TestExcel

## Meme Generator (Генератор Мемов) 🎭

A simple, web-based meme generator that allows you to create memes with custom text overlays on images.

### Features

- 📁 Upload your own images
- 🖼️ Use popular meme templates
- ✏️ Add top and bottom text
- 🎨 Customize font size, color, and style
- 💾 Download your memes as PNG files
- 🌐 Works entirely in the browser (no server needed)

### How to Use

1. **Open the Meme Generator**
   - Simply open `meme-generator.html` in your web browser
   - Or visit the GitHub Pages URL (if hosted)

2. **Choose an Image**
   - Click "Загрузить изображение" to upload your own image
   - Or select one of the popular meme templates

3. **Add Text**
   - Enter text in the "Верхний текст" field for top text
   - Enter text in the "Нижний текст" field for bottom text
   - The meme updates automatically as you type

4. **Customize**
   - Adjust font size (10-100)
   - Change text color
   - Change stroke/outline color
   - Select different fonts

5. **Download**
   - Click "Скачать мем" to download your creation

### Local Usage

No installation required! Just open the HTML file:

```bash
# On macOS
open meme-generator.html

# On Linux
xdg-open meme-generator.html

# On Windows
start meme-generator.html
```

Or use a local web server:

```bash
# Python 3
python -m http.server 8000

# Then open http://localhost:8000/meme-generator.html
```

---

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
