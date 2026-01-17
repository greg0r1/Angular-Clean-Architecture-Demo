# 📱 App - Racine de l'application Angular

## 📋 Description

Le dossier **app** est la racine de l'application Angular, contenant toutes les couches de la Clean Architecture.

## 🏗️ Structure complète

```
app/
├── core/                    # Cœur de l'application
│   ├── domain/             # Logique métier pure
│   │   ├── entities/       # Entités avec identité
│   │   ├── repositories/   # Interfaces de persistence
│   │   └── value-objects/  # Objets immuables
│   └── application/        # Cas d'utilisation
│       ├── use-cases/      # Scénarios métier
│       └── ports/          # Interfaces services externes
├── infrastructure/         # Implémentations techniques
│   ├── repositories/       # Implémentation repositories
│   ├── http/              # Services HTTP, intercepteurs
│   ├── storage/           # LocalStorage, etc.
│   └── mappers/           # DTO ↔ Domain
├── presentation/          # Interface utilisateur
│   ├── pages/            # Composants de niveau page
│   ├── components/       # Composants réutilisables
│   ├── state/            # Gestion d'état
│   └── view-models/      # Logique de présentation
└── shared/               # Code partagé
    ├── utils/            # Fonctions utilitaires
    └── types/            # Types TypeScript
```

## 🔄 Flux de dépendances

```
┌─────────────────────────────────────────┐
│          PRESENTATION                   │
│     (UI, Components, Pages)             │
└─────────────────────────────────────────┘
              ↓ dépend de
┌─────────────────────────────────────────┐
│         INFRASTRUCTURE                  │
│  (Repositories, HTTP, Storage)          │
└─────────────────────────────────────────┘
              ↓ implémente
┌─────────────────────────────────────────┐
│          APPLICATION                    │
│      (Use Cases, Ports)                 │
└─────────────────────────────────────────┘
              ↓ utilise
┌─────────────────────────────────────────┐
│            DOMAIN                       │
│   (Entities, Value Objects)             │
│         (AUCUNE DÉPENDANCE)             │
└─────────────────────────────────────────┘
```

## 📖 Règles fondamentales

### 1. Règle de dépendance
**Les dépendances pointent toujours vers l'intérieur (vers le Domain)**

- ✅ Presentation → Application → Domain
- ✅ Infrastructure → Domain
- ❌ Domain → Infrastructure (INTERDIT !)
- ❌ Domain → Presentation (INTERDIT !)

### 2. Le Domain ne dépend de rien

```typescript
// ✅ CORRECT - Domain pur
export class User {
  constructor(
    public readonly id: string,
    public readonly email: Email
  ) {}
}

// ❌ INCORRECT - Dépendance externe
import { HttpClient } from '@angular/common/http'; // ❌ NON !

export class User {
  constructor(private http: HttpClient) {} // ❌ NON !
}
```

### 3. Inversion de dépendances

L'Infrastructure implémente les interfaces du Domain.

```typescript
// Domain définit l'interface
export interface UserRepository {
  findById(id: string): Observable<User>;
}

// Infrastructure implémente
@Injectable()
export class UserRepositoryImpl implements UserRepository {
  constructor(private http: HttpClient) {}

  findById(id: string): Observable<User> {
    return this.http.get<UserDTO>(`/api/users/${id}`).pipe(
      map(dto => UserMapper.toDomain(dto))
    );
  }
}
```

## 🎯 Workflow de développement

### Ajouter une nouvelle fonctionnalité

1. **Domain** : Créer l'entité et le repository interface
2. **Application** : Créer le use case
3. **Infrastructure** : Implémenter le repository
4. **Presentation** : Créer le component qui utilise le use case

### Exemple : Ajouter "Gestion de produits"

```typescript
// 1. DOMAIN - entities/product.entity.ts
export class Product {
  constructor(
    public readonly id: string,
    public readonly name: string,
    public readonly price: Money
  ) {}
}

// 1. DOMAIN - repositories/product.repository.ts
export interface ProductRepository {
  findAll(): Observable<Product[]>;
  save(product: Product): Observable<Product>;
}

// 2. APPLICATION - use-cases/get-all-products.use-case.ts
@Injectable({ providedIn: 'root' })
export class GetAllProductsUseCase {
  constructor(private productRepository: ProductRepository) {}

  execute(): Observable<Product[]> {
    return this.productRepository.findAll();
  }
}

// 3. INFRASTRUCTURE - repositories/product-repository.impl.ts
@Injectable({ providedIn: 'root' })
export class ProductRepositoryImpl implements ProductRepository {
  constructor(private http: HttpClient) {}

  findAll(): Observable<Product[]> {
    return this.http.get<ProductDTO[]>('/api/products').pipe(
      map(dtos => dtos.map(ProductMapper.toDomain))
    );
  }
}

// 4. PRESENTATION - pages/products/products.page.ts
@Component({
  selector: 'app-products-page',
  standalone: true,
  template: `
    @for (product of products(); track product.id) {
      <div>{{ product.name }} - {{ product.price.format() }}</div>
    }
  `
})
export class ProductsPage implements OnInit {
  private getAllProductsUseCase = inject(GetAllProductsUseCase);

  products = signal<Product[]>([]);

  ngOnInit(): void {
    this.getAllProductsUseCase.execute().subscribe(products => {
      this.products.set(products);
    });
  }
}
```

## ⚙️ Configuration

### app.config.ts - Injection de dépendances

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient, withInterceptors } from '@angular/common/http';

// Repositories
import { UserRepository } from './core/domain/repositories/user.repository';
import { UserRepositoryImpl } from './infrastructure/repositories/user-repository.impl';

// Intercepteurs
import { authInterceptor } from './infrastructure/http/auth.interceptor';
import { errorInterceptor } from './infrastructure/http/error.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(
      withInterceptors([authInterceptor, errorInterceptor])
    ),

    // Binding repositories
    {
      provide: UserRepository,
      useClass: UserRepositoryImpl
    },
    {
      provide: ProductRepository,
      useClass: ProductRepositoryImpl
    }
  ]
};
```

## 🎓 Principes clés

1. **Separation of Concerns** : Chaque couche a sa responsabilité
2. **Dependency Inversion** : Dépendre des abstractions
3. **Single Responsibility** : Une classe = une responsabilité
4. **Open/Closed** : Ouvert à l'extension, fermé à la modification
5. **Interface Segregation** : Interfaces spécifiques
6. **Don't Repeat Yourself** : Éviter la duplication
7. **Keep It Simple** : Simplicité avant tout

## 📚 Navigation rapide

- **[Domain](./core/domain/README.md)** : Logique métier pure
- **[Application](./core/application/README.md)** : Cas d'utilisation
- **[Infrastructure](./infrastructure/README.md)** : Implémentations techniques
- **[Presentation](./presentation/README.md)** : Interface utilisateur
- **[Shared](./shared/README.md)** : Code partagé

---

**Bienvenue dans votre application Angular Clean Architecture ! 📱**
