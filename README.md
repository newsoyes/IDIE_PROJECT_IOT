# GlobalByte Projects — Project Voting Portal

> A lightweight, responsive project-voting web page for browsing, searching, filtering, sorting, and locally voting on a curated collection of AI, ESP32, Raspberry Pi, Arduino, sensor, IoT, and 3D-print projects.

## Overview

**GlobalByte Projects — Project Voting Portal** is a self-contained front-end web page built with standard **HTML, CSS, and vanilla JavaScript**.

The application presents a collection of project ideas as cards. Visitors can search the project catalog, filter by category, sort by vote count or name, cast a vote, remove their own vote, and open the original project page in a new browser tab.

The current `index.html` source contains **46 project entries** across **7 categories**:

| Category | Projects |
|---|---:|
| AI | 11 |
| ESP32 | 11 |
| Sensors | 7 |
| Raspberry Pi | 6 |
| Arduino | 6 |
| IoT | 4 |
| 3D Print | 1 |
| **Total** | **46** |

The interface is intentionally simple to deploy: there is no build system, package manager, application server, database, or server-side API in the supplied project.

---

## Key Features

### Project Catalog

Each project is represented by structured data containing:

- Numeric project ID
- Project title
- Description
- Category
- Emoji
- Local image path
- External project URL

Projects are rendered dynamically from the `PROJECTS` JavaScript array.

### Search

The search field filters projects in real time.

A search is performed against:

- Project title
- Project description

Matching is case-insensitive.

### Category Filtering

The page provides the following category filters:

- All
- AI
- ESP32
- Raspberry Pi
- Arduino
- Sensors
- IoT
- 3D Print

Filtering is performed entirely in the browser.

### Sorting

Users can choose:

- Default order
- Most votes
- Name

Name sorting uses JavaScript `localeCompare()` with the Thai locale (`th`), which is suitable for the multilingual project names used in the catalog.

### Voting

Each project has a vote button.

When a user votes:

1. The project's vote count increases by one.
2. The project is added to the user's local voted list.
3. The card receives a visual voted state.
4. A toast notification is displayed.
5. The interface re-renders immediately.

The same button can be used to remove the vote.

### Ranking

Projects are ranked dynamically by their current vote count.

The top three projects receive a rank badge when they have at least one vote.

The project currently ranked #1 receives an additional winner-style visual highlight.

### Vote Progress Bar

Each project displays a horizontal bar whose width represents its vote count relative to the highest vote count currently recorded in the browser.

### External Project Links

Every card includes a link that opens the corresponding GlobalByte Shop project page in a new browser tab.

### Responsive Layout

The layout adapts to smaller screens.

On screens up to 640px wide:

- The header becomes more compact.
- Header statistics are hidden.
- The hero section uses reduced spacing.
- The project grid becomes a single column.
- Controls use smaller horizontal padding.

---

# How the Application Works

The complete application flow is client-side:

```text
                ┌─────────────────────┐
                │      Browser        │
                │                     │
                │  index.html         │
                │  HTML + CSS + JS    │
                └─────────┬───────────┘
                          │
             ┌────────────┼─────────────┐
             │            │             │
             ▼            ▼             ▼
        Project Data   User Input   Local Storage
        PROJECTS[]     Search       project_votes
                       Filter        my_votes
                       Sort
             │
             ▼
        Dynamic Render
             │
      ┌──────┴───────┐
      │              │
      ▼              ▼
 Project Cards   External Links
      │              │
      │              └──────► GlobalByte Shop
      │
      └──────► Vote / Unvote
```

There is no request from the voting code to a backend service. Vote state is stored in the browser using the Web Storage API.

---

# Data Model

The main project dataset is declared in JavaScript as:

```js
const PROJECTS = [
  {
    id: 1,
    title: "...",
    desc: "...",
    category: "ESP32",
    emoji: "⌨️",
    img: "img/1.jpg",
    url: "https://..."
  }
];
```

### Project Object Fields

| Field | Type | Purpose |
|---|---|---|
| `id` | Number | Unique project identifier |
| `title` | String | Display name |
| `desc` | String | Short project description |
| `category` | String | Category used by the filter |
| `emoji` | String | Visual project icon |
| `img` | String | Relative image path |
| `url` | String | Original project URL |

