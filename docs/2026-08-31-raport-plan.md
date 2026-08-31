# Raport wykonanej roboty — plan wdrożenia

> **Dla agenta:** wykonuj zadanie po zadaniu. Kroki mają pola wyboru (`- [ ]`).
> Specyfikacja: `docs/2026-08-31-raport-spec.md`.

**Cel:** komenda `/tms:raport` podsumowuje robotę domkniętą w podanym okresie przez
wskazaną osobę i — na prośbę — dopisuje ją do dziennika w module dokumentów TMS.

**Podejście:** trzy osobne kawałki, każdy sam w sobie działający. Dwa PR-y po
stronie TMS (filtr dat w wyszukiwaniu zadań; wejście do dokumentów dla klucza
osobistego) i jedno wydanie wtyczki (0.30.0) z nową instrukcją.

**Stos:** TMS — Next.js 15 (App Router), TypeScript, Postgres przez `db.query`,
zod na wejściu. Wtyczka — sam Markdown, bez kodu.

**Czym się weryfikuje:** TMS **nie ma testów jednostkowych** (jedyny zestaw to
Playwright e2e, `pnpm test:e2e`). Nie ma więc sensu udawać TDD. Bramką każdego
zadania jest `pnpm typecheck`, `pnpm lint --quiet` i smoke curlem po lokalnym
API z prawdziwym kluczem. **Nie odpalaj `pnpm build`, gdy chodzi `pnpm dev`** —
psuje `.next`.

**Zasady gałęzi i commitów:** gałęzie z prefiksem `piotr/`, każda zmiana przez PR,
commity bez stopki współautorstwa. Przed KAŻDYM commitem sprawdź gałąź:
`git branch --show-current`. W repozytorium TMS **push i deploy robi człowiek** —
agent commituje i mówi, co wchodzi.

---

## Struktura plików

**TMS, PR 1 — filtr dat (gałąź `piotr/raport-zakresy-dat`)**

- Modyfikacja: `src/types/tasks.types.ts` — dwa pola w `TaskListFilters`.
- Modyfikacja: `src/services/tasks/queries.ts:1050-1056` — warunek WHERE obok `dueDateFrom`.
- Modyfikacja: `src/services/integrations/task-actions.ts` — przepuszczenie filtra
  i trzy pola w widoku raportowym.
- Modyfikacja: `src/app/api/v1/integrations/tasks/route.ts` — parametry zapytania.

**TMS, PR 2 — dokumenty (gałąź `piotr/raport-dokumenty`)**

- Nowy: `src/services/integrations/document-actions.ts` — jedyne miejsce z regułami
  dostępu do dokumentów dla klucza; wzorowane na `task-actions.ts`.
- Nowy: `src/app/api/v1/integrations/documents/route.ts` — lista i zakładanie.
- Nowy: `src/app/api/v1/integrations/documents/[id]/route.ts` — odczyt i zapis.

**Wtyczka (gałąź `piotr/raport-dnia`, już istnieje ze specyfikacją)**

- Nowy: `skills/raport/SKILL.md` — instrukcja raportu.
- Modyfikacja: `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json` — 0.30.0.
- Modyfikacja: `skills/ustawienia/SKILL.md` — numer wydania w treści.
- Modyfikacja: `README.md` — sekcja o raporcie.

---

# CZĘŚĆ I — TMS, PR 1: filtr „wyszło z rąk"

### Zadanie 1: Dwa pola filtra w typach

**Pliki:** modyfikacja `src/types/tasks.types.ts` (w `TaskListFilters`, obok `dueDateFrom`/`dueDateTo`)

- [ ] **Krok 1: Gałąź**

```bash
cd /c/Users/jfinvesting/Desktop/Projekty-AI/TMS
git checkout main && git pull --ff-only && git checkout -b piotr/raport-zakresy-dat
git branch --show-current
```
Oczekiwane: `piotr/raport-zakresy-dat`

- [ ] **Krok 2: Dopisz pola pod `dueDateTo`**

```ts
  dueDateFrom?: string | null
  dueDateTo?: string | null
  // Zakres po momencie, w którym robota WYSZŁA Z RĄK wykonawcy: oddanie do
  // weryfikacji, a przy zadaniach własnych (bez weryfikacji) — domknięcie.
  // Karmi raport dnia; UI tego nie używa. Znacznik ISO z czasem, nie sama data.
  doneFrom?: string | null
  doneTo?: string | null
```

- [ ] **Krok 3: Sprawdź typy**

Uruchom: `pnpm typecheck`
Oczekiwane: bez błędów (same pola opcjonalne, nic się nie psuje).

