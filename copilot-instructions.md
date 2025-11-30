# Copilot Instructions - Planning Pro

## Project Overview

**Planning Pro** is a single-file Vue-less web application for annual shift planning with an 8-day cycle (5 work days / 3 rest days). It's a fully client-side HTML/CSS/JavaScript application using localStorage for persistence and external CDN libraries for export features.

### Core Architecture

- **File Structure**: Single `index.html` file (~2,300 lines) containing all markup, styles, and logic
- **Deployment**: Static HTML file (no build step required)
- **Data Layer**: Browser localStorage with year-based keys (`planningPro_${year}`)
- **Language**: French (UI, variable names, comments)

## Critical Patterns & Conventions

### 1. Shift Cycle Logic (`getDayStatus()`)
The application uses a specific shift pattern - **understand this before modifying scheduling**:
- **Reference date**: April 1, 2025 starts a new cycle
- **Pattern**: 5 consecutive work days, followed by 3 consecutive rest days
- **Before April 1, 2025**: Different legacy pattern (2 rest, 5 work, 1 rest)
- Code location: Lines ~1250-1270

**Key function**: `getDayStatus(date)` determines day type (travail/repos) based on math: `diffDays % 8`

### 2. Data Structure
```javascript
planningData = {
  "2025-01-15": { status: "repos", note: "Optional text" },
  "2025-01-16": { status: "travail", note: "" }
}
```
- Keys: ISO date format (`YYYY-MM-DD`)
- Status values: `repos`, `travail`, `formation`, `conges`, `maladie`
- Always use `formatDate(date)` helper to generate keys

### 3. Color Mapping
CSS variables (lines 14-22) define all day type colors:
- `--repos: #58d68d` (green)
- `--travail: #faef03` (yellow)
- `--formation: #FFC300` (gold)
- `--conges: #c443f5` (purple)
- `--maladie: #ffffff` (white, bordered)

Use `getColorByStatus(status)` to resolve status to color variable.

### 4. Export Implementations
**Excel Export** (`exportToExcel()`, lines ~1330-1380):
- Uses XLSX library from CDN
- Iterates all 365 days, applies statuses, includes notes
- Status values displayed in French

**PDF Export** (`exportToPDF()`, lines ~1390-1480):
- Uses html2canvas + jsPDF from CDN
- First generates 4-page layout with 3 months per page via `createPDFContent()`
- Pages created dynamically before rendering to canvas
- Includes legend on first page only

### 5. Modal System
Three modals share common patterns (all use `display: none`/`flex`):
- **Day Modal** (`#day-modal`): Edit single day - status + note
- **Share Modal** (`#share-modal`): Export/share options
- **Close pattern**: Click outside modal or button listener

Modal data binding happens via selected DOM element state (e.g., `.color-option.selected`).

### 6. UI Navigation States
- `currentYear` and `currentMonth` globals track view state
- `scrollToCurrentMonth()` auto-scrolls calendar to current month
- Today's date marked with `.today` class and red border
- Current month card highlighted with `.current-month` class

## Developer Workflows

### Adding a New Day Status Type
1. Add color variable to `:root` (lines 14-22)
2. Add option button in day-modal `color-options` (lines ~670-675)
3. Update `getColorByStatus()` switch statement (lines ~1285-1295)
4. French label appears in legend (lines ~580-600)

### Modifying Shift Cycle
Edit `getDayStatus()` logic (~1250-1270):
- Cycle day determined by: `diffDays % 8` (8-day repeating pattern)
- First 5 values (0-4) = work, last 3 (5-7) = rest
- Change modulo or condition thresholds to adjust pattern

### Common Commands (no build step needed)
- **Testing**: Open `index.html` directly in browser
- **Deployment**: Upload single `index.html` file
- **Debugging**: Browser DevTools console (check localStorage with `JSON.parse(localStorage.getItem('planningPro_2025'))`)

## Integration Points & Dependencies

### External Libraries (loaded via CDN)
- **XLSX** (line ~2240): Excel export - version 0.18.5
- **jsPDF** (line ~2241): PDF generation - version 2.5.1
- **html2canvas** (line ~2242): Canvas rendering - version 1.4.1

### Browser APIs Used
- `localStorage` for persistence
- `Intl.DateTimeFormat` for French locale dates
- `html2canvas` + Canvas API for PDF rendering
- `navigator.clipboard` for copy-to-clipboard (with fallback)
- `window.print()` for printing

## Notable Implementation Details

### Share Feature
- Generates shareable link by Base64 encoding planning data: `btoa(JSON.stringify(shareData))`
- No server required - data embedded in URL
- Query param: `?share=<base64_data>`

### Today Highlight
- Compared as `.toDateString()` (ignores time)
- Marked with `.today` class + red border via CSS
- Updated on calendar regeneration

### Responsive Design
- Mobile breakpoint: `768px` (lines ~450-475)
- Calendar grid: `repeat(auto-fill, minmax(300px, 1fr))`
- Toolbar stacks vertically on mobile

## Code Style Notes
- Variables in camelCase, CSS in kebab-case
- Comments in French
- Date operations use native `Date` object (no external lib)
- Event listeners added via `.addEventListener()` in `setupEventListeners()` function
