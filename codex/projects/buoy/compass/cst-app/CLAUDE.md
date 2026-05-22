<!-- GENERATED FROM claude/projects/buoy/compass/cst-app/CLAUDE.md. Do not edit directly. -->

# TypeScript/React Conventions (compass/cst-app/)

- Framework: Next.js 14 with App Router, React 18, TypeScript strict mode
- Component files: PascalCase directories with `index.tsx` (e.g., `StudentCard/index.tsx`)
- Components: functional components typed as `React.FC<IPropsInterface>`
- Props interfaces: PascalCase with `I` prefix (e.g., `IStudentCard`)
- State management: Redux Toolkit — slices in `app/features/`, typed hooks in `hooks/useRedux.ts`
- Styling: Tailwind CSS with custom color palette and font families in `tailwind.config.ts`
- Path alias: `@/` resolves to `./src/`
- API layer: generic `fetchData<T>()` helper with typed `FetchResponse<T>`; API route constants in `constants/apiRoutes.ts`
- Custom hooks: `use` prefix, camelCase filenames (e.g., `useAuth.ts`, `useAlert.ts`)
- Component organization: `/components/Base/` for reusable primitives, `/components/{Feature}/` for feature-specific
- Auth: Auth0 via `@auth0/auth0-react` with `withAuth` HOC pattern
- Navigation: `useRouter()` from `next/navigation`; route constants in `constants/routes.ts`
- Enums: PascalCase names, UPPER_SNAKE_CASE values
- Build: static export (`output: 'export'`), Capacitor for iOS/Android wrappers
- Testing: Jest + React Testing Library; mocks in `__mocks__/` directories
- ESLint: extends `next/core-web-vitals` and `next/typescript`
- Before pushing: run `npm run lint` and `npm test` from `compass/cst-app/`. Type errors surface when the dev server compiles or via `npm run build`.