---

# Client-Side State

The page maintains three major pieces of application state.

```js
let votes = JSON.parse(
  localStorage.getItem('project_votes') || '{}'
);

let myVotes = JSON.parse(
  localStorage.getItem('my_votes') || '[]'
);

let activeFilter = 'all';
let searchQuery = '';
let sortMode = 'default';
```

## `votes`

An object containing vote totals keyed by project ID.

Example:

```json
{
  "3": 4,
  "6": 2,
  "12": 7
}
```

## `myVotes`

An array containing project IDs voted for by the current browser.

Example:

```json
[3, 6, 12]
```

## `activeFilter`

Stores the currently selected category.

Example:

```js
"AI"
```

## `searchQuery`

Stores the current search text.

Example:

```js
"ESP32"
```

## `sortMode`

Stores the current sorting mode.

Possible values:

```text
default
votes
name
```

---

# Voting Logic

The voting function toggles the user's local vote state.

```js
function vote(id) {
  if (hasVoted(id)) {
    votes[id] = Math.max(0, (votes[id] || 0) - 1);
    myVotes = myVotes.filter(v => v !== id);
  } else {
    votes[id] = (votes[id] || 0) + 1;
    myVotes.push(id);
  }

  saveVotes();
  render();
}
```

### Vote

When the user has not voted for a project:

```text
Vote button
    ↓
Increase vote count
    ↓
Add project ID to myVotes
    ↓
Save to localStorage
    ↓
Re-render page
```

### Unvote

When the user has already voted:

```text
Vote button
    ↓
Decrease vote count
    ↓
Remove project ID from myVotes
    ↓
Save to localStorage
    ↓
Re-render page
```

Vote counts are never allowed to become negative because the implementation uses `Math.max(0, ...)`.

---

# Important Architecture Limitation

## Votes Are Local to Each Browser

The supplied implementation stores voting data in:

```text
localStorage
```

using these keys:

```text
project_votes
my_votes
```

This means the current implementation **does not provide shared voting across multiple users or devices**.

For example:

```text
User A's Browser
project_votes → 20 votes

User B's Browser
project_votes → 3 votes
```

These values are independent.

### What This Means

The current version is suitable for:

- A local demo
- A classroom demonstration
- A prototype
- A single-device voting exercise
- UI/UX testing

It is **not yet suitable for a real public voting system** where all users need to contribute to the same shared vote totals.

A shared voting system would require a backend and persistent server-side storage.

---

# Project Rendering

The `render()` function is responsible for updating the page.

Its workflow is:

```text
Get filtered project list
        ↓
Calculate maximum vote count
        ↓
Update header statistics
        ↓
Generate project cards
        ↓
Calculate project ranking
        ↓
Calculate vote-bar percentage
        ↓
Insert generated HTML into #projectGrid
```

For every project, the renderer calculates:

- Vote count
- Whether the current browser has voted
- Current ranking
- Whether the project is the winner
- Vote progress-bar percentage
- Category CSS class
- Fallback image URL

---

# Image Handling

Project images are primarily loaded from the local `img/` directory.

Example:

```text
img/1.jpg
img/2.jpg
img/3.jpg
...
```

The image element uses an `onerror` handler:

```html
onerror="this.onerror=null; this.src='fallback-image';"
```

When a local project image fails to load, the application substitutes a category-specific image hosted by **Unsplash**.

## Included Image Assets

The archive contains local JPG assets under:

```text
img/
```

The current `index.html` references project images `img/1.jpg` through `img/46.jpg`.

The following referenced images are missing from the supplied archive:

```text
img/31.jpg
img/41.jpg
```

The built-in fallback logic is intended to handle such missing assets.

### Offline Consideration

Because the fallback images use external Unsplash URLs, the fallback itself requires an internet connection.

---

# UI / Design System

The interface uses a dark technology-oriented visual theme.

## Main Visual Direction

- Dark navy/black background
- Cyan primary accent
- Purple secondary accent
- Green success state
- Red removal/error state
- Orange/gold ranking highlight
- Card-based project layout
- Subtle grid background
- Glass-like header effect
- Hover animation
- Gradient typography
- Rounded cards and controls

## Fonts

The project imports two Google Fonts:

```text
Sarabun
Kanit
```

These are loaded from:

```text
fonts.googleapis.com
```

The body primarily uses **Sarabun**, while the logo, project titles, and selected numerical elements use **Kanit**.

---

# Main Page Structure

```text
Header
│
├── Application logo
├── Total project count
├── Number of projects voted for
└── Total vote count
│
Hero Section
│
├── Voting badge
├── Main heading
└── Description
│
Controls
│
├── Search box
├── Category filters
└── Sort selector
│
Project Grid
│
└── Dynamic Project Cards
    ├── Image
    ├── Category
    ├── Rank
    ├── Title
    ├── Description
    ├── Vote button
    ├── Vote progress bar
    ├── Vote count
    └── External project link
```

---

# Project Card Behavior

Each rendered card contains:

### Image Area

- Local project image
- Lazy loading
- Category badge
- Top-three rank badge when applicable

### Body

- Project emoji
- Project title
- Short project description

The description is visually limited to three lines using CSS line clamping.

### Footer

- Vote / unvote button
- Vote progress bar
- Vote count
- External project link

---

# Categories

The current project catalog uses seven categories.

## AI

Projects involving artificial intelligence, computer vision, AI assistants, or AI-powered devices.

**11 projects**

## ESP32

Projects centered around ESP32-family microcontrollers.

**11 projects**

## Sensors

Projects focused on physical sensors and measurements.

**7 projects**

## Raspberry Pi

Projects using Raspberry Pi boards or related Raspberry Pi platforms.

**6 projects**

## Arduino

Projects built around Arduino boards and Arduino-oriented hardware.

**6 projects**

## IoT

Projects focusing on connected devices, cloud dashboards, communication, or Internet-connected monitoring.

**4 projects**

## 3D Print

Projects where 3D printing is a major part of the project.

**1 project**

---

# Project Catalog

The following catalog reflects the project entries currently embedded in `index.html`.

| ID | Project | Category | Image |
|---:|---|---|---|
| 1 | DIY Macropad 9 Keys + OLED RP2350 | ESP32 | `img/1.jpg` |
| 2 | Long-Range Infrared Night Vision Telescope | Sensors | `img/2.jpg` |
| 3 | Claude Desktop Buddy — AI Assistant on ESP32-S3 | AI | `img/3.jpg` |
| 4 | AI People Counter + Home Assistant | AI | `img/4.jpg` |
| 5 | Smart Roller Shades ESP32-S3 | ESP32 | `img/5.jpg` |
| 6 | DIY AI Pin — ESP32-S3 + OpenAI Vision + Voice | AI | `img/6.jpg` |
| 7 | Hands-Free Call Button for ALS Patients | Sensors | `img/7.jpg` |
| 8 | Omnibus 4×8 Power Bank + ESP32 + Inverter | ESP32 | `img/8.jpg` |
| 9 | Pi Slate — Raspberry Pi 5 Handheld Cyberdeck | Raspberry Pi | `img/9.jpg` |
| 10 | M5Stack Pixel Pets — AI Virtual Pet | AI | `img/10.jpg` |
| 11 | AI Computer Vision CCTV Visitor Counter | AI | `img/11.jpg` |
| 12 | ESP32 Air Quality Monitoring System | ESP32 | `img/12.jpg` |
| 13 | DIY PM2.5 Meter | Sensors | `img/13.jpg` |
| 14 | ESP32-S3 Speaking Alarm Clock + OLED + TTS | ESP32 | `img/14.jpg` |
| 15 | Moai 3D-Printed Soap Dispenser | 3D Print | `img/15.jpg` |
| 16 | ESP32 + Blynk Water Level Monitor | IoT | `img/16.jpg` |
| 17 | DIY ESP8266 Lightning Detector | Sensors | `img/17.jpg` |
| 18 | Raspberry Pi Smart Door with Face Recognition | Raspberry Pi | `img/18.jpg` |
| 19 | Automatic Water Pump with Arduino Uno | Arduino | `img/19.jpg` |
| 20 | Raspberry Pi Pico Fortune Ball | Raspberry Pi | `img/20.jpg` |
| 21 | Arduino TPMS Tire Pressure Display | Arduino | `img/21.jpg` |
| 22 | Arduino Nano Automatic Mosquito Trap | Arduino | `img/22.jpg` |
| 23 | Q AI Mobility Companion — Arduino Uno | AI | `img/23.jpg` |
| 24 | Real-Time Arduino UV Meter | Arduino | `img/24.jpg` |
| 25 | ESP32 SMS Sender | IoT | `img/25.jpg` |
| 26 | Raspberry Pi Pico GPS Tracker | Raspberry Pi | `img/26.jpg` |
| 27 | ESP32 GPS Tracker | ESP32 | `img/27.jpg` |
| 28 | AI Person Detection Camera + Stack Light | AI | `img/28.jpg` |
| 29 | Sign Language Translation Robot | AI | `img/29.jpg` |
| 30 | Sensirion Air Pressure Sensor Kit | Sensors | `img/30.jpg` |
| 31 | 5 Fun Raspberry Pi Projects | Raspberry Pi | `img/31.jpg` |
| 32 | Health Band | Sensors | `img/32.jpg` |
| 33 | Open-Source RFID System | IoT | `img/33.jpg` |
| 34 | Brain-Controlled Toy Hack with Arduino | Arduino | `img/34.jpg` |
| 35 | Raspberry Pi Smart Security Camera | Raspberry Pi | `img/35.jpg` |
| 36 | ESP32-CAM Email Notification System | ESP32 | `img/36.jpg` |
| 37 | DIY Smartphone with ESP32 | ESP32 | `img/37.jpg` |
| 38 | Mini Arduino Voice Recorder | Arduino | `img/38.jpg` |
| 39 | AI Posture and Movement Detection Camera | AI | `img/39.jpg` |
| 40 | Arduino IoT Cloud Environment Dashboard | IoT | `img/40.jpg` |
| 41 | SoundSense AR — Directional Audio with Arduino | AI | `img/41.jpg` |
| 42 | OAQ — Open-Source PM2.5 Air Quality Monitor | Sensors | `img/42.jpg` |
| 43 | ESP32 Pocket AI Voice Assistant (小智) | AI | `img/43.jpg` |
| 44 | Gamer Bug Zapper with Kill Counter | ESP32 | `img/44.jpg` |
| 45 | Smart Bug Zapper with ESP32 Kill Counter | ESP32 | `img/45.jpg` |
| 46 | Refrigerator Assistant with ESP32-CAM | ESP32 | `img/46.jpg` |

