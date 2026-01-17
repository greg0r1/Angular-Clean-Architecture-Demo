# 📄 Pages - Composants de niveau page

## 📋 Description

Les **Pages** sont des composants de niveau route qui représentent des vues complètes de l'application.

## ✅ Ce qu'on doit mettre

- ✅ Composants associés à des routes
- ✅ Layout des pages
- ✅ Orchestration de components enfants
- ✅ Gestion du state de la page
- ✅ Appels aux use cases

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Logique métier (→ Domain)
- ❌ Composants réutilisables (→ components/)
- ❌ Appels HTTP directs (→ Infrastructure)

## 📊 Exemple

```typescript
// pages/user-detail/user-detail.page.ts
@Component({
  selector: 'app-user-detail-page',
  standalone: true,
  imports: [
    CommonModule,
    UserProfileComponent,
    UserOrdersComponent,
    LoaderComponent
  ],
  template: `
    <div class="user-detail-page">
      <h1>Détail utilisateur</h1>

      @if (loading()) {
        <app-loader />
      } @else if (error()) {
        <div class="error">{{ error() }}</div>
      } @else if (user()) {
        <app-user-profile [user]="user()!" />
        <app-user-orders [userId]="user()!.id" />
      }
    </div>
  `,
  styles: [`
    .user-detail-page {
      max-width: 1200px;
      margin: 0 auto;
      padding: 20px;
    }
  `]
})
export class UserDetailPage implements OnInit {
  private route = inject(ActivatedRoute);
  private router = inject(Router);
  private getUserUseCase = inject(GetUserByIdUseCase);

  user = signal<User | null>(null);
  loading = signal(false);
  error = signal<string | null>(null);

  ngOnInit(): void {
    const userId = this.route.snapshot.params['id'];
    this.loadUser(userId);
  }

  private loadUser(id: string): void {
    this.loading.set(true);

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

// Routes
export const routes: Routes = [
  {
    path: 'users/:id',
    component: UserDetailPage
  }
];
```

---

**Les pages orchestrent vos composants pour créer des vues complètes ! 📄**