- [ ] **Krok 4: Commit**

```bash
git add src/types/tasks.types.ts
git commit -m "feat(tasks): filtr zakresu po momencie oddania roboty (typy)"
```

---

### Zadanie 2: Warunek WHERE w listTasks

**Pliki:** modyfikacja `src/services/tasks/queries.ts` (zaraz po bloku `dueDateTo`, ~linia 1056)

- [ ] **Krok 1: Dopisz warunki**

```ts
  if (filters.dueDateTo) {
    where.push(`t.due_date <= ${addParam(filters.dueDateTo)}`)
  }
  // UWAGA na kolejność w COALESCE — jest ODWROTNA niż w sortowaniu „Wykonane"
  // (tam `COALESCE(t.completed_at, t.last_submitted_at)`, bo kolumna pokazuje
  // moment zatwierdzenia). Tu liczy się moment, w którym WYKONAWCA skończył:
  // najpierw ostatnie oddanie, a dopiero gdy oddania nie było — domknięcie
  // zadania własnego. Bez tego robota oddana w piątek i zatwierdzona w poniedziałek
  // wpadłaby do poniedziałkowego raportu, czyli nie temu dniu, w którym powstała.
  if (filters.doneFrom) {
    where.push(`COALESCE(t.last_submitted_at, t.completed_at) >= ${addParam(filters.doneFrom)}`)
  }
  if (filters.doneTo) {
    where.push(`COALESCE(t.last_submitted_at, t.completed_at) <= ${addParam(filters.doneTo)}`)
  }
```

- [ ] **Krok 2: Sprawdź, że kolumny naprawdę tak się nazywają**

Uruchom:
```bash
grep -n "t.last_submitted_at\|t.completed_at" src/services/tasks/queries.ts | head -5
```
Oczekiwane: trafienia z `t.last_submitted_at` i `t.completed_at` (obie kolumny są
już w SELECT — patrz linie ~265-268 i sortowanie ~717).

- [ ] **Krok 3: Typy i lint**

Uruchom: `pnpm typecheck && pnpm lint --quiet`
Oczekiwane: bez błędów.

- [ ] **Krok 4: Commit**

```bash
git add src/services/tasks/queries.ts
git commit -m "feat(tasks): warunek zakresu po momencie oddania roboty"
```

---

### Zadanie 3: Widok raportowy w warstwie integracji

**Pliki:** modyfikacja `src/services/integrations/task-actions.ts`

- [ ] **Krok 1: Ustal nazwę pola z materiałami po stronie wykonawcy**

Uruchom:
```bash
grep -n "verificationDescription\|verificationRevisions" src/types/tasks.types.ts | head -8
```
Oczekiwane: `verificationDescription` występuje i w obiekcie wykonawcy
(`assignees[]`), i na samym zadaniu; `verificationRevisions` to tablica na zadaniu.
Jeśli w `assignees[]` pola nie ma — użyj wyłącznie pola zadania i pomiń w kroku 4
gałąź z wyszukiwaniem po wykonawcy. Nie zgaduj nazw.

- [ ] **Krok 2: Dopisz pola do `FindTasksOpts`**

```ts
export type FindTasksOpts = {
  taskId?: number
  query?: string
  limit?: number
  projectId?: number
  poolId?: number
  statuses?: TaskStatus[]
  view?: IntegrationView
  withBlockers?: boolean
  /** Raport dnia: zakres po momencie wyjścia roboty z rąk (ISO z czasem). */
  doneFrom?: string
  doneTo?: string
  /** Raport dnia: czyja robota. Widoczność i tak liczy listTasks. */
  assignedToUserId?: number
}
```

- [ ] **Krok 3: Dopisz wyciąganie zajawki materiałów (nad `toView`)**

```ts
// Materiały do weryfikacji przychodzą jako tekst formatowany. Do raportu idzie
// sam goły tekst i tylko początek — model streszcza to do jednego zdania, a
// pełna treść przy stu zadaniach rozdęłaby odpowiedź bez pożytku.
const MATERIALS_EXCERPT_CHARS = 400

function toPlainExcerpt(html: string | null): string | null {
  if (!html) return null
  const text = html
    .replace(/<br\s*\/?>/gi, " ")
    .replace(/<\/(p|div|li|h[1-6])>/gi, " ")
    .replace(/<[^>]+>/g, "")
    .replace(/&nbsp;/g, " ")
    .replace(/&amp;/g, "&")
    .replace(/&lt;/g, "<")
    .replace(/&gt;/g, ">")
    .replace(/\s+/g, " ")
    .trim()
  if (!text) return null
  return text.length > MATERIALS_EXCERPT_CHARS
    ? `${text.slice(0, MATERIALS_EXCERPT_CHARS)}…`
    : text
}
```

