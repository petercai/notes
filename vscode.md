# extension Rewrap

In VS Code, you can automatically wrap pasted text at 80 characters using the **Rewrap** extension. Here's how to set it up:

1. **Install the Rewrap extension**:
   - Press `Ctrl+Shift+X` (or `Cmd+Shift+X` on Mac) to open Extensions
   - Search for "Rewrap" by stkb
   - Click Install

2. **Configure the wrap column**:
   - Open Settings (`Ctrl+,` or `Cmd+,`)
   - Search for "rewrap"
   - Set "Rewrap: Wrap Column" to 80

3. **Use it when pasting**:
   - Paste your text normally (`Ctrl+V`)
   - Then immediately run Rewrap: `Alt+Q` (or `Option+Q` on Mac)
   - Or use Command Palette: `Ctrl+Shift+P` → "Rewrap Comment/Text"

**Alternative: Manual word wrap setting**

If you just want visual wrapping (not actual line breaks), you can:
- Open Settings and search for "word wrap"
- Set "Editor: Word Wrap" to "wordWrapColumn"
- Set "Editor: Word Wrap Column" to 80

This won't insert actual newline characters but will display text wrapped at 80 characters.

**For automatic formatting on paste**, you could also create a keyboard shortcut or macro that pastes and then immediately rewraps, but this requires additional setup with extensions like "multi-command".

Which approach would work best for your use case?