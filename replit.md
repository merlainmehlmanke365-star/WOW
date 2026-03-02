# 生活记录 — Life Journal App

A personal life management web app built with React, Express, and PostgreSQL.

## Features

- **昼夜感知 Day/Night Mode**: Home page hero auto-switches based on local time. Day (06:00–18:00) shows 时光报社; night (18:00–06:00) shows 微光星空. Manual toggle (☀️/🌙 buttons in section label). `getAutoMode()` checks hourly via `setInterval`.
- **微光星空 Star Galaxy**: Night-mode hero — all `small_joy` milestones rendered as twinkling stars on deep navy canvas. Deterministic positions via `hashNum()`. Click a star → floating bubble (React portal, fixed positioning) shows title + date. Empty-state prompt if no small joys recorded.
- **时光报社 Daily Echo**: AI-powered daily newspaper — picks one past milestone each day, uses GPT-5.2 to generate a New Yorker-style Chinese title, curated quote on sticky note. Auto-pops as a full-screen modal on first app open each day (localStorage key: `echo_shown_date`). Reader comment section below newspaper, synced with archive.
- **报纸档案馆 (Archive)**: Long-strip index view with IndexRow components. Category badges: gold "◆ 里程碑" for milestone, "▲ 成就" for achievement. Click to read full paper in modal. Reader comment block (parchment style, Lora italic, signature).
- **成就博物院 (Milestone Museum)**: Add/view/manage achievements with pride score (1-5 stars), companions field, optional cover photo. Gold 🔒✉ indicator shows when a Time Capsule message is hidden inside.
- **时光锦囊 (Time Capsule)**: When adding a milestone/achievement, optionally write a private message "to your future tired self." Shown as a gold sealed envelope on the card. In the Todos page, click "我需要动力" or auto-triggers when >5 tasks are pending — an animated envelope opens revealing the message with New Yorker EN translation.
- **灵感便利贴 — 软木板 Corkboard**: Cork-texture board with draggable Post-it notes (4 pastel colors, ±5° tilt, masking tape strip, folded corner). Lightning quick-add (Enter key), drag to reposition (in-memory). Hover reveals: edit, delete, "转化为里程碑". Enhanced hover: scale(1.045) + big shadow + rotation reduces toward 0 (peel-off-wall effect). 📷 Polaroid mode: camera button → upload image → note displays as white Polaroid frame with caption. 🧵 红线连接 (Red Thread): toggle connection mode (Link2 icon) → click note A (pulsing ring) → click note B → SVG bezier thread drawn with gravity sag + pin circles + textured wool overlay; × delete button appears only while in connection mode; threads are decorative (pointer-events: none) outside connection mode. Schema: inspirations.imageUrl + inspiration_connections table (fromId, toId). 🌫️ 灵感褪色 (Fading Memory): localStorage tracks `inspr-active-{id}` per note; notes >7/14/21 days inactive get progressive CSS `saturate+sepia` fade + growing fold-corner (16→42px); clicking/dragging any note calls `setLocalLastActive()` + `revivedId` state triggers 1.3s `saturate(160%) brightness(1.18)` revival flash. 🕯️ 深夜台灯 (Desk Lamp): `isNight` (18:00–06:00) adds dark overlay + `radial-gradient` warm spotlight following cursor `lampPos` (viewport-relative, pointerEvents: none); board darkens to 50% at night; lamp hint shown in header. 🫙 漂流瓶 (Drift Bottle): FlaskConical button replaces blindbox; `driftCandidates` = notes where `createdAt < 7 days ago`; 3-phase dialog: "shaking" (🫙 bounce) → "opening" (cork flies up) → "revealed" (post-it floats in with days-ago label); DriftBottleDialog component with dark parchment background.
- **todo小森林 + 番茄钟小森林 (Pomodoro Grove)**: Todo list with a live forest at the bottom of the page. Each todo has a "专注" button that starts a 25-minute Pomodoro timer. Timer shows a circular ring countdown + a growing tree (Ginkgo for medium, Oak for high, Fruit tree for low priority). Completing a Pomodoro: bell sound + confetti + "你用25分钟的专注，为你的世界添了一抹绿" toast + permanent tree in the DB. Abandoning: withered/dead tree planted instead. Forest canvas (210px) renders all planted trees with hash-stable positions and sizes. Night mode (18:00–06:00): dark sky + firefly animations. Winter (Dec/Jan/Feb): falling snowflakes. Tree schema: `forest_trees` table (todoId, treeType, isWithered). Routes: GET/POST/DELETE /api/forest-trees. `canvas-confetti` celebration on todo completion too.
- **全览 (Unified View)**: View all content in one place with type-based filtering and stats

## Tech Stack

- **Frontend**: React + TypeScript, TanStack Query, wouter, shadcn/ui, Tailwind CSS, framer-motion
- **Backend**: Express.js, Drizzle ORM, PostgreSQL, multer (file uploads)
- **AI**: OpenAI via Replit AI Integrations (gpt-5.2) for New Yorker-style title generation
- **Design**: Lora (serif) + Space Grotesk (mono) + Architects Daughter (handwriting) fonts, warm apricot light mode / cool blue dark mode

## Architecture

- `shared/schema.ts` — Drizzle schemas for milestones (with prideScore, companions, imageUrl), inspirations, todos
- `server/storage.ts` — DatabaseStorage class with full CRUD operations
- `server/routes.ts` — REST API endpoints + /api/upload (multer) + /api/daily-echo (AI)
- `server/openai.ts` — OpenAI client factory using Replit AI Integrations env vars
- `server/index.ts` — Express app + static /uploads serving + seed data on startup
- `client/src/App.tsx` — Root app with SidebarProvider + routing
- `client/src/components/app-sidebar.tsx` — Navigation sidebar with theme toggle
- `client/src/pages/home.tsx` — Unified overview + DailyEcho newspaper component
- `client/src/pages/milestones.tsx` — Milestone Museum with ImageUploader, StarPicker, animated expand/collapse
- `client/src/pages/inspirations.tsx` — Inspiration grid with tag filtering
- `client/src/pages/todos.tsx` — Todo list with confetti on completion
- `uploads/` — User-uploaded images (served at /uploads/*)

## API Endpoints

- `GET /api/daily-echo` — Today's milestone with AI-generated title and curated quote (cached per day)
- `POST /api/upload` — Image upload (multipart/form-data), returns { url }
- `GET/POST /api/milestones` — List and create milestones
- `PATCH/DELETE /api/milestones/:id` — Update and delete
- `GET/POST /api/inspirations` — List and create inspirations
- `PATCH/DELETE /api/inspirations/:id` — Update and delete
- `GET/POST /api/todos` — List and create todos
- `PATCH/DELETE /api/todos/:id` — Update and delete

## Design Notes

- Light mode: warm apricot primary hsl(38,88%,75%); Dark mode: bright blue primary hsl(204,88%,53%)
- Milestone cards: hover lift (-translate-y-1 + shadow), 5-star = amber border + title
- Daily Echo: inline CSS for newspaper aesthetic (cream bg, double-rule, drop cap, sticky note)
- Sticky note: Architects Daughter font, rotate(-5deg), color cycles per issue number
- DO NOT add hover:bg-* on Buttons/Badges — use hover-elevate utility from index.css instead
