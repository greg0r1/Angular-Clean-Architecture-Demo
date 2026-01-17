# 🔄 State - Gestion d'état

## 📋 Description

Le dossier **State** contient la gestion d'état global de l'application avec Signals ou state management libraries.

## ✅ Ce qu'on doit mettre

- ✅ State stores avec Signals
- ✅ State services
- ✅ Actions et reducers (si NgRx)
- ✅ Selectors

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Logique métier (→ Domain)
- ❌ Use cases (→ Application)
- ❌ Components (→ components/)

## 📊 Exemples

### Simple Signal Store

```typescript
// user.store.ts
@Injectable({ providedIn: 'root' })
export class UserStore {
  private getUserUseCase = inject(GetUserByIdUseCase);

  // State
  private _currentUser = signal<User | null>(null);
  private _loading = signal(false);
  private _error = signal<string | null>(null);

  // Public readonly signals
  currentUser = this._currentUser.asReadonly();
  loading = this._loading.asReadonly();
  error = this._error.asReadonly();

  // Computed
  isAuthenticated = computed(() => this._currentUser() !== null);
  userName = computed(() =>
    this._currentUser()?.name.getFullName() ?? 'Guest'
  );

  // Actions
  loadUser(id: string): void {
    this._loading.set(true);
    this._error.set(null);

    this.getUserUseCase.execute(id).subscribe({
      next: (user) => {
        this._currentUser.set(user);
        this._loading.set(false);
      },
      error: (error) => {
        this._error.set(error.message);
        this._loading.set(false);
      }
    });
  }

  clearUser(): void {
    this._currentUser.set(null);
  }
}
```

### Cart Store

```typescript
// cart.store.ts
@Injectable({ providedIn: 'root' })
export class CartStore {
  private addToCartUseCase = inject(AddToCartUseCase);

  // State
  private _items = signal<CartItem[]>([]);

  // Public
  items = this._items.asReadonly();

  // Computed
  itemCount = computed(() =>
    this._items().reduce((sum, item) => sum + item.quantity, 0)
  );

  total = computed(() =>
    this._items().reduce((sum, item) =>
      sum + item.price.amount * item.quantity, 0
    )
  );

  // Actions
  addItem(item: CartItem): void {
    const existingItem = this._items().find(i => i.productId === item.productId);

    if (existingItem) {
      this._items.update(items =>
        items.map(i =>
          i.productId === item.productId
            ? { ...i, quantity: i.quantity + item.quantity }
            : i
        )
      );
    } else {
      this._items.update(items => [...items, item]);
    }
  }

  removeItem(productId: string): void {
    this._items.update(items =>
      items.filter(item => item.productId !== productId)
    );
  }

  clear(): void {
    this._items.set([]);
  }
}
```

---

**Le state management rend votre application réactive et prédictible ! 🔄**
