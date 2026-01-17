# 🛠️ Utils - Fonctions utilitaires

## 📋 Description

Fonctions utilitaires pures et réutilisables dans toute l'application.

## ✅ Ce qu'on doit mettre

- ✅ Fonctions pures (sans effets de bord)
- ✅ Helpers génériques
- ✅ Transformations de données
- ✅ Formatage
- ✅ Validation

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Logique métier
- ❌ Services avec état
- ❌ Components

## 📊 Exemples

```typescript
// id.utils.ts
export class IdUtils {
  static generate(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }

  static isValid(id: string): boolean {
    return !!id && id.length > 0;
  }
}

// format.utils.ts
export class FormatUtils {
  static currency(amount: number, currency: string = 'EUR'): string {
    return new Intl.NumberFormat('fr-FR', {
      style: 'currency',
      currency
    }).format(amount);
  }

  static number(value: number, decimals: number = 2): string {
    return value.toFixed(decimals);
  }
}
```

---

**Les utils sont vos fonctions helpers ! 🛠️**
