# LG TV Remote

[![Buy Me A Coffee](https://img.shields.io/badge/Buy_Me_A_Coffee-FFDD00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black)](https://www.buymeacoffee.com/cmtrkr)

A local app that discovers and controls LG NetCast 3.0 / 4.0 TVs over the same Wi-Fi network. Install the Android APK on your phone and use it right away. To modify the code, follow the steps below.

No ads or telemetry. Free to use.

## Stack

- pnpm workspaces, Node.js >=24.3.0, TypeScript 6.0.3
- API: Express 5
- Database: PostgreSQL + Drizzle ORM
- App: Expo 57 + React Native 0.86 + expo-router
- Discovery: `expo-ssdp` (SSDP M-SEARCH/B-SEARCH) + `expo-network` subnet fallback (ROAP `http://<tv>:8080/roap/api/`)
- Validation: Zod, Orval (OpenAPI)

## Running

```bash
corepack enable
pnpm install --frozen-lockfile
pnpm run typecheck
pnpm run test
pnpm run lint
PUBLIC_ORIGIN=https://example.test pnpm run build

# API server (PORT is required)
PORT=5000 pnpm --filter @workspace/api-server run dev
# Or after building
# pnpm --filter @workspace/api-server run build && pnpm --filter @workspace/api-server run start

# NetCast Remote (Expo)
pnpm --filter @workspace/netcast-remote run dev
# PUBLIC_ORIGIN=https://example.test pnpm --filter @workspace/netcast-remote run build
# PUBLIC_ORIGIN=https://example.test pnpm --filter @workspace/netcast-remote run serve

# Mockup sandbox
pnpm --filter @workspace/mockup-sandbox run dev
```

You can also use `pnpm run verify`, `pnpm run audit`, and `pnpm run doctor` for CI and local validation. `verify` runs the Orval generated-code diff check, typecheck, tests, lint, and format check in order.

## Language

NetCast Remote opens in Turkish by default. You can switch languages instantly with the `TR / EN` toggle on the setup screen. The choice is stored under the `netcast-remote-language` AsyncStorage key and restored on subsequent launches. UI strings resolve through the `LanguageProvider` and `useTranslation` (`t`) hook in `artifacts/netcast-remote/i18n`.

## Environment variables

- `PUBLIC_ORIGIN` — required canonical origin when building and serving the NetCast static bundle; it must not contain a path, query, or fragment. CI smoke builds can use `https://example.test`.
- `BASE_PATH` — safe absolute path when building/serving the NetCast static bundle and for mockup Vite; defaults to `/`. It must not contain `..`, backslashes, or invalid percent encoding.
- `DATABASE_URL` — Postgres connection string (for api-server and the reviewed DB workflow).
- `PORT` — port for the API/mockup (defaults: api 5000, mockup 5173).
- `EXPO_PUBLIC_DOMAIN` / `DEPLOYMENT_DOMAIN` — fallback domain for the netcast-remote static build when `PUBLIC_ORIGIN` is not provided.

## Expo Doctor

Expo Doctor runs in the netcast-remote package and validates SDK 57 version alignment. Use Node >=24.3.0 and the repository `packageManager` version (`pnpm@12.6.0`). The expected linked versions are `expo ~57.0.24`, `expo-build-properties ~57.0.21`, `expo-constants ~57.0.19`, `expo-image-picker ~57.0.19`, `expo-location ~57.0.19`, and `expo-router ~57.0.22`:

```bash
pnpm run doctor
```

## Security overrides

`uuid@11.1.1` is overridden for `xcode@3.0.1`, and the official `decode-uri-component@0.5.0` is overridden for `query-string@7.1.3`. Because the latter package is ESM, `tests/fixtures/patches/query-string@7.1.3.patch` adds only default-export interop to the CJS `query-string` call. This compatibility is verified by `tests/dependency-compatibility.test.ts` and the Expo build.

## Database workflow

Do not apply database changes directly. Generate, review, and apply schema changes:

```bash
pnpm --filter @workspace/db run generate
pnpm --filter @workspace/db run check
pnpm --filter @workspace/db run migrate
```

## Project structure

```text
artifacts/api-server      -> Express API (/api/healthz)
artifacts/netcast-remote  -> Expo remote app (main product)
artifacts/mockup-sandbox  -> Vite component preview
lib/api-spec              -> OpenAPI spec (openapi.yaml) + Orval
lib/api-client-react      -> React Query client (codegen)
lib/api-zod               -> Zod schemas (codegen)
lib/db                    -> Drizzle schema
```

## Notes

- SSDP scanning works on a physical device; web/Expo Go uses the manual IP fallback.
- NetCast ROAP uses port 8080 and cleartext HTTP (Android `usesCleartextTraffic`).

## Support

If you find this project useful, you can buy me a coffee:

[![Buy Me A Coffee](https://img.shields.io/badge/Buy_Me_A_Coffee-FFDD00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black)](https://www.buymeacoffee.com/cmtrkr)