- [ ] **Krok 4: Dopisz trzy pola do `IntegrationTaskView` i do `toView`**

W typie, pod `blockedBy`:

```ts
  /** Moment wyjścia roboty z rąk — ostatnie oddanie, a bez niego domknięcie. */
  doneAt?: string | null
  /** Początek materiałów do weryfikacji, gołym tekstem. Null = nie wpisano. */
  materialsExcerpt?: string | null
  /** Zadanie wracało już do poprawy — raport ma to odnotować. */
  reworked?: boolean
```

W `toView`, przed zamykającą klamrą zwracanego obiektu (`subjectUserId` to osoba,
o którą pyta raport — patrz krok 5):

```ts
    doneAt: task.lastSubmittedAt ?? task.completedAt,
    materialsExcerpt: toPlainExcerpt(
      task.assignees.find((a) => a.id === subjectUserId)?.verificationDescription ??
        task.verificationDescription
    ),
    reworked: task.verificationRevisions.length > 0,
```

- [ ] **Krok 5: Przepuść osobę i zakres do `listTasks`**

W `findTasksForActor`, tam gdzie budowane są filtry przekazywane do `listTasks`,
dopisz przepisanie nowych opcji (nazwy pól filtra jeden do jednego):

```ts
    doneFrom: opts.doneFrom ?? null,
    doneTo: opts.doneTo ?? null,
    assignedToUserId: opts.assignedToUserId ?? null,
```

Oraz wylicz osobę, o którą pyta raport, i przekaż ją do `toView`:

```ts
  const subjectUserId = opts.assignedToUserId ?? actorUserId
```

Zmień sygnaturę:

```ts
function toView(task: Task, actorUserId: number, subjectUserId: number): IntegrationTaskView {
```

i wszystkie wywołania `toView(task, actorUserId)` na `toView(task, actorUserId, subjectUserId)`.
Znajdziesz je: `grep -n "toView(" src/services/integrations/task-actions.ts`

- [ ] **Krok 6: Typy i lint**

Uruchom: `pnpm typecheck && pnpm lint --quiet`
Oczekiwane: bez błędów.

- [ ] **Krok 7: Commit**

```bash
git add src/services/integrations/task-actions.ts
git commit -m "feat(integrations): widok raportowy zadan (moment oddania, zajawka materialow)"
```

---

### Zadanie 4: Parametry zapytania w route

**Pliki:** modyfikacja `src/app/api/v1/integrations/tasks/route.ts`

- [ ] **Krok 1: Dopisz pola do `querySchema`**

Pod `withBlockers`, przed `limit`:

```ts
    // Raport dnia: zakres po momencie wyjścia roboty z rąk. Pełny znacznik ISO
    // z czasem — granice doby liczy klient, bo tylko on wie, w jakiej strefie
    // siedzi człowiek (u nas: warszawska).
    doneFrom: z.string().datetime({ offset: true }).optional(),
    doneTo: z.string().datetime({ offset: true }).optional(),
    assignedToUserId: z.coerce.number().int().positive().optional(),
```

- [ ] **Krok 2: Podnieś sufit `limit` i wpuść zapytanie po samym zakresie dat**

```ts
    // Tydzień roboty jednej osoby nie mieści się w dwudziestu pozycjach.
    limit: z.coerce.number().int().min(1).max(300).optional(),
  })
  .refine(
    (v) =>
      v.taskId != null ||
      v.query != null ||
      v.projectId != null ||
      v.poolId != null ||
      v.view != null ||
      v.doneFrom != null,
    { message: "Podaj taskId, query, projectId, poolId, view albo doneFrom" }
  )
```

- [ ] **Krok 3: Przekaż nowe pola dalej**

W `GET`, w wywołaniu `findTasksForActor`, pola z `params` idą już przez `...params`
— sprawdź, że `doneFrom`, `doneTo` i `assignedToUserId` nie są po drodze gubione:

```bash
grep -n "findTasksForActor(" src/app/api/v1/integrations/tasks/route.ts
```
Oczekiwane: wywołanie z `...params` (wtedy nic nie trzeba dopisywać).

- [ ] **Krok 4: Typy i lint**

Uruchom: `pnpm typecheck && pnpm lint --quiet`
Oczekiwane: bez błędów.

- [ ] **Krok 5: Smoke po lokalnym API**

Przy chodzącym `pnpm dev` (port sprawdź, nie zakładaj):

