# 🎨 Presentation - Interface utilisateur

## 📋 Description

La couche **Presentation** contient tout ce qui est visible et interactif pour l'utilisateur : components, pages, state management, et logique de présentation.

> **Principe clé** : La Presentation **orchestre l'UI** et **appelle les Use Cases**.

## 🏗️ Structure

```
presentation/
├── pages/          # Composants de niveau page (routes)
├── components/     # Composants réutilisables
├── state/          # Gestion d'état (Signals, Stores)
├── view-models/    # Logique de présentation
└── README.md
```

## ✅ Ce qu'on doit mettre

- ✅ Components standalone Angular
- ✅ Pages (routed components)
- ✅ State management (Signals)
- ✅ View Models
- ✅ Formulaires (Reactive Forms)
- ✅ Directives et Pipes
- ✅ Templates HTML
- ✅ Styles CSS/SCSS

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Logique métier (→ Domain)
- ❌ Use cases (→ Application)
- ❌ Appels HTTP directs (→ Infrastructure)
- ❌ Implémentations de repositories (→ Infrastructure)

## 🔄 Flux de données

```
User
 ↓ (interaction)
Component
 ↓ (appelle)
Use Case
 ↓ (retourne Observable)
Component
 ↓ (met à jour)
Signal/State
 ↓ (réactive)
Template (affichage)
```

## 📖 Bonnes pratiques

### 1. Standalone Components avec Signals

```typescript
// user-profile.component.ts
@Component({
  selector: 'app-user-profile',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  template: `
    <div class="profile">
      @if (loading()) {
        <p>Chargement...</p>
      } @else if (error()) {
        <p class="error">{{ error() }}</p>
      } @else if (user()) {
        <h1>{{ user()!.name.getFullName() }}</h1>
        <p>{{ user()!.email.getValue() }}</p>
      }

      <button (click)="loadProfile()">Recharger</button>
    </div>
  `,
  styles: [`
    .profile { padding: 20px; }
    .error { color: red; }
  `]
})
export class UserProfileComponent implements OnInit {
  private getUserUseCase = inject(GetUserByIdUseCase);
  private route = inject(ActivatedRoute);

  // Signals
  user = signal<User | null>(null);
  loading = signal(false);
  error = signal<string | null>(null);

  ngOnInit(): void {
    const userId = this.route.snapshot.params['id'];
    this.loadProfile(userId);
  }

  loadProfile(userId?: string): void {
    this.loading.set(true);
    this.error.set(null);

    const id = userId || this.route.snapshot.params['id'];

    this.getUserUseCase.execute(id).subscribe({
      next: (user) => {
        this.user.set(user);
        this.loading.set(false);
      },
      error: (error) => {
        this.error.set(error.message);
        this.loading.set(false);
      }
    });
  }
}
```

### 2. Component avec formulaire

```typescript
@Component({
  selector: 'app-user-form',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <input formControlName="firstName" placeholder="Prénom">
      @if (form.get('firstName')?.invalid && form.get('firstName')?.touched) {
        <span class="error">Prénom requis</span>
      }

      <input formControlName="lastName" placeholder="Nom">
      <input formControlName="email" type="email" placeholder="Email">

      <button type="submit" [disabled]="form.invalid || submitting()">
        {{ submitting() ? 'Envoi...' : 'Enregistrer' }}
      </button>
    </form>
  `
})
export class UserFormComponent {
  private createUserUseCase = inject(CreateUserUseCase);
  private fb = inject(FormBuilder);

  submitting = signal(false);

  form = this.fb.group({
    firstName: ['', Validators.required],
    lastName: ['', Validators.required],
    email: ['', [Validators.required, Validators.email]],
    password: ['', [Validators.required, Validators.minLength(8)]]
  });

  onSubmit(): void {
    if (this.form.invalid) return;

    this.submitting.set(true);

    const dto: CreateUserDTO = this.form.value as CreateUserDTO;

    this.createUserUseCase.execute(dto).subscribe({
      next: (user) => {
        console.log('User created:', user);
        this.form.reset();
        this.submitting.set(false);
      },
      error: (error) => {
        console.error('Error:', error);
        this.submitting.set(false);
      }
    });
  }
}
```

### 3. Computed Signals

```typescript
@Component({
  selector: 'app-cart',
  standalone: true,
  template: `
    <div>
      <h2>Panier ({{ itemCount() }} articles)</h2>
      <p>Total : {{ total() }}</p>

      @for (item of items(); track item.id) {
        <div>{{ item.name }} - {{ item.price }}</div>
      }
    </div>
  `
})
export class CartComponent {
  items = signal<CartItem[]>([]);

  // Computed signals
  itemCount = computed(() => {
    return this.items().reduce((sum, item) => sum + item.quantity, 0);
  });

  total = computed(() => {
    return this.items()
      .reduce((sum, item) => sum + item.price * item.quantity, 0)
      .toFixed(2);
  });
}
```

### 4. Effect pour side-effects

```typescript
@Component({
  selector: 'app-search',
  standalone: true,
  template: `
    <input [value]="searchTerm()" (input)="onSearchChange($event)">

    @for (result of searchResults(); track result.id) {
      <div>{{ result.name }}</div>
    }
  `
})
export class SearchComponent {
  private searchUseCase = inject(SearchProductsUseCase);

  searchTerm = signal('');
  searchResults = signal<Product[]>([]);

  constructor() {
    // Effect pour recherche automatique
    effect(() => {
      const term = this.searchTerm();

      if (term.length >= 3) {
        this.searchUseCase.execute(term).subscribe(results => {
          this.searchResults.set(results);
        });
      } else {
        this.searchResults.set([]);
      }
    });
  }

  onSearchChange(event: Event): void {
    const input = event.target as HTMLInputElement;
    this.searchTerm.set(input.value);
  }
}
```

## 🎓 Points clés

1. **Standalone** : Components standalone Angular
2. **Signals** : State management réactif
3. **Inject()** : Injection de dépendances
4. **Use Cases** : Appeler les use cases
5. **@if/@for** : Nouvelle syntaxe de contrôle
6. **Computed** : Valeurs dérivées
7. **Effect** : Side-effects réactifs
8. **Reactive Forms** : Formulaires typés

---

**La Presentation rend votre application vivante ! 🎨**
