# 🧩 Components - Composants réutilisables

## 📋 Description

Les **Components** sont des composants réutilisables qui peuvent être utilisés dans plusieurs pages.

## ✅ Ce qu'on doit mettre

- ✅ Composants réutilisables
- ✅ Components avec @Input/@Output
- ✅ UI components (boutons, cards, modals...)
- ✅ Components métier réutilisables

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Pages complètes (→ pages/)
- ❌ Logique métier (→ Domain)
- ❌ Use cases (→ Application)

## 📊 Exemples

### Component simple

```typescript
// user-card.component.ts
@Component({
  selector: 'app-user-card',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="user-card">
      <h3>{{ user().name.getFullName() }}</h3>
      <p>{{ user().email.getValue() }}</p>
      <button (click)="onViewDetails()">Voir détails</button>
    </div>
  `,
  styles: [`
    .user-card {
      border: 1px solid #ccc;
      padding: 16px;
      border-radius: 8px;
    }
  `]
})
export class UserCardComponent {
  user = input.required<User>();
  viewDetails = output<string>();

  onViewDetails(): void {
    this.viewDetails.emit(this.user().id);
  }
}
```

### Component avec formulaire

```typescript
// order-form.component.ts
@Component({
  selector: 'app-order-form',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <!-- Form fields -->
      <button type="submit" [disabled]="form.invalid">
        Créer commande
      </button>
    </form>
  `
})
export class OrderFormComponent {
  orderCreated = output<Order>();

  private createOrderUseCase = inject(CreateOrderUseCase);
  private fb = inject(FormBuilder);

  form = this.fb.group({
    customerId: ['', Validators.required],
    items: this.fb.array([])
  });

  onSubmit(): void {
    if (this.form.invalid) return;

    const dto = this.form.value as CreateOrderDTO;

    this.createOrderUseCase.execute(dto).subscribe({
      next: (order) => this.orderCreated.emit(order)
    });
  }
}
```

---

**Les components sont vos briques réutilisables ! 🧩**
