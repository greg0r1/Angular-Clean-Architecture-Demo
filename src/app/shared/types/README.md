# 📐 Types - Types TypeScript

## 📋 Description

Types TypeScript génériques et réutilisables.

## ✅ Ce qu'on doit mettre

- ✅ Types génériques TypeScript
- ✅ Interfaces communes
- ✅ Type aliases
- ✅ Utility types

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Entités du Domain
- ❌ DTOs spécifiques
- ❌ Interfaces de ports

## 📊 Exemples

```typescript
// common.types.ts
export type ID = string;
export type Timestamp = number;

export type Nullable<T> = T | null;
export type Optional<T> = T | undefined;

export type ReadonlyDeep<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? ReadonlyDeep<T[P]>
    : T[P];
};

// api.types.ts
export interface ApiResponse<T> {
  data: T;
  status: number;
  message?: string;
}

export interface ApiError {
  code: string;
  message: string;
  details?: Record<string, any>;
}

// pagination.types.ts
export interface Paginated<T> {
  items: T[];
  total: number;
  page: number;
  pageSize: number;
}
```

---

**Les types renforcent la sécurité de votre code ! 📐**