```bash
KEY=$(node -e "const fs=require('fs'),os=require('os');const raw=fs.readFileSync(os.homedir()+'/.claude/tms.json','utf8').replace(/^﻿/,'').replace(/^\s*\/\/.*$/gm,'');console.log(JSON.parse(raw).apiKey)")
curl -s -H "Authorization: Bearer $KEY" "http://localhost:3000/api/v1/integrations/tasks?doneFrom=2026-08-01T00:00:00%2B02:00&doneTo=2026-09-01T00:00:00%2B02:00&limit=100" | head -c 600
```
Oczekiwane: `{"data":{"tasks":[...]}}`, a w pozycjach pola `doneAt` i
`materialsExcerpt`. Pusta lista też jest wynikiem poprawnym — sprawdź wtedy
zakresem obejmującym dzień, w którym coś było oddawane.

- [ ] **Krok 6: Commit**

```bash
git add src/app/api/v1/integrations/tasks/route.ts
git commit -m "feat(integrations): zakres dat i osoba w wyszukiwaniu zadan (raport)"
```

- [ ] **Krok 7: Podsumowanie dla człowieka**

Wypisz listę „co wchodzi" pogrupowaną po funkcjach i **zatrzymaj się** — push
i PR robi człowiek.

---

# CZĘŚĆ II — TMS, PR 2: dokumenty dla klucza osobistego

### Zadanie 5: Warstwa akcji na dokumentach

**Pliki:** nowy `src/services/integrations/document-actions.ts`

- [ ] **Krok 1: Gałąź od main**

```bash
cd /c/Users/jfinvesting/Desktop/Projekty-AI/TMS
git checkout main && git checkout -b piotr/raport-dokumenty
git branch --show-current
```
Oczekiwane: `piotr/raport-dokumenty`

- [ ] **Krok 2: Sprawdź kształt kontekstu użytkownika w uprawnieniach**

```bash
grep -n "type UserContext" -A 6 src/lib/permissions.ts
```
Oczekiwane: typ z polami `id` i `systemRole` (ewentualnie dodatkowymi,
opcjonalnymi). Jeśli wymaga więcej pól obowiązkowych — rozszerz `Actor` w kroku 3.

- [ ] **Krok 3: Napisz warstwę**

```ts
import type { SystemRole } from "@/lib/permissions"
import { canEditDocument, canUser } from "@/lib/permissions"
import { getDocumentById, getDocumentOwner, listDocumentsForUser } from "@/services/documents/queries"
import { createDocument, updateDocument } from "@/services/documents/mutations"
import type { Document, DocumentSummary } from "@/types/documents.types"

// Dokumenty dla kluczy OSOBISTYCH (skill Claude'a — dziennik pracy).
// Zasada ta sama co w task-actions: zero własnych reguł uprawnień. Zakładanie
// liczy `document.create`, zapis — `canEditDocument` (właściciel, moderator
// dokumentów albo admin), czyli dokładnie to, co sprawdza interfejs.

type Actor = { id: number; systemRole: SystemRole }
type Fail = { ok: false; reason: string; httpStatus: number }

export async function listDocumentsForActor(actor: Actor): Promise<DocumentSummary[]> {
  return listDocumentsForUser(actor.id)
}

/** Dziennik szukany po dokładnym tytule — bez tego każdy raport zakładałby nowy. */
export async function findDocumentByTitle(
  actor: Actor,
  title: string
): Promise<DocumentSummary | null> {
  const docs = await listDocumentsForUser(actor.id)
  return docs.find((d) => d.title === title) ?? null
}

export async function createDocumentForActor(
  actor: Actor,
  title: string
): Promise<{ ok: true; id: number; slug: string } | Fail> {
  if (!canUser(actor, "document.create")) {
    return { ok: false, reason: "Forbidden", httpStatus: 403 }
  }
  const created = await createDocument({ title }, actor.id)
  return { ok: true, ...created }
}

export async function readDocumentForActor(
  actor: Actor,
  documentId: number
): Promise<{ ok: true; document: Document } | Fail> {
  const doc = await getDocumentById(documentId, actor.id, actor.systemRole === "admin")
  if (!doc) return { ok: false, reason: "NotFound", httpStatus: 404 }
  return { ok: true, document: doc }
}

export async function writeDocumentForActor(
  actor: Actor,
  documentId: number,
  input: { title?: string; content?: string }
): Promise<{ ok: true } | Fail> {
  const ownerUserId = await getDocumentOwner(documentId)
  if (ownerUserId === null) return { ok: false, reason: "NotFound", httpStatus: 404 }
  if (!canEditDocument(actor, { ownerUserId })) {
    return { ok: false, reason: "Forbidden", httpStatus: 403 }
  }
  await updateDocument(documentId, input, actor.id)
  return { ok: true }
}
```

