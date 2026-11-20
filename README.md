# NoteFlow 🚀
> **Capture Your Thoughts. Organize Your Life.**

NoteFlow is a modern personal notes management web application inspired by Notion, Linear, and Craft. It features rich-text document editing powered by TipTap, real-time autosave indicators, tags, categories, pinning, archiving, soft-delete trash recovery, responsive dashboards, interactive analytics with Recharts, and dark/light themes.

---

## 📸 Core Features

- **Full Authentication System:**
  - Secure email registration with duplicate prevention.
  - Password hashing via `bcryptjs` (10 rounds).
  - JWT token-based authentication with persistent sessions.
  - Automatic provisioning of 5 default categories (*Personal, Work, Study, Ideas, Important*) and an interactive welcome note upon registration.
  - Complete user data isolation at database and middleware levels.

- **TipTap Rich Text Editor:**
  - Headings (H1, H2, H3), Bold, Italic, Underline.
  - Bulleted and numbered lists, blockquotes, code blocks.
  - Text alignment (left, center, right), undo, and redo.
  - Debounced autosave with live status indicators (`Saving...`, `Saved`, `Save Failed`).

- **Flexible Notes Organization:**
  - Note cards with custom gentle color tints.
  - Pin important notes to the top of your dashboard.
  - Archive completed notes without deleting them.
  - Soft-delete to Trash with one-click restoration or permanent destruction.
  - Empty Trash with safety confirmation modal.
  - Duplicate notes in one click.

- **Categories & Tags:**
  - Create, customize, and color-code categories.
  - Multi-tag selector with inline creation (`#tag`).
  - Filter notes instantly by category or tag.

- **Search & Filtering:**
  - Real-time global search across note titles, contents, and tags (`Ctrl/Cmd + K`).
  - Sort by *Recently Updated*, *Newest First*, *Oldest First*, or *Alphabetical (A–Z)*.
  - Grid View and List View toggle with persisted preference.

- **Interactive Dashboard:**
  - Live statistics: Total active notes, pinned notes, archived count, and categories.
  - 7-Day activity timeline with smooth gradient area charts via Recharts.
  - Category breakdown and quick note creator.

- **Personalization & Settings:**
  - Profile customization (name, email).
  - Secure password changes.
  - Appearance preferences: Light mode, Dark mode, System sync, and accent color palette.

---

## 🛠 Technology Stack

### Frontend
- **React.js (v18)** with **Vite** (Plain JavaScript ES6+, no TypeScript)
- **Tailwind CSS** with custom Notion/Linear-inspired palette & Dark Mode
- **React Router DOM (v6)** for client-side routing & route guards
- **TipTap Editor** (`@tiptap/react`, `@tiptap/starter-kit`, `@tiptap/extension-underline`, `@tiptap/extension-text-align`, `@tiptap/extension-placeholder`)
- **Lucide React** for clean iconography
- **Recharts** for analytics and activity graphs
- **React Hot Toast** for notifications

### Backend
- **Node.js** & **Express.js** (ES Modules)
- **Prisma ORM** with **SQLite**
- **JSON Web Tokens (jsonwebtoken)** & **bcryptjs**
- **Zod** schema validation middleware
- **Helmet**, **CORS**, and **express-rate-limit** for API security
- **Morgan** for HTTP request logging

---

## 🗄 Database Architecture

```prisma
model User {
  id              String     @id @default(uuid())
  name            String
  email           String     @unique
  password        String
  themePreference String     @default("light")
  accentColor     String     @default("#6366F1")
  createdAt       DateTime   @default(now())
  updatedAt       DateTime   @updatedAt
  notes           Note[]
  categories      Category[]
  tags            Tag[]
}

model Note {
  id         String     @id @default(uuid())
  title      String
  content    String     @default("")
  color      String     @default("#ffffff")
  isPinned   Boolean    @default(false)
  isArchived Boolean    @default(false)
  isDeleted  Boolean    @default(false)
  userId     String
  categoryId String?
  createdAt  DateTime   @default(now())
  updatedAt  DateTime   @updatedAt
  deletedAt  DateTime?
  user       User       @relation(fields: [userId], references: [id], onDelete: Cascade)
  category   Category?  @relation(fields: [categoryId], references: [id], onDelete: SetNull)
  tags       NoteTag[]
}

model Category {
  id        String   @id @default(uuid())
  name      String
  color     String   @default("#6366F1")
  userId    String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  notes     Note[]
  @@unique([userId, name])
}

model Tag {
  id        String    @id @default(uuid())
  name      String
  color     String    @default("#8B5CF6")
  userId    String
  createdAt DateTime  @default(now())
  user      User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  notes     NoteTag[]
  @@unique([userId, name])
}

model NoteTag {
  noteId    String
  tagId     String
  createdAt DateTime @default(now())
  note      Note     @relation(fields: [noteId], references: [id], onDelete: Cascade)
  tag       Tag      @relation(fields: [tagId], references: [id], onDelete: Cascade)
  @@id([noteId, tagId])
}
```

