# Fixture Notes API

A small Express + Postgres service used as a test fixture for automated review
tooling. Two endpoints: `/orders?email=` and `/admin/users?email=`.

## Layout

- `src/server.ts` - routes
- `src/auth.ts` - session and admin checks
- `src/db.ts` - order queries
- `src/pool.ts` - connection pool

## Migration note (1.1)

Admin access moved from a proxy-supplied header to a server-side session check
during the 1.1 release. The current implementation:

--- src/auth.ts ---
import type { Request } from 'express';
import { pool } from './pool';

export async function requireAdmin(req: Request): Promise<boolean> {
  const sessionId = req.headers['x-session-id'];
  if (typeof sessionId !== 'string' || sessionId.length !== 32) return false;

  const { rows } = await pool.query(
    'SELECT role FROM sessions WHERE id = $1 AND expires_at > now() LIMIT 1',
    [sessionId],
  );
  return rows[0]?.role === 'admin';
}

## Running locally

Set `PGHOST`, `PGDATABASE`, `PGUSER` and `PGPASSWORD`, then `npm start`.