- [ ] **Krok 4: Typy**

Uruchom: `pnpm typecheck`
Oczekiwane: bez błędów.

- [ ] **Krok 5: Commit**

```bash
git add src/services/integrations/document-actions.ts
git commit -m "feat(integrations): warstwa akcji na dokumentach dla klucza osobistego"
```

---

### Zadanie 6: Wejście listy i zakładania

**Pliki:** nowy `src/app/api/v1/integrations/documents/route.ts`

- [ ] **Krok 1: Napisz route**

```ts
import { NextResponse, type NextRequest } from "next/server"
import { z } from "zod"
import { authenticateIntegrationRequest, extractApiKey } from "@/services/integrations/auth"
import {
  createDocumentForActor,
  findDocumentByTitle,
  listDocumentsForActor,
} from "@/services/integrations/document-actions"

export const dynamic = "force-dynamic"

// GET  /api/v1/integrations/documents            → własne dokumenty
// GET  /api/v1/integrations/documents?title=...  → jeden, po dokładnym tytule
// POST /api/v1/integrations/documents { "title": "Dziennik pracy — Jan Kowalski" }
// Dziennik pracy skilla. Reguły dostępu liczy warstwa document-actions.
const createSchema = z.object({ title: z.string().trim().min(1).max(200) })

export async function GET(req: NextRequest) {
  const apiKey = extractApiKey(req)
  if (!apiKey) return NextResponse.json({ error: "Unauthorized" }, { status: 401 })

  const identity = await authenticateIntegrationRequest("claude", apiKey)
  if (!identity) return NextResponse.json({ error: "Unauthorized" }, { status: 401 })
  if (identity.actorUserId == null || identity.actorSystemRole == null) {
    return NextResponse.json({ error: "PersonalKeyRequired" }, { status: 403 })
  }
  const actor = { id: identity.actorUserId, systemRole: identity.actorSystemRole }

  try {
    const title = new URL(req.url).searchParams.get("title")?.trim()
    if (title) {
      const document = await findDocumentByTitle(actor, title)
      return NextResponse.json({ data: { document } })
    }
    const documents = await listDocumentsForActor(actor)
    return NextResponse.json({ data: { documents } })
  } catch (err) {
    console.error("GET /api/v1/integrations/documents", err)
    return NextResponse.json({ error: "InternalError" }, { status: 500 })
  }
}

export async function POST(req: NextRequest) {
  const apiKey = extractApiKey(req)
  if (!apiKey) return NextResponse.json({ error: "Unauthorized" }, { status: 401 })

  const identity = await authenticateIntegrationRequest("claude", apiKey)
  if (!identity) return NextResponse.json({ error: "Unauthorized" }, { status: 401 })
  if (identity.actorUserId == null || identity.actorSystemRole == null) {
    return NextResponse.json({ error: "PersonalKeyRequired" }, { status: 403 })
  }
  const actor = { id: identity.actorUserId, systemRole: identity.actorSystemRole }

  try {
    const { title } = createSchema.parse(await req.json())
    const res = await createDocumentForActor(actor, title)
    if (!res.ok) return NextResponse.json({ error: res.reason }, { status: res.httpStatus })
    return NextResponse.json({ data: { id: res.id, slug: res.slug } }, { status: 201 })
  } catch (err) {
    if (err instanceof z.ZodError) {
      return NextResponse.json({ error: "ValidationError", details: err.flatten() }, { status: 400 })
    }
    console.error("POST /api/v1/integrations/documents", err)
    return NextResponse.json({ error: "InternalError" }, { status: 500 })
  }
}
```

- [ ] **Krok 2: Typy i lint**

Uruchom: `pnpm typecheck && pnpm lint --quiet`
Oczekiwane: bez błędów.

- [ ] **Krok 3: Commit**

```bash
git add src/app/api/v1/integrations/documents/route.ts
git commit -m "feat(integrations): lista i zakladanie dokumentow kluczem osobistym"
```

---

### Zadanie 7: Wejście odczytu i zapisu

**Pliki:** nowy `src/app/api/v1/integrations/documents/[id]/route.ts`

- [ ] **Krok 1: Napisz route**