> Project names above are English descriptions/translations of the names embedded in the source. The source data remains the authoritative wording for the actual page.

---

# External Project Links

Every project includes an external URL pointing to a related page on:

```text
https://globalbyteshop.com/blogs/projects/
```

The application does not fetch project details from those pages. The project title, description, category, emoji, image path, and URL are already embedded in the client-side `PROJECTS` array.

This means the catalog is **static data** from the application's point of view.

---

# Files and Directory Structure

The supplied archive contains:

```text
IDIE_PROJECT_IOT-main/
│
├── index.html
├── project_vote.html
│
└── img/
    ├── 1.jpg
    ├── 2.jpg
    ├── 3.jpg
    ├── ...
    └── 46.jpg
```

## `index.html`

The primary page implementation inspected for this README.

It contains:

- Page markup
- All CSS
- Project dataset
- Voting logic
- Search logic
- Filter logic
- Sorting logic
- Rendering logic
- Local storage persistence

The file is self-contained except for its external Google Fonts and fallback image URLs.

## `project_vote.html`

A second HTML file with substantially the same voting-page implementation.

One important difference in the supplied archive is the number of project records:

```text
index.html        → 46 projects
project_vote.html → 43 projects
```

Therefore, these two files should not be assumed to contain identical catalog data.

## `img/`

Contains local JPEG project images referenced by the page.

---

# Local Development

No package installation is required.

## Option 1 — Open Directly

You can open the page directly in a modern browser:

```text
index.html
```

This is sufficient for the core client-side functionality.

However, using a local HTTP server is preferable for testing static assets in a browser-like deployment environment.

---

## Option 2 — Run a Local HTTP Server

### Python

From the project root:

```bash
python -m http.server 8000
```

Then open:

```text
http://localhost:8000/
```

### Node.js

If Node.js is already installed:

```bash
npx serve .
```

Then open the local URL printed by the tool.

