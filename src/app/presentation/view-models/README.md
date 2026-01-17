# 🎭 View Models - Logique de présentation

## 📋 Description

Les **View Models** contiennent la logique de présentation qui ne doit pas être dans les components.

## ✅ Ce qu'on doit mettre

- ✅ Transformation de données pour l'affichage
- ✅ Formatage de valeurs
- ✅ Logique de validation UI
- ✅ Calculs pour l'interface

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Logique métier (→ Domain)
- ❌ Use cases (→ Application)
- ❌ Appels HTTP (→ Infrastructure)

## 📊 Exemples

### User View Model

```typescript
// user.view-model.ts
export class UserViewModel {
  constructor(private user: User) {}

  getDisplayName(): string {
    return this.user.name.getFullName();
  }

  getFormattedEmail(): string {
    return this.user.email.getValue().toLowerCase();
  }

  getAge(): number {
    return this.user.birthDate.calculateAge();
  }

  getStatusBadge(): { text: string; color: string } {
    return this.user.isActive
      ? { text: 'Actif', color: 'green' }
      : { text: 'Inactif', color: 'red' };
  }

  canEdit(): boolean {
    return this.user.isActive;
  }

  canDelete(): boolean {
    return this.user.canBeDeleted();
  }
}
```

### Order View Model

```typescript
// order.view-model.ts
export class OrderViewModel {
  constructor(private order: Order) {}

  getFormattedTotal(): string {
    const total = this.order.calculateTotal();
    return `${total.amount.toFixed(2)} ${total.currency}`;
  }

  getStatusLabel(): string {
    const labels: Record<OrderStatus, string> = {
      [OrderStatus.Pending]: 'En attente',
      [OrderStatus.Confirmed]: 'Confirmée',
      [OrderStatus.Shipped]: 'Expédiée',
      [OrderStatus.Delivered]: 'Livrée',
      [OrderStatus.Cancelled]: 'Annulée'
    };
    return labels[this.order.status];
  }

  getStatusColor(): string {
    const colors: Record<OrderStatus, string> = {
      [OrderStatus.Pending]: 'orange',
      [OrderStatus.Confirmed]: 'blue',
      [OrderStatus.Shipped]: 'purple',
      [OrderStatus.Delivered]: 'green',
      [OrderStatus.Cancelled]: 'red'
    };
    return colors[this.order.status];
  }

  canBeCancelled(): boolean {
    return this.order.canBeCancelled();
  }

  getFormattedDate(): string {
    return this.order.createdAt.toLocaleDateString('fr-FR');
  }
}
```

### Utilisation dans un component

```typescript
@Component({
  selector: 'app-order-list',
  template: `
    @for (vm of orderViewModels(); track vm.order.id) {
      <div class="order-card">
        <h3>Commande {{ vm.order.id }}</h3>
        <p>Total : {{ vm.getFormattedTotal() }}</p>
        <span [style.color]="vm.getStatusColor()">
          {{ vm.getStatusLabel() }}
        </span>
        @if (vm.canBeCancelled()) {
          <button (click)="cancelOrder(vm.order)">Annuler</button>
        }
      </div>
    }
  `
})
export class OrderListComponent {
  orders = signal<Order[]>([]);

  // Computed view models
  orderViewModels = computed(() =>
    this.orders().map(order => new OrderViewModel(order))
  );

  cancelOrder(order: Order): void {
    // Logic...
  }
}
```

---

**Les view models séparent la logique de présentation des components ! 🎭**