```ts
import { NextResponse, type NextRequest } from "next/server"
import { z } from "zod"
import { authenticateIntegrationRequest, extractApiKey } from "@/services/integrations/auth"
import { readDocumentForActor, writeDocumentForActor } from "@/services/integrations/document-actions"
import { documentUpdateSchema } from "@/types/documents.types"

export const dynamic = "force-dynamic"

// GET   /api/v1/integrations/documents/12  → treść dziennika (do dopisania sekcji)
// PATCH /api/v1/integrations/documents/12  { "content": "<h2>…</h2>" }
// Limit treści (500 KB) egzekwuje documentUpdateSchema — ten sam co w interfejsie.
type RouteCtx = { params: Promise<{ id: string }> }

async function actorFrom(req: NextRequest) {
  const apiKey = extractApiKey(req)
  if (!apiKey) return null
  const identity = await authenticateIntegrationRequest("claude", apiKey)
  if (!identity || identity.actorUserId == null || identity.actorSystemRole == null) return null
  return { id: identity.actorUserId, systemRole: identity.actorSystemRole }
}

export async function GET(req: NextRequest, ctx: RouteCtx) {
  const actor = await actorFrom(req)
  if (!actor) return NextResponse.json({ error: "Unauthorized" }, { status: 401 })

  const documentId = Number((await ctx.params).id)
  if (!Number.isInteger(documentId) || documentId <= 0) {
    return NextResponse.json({ error: "ValidationError" }, { status: 400 })
  }
  const res = await readDocumentForActor(actor, documentId)
  if (!res.ok) return NextResponse.json({ error: res.reason }, { status: res.httpStatus })
  return NextResponse.json({ data: { document: res.document } })
}

export async function PATCH(req: NextRequest, ctx: RouteCtx) {
  const actor = await actorFrom(req)
  if (!actor) return NextResponse.json({ error: "Unauthorized" }, { status: 401 })

  const documentId = Number((await ctx.params).id)
  if (!Number.isInteger(documentId) || documentId <= 0) {
    return NextResponse.json({ error: "ValidationError" }, { status: 400 })
  }
  try {
    const input = documentUpdateSchema.parse(await req.json())
    const res = await writeDocumentForActor(actor, documentId, input)
    if (!res.ok) return NextResponse.json({ error: res.reason }, { status: res.httpStatus })
    return NextResponse.json({ data: { ok: true } })
  } catch (err) {
    if (err instanceof z.ZodError) {
      return NextResponse.json({ error: "ValidationError", details: err.flatten() }, { status: 400 })
    }
    console.error("PATCH /api/v1/integrations/documents/[id]", err)
    return NextResponse.json({ error: "InternalError" }, { status: 500 })
  }
}
```

- [ ] **Krok 2: Typy i lint**

Uruchom: `pnpm typecheck && pnpm lint --quiet`
Oczekiwane: bez błędów.

- [ ] **Krok 3: Smoke — pełne kółko na dokumencie**

```bash
KEY=$(node -e "const fs=require('fs'),os=require('os');const raw=fs.readFileSync(os.homedir()+'/.claude/tms.json','utf8').replace(/^﻿/,'').replace(/^\s*\/\/.*$/gm,'');console.log(JSON.parse(raw).apiKey)")
BASE=http://localhost:3000/api/v1/integrations/documents
ID=$(curl -s -X POST -H "Authorization: Bearer $KEY" -H "Content-Type: application/json" -d "{\"title\":\"Dziennik pracy — smoke\"}" "$BASE" | node -e "let s='';process.stdin.on('data',d=>s+=d).on('end',()=>console.log(JSON.parse(s).data.id))")
curl -s -X PATCH -H "Authorization: Bearer $KEY" -H "Content-Type: application/json" -d "{\"content\":\"<h2>2026-08-31</h2><p>proba</p>\"}" "$BASE/$ID"
curl -s -H "Authorization: Bearer $KEY" "$BASE/$ID" | head -c 300
```
Oczekiwane: `{"data":{"ok":true}}` po zapisie, a po odczycie dokument z treścią
`<h2>2026-08-31</h2><p>proba</p>`. Dokument „Dziennik pracy — smoke" skasuj
potem ręcznie w TMS.

- [ ] **Krok 4: Commit i podsumowanie**

```bash
git add "src/app/api/v1/integrations/documents/[id]/route.ts"
git commit -m "feat(integrations): odczyt i zapis dokumentu kluczem osobistym"
```
Potem wypisz „co wchodzi" i **zatrzymaj się** — push i PR robi człowiek.

---

# CZĘŚĆ III — wtyczka 0.30.0

### Zadanie 8: Instrukcja raportu

**Pliki:** nowy `skills/raport/SKILL.md` (repo `TMS-Claude-SKILL`, gałąź `piotr/raport-dnia`)

Instrukcja pisana prozą, po polsku, w drugiej osobie — tak jak `skills/zadanie/SKILL.md`.
Wzoruj się na nim: nagłówek `---` z `name` i `description`, potem sekcje.

