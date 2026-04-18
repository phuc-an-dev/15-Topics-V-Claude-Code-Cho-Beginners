# Topic 2: Best Claude Skills Cho React + TypeScript Developer

> **Nguồn**: `anthropics/skills`, `mattpocock/skills` (Total TypeScript), `Jeffallan/claude-skills`, Vercel Engineering, Callstack, Reddit r/ClaudeAI

---

## 🎯 Starter Pack cho React + TypeScript Developer

### Must-Have (Top 5 - install ngay)

#### 1. **Anthropic Frontend Design** 🔥 (65k ⭐)
**Repo**: https://github.com/anthropics/skills (path: `skills/frontend-design/`)

- **Skill số 1** cho frontend work
- Thoát khỏi "visual signature" của AI-generated UI
- Push Claude tạo designs **bold, distinctive** thay vì safe defaults
- Includes: CSS-only animations, Motion library for React, spatial composition, gradient meshes, noise textures

**Install**:
```bash
git clone https://github.com/anthropics/skills.git
cp -r skills/skills/frontend-design ~/.claude/skills/
```

#### 2. **Vercel React Best Practices** 🔥
- **57 performance rules** across 8 categories, prioritized by impact
- Built by Vercel Engineering team
- Covers: bundle size, render optimization, waterfall elimination, Suspense boundaries

**Install**:
```bash
npx claude-code-templates@latest --skill development/react-best-practices
```

**Key rules**:
- `async-defer-await`: Move await vào branches actually used
- `async-parallel`: `Promise.all()` cho independent operations
- `async-suspense-boundaries`: Stream content với Suspense thay vì wait hết data

#### 3. **mattpocock/skills** (Total TypeScript) ⭐
**Repo**: https://github.com/mattpocock/skills

**17 dev workflow skills** từ Matt Pocock (creator Total TypeScript):
- `tdd`: Stricter TDD, closer to real PR review standards
- `refactoring-plan`: Step-by-step plan trước khi touch code
- `write-a-prd`: Product requirement documents
- `request-refactor-plan`: Structured refactoring approach
- `git-guardrails-claude-code`: Git safety

**Perspective**: Strict type discipline, clean interface boundaries, honest refactoring.

**Install individually**:
```bash
npx skills@latest add mattpocock/skills/tdd
npx skills@latest add mattpocock/skills/refactoring-plan
npx skills@latest add mattpocock/skills/write-a-prd
```

#### 4. **mastering-typescript-skill** (SpillwaveSolutions)
**Repo**: https://github.com/SpillwaveSolutions/mastering-typescript-skill

TypeScript 5.9+ enterprise-grade guidance:
- Type System Mastery: Generics, mapped types, conditional types, `satisfies` operator
- Enterprise Patterns: Error handling, Zod validation, project architecture
- React Integration: Type-safe components, hooks, state management (Zustand, Redux Toolkit)
- NestJS Development: DTOs, RBAC, authentication
- Modern Toolchain: Vite 7, pnpm, ESLint 9 flat config, Vitest

#### 5. **spartan-ai-toolkit** (Quality Gates) 🌟
**Repo**: https://github.com/spartan-stratos/spartan-ai-toolkit

**Key feature**: Chạy **typecheck → lint → test → review** in sequence. Won't move to next step nếu previous fail.

> "Without quality gates, Claude Code will write code, notice failing test, patch the test to pass, and present as done. With this toolkit, that path doesn't exist." — welcomedeveloper blog

**React stack profile** available - gives Claude React + TypeScript conventions thay vì generic JS.

---

## 🎨 UI/UX Focused Skills

### 6. **Web Design Guidelines**
- Accessibility compliance (WCAG 2.1/2.2)
- Contrast ratios, keyboard nav, screen reader support
- Pair với Frontend Design skill (complementary, không conflict)

### 7. **Nothing Design Language** (1.2k ⭐)
**Repo**: https://github.com/nothing-design-skill
- Generate UI in Nothing design language
- Monochrome, typographic, industrial aesthetic

### 8. **GSAP Skills** 🔥 (1.5k ⭐)
**Repo**: Official GSAP skills
- Correctly use GreenSock Animation Platform
- Best practices, animation patterns, plugin usage
- Cho React: Integration với useGSAP hook

### 9. **Remotion Agent Skill**
- Translate natural language → Remotion components (React-based video)
- Workflow: describe → Claude generates → preview in Remotion Studio → render MP4

---

## 🔧 TypeScript-Specific Deep Dive

### 10. **typescript-pro Subagent** (VoltAgent)
**Repo**: https://github.com/VoltAgent/awesome-claude-code-subagents

Subagent chuyên TypeScript 5.0+:
- Advanced type system features
- Full-stack type safety (tRPC, OpenAPI → TS, GraphQL codegen)
- Build tooling: Vite, Turbopack, esbuild
- Module augmentation, ambient declarations
- JS interop, migration approaches

**Collaborate với**:
- `react-developer` subagent - component types
- `api-designer` - contract sharing
- `fullstack-developer` - type sharing across stack

---

## 🏗️ Project Structure Recommendations

### Feature Slicing (từ Medium article của Principal Engineer)
```
src/
├── features/
│   └── <feature-name>/
│       ├── screens/         # Screen/page components
│       ├── components/      # Feature-specific components
│       ├── hooks/          # Custom hooks
│       ├── types/          # TypeScript interfaces
│       ├── services/       # API calls (TanStack Query)
│       ├── __tests__/      # Colocated tests
│       └── index.ts        # Barrel export
├── components/
│   ├── ui/                # shadcn/ui components
│   ├── common/            # Reusable
│   └── forms/             # Form components
├── hooks/                 # Global hooks
├── context/               # React Context providers
├── types/                 # Global types
└── utils/                 # Utilities
```

