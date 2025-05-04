# UFO Replacer

![{D56818B9-8DA8-47E0-8F9C-D2FE9BFAF8D9}](https://github.com/user-attachments/assets/c71cf01f-e04c-45fb-9692-7d8f1d194ccc)



🔎 **UFO Replacer** is a powerful and intuitive web-based tool for performing bulk find-and-replace operations on text. Built with Vue.js and TypeScript, it offers a modern interface for managing multiple replacement rules, real-time highlighting, file uploads, and preset management. Whether you're editing code, cleaning data, or transforming text, UFO Replacer makes the process efficient and user-friendly.

🌐 **Live Demo**: https://uforeplacer.vercel.app
📂 **Repository**: [github.com/aisurf3r/UFOreplacer](https://github.com/aisurf3r/UFOreplacer)

## Features

- **Bulk Find & Replace**: Define multiple find-and-replace pairs to process text in one go.
- **Real-Time Highlighting**: Matches are highlighted in the input text with customizable colors for each replacement rule.
- **Regex Support**: Use regular expressions for advanced pattern matching, with options for case sensitivity and whole-word matching.
- **File Upload**: Import text files (TXT, CSV, HTML, CSS, JS, JSON, XML, MD) up to 150kb for processing.
- **Preset Management**: Save, load, and delete replacement rule sets for reuse.
- **Undo/Redo**: Easily revert or reapply changes with a robust history system.
- **Multilingual**: Supports English and Spanish interfaces.
- **Syntax Highlighting**: Processed output is displayed with code-friendly formatting using Highlight.js.

## Installation

To set up UFO Replacer locally, follow these steps:

### Prerequisites
- [Node.js](https://nodejs.org/) (v16 or higher)
- [npm](https://www.npmjs.com/) or [yarn](https://yarnpkg.com/)

### Steps
1. **Clone the repository**:
   ```bash
   git clone https://github.com/aisurf3r/UFOreplacer.git
   cd UFOreplacer
   ```

2. **Install dependencies**:
   ```bash
   npm install
   ```

3. **Run the development server**:
   ```bash
   npm run dev
   ```
   Open your browser and navigate to `http://localhost:5173` (or the port specified in the console).

4. **Build for production**:
   ```bash
   npm run build
   ```
   The compiled files will be generated in the `dist` folder.

5. **Preview the production build**:
   ```bash
   npm run preview
   ```
   This serves the `dist` folder locally, typically at `http://localhost:4173`.

## Usage

1. **Input Text**:
   - Paste text directly into the input panel or upload a supported file (TXT, CSV, HTML, etc.).
   - The input panel supports real-time editing with match highlighting.

2. **Define Replacements**:
   - Add find-and-replace pairs in the sidebar.
   - Enable options like **Case Sensitive**, **Whole Words**, or **Use Regex** for precise matching.
   - Navigate matches using the **Next** and **Previous** buttons (long-press to jump to first/last match).

3. **Process Output**:
   - View the processed text in the output panel with syntax highlighting.
   - Copy the output to the clipboard or download it as a `.txt` file.

4. **Manage Presets**:
   - Save replacement rules as presets for reuse.
   - Load or delete existing presets from the sidebar.

5. **Undo/Redo**:
   - Use the **Undo** (Ctrl+Z) and **Redo** (Ctrl+Y) buttons to navigate the history of changes.

## Project Structure

```
UFOreplacer/
├── src/
│   ├── components/
│   │   └── ReplacementForm.vue  # Main component for the app
│   ├── App.vue                 # Root component
│   ├── main.ts                 # Entry point
│   └── index.css               # Global styles
├── public/                     # Static assets
├── dist/                       # Compiled output (after build)
├── vite.config.ts              # Vite configuration
├── package.json                # Project dependencies and scripts
└── README.md                   # This file
```

## Configuration

The project uses [Vite](https://vitejs.dev/) as the build tool. To customize the build process, edit `vite.config.ts`:

```typescript
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
  plugins: [vue()],
  base: './' // Use relative paths for local testing
});
```

- Set `base: '/'` for deployment to a domain root or `base: '/subpath/'` for a subdirectory.
- Ensure the `dist` folder is served via a web server (e.g., `http-server` or `npm run preview`) for production testing, as opening `index.html` directly with `file://` will not work.

## Contributing

Contributions are welcome! To contribute to UFO Replacer:

1. Fork the repository.
2. Create a new branch:
   ```bash
   git checkout -b feature/your-feature
   ```
3. Make your changes and commit:
   ```bash
   git commit -m "Add your feature"
   ```
4. Push to your fork:
   ```bash
   git push origin feature/your-feature
   ```
5. Open a pull request on the [main repository](https://github.com/aisurf3r/UFOreplacer).

Please include a detailed description of your changes and ensure your code follows the project's coding style.

## Issues

If you encounter bugs or have feature requests, please open an issue on the [GitHub repository](https://github.com/aisurf3r/UFOreplacer/issues). Include:
- A clear description of the issue.
- Steps to reproduce.
- Screenshots or error messages, if applicable.

## License

This project is licensed under the [MIT License](LICENSE). See the LICENSE file for details.

## Acknowledgements

- Built with [Vue.js](https://vuejs.org/) and [TypeScript](https://www.typescriptlang.org/).
- Syntax highlighting powered by [Highlight.js](https://highlightjs.org/).
- Developed by [aisurf3r](https://github.com/aisurf3r).

---
TODO: Fix placeholders in main boxes.
      Text edition and focus lost in main in input.

🚀 Happy text replacing with UFO Replacer! 🛸