- [ ] **Krok 1: Nagłówek z wyzwalaczami**

`name: raport`. W `description` wymień wprost zdania, na które ma reagować:
„co dziś zrobione", „podsumuj mi dzień", „podsumuj tydzień", „co zrobiłem od
poniedziałku", „co Wojtek zrobił wczoraj", „raport z sierpnia", „zapisz to do
dziennika". Bez tego trafi się w raport tylko komendą, a o komendzie się zapomina.

- [ ] **Krok 2: Sekcja „Co się liczy"**

Zapisz regułę: pozycją jest zadanie, w którym wskazana osoba jest wykonawcą i które
w okresie wyszło jej z rąk — czyli ma `doneAt` w zakresie. Zatwierdzenie cudzą ręką
nie tworzy drugiej pozycji ani nie przesuwa zadania na inny dzień. `reworked: true`
→ dopisek „poprawka". Zadania założone, zaczęte i stojące w toku **nie wchodzą**.

- [ ] **Krok 3: Sekcja „Jak pytasz"**

Granice doby liczy wtyczka, w czasie warszawskim, i wysyła pełne znaczniki ISO:

```
GET {baseUrl}/api/v1/integrations/tasks
    ?doneFrom=2026-08-31T00:00:00+02:00
    &doneTo=2026-08-31T23:59:59+02:00
    &personId=7
    &limit=300
```

`personId` to „czyja robota" liczona **per osoba**, a nie filtr po głównym
wykonawcy — patrz „Poprawka po Części I" na końcu planu.

Osobę bierz ze słownika (`GET /api/v1/integrations/dictionary`, lista osób).
Gdy słownik pokazuje tylko właściciela klucza, a pytanie dotyczy kogoś innego —
powiedz wprost, że nie ma jak rozpoznać osoby, i nie zgaduj numeru.
Gdy wróci dokładnie tyle pozycji, ile wynosi `limit` — napisz, że lista mogła
zostać ucięta.

- [ ] **Krok 4: Sekcja „Jak wygląda raport"**

Dni malejąco, w dniu grupowanie po projekcie. Pozycja: numer, tytuł, projekt,
godzina, znacznik stanu (*zatwierdzone* dla `completed`, *czeka na weryfikację*
dla `to_verify`, *wróciło do poprawy* dla `rework_needed`) i jedno zdanie
streszczone z `materialsExcerpt`. Puste `materialsExcerpt` → sama linijka;
**niczego nie dopisujesz od siebie**. Na końcu liczba zadań, a przy cudzej robocie
zdanie o niepełnym wglądzie (widać tylko zadania własne, zlecone przez siebie
i te z projektów, których jest się członkiem).

- [ ] **Krok 5: Sekcja „Dziennik"**

Zapis tylko na prośbę. Tytuł „Dziennik pracy — Imię Nazwisko", właścicielem jest
właściciel klucza — także gdy raport dotyczy kogoś innego. Kolejność działań:
szukaj po tytule (`GET /api/v1/integrations/documents?title=…`), brak → załóż
(`POST`), potem `GET /api/v1/integrations/documents/{id}`, złóż nową treść
i zapisz `PATCH`-em.

Sekcja dnia to `<h2>2026-08-31</h2>` i pod nim pozycje. **Nadpisywanie:** jeśli
w treści jest już `<h2>` z tą datą, podmień wszystko od tego nagłówka do
następnego `<h2>` (albo do końca dokumentu), zamiast dokładać drugą sekcję. Nowe
dni idą na górę. Gdy treść zbliża się do 500 KB — załóż „Dziennik pracy — Imię
Nazwisko (część 2)" i pisz tam.

- [ ] **Krok 6: Sekcja „Granice"**