### Path Aliases (tsconfig.json)
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@/components/*": ["src/components/*"],
      "@/hooks/*": ["src/hooks/*"],
      "@/types/*": ["src/types/*"]
    }
  }
}
```

---

## 💡 CLAUDE.md Template cho React + TS Project

```markdown
# Project: [Tên]

## Stack
- React 18+ with TypeScript 5.x
- Vite for build (not CRA)
- TanStack Query for server state
- Zustand for client state
- shadcn/ui + Tailwind CSS
- Vitest for testing

## Code Style
- **NO** use of `any` - use `unknown` or proper types
- Prefer `type` for unions, `interface` for object shapes extendable
- Use `satisfies` operator for type validation without widening
- Destructure imports
- Colocate tests with components (`Component.test.tsx`)

## Architecture
- Feature-based folders (src/features/<n>/)
- Every API call goes through custom hook + TanStack Query
- Every screen wrapped in ErrorBoundary
- Use Suspense boundaries to stream, don't block

## Commands
- `npm run dev`: Start Vite dev server
- `npm run typecheck`: `tsc --noEmit`
- `npm run lint`: ESLint 9 flat config
- `npm test`: Vitest
- `npm run build`: Production build

## IMPORTANT Rules
- Run `tsc --noEmit` AFTER every .tsx file write
- NEVER import from barrel files in hot paths (use deep imports)
- Prefer React Server Components when possible (Next.js)
- Use React.memo sparingly - only after profiling
```

---

## 🪝 Useful Hooks cho Frontend Dev

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "command": "echo 'Remember: Run tsc --noEmit after writing .tsx files'"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash(npm install*)",
        "command": "echo 'Verify package.json for unexpected dependency additions'"
      }
    ],
    "Stop": [
      {
        "command": "terminal-notifier -message 'Claude finished the task' -title 'Claude Code'"
      }
    ]
  }
}
```

**Game-changer**: Stop hook với desktop notification - bạn có thể làm việc khác, nhận notification khi Claude xong.

---

## 🎯 Recommended Install Order (theo tuần)

### Week 1 - Foundations
1. obra/superpowers (TDD + debugging + collaboration)
2. frontend-design (Anthropic)
3. spartan-ai-toolkit (quality gates)

### Week 2 - Stack-specific
4. mattpocock/skills (TypeScript discipline)
5. Vercel react-best-practices (performance)
6. mastering-typescript-skill (patterns)

### Week 3 - UI/UX polish
7. web-design-guidelines (accessibility)
8. GSAP skills nếu dùng animations

### Week 4 - Advanced
9. typescript-pro subagent
10. Custom project-specific skills

---

## 📋 Example Prompts Tận Dụng Skills

### "Build a component" - skills auto-combine
```
Build a UserProfile card component với avatar, name, role, bio.
Must be accessible, responsive, animated on hover.
Use TypeScript with strict types.
```
→ Triggers: frontend-design + web-design-guidelines + mastering-typescript-skill + TDD

### "Refactor this code"
```
Refactor this 200-line component into smaller pieces.
Follow feature slicing pattern.
Maintain strict TypeScript types.
```
→ Triggers: refactoring-plan + typescript-pro + software-architecture

### "Fix performance"
```
This list rendering is slow with 500+ items.
Profile first, then fix root cause.
```
→ Triggers: Vercel react-best-practices + systematic-debugging

---

## 🚫 Common Pitfalls

### 1. **Over-using React.memo / useMemo**
Vercel skill cảnh báo: Dev thường jump vào memoization trong khi root cause là waterfall API calls hoặc barrel imports.

### 2. **Using `any` cho "just make it work"**
mattpocock skill enforces strict discipline - không có `any`.

### 3. **Testing implementation details**
TDD skills encourage testing behavior, không phải internal state.

### 4. **Ignoring Error Boundaries**
Every screen cần wrap trong ErrorBoundary - không để bug crash entire app.

### 5. **Barrel imports trong hot paths**
```tsx
❌ import { Icon1, Icon2 } from '@/components' // loads ALL
✅ import { Icon1 } from '@/components/Icon1' // tree-shakeable
```

---

## 📚 Key Repos Summary

| Repo | Stars | Focus |
|------|-------|-------|
| [anthropics/skills](https://github.com/anthropics/skills) | 65k | Official frontend-design |
| [Vercel React Best Practices](https://www.aitmpl.com/component/skill/react-best-practices) | - | 57 perf rules |
| [mattpocock/skills](https://github.com/mattpocock/skills) | - | TypeScript discipline |
| [SpillwaveSolutions/mastering-typescript-skill](https://github.com/SpillwaveSolutions/mastering-typescript-skill) | - | TS 5.9+ enterprise |
| [Jeffallan/claude-skills](https://github.com/Jeffallan/claude-skills) | 1.5k | 66 full-stack skills |
| [spartan-ai-toolkit](https://github.com/spartan-stratos/spartan-ai-toolkit) | - | Quality gates |
| [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) | - | Specialized subagents |

---

## 📚 Sources
- Snyk "Top 8 Claude Skills for UI/UX Engineers": https://snyk.io/articles/top-claude-skills-ui-ux-engineers/
- "The 10 Claude Code Skills I Actually Use": https://www.welcomedeveloper.com/posts/the-10-claude-code-skills/
- Cars24 "Claude Code for React & React Native": Medium article by Ankit Bhalla
- unicodeveloper "10 Must-Have Skills for Claude": Medium
