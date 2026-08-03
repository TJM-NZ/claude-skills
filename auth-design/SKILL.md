---
name: auth-design
description: Design authentication and authorization architecture before writing code — asks discovery questions, outputs entity model, permission matrix, auth flows, protected routes, and managed provider recommendation. Use at the start of any new webapp that needs auth.
when_to_use: User is starting a new webapp and needs to design auth, mentions "authentication", "login", "auth model", "workspace", "multi-tenant", "roles", or asks how to structure auth before building.
allowed-tools: Read
---

Output a design spec only — no code. Preferred providers: Clerk, Better Auth, Supabase Auth.

## Phase 1: Discovery

Ask these in one message:

1. **Auth pattern**
   - A) Simple — users only, no org hierarchy
   - B) Single-tenant — one shared workspace for all users
   - C) Multi-tenant — users belong to one or more orgs/workspaces
   - D) Multi-tenant + teams — workspaces contain sub-groups

2. **Sign-in methods** (can be multiple): email+password, magic link, OAuth (which providers?), SSO/SAML

3. **Roles**: none | admin/member | named roles (list them) | per-resource permissions

4. **Special requirements**: public content, invite-only signup, email domain restriction, MFA, API keys

5. **Stack**: Next.js App Router | Remix | Express + separate frontend | other

---

## Phase 2: Spec

### Entity Model

Text ER diagram. Only include entities this pattern actually needs.

Reference shapes:
- A: `User — Session`
- B: `User(role: admin|member) — Session`
- C: `User <─ Membership(role) ─> Workspace — Invitation(email, role, expires_at)`
- D: C + `Workspace <─ Team <─ TeamMembership(role) ─> User`

List key fields per entity.

### Permission Matrix

Table: roles × actions. Columns = the app's actual resources. Mark ✓ / —.
Always include: view content, invite users, manage roles, delete workspace, billing.
Flag if per-resource permissions are needed — note this requires an ACL table and is significantly more complex.

### Auth Flows

Step-by-step for each relevant flow. Always include: sign up, sign in, sign out.
Add if applicable: invite flow, workspace creation, workspace switching, team management, role change, MFA.

### Protected Routes

| Route | Auth required | Min role | Notes |
|-------|--------------|----------|-------|

Multi-tenant flag: **every data query must scope to the active workspace via session, not URL alone** — most common multi-tenant bug.

### Provider Recommendation

Decision guide:
- A/B → Better Auth or Supabase Auth (no org features needed)
- C → Clerk (orgs, invitations, switching built-in)
- D → Clerk (if teams GA) or Better Auth org plugin

Stack notes:
- Next.js App Router: all three work; Clerk has best middleware integration
- Remix: Better Auth adapter is solid; Clerk works; Supabase is more manual
- Express + separate frontend: Better Auth most flexible

Output: what it handles natively, what needs custom logic, one key gotcha for this pattern + provider.

---

## Phase 3: Flag Anti-patterns

Check and call out any that apply:
- Roles in JWT → role changes don't take effect until token expiry; use server-side lookup
- URL-only workspace scoping → always verify membership server-side, not from `:workspaceId` alone
- Shared sessions across workspaces → active workspace must be session state, not client state
- Invitation tokens with no expiry → must be single-use and time-limited
- Per-resource permissions added late → design the policy table now if likely needed; retrofitting is expensive
