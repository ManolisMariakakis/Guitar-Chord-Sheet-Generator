
# 🎸 Guitar Chord Sheet Generator

This project defines a structured format for representing guitar chord sheets using plain text, with support for transposing, rendering, printing, and exporting.

---

## 📝 Input Format

Each line follows the format:

```
Text@Type
```

- `Text`: The actual content (lyrics, chords, section names, etc.)
- `Type`: A single character defining how the line is interpreted

A blank line is automatically inserted before every new section.

All content is rendered using a **monospace font** to preserve alignment between chords and lyrics.

---

## 📘 Type Definitions

| Type | Name       | Description                                                                                     |
| ---- | ---------- | ----------------------------------------------------------------------------------------------- |
| `T`  | Title      | Indicates the start of a new section. May include structural tags (see below).                  |
| `H`  | Header Tag | Only used once to extract the internal song title                                               |
| `F`  | Footer     | Free-text lines (e.g. metadata) added at the end of the song in a separate section |
| `L`  | Lyrics     | A lyrics line, typically displayed beneath its corresponding chord line                         |
| `C`  | Chords     | A line containing chords aligned to the next lyrics line (`L`)                                  |

> ℹ️ Multiple lines ending in `@F` are grouped together into a footer block. These appear in a dedicated section after all lyrics, within a `<w>` marker.

---

## 🔖 Special Section Tags (`T`)

When the `Type` is `T`, the `Text` field may contain **section tags** to indicate structural parts of the song.

These are handled by the following label mapping:

| T Value  | Section Label |
| -------- | ------------- |
| `<w>`    | *(none)*      |
| `<w1>`   | *(none)*      |
| `<w2>`   | *(none)*      |
| `<w3>`   | *(none)*      |
| `<w4>`   | *(none)*      |
| `<c>`    | Refrain       |
| `<d>`    | Refrain B     |
| `<CODA>` | Coda          |

- If the label is empty (`''`), no label is shown.
- If a label exists, the section is rendered with it's label.
- A line separator appears between sections.

---

## 🔒 Special Internal Tags (`H`)

- There is exactly **one line** of type `H`.
- It is used to **extract the song title**.

Example:

```text
My Song Title@H
```

---

## 🎥 Example

```text
The Song Title@H
<w>@T
G       D       Em@C
This is the first verse line@L
A     D       Em@C
This is the second verse line@L
<c>@T
C       G       Am@C
This is the first refrain line@L
C         E       Am@C
This is the second refrain line@L
<w>@T
This is the first of next verse line@L
This is the second of next verse line@L
<w>@T
### Author: Author Name@F
### Composer: Composer Name@F
### Comments: Some comment here@F
```

In this example:

- The last `<w>` section begins the footer section.
- All `@F` lines under it are rendered as free text.
- These lines begin with `###` and contain metadata (e.g., Author, Composer, Comments).

---

## 📈 Rendering Input Text

The rendering process converts the input into structured HTML and performs the following steps:

1. **Splits the input** into lines.
2. **Parses each line** into `Text` and `Type`.
3. **Processes each type differently**:
   - `T`: Uses a label mapping to render a section heading, or just a break if the label is empty.
   - `C`: Detects and highlights chords, storing them for the next lyric line.
   - `L`: Displays lyrics, along with any chords collected just before.
   - `F`: Appends one or more lines of footer text at the end of lyrics, typically used for metadata..
   - `H`: Extracts the title.
4. **Renders all text using monospace font** for alignment.
5. **Triggers rendering of the chord chart** with diagrams for detected chords.

---

## 🎶 Process of Type `C` Chords

Lines of type `C` contain chords that are:

- Displayed in **bold blue monospace font** (Highlighted)
- Linked to a small hoverable **chord diagram** if a match is found
- Marked visually if no chord diagram is available (e.g., red or with a warning)

This makes it easier for users to recognize and learn the chords interactively.

---

## 🎵 Enharmonics: Normalizing Equivalent Notes

Some notes sound the same but are written differently — these are called **enharmonic equivalents**. To simplify processing, the system normalizes flat notes to their sharp equivalents using:

```js
const enharmonics = {
    'Bb': 'A#', 'Eb': 'D#', 'Ab': 'G#',
    'Db': 'C#', 'Gb': 'F#'
};
```

### Example:

- `Bb7` becomes `A#7`
- `Ebmaj7` becomes `D#maj7`
- `Db/F` becomes `C#/F`

This normalized form is then used for matching and transposing chords consistently.

---

## 🫠 Chord Shapes Collection


Chord Shapes Collection contains more than 12.000 items. Each chord name maps to two arrays of 6 values each:

- `frets`: array of fret numbers per string
- `fingers`: array of finger numbers (1–4), or null for open/muted

**The collection originally sourced** from [T-vK/chord-collection](https://github.com/T-vK/chord-collection)

### 🎸 **Fretboard array** starts from high E to low E:

Each chord shape is represented by an array of 6 values — one for each string, ordered from **high E** (string 1) to **low E** (string 6).

Each value means:

- A number: play this string
- `'x'`: mute the string
- `'0'`: open string

### ✋ Fingering Variations

There are often **multiple valid fingerings** for the same chord shape. This collection uses the most **common and simple** fingering option for each chord.

Each value of the **`fingers` array** corresponds to one of the six strings (from high E to low E) and indicates **which finger** presses the string — if any:

- `0`: no finger used
- `1`: index finger
- `2`: middle finger
- `3`: ring finger
- `4`: pinky


#### Chord Item Example:

```js
"C": {
  frets: ['0', '1', '0', '2', '3', 'x'],
  fingers: ['0', '1', '0', '2', '3', '0']
}
```

Means:

- String 1 (high E): '0' → open → finger: 0
- String 2 (B): '1' → fret 1 → finger: 1 (index finger)
- String 3 (G): '0' → open → finger: 0
- String 4 (D): '2' → fret 2 → finger: 2 (middle finger)
- String 5 (A): '3' → fret 3 → finger: 3 (ring finger)
- String 6 (low E): 'x' → muted → finger: 0

---

## 🎨 How Chord Shapes Are Rendered to Diagrams

Once a chord item is available, a process draws a fretboard diagram:

- A 5-row by 6-column **grid** represents 5 frets across 6 strings.
- The function loops through the `shape` array.
- Wherever a fret value matches a row, it draws a **black dot**.
- If a string is marked `'x'`, it's shown as **muted** (×) above the grid.
- If the value is `'0'`, it's shown as an **open circle** (○) above the string.
- The `fr` (fret number) determines the **starting fret** displayed on the diagram.
- The `fingers` array is used to **label dots with finger numbers** (1–4).
- If multiple strings are pressed on the same fret with the same finger, it draws a **barre line** across those strings.

### 🎸 Example: G♯ Major Barre Chord

This diagram shows a G♯ chord starting at the 4th fret, using a barre and multiple fingers:

![G# Barre Chord](./G%23.png)

- `4fr` means the diagram starts at the **4th fret**.
- Finger `1` is used as a **barre** across strings E, B, and e.
- Other fingers press the appropriate frets on A (3), D (4), and G (2).

This is a common **E-shape barre chord**, used widely across styles.

---

## 🔄 How Transposing Works

The system can **transpose chords up or down** by semitones. This is useful for adapting songs to different vocal ranges or instruments.

### 👥 Step-by-step Process:

1. **Parse the chord** into:
   - Root note (e.g. `C`, `D#`)
   - Suffix (e.g. `m7`, `sus4`, `7`, etc.)
   - Optional bass note (e.g. `C/E` → bass is `E`)
2. **Normalize the root and bass** to sharp equivalents (e.g. `Bb` → `A#`).
3. **Transpose** the root and bass by a number of semitones using a 12-note scale:
   ```
   ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B']
   ```
4. **Rebuild the transposed chord** by combining:
   - The new root note
   - The unchanged suffix
   - The transposed bass (if any)
5. **Look up the transposed chord** in `Chord Shapes` table.

---

### 🎹 Transposition Table (12 semitone steps)

| Original | +1 | +2 | +3 | +4 | +5 | +6 | +7 | +8 | +9 | +10 | +11 |
| -------- | -- | -- | -- | -- | -- | -- | -- | -- | -- | --- | --- |
| C        | C# | D  | D# | E  | F  | F# | G  | G# | A  | A#  | B   |
| D        | D# | E  | F  | F# | G  | G# | A  | A# | B  | C   | C#  |
| E        | F  | F# | G  | G# | A  | A# | B  | C  | C# | D   | D#  |
| F        | F# | G  | G# | A  | A# | B  | C  | C# | D  | D#  | E   |
| G        | G# | A  | A# | B  | C  | C# | D  | D# | E  | F   | F#  |
| A        | A# | B  | C  | C# | D  | D# | E  | F  | F# | G   | G#  |
| B        | C  | C# | D  | D# | E  | F  | F# | G  | G# | A   | A#  |

---

### 🎼 Chord Line Transposition Example

Original:

```
C       G       Am      F
```

Transpose +2:

```
D       A       Bm      G
```

---

### 📊 Visual: Semitone Movement

Here’s how a single note (e.g. `F`) moves up the scale by semitones:

```
F → F# → G → G# → A → A# → B → C → C# → D → D# → E → F
```

Each step to the right represents a +1 semitone shift.

This visual helps understand how any root or bass note is transposed.

---

## 🧩 Core JavaScript Functions

This project includes several core functions that handle parsing, rendering, transposition, and chord diagram generation.

### 📦 Main Functions

| Function | Purpose |
|---------|---------|
| `render()` | The main rendering function. Parses each `Text@Type` line and builds the structured HTML layout of the chord sheet. |
| `highlightChords(line)` | Parses a chord line and wraps each chord in a span with optional hoverable diagram support. |
| `renderDiagram(shape, finger)` | Renders a 5x6 fretboard grid using a chord shape array and an finger array. Places dots and finger numbers on the grid. |
| `renderChordChart()` | Shows all used chords in a visual chart below the lyrics. Uses `renderDiagram()` to display each chord. |
| `transposeNote(note, step)` | Transposes a musical note by a number of semitone steps. Supports enharmonic normalization (e.g., `Bb → A#`). |
| `transposeChord(chord, step)` | Transposes a full chord (root + suffix + optional bass note). Uses `transposeNote()` internally. |
| `transposeChords(step)` | Transposes all chords in the document by a given number of semitones and refreshes the diagrams and layout. |
| `getOrFetchChordShape(chord)` | Retrieves the shape array for a chord from `chordShapes`, or fetches it from a server if not cached. |
| `applyFretSize()` | Adjusts the CSS grid sizing of the chord diagrams based on the user’s selected fret size. |
| `populateDebugTableWithoutTranspose()` | Debug function that logs original chords, parsed components, and their existence in `chordShapes`. |

---

### 🧠 Utility & Event Functions

| Function or Handler | Description |
|---------------------|-------------|
| `drawHeader(pdf)` | Adds a custom header (project name and link) to each page of the exported PDF. |
| `drawFooter(pdf)` | Adds a footer (including URL and page number) to the exported PDF. |
| `#download-pdf` Click Handler | Captures the rendered DOM as an image and uses `jsPDF` to create a downloadable PDF. |
| `.step-button` Click Handler | Increases or decreases font size or fretboard size and applies the update live. |

## 🌐 AJAX Requests and Data Fetching

This project uses two AJAX calls to dynamically load data needed for rendering the chord sheet:

### 🎸 1. Chord Diagram Fetching

When a chord appears in the input that is not already stored in memory, the system performs a request to retrieve its shape (a list of fret positions for each string). This shape data is used to draw the corresponding chord diagram. The request is synchronous to ensure the shape is available before rendering continues.

In the current demo setup, this data is loaded from a static local file (`guitar.html`). In a full implementation, this would be replaced by a server-side script that returns the appropriate chord diagram in JSON format.

---

### 🎼 2. Chord Sheet Content Loading

When the page loads, another request is triggered to load the full chord sheet data. This includes all the `Text@Type` lines that define the structure of the song. Once retrieved, the data is passed to the `render()` function to build the song view.

This fetch is performed using a static file (`chord_sheet.html`) in test mode. In production, the sheet would be dynamically selected via a URL parameter:

```
?c=551
```

The `c` parameter represents the **song ID** used to fetch the correct song content from the server.

For testing purposes, a fixed value (`1`) is used, and the file is loaded statically without any real query resolution.

---

> ⚙️ These AJAX calls are essential for loading and displaying data without requiring a full page reload, and allow the system to remain flexible and modular in how it handles chord diagrams and song content.

---

## 📁 Project Files

This project includes the following key files:

| File Name              | Description                                                                 |
|------------------------|-----------------------------------------------------------------------------|
| `index.html`           | The main application file that loads and renders chord sheets in the browser |
| `guitar.html`          | A sample static file returning chord item data in JSON format (used for testing) |
| `chord_sheet.html`     | A sample static file containing the input song text in `Text@Type` format    |
| `import2table.sql`     | SQL statements for importing the chord shape collection into a `quitar` table. Example: `INSERT INTO guitar(note, fret, finger) VALUES ('C', '["x", "3", "2", "0", "1", "0"]', '["0", "3", "2", "0", "1", "0"]');` |
| `logo.png`             | Logo used in the header           |