> These commands are generic ways to serve static files; the supplied project does not include a custom server.

---

# No Build Step

The supplied source does not include:

- `package.json`
- `vite.config.*`
- `webpack.config.*`
- `next.config.*`
- `tsconfig.json`

Therefore, there is no project-specific build pipeline in the supplied archive.

The browser executes the HTML, CSS, and JavaScript directly.

---

# Dependencies

## Runtime Dependencies

The core application has no JavaScript package dependencies.

It uses browser APIs and standard web technologies.

### Browser APIs Used

```text
localStorage
DOM API
Event Listeners
Template Literals
Array methods
String methods
```

## External Resources

### Google Fonts

The page imports:

```text
Sarabun
Kanit
```

from Google Fonts.

### Unsplash

Category-based fallback images are loaded from Unsplash when a local project image cannot be loaded.

---

# Main JavaScript Functions

The application is organized around a small set of functions.

## `saveVotes()`

Persists vote state:

```js
localStorage.setItem('project_votes', JSON.stringify(votes));
localStorage.setItem('my_votes', JSON.stringify(myVotes));
```

## `getVoteCount(id)`

Returns the stored vote count for a project.

```js
function getVoteCount(id) {
  return votes[id] || 0;
}
```

## `hasVoted(id)`

Checks whether the current browser has voted for a project.

```js
function hasVoted(id) {
  return myVotes.includes(id);
}
```

## `totalVoteSum()`

Calculates the total number of stored votes.

```js
function totalVoteSum() {
  return Object.values(votes).reduce((a, b) => a + b, 0);
}
```

## `maxVotes()`

Finds the largest current project vote count.

This value is used to scale the vote progress bars.

## `vote(id)`

Toggles vote/unvote state.

## `showToast(msg)`

Displays a temporary status message at the bottom-right of the page.

The toast disappears after approximately 2.5 seconds.

## `getFiltered()`

Applies:

1. Category filter
2. Search query
3. Sort mode

and returns the resulting project list.

## `getRank(id)`

Ranks the project against all projects using current local vote counts.

## `render()`

Rebuilds the project grid and updates statistics.

---

# Event Handling

The page attaches listeners to three main controls.

## Category Filters

The filter bar listens for clicks on `.filter-tab` elements and updates:

```js
activeFilter
```

Then calls:

```js
render()
```

## Search Input

The search field listens for `input` events and updates:

```js
searchQuery
```

Then calls:

```js
render()
```

This creates live search behavior while typing.

## Sort Selector

The sort dropdown listens for `change` events and updates:

```js
sortMode
```

Then calls:

```js
render()
```

---

# Empty Search Result

When filtering/searching produces no matching projects, the page displays an empty state rather than leaving the grid blank.

The intended message tells the user that no project was found and suggests changing the filter or search query.

---

# Responsive Behavior

The stylesheet includes a mobile breakpoint at:

```css
@media (max-width: 640px)
```

At that breakpoint:

- Header spacing is reduced.
- Project grid becomes one column.
- Header statistics are hidden.
- Hero padding is reduced.
- Controls use reduced horizontal padding.

The project grid uses:

```css
grid-template-columns:
  repeat(auto-fill, minmax(300px, 1fr));
```

on larger screens.

---

# Customizing the Project List

Projects can be added or edited directly inside:

```js
const PROJECTS = [
  ...
];
```

Example:

```js
{
  id: 47,
  title: "My New IoT Project",
  desc: "A new project description.",
  category: "IoT",
  emoji: "🌐",
  img: "img/47.jpg",
  url: "https://example.com/project"
}
```

### Requirements

When adding a new project:

1. Use a unique numeric ID.
2. Use one of the supported category names.
3. Place the corresponding image in `img/`.
4. Provide a valid project URL.
5. Ensure the image path matches the file name.

Supported category values are:

```text
AI
ESP32
Raspberry Pi
Arduino
เซ็นเซอร์
IoT
3D Print
```

> The page's display labels for some categories are Thai even though the README uses English terminology.

---

# Adding a New Category

To support an additional category, update at least three parts of the source.

## 1. Add a Filter Button

Example:

```html
<button class="filter-tab" data-cat="Robotics">
  🤖 Robotics
</button>
```

## 2. Add a CSS Category Class

Example:

```css
.cat-robotics {
  color: #38bdf8;
  border-color: #38bdf8;
}
```

## 3. Add a Category Color Mapping

Example:

```js
const CAT_COLORS = {
  ...
  "Robotics": "cat-robotics"
};
```

## 4. Add a Fallback Image

Example:

```js
const FALLBACK_IMGS = {
  ...
  "Robotics": "https://example.com/robotics-fallback.jpg"
};
```

---

# Storage Behavior

The project uses the browser's local storage namespace:

```text
project_votes
my_votes
```

Because no server-side database is present, clearing browser storage will remove the stored voting state.

Possible actions that can affect local state include:

- Clearing site data
- Clearing browser storage
- Using a different browser
- Using a different device
- Using a different browser profile

---

# Deployment

Because the project is static, it can be deployed to a static web host.

Possible deployment approaches include:

- GitHub Pages
- Netlify
- Vercel static hosting
- Cloudflare Pages
- Any standard web server capable of serving HTML/CSS/JS/JPG files

No server-side application is required for the current implementation.

## Required Files for Deployment

At minimum:

```text
index.html
img/
```

Make sure the relative paths remain unchanged:

```text
index.html
img/1.jpg
img/2.jpg
...
```

---

# Production Considerations

The current implementation is a front-end prototype rather than a secure shared voting platform.

For production use, consider adding:

## Shared Vote Database

Replace browser-only local storage with a server-side database.

Possible architecture:

```text
Browser
   ↓
REST API
   ↓
Backend
   ↓
Database
```

## User Authentication

A real voting system may need:

- User accounts
- Login
- Session management
- Identity verification
- Vote-per-user rules
- Duplicate-vote prevention

## Server-Side Validation

Votes should be validated on the server rather than trusting browser state.

## Rate Limiting

A public endpoint should limit automated or excessive vote submissions.

## Audit Logging

For an important group decision, store:

- User
- Project
- Timestamp
- Vote action
- Source/device information as appropriate

## Shared Ranking

When server-side storage is implemented, ranking should be calculated from the same central dataset for every user.

---

# Suggested Production Architecture

A future shared version could use:

```text
┌──────────────────────────┐
│       Frontend           │
│ HTML/CSS/JavaScript      │
└────────────┬─────────────┘
             │ HTTPS
             ▼
┌──────────────────────────┐
│       Voting API         │
│ Authentication + Rules   │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│       Database           │
│ projects / votes / users │
└──────────────────────────┘
```

This would allow all participants to see the same vote totals.

---

# Accessibility Notes

The source already includes some positive accessibility-oriented practices:

- Semantic `<button>` elements for interactive voting/filter controls
- Text alternatives on project images through `alt`
- Standard form controls
- Responsive layout for small screens

Further improvements could include:

- Visible keyboard focus states
- More descriptive link labels
- ARIA labels where icon-only controls are used
- Better color contrast verification
- Screen-reader-friendly live regions for toast messages
- Keyboard navigation testing

---

# Browser Considerations

The application depends on modern browser features such as:

- `localStorage`
- Template literals
- Arrow functions
- `Array.prototype.map`
- `Array.prototype.filter`
- `Array.prototype.sort`
- `Object.values`
- CSS Grid
- CSS custom properties
- `backdrop-filter`
- CSS line clamping

A modern version of Chrome, Edge, Firefox, or Safari is recommended.

---

# Known Source-Level Notes

The following observations are based directly on the supplied archive.

## 1. Two Similar HTML Files

The archive includes:

```text
index.html
project_vote.html
```

They implement very similar voting pages, but their embedded project counts differ.

```text
index.html        → 46 projects
project_vote.html → 43 projects
```

For the catalog described in this README, `index.html` is treated as the main source because it contains the larger current project list in the supplied archive.

## 2. Two Missing Local Images

The current `index.html` references:

```text
img/31.jpg
img/41.jpg
```

but these files are not present in the supplied `img/` directory.

The code provides a category-based fallback mechanism.

## 3. Source Comment vs Actual Catalog Count

A source comment describes image numbering up to `43`, while the actual `PROJECTS` array in `index.html` contains project IDs through `46`.

The implementation itself is what determines the rendered project count.

## 4. Voting Is Not Centralized