---

## 🔌 API Endpoints Reference

| Method | Endpoint | Description | Auth Required |
|---|---|---|---|
| `POST` | `/api/auth/register` | Register new user & seed categories | No |
| `POST` | `/api/auth/login` | Login and obtain JWT token | No |
| `GET` | `/api/auth/me` | Fetch authenticated user data | Yes |
| `POST` | `/api/auth/logout` | Terminate session | Yes |
| `GET` | `/api/notes` | List notes (search, category, tag, sort) | Yes |
| `GET` | `/api/notes/:id` | Get single note with tags & category | Yes |
| `POST` | `/api/notes` | Create a note | Yes |
| `PUT` | `/api/notes/:id` | Update note fields & tags | Yes |
| `DELETE` | `/api/notes/:id` | Move note to trash (soft delete) | Yes |
| `PATCH` | `/api/notes/:id/pin` | Toggle or set pin status | Yes |
| `PATCH` | `/api/notes/:id/archive` | Toggle or set archive status | Yes |
| `PATCH` | `/api/notes/:id/restore` | Restore note from trash/archive | Yes |
| `POST` | `/api/notes/:id/duplicate`| Clone note with `(Copy)` suffix | Yes |
| `DELETE` | `/api/notes/:id/permanent`| Permanently delete note | Yes |
| `DELETE` | `/api/notes/trash/empty` | Empty all notes in trash bin | Yes |
| `GET` | `/api/categories` | List categories with note counts | Yes |
| `POST` | `/api/categories` | Create category | Yes |
| `PUT` | `/api/categories/:id` | Rename / update category color | Yes |
| `DELETE` | `/api/categories/:id` | Delete category (notes preserved) | Yes |
| `GET` | `/api/tags` | List all tags with note counts | Yes |
| `POST` | `/api/tags` | Create a tag | Yes |
| `DELETE` | `/api/tags/:id` | Delete tag | Yes |
| `GET` | `/api/dashboard/stats` | Real-time counts & 7-day activity | Yes |
| `GET` | `/api/dashboard/recent-notes`| Fetch recently modified notes | Yes |
| `PUT` | `/api/users/profile` | Update profile information | Yes |
| `PUT` | `/api/users/password` | Update account password | Yes |
| `DELETE` | `/api/users/account` | Permanently delete account & notes | Yes |

---

## ⌨️ Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl + K` / `Cmd + K` | Open global search modal |
| `Ctrl + N` / `Cmd + N` | Open new note creator |
| `Ctrl + S` / `Cmd + S` | Save current note changes |
| `Escape` | Close active modal dialog |

---

## 🚀 Getting Started

### Prerequisites
- Node.js (v18.0.0 or higher)
- npm (v9.0.0 or higher)

### 1. Clone & Install Dependencies

```bash
# Clone the repository
git clone <repository-url>
cd web

# Install server dependencies
cd server
npm install

# Install client dependencies
cd ../client
npm install
```

### 2. Configure Environment Variables

In `server/.env`:
```env
PORT=5000
NODE_ENV=development
DATABASE_URL="file:./dev.db"
JWT_SECRET="noteflow_super_secret_jwt_key_2026_secure_production"
JWT_EXPIRES_IN="7d"
CORS_ORIGIN="http://localhost:5173"
```

### 3. Initialize Database & Run Migrations

```bash
cd server
npx prisma generate
npx prisma db push
```

### 4. Run Automated Backend Integration Tests

```bash
cd server
npm run test:api
```

### 5. Start Frontend and Backend

**Backend Server (Port 5000):**
```bash
cd server
npm run dev
# Server runs on http://localhost:5000
```

**Frontend Application (Port 5173):**
```bash
cd client
npm run dev
# Frontend runs on http://localhost:5173
```

### 6. Build for Production

```bash
cd client
npm run build
```

---

## 🛡 Security & Reliability Highlights
1. **Zero Plaintext Passwords:** Strict bcrypt hashing with 10 salt rounds.
2. **User Data Isolation:** Every query (`findMany`, `findFirst`, `update`, `delete`) filters by `userId: req.user.id`.
3. **Input Validation:** Every payload is validated against strict Zod schemas before touching controllers.
4. **Resilient Autosave:** Keystrokes in TipTap are debounced (800ms) and reflect status badges (*Saving...*, *Saved*, *Save Failed*).
5. **Soft Deletions:** Notes are safeguarded with `isDeleted` and `deletedAt` timestamps, preventing accidental permanent loss.