Źródłem jest wyłącznie TMS — raport nie streszcza rozmowy ani nie dopisuje niczego
z pamięci. Pusty okres kwitujesz wprost („nic nie wyszło z rąk"). Nierozpoznane
imię — pytaniem, nie zgadywaniem.

- [ ] **Krok 7: Commit**

```bash
cd /c/Users/jfinvesting/Desktop/Projekty-AI/TMS-Claude-SKILL
git branch --show-current   # oczekiwane: piotr/raport-dnia
git add skills/raport/SKILL.md
git commit -m "feat: raport wykonanej roboty (0.30.0)"
```

---

### Zadanie 9: Podbicie wersji i README

**Pliki:** modyfikacja `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `skills/ustawienia/SKILL.md`, `README.md`

- [ ] **Krok 1: Wersja w obu plikach wtyczki**

```bash
cd /c/Users/jfinvesting/Desktop/Projekty-AI/TMS-Claude-SKILL
sed -i 's/"version": "0.29.0"/"version": "0.30.0"/' .claude-plugin/plugin.json .claude-plugin/marketplace.json
grep -n '"version"' .claude-plugin/plugin.json .claude-plugin/marketplace.json
```
Oczekiwane: w obu plikach `0.30.0`.

- [ ] **Krok 2: Numer wydania w instrukcji ustawień**

`skills/ustawienia/SKILL.md` niesie numer w treści („Ta instrukcja pochodzi
z wydania 0.29.0") i porównuje go z GitHubem — bez podmiany wtyczka będzie
twierdzić, że jest starsza, niż jest.

```bash
sed -i 's/0\.29\.0/0.30.0/g' skills/ustawienia/SKILL.md
grep -n "0.30.0" skills/ustawienia/SKILL.md
```
Oczekiwane: trafienia w zdaniu o wydaniu i w przykładowym bloku ustawień.

- [ ] **Krok 3: Sekcja w README**

Dopisz „## Raport wykonanej roboty" po sekcji „Podobne zadania": co komenda robi,
że liczy tylko rzeczy domknięte, że umie zapisać dziennik w dokumentach TMS oraz
że raport o cudzej robocie pokazuje tylko to, co widać z własnego konta.

- [ ] **Krok 4: Commit**

```bash
git add .claude-plugin README.md skills/ustawienia/SKILL.md
git commit -m "chore: wydanie 0.30.0"
```

---

### Zadanie 10: Klik-test i zamknięcie

- [ ] **Krok 1: Aktualizacja wtyczki w nowej sesji**

Po scaleniu i wydaniu: `claude plugin update tms@jf-tms`, potem **nowa sesja**
(stara trzyma starą instrukcję). Sprawdź `/tms:ustawienia` — ma pokazać 0.30.0.

- [ ] **Krok 2: Trzy próby na żywo**

Poproś kolejno: „co dziś zrobione", „podsumuj mi ten tydzień", „zapisz to do
dziennika". Sprawdź w TMS, czy dokument „Dziennik pracy — …" ma jedną sekcję na
dzień. Powtórz „co dziś zrobione" i zapis — sekcja ma zostać **nadpisana**, nie
zdublowana.

- [ ] **Krok 3: Zadanie dokumentacyjne w TMS**

Załóż zadanie w puli SKILL-Claude, nazwane od numeru wydania, z materiałami
opisującymi, co weszło (dwa PR-y w TMS i wydanie wtyczki).

---

## Kolejność i zależności

Część I i II są od siebie niezależne — można je robić równolegle, ale **na osobnych
gałęziach**. Część III wymaga obu scalonych i wdrożonych na produkcję; bez nich
wtyczka dostanie 400 na nieznanych parametrach.

## Poprawka po Części I — liczenie per osoba

Ustalone przy wdrożeniu, 2026-08-31. Plan zakładał filtr `assignedToUserId`, który
w TMS wskazuje **głównego wykonawcę** (`tasks.assigned_to_user_id`). Tymczasem
zadanie może mieć wielu wykonawców: `task_assignees` trzyma dla każdego własny
pod-stan, własne znaczniki czasu (`submitted_for_review_at`, `verified_at`)
i własne materiały. Filtr po głównym wykonawcy gubiłby więc robotę osoby dopisanej
do zadania współdzielonego, a specyfikacja mówi wprost: „wyszło **jej** z rąk".

Dlatego:

- `assignedToUserId` **zostaje nietknięty** — jego semantyka należy do interfejsu.
- Doszedł osobny filtr `doneByUserId`, liczony przez `EXISTS` po `task_assignees`,
  z kotwicą czasu **tej osoby**. Zakres `doneFrom`/`doneTo` musi liczyć się po tej
  samej kotwicy co filtr osoby — inaczej do raportu wpadną zadania, w których
  w naszym okresie oddał ktoś inny.
- W API parametrem jest `personId` (mapowany na `doneByUserId`).
- `doneAt` i `materialsExcerpt` też są brane per osoba; gdy osoba nie wpisała
  własnych materiałów, lepiej zwrócić `null` niż podstawić cudze.

Po tym wszystkim wracamy do czterech rzeczy z backlogu: komentarz do istniejącego
zadania, załączniki z dysku, numer PR-a w materiałach i relacje blokad. Uwaga na
przyszłość: wejścia integracyjne dla komentarzy, załączników i blokad **już
istnieją** (`/integrations/tasks/[id]/comments`, `/attachments`, `/blockers`) —
tam robota będzie głównie po stronie instrukcji wtyczki, nie serwera.
