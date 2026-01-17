# 🔧 Shared - Code partagé

## 📋 Description

Le dossier **Shared** contient le code réutilisable et partagé entre toutes les couches de l'application.

## 🏗️ Structure

```
shared/
├── utils/      # Fonctions utilitaires
├── types/      # Types TypeScript communs
└── README.md
```

## ✅ Ce qu'on doit mettre

- ✅ Fonctions utilitaires pures
- ✅ Types TypeScript génériques
- ✅ Constantes globales
- ✅ Helpers
- ✅ Validators custom
- ✅ Pipes réutilisables
- ✅ Directives réutilisables

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Logique métier (→ Domain)
- ❌ Components (→ Presentation)
- ❌ Use cases (→ Application)
- ❌ Services avec état

## 📊 Exemples

### Utils

```typescript
// utils/string.utils.ts
export class StringUtils {
  static capitalize(str: string): string {
    if (!str) return '';
    return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
  }

  static truncate(str: string, maxLength: number): string {
    if (str.length <= maxLength) return str;
    return str.slice(0, maxLength) + '...';
  }

  static slugify(str: string): string {
    return str
      .toLowerCase()
      .normalize('NFD')
      .replace(/[\u0300-\u036f]/g, '')
      .replace(/[^a-z0-9]+/g, '-')
      .replace(/(^-|-$)/g, '');
  }
}

// utils/date.utils.ts
export class DateUtils {
  static formatDate(date: Date, format: 'short' | 'long' = 'short'): string {
    const options: Intl.DateTimeFormatOptions = format === 'long'
      ? { year: 'numeric', month: 'long', day: 'numeric' }
      : { year: 'numeric', month: '2-digit', day: '2-digit' };

    return date.toLocaleDateString('fr-FR', options);
  }

  static isToday(date: Date): boolean {
    const today = new Date();
    return date.toDateString() === today.toDateString();
  }

  static daysBetween(date1: Date, date2: Date): number {
    const diff = Math.abs(date2.getTime() - date1.getTime());
    return Math.ceil(diff / (1000 * 60 * 60 * 24));
  }
}

// utils/array.utils.ts
export class ArrayUtils {
  static unique<T>(array: T[]): T[] {
    return [...new Set(array)];
  }

  static groupBy<T>(array: T[], key: keyof T): Record<string, T[]> {
    return array.reduce((groups, item) => {
      const groupKey = String(item[key]);
      if (!groups[groupKey]) {
        groups[groupKey] = [];
      }
      groups[groupKey].push(item);
      return groups;
    }, {} as Record<string, T[]>);
  }

  static chunk<T>(array: T[], size: number): T[][] {
    const chunks: T[][] = [];
    for (let i = 0; i < array.length; i += size) {
      chunks.push(array.slice(i, i + size));
    }
    return chunks;
  }
}
```

### Types

```typescript
// types/common.types.ts
export type Nullable<T> = T | null;
export type Optional<T> = T | undefined;
export type Maybe<T> = T | null | undefined;

export type Dictionary<T> = Record<string, T>;

export type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

export type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

// types/result.type.ts
export type Result<T, E = Error> = Success<T> | Failure<E>;

export class Success<T> {
  readonly success = true as const;
  constructor(public readonly value: T) {}
}

export class Failure<E> {
  readonly success = false as const;
  constructor(public readonly error: E) {}
}

// types/pagination.types.ts
export interface PaginationParams {
  page: number;
  pageSize: number;
}

export interface PaginatedResponse<T> {
  items: T[];
  total: number;
  page: number;
  pageSize: number;
  totalPages: number;
}
```

### Pipes

```typescript
// pipes/truncate.pipe.ts
@Pipe({
  name: 'truncate',
  standalone: true
})
export class TruncatePipe implements PipeTransform {
  transform(value: string, maxLength: number = 50): string {
    if (!value) return '';
    return StringUtils.truncate(value, maxLength);
  }
}

// pipes/format-date.pipe.ts
@Pipe({
  name: 'formatDate',
  standalone: true
})
export class FormatDatePipe implements PipeTransform {
  transform(value: Date, format: 'short' | 'long' = 'short'): string {
    if (!value) return '';
    return DateUtils.formatDate(value, format);
  }
}
```

### Validators

```typescript
// validators/custom.validators.ts
export class CustomValidators {
  static strongPassword(control: AbstractControl): ValidationErrors | null {
    const value = control.value as string;

    if (!value) return null;

    const hasUpperCase = /[A-Z]/.test(value);
    const hasLowerCase = /[a-z]/.test(value);
    const hasNumeric = /[0-9]/.test(value);
    const hasSpecialChar = /[!@#$%^&*]/.test(value);
    const isLongEnough = value.length >= 8;

    const passwordValid =
      hasUpperCase && hasLowerCase && hasNumeric && hasSpecialChar && isLongEnough;

    return passwordValid ? null : { weakPassword: true };
  }

  static noWhitespace(control: AbstractControl): ValidationErrors | null {
    const value = control.value as string;
    if (!value) return null;

    const hasWhitespace = /\s/.test(value);
    return hasWhitespace ? { whitespace: true } : null;
  }
}
```

## 🎓 Points clés

1. **Pureté** : Fonctions pures sans effets de bord
2. **Réutilisabilité** : Code utilisable partout
3. **Sans état** : Pas de state dans shared
4. **Types** : Types génériques TypeScript
5. **Utils** : Fonctions helpers
6. **Pipes** : Pipes Angular standalone
7. **Validators** : Validators custom pour formulaires

---

**Le shared contient vos outils réutilisables ! 🔧**