There is no server-side vote endpoint in the supplied source.

All vote persistence uses browser-local storage.

## 5. Fallback Images Are Remote

The fallback images are external Unsplash URLs.

A fully offline deployment will therefore not have working category fallback images unless those images are downloaded and stored locally.

---

# Maintenance Guide

## Editing Project Information

Edit:

```js
const PROJECTS = [...]
```

## Editing Category Filters

Edit:

```html
<div class="filter-tabs" id="filterTabs">
```

## Editing Category Colors

Edit:

```js
const CAT_COLORS = {
  ...
};
```

and the corresponding CSS classes.

## Editing Fallback Images

Edit:

```js
const FALLBACK_IMGS = {
  ...
};
```

## Editing Branding

Update:

```html
<div class="logo">
```

and the related CSS variables:

```css
--accent
--accent2
--accent3
--green
--red
--text
```

## Editing Layout

The main grid and responsive behavior are defined in the stylesheet near:

```css
.grid-container
```

and:

```css
@media (max-width: 640px)
```

---

# Recommended Git Workflow

A simple workflow for maintaining the project:

```bash
git clone <repository-url>
cd IDIE_PROJECT_IOT-main
```

Edit the files, test locally, then:

```bash
git add .
git commit -m "Update project voting portal"
git push
```

The project has no package installation step in the supplied archive.

---

# Project Goals

The page is designed to make group project selection easier by putting a large collection of project ideas into one interface.

The intended user flow is:

```text
Browse projects
      ↓
Search or filter
      ↓
Compare ideas
      ↓
Open the original project
      ↓
Vote for a preferred project
      ↓
Review rankings
```

This makes the page useful for:

- Student project selection
- Team project planning
- Classroom voting
- Hackathon idea selection
- Personal project curation
- Technology showcase catalogs

---

# Current Status

## Implemented

- [x] Responsive project catalog
- [x] 46 project records in `index.html`
- [x] 7 project categories
- [x] Search
- [x] Category filtering
- [x] Sorting
- [x] Vote / unvote
- [x] Local vote persistence
- [x] Project ranking
- [x] Top-three badges
- [x] Winner highlight
- [x] Vote progress bars
- [x] Toast notifications
- [x] Lazy-loaded project images
- [x] Image fallbacks
- [x] External project links
- [x] Mobile responsive layout

## Not Included

- [ ] Backend
- [ ] Shared multi-user voting
- [ ] Database
- [ ] User authentication
- [ ] Admin panel
- [ ] Server-side vote validation
- [ ] Vote history
- [ ] Cloud synchronization
- [ ] API for project management

---

# Future Improvements

A future version could evolve into a full project-management and voting platform.

### Shared Voting

Store votes centrally so all participants see the same totals.

### Authentication

Allow each participant to sign in and associate votes with a user account.

### Admin Dashboard

Provide an interface for:

- Adding projects
- Editing projects
- Removing projects
- Managing categories
- Reviewing voting results

### Database-backed Projects

Move the `PROJECTS` array into a database so project content can be updated without editing HTML.

### Real-Time Results

Use WebSockets or another real-time mechanism so rankings update immediately for everyone.

### Voting Rules

Possible rules include:

- One vote per user
- Maximum number of selections
- Voting deadlines
- Weighted votes
- Anonymous voting
- Restricted project categories

---

# License

No software license is specified in the supplied archive.

Until a license is added, assume normal copyright restrictions apply to the project source and bundled assets.

---

# Credits

The page branding in the source is:

```text
GlobalByte Projects
```

External project references point to GlobalByte Shop project pages.

---

# Summary

**GlobalByte Projects — Project Voting Portal** is a lightweight static voting interface built without a framework.

Its main strengths are:

- Very simple deployment
- No backend required for demonstration
- Fast client-side filtering
- Responsive project cards
- Local vote persistence
- Easy project-data customization
- Minimal technical overhead

Its primary limitation is that voting is stored only in the user's browser. For real multi-user voting, the application should be extended with authentication, a backend API, and centralized persistent storage.

---

## Quick Start

```bash
# Option 1: open index.html directly

# Option 2: serve the project locally
python -m http.server 8000
```

Then visit:

```text
http://localhost:8000/
```

That's all that is required to run the supplied front-end version.
