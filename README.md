# 🏗️ Angular Clean Architecture - Projet de Démonstration

Un projet de démonstration complet illustrant l'implémentation de la **Clean Architecture** dans une application Angular moderne, avec des composants standalone et les Signals Angular.

## 📋 Table des matières

- [Vue d'ensemble](#-vue-densemble)
- [Architecture](#-architecture)
- [Les couches](#-les-couches)
- [Diagramme de dépendances](#-diagramme-de-dépendances)
- [Structure du projet](#-structure-du-projet)
- [Principes appliqués](#-principes-appliqués)
- [Avantages de cette architecture](#-avantages-de-cette-architecture)
- [Comment démarrer](#-comment-démarrer)
- [Bonnes pratiques](#-bonnes-pratiques)

## 🎯 Vue d'ensemble

Ce repository démontre comment structurer une application Angular en respectant les principes de la **Clean Architecture** d'Uncle Bob (Robert C. Martin). L'objectif est de créer une application maintenable, testable et indépendante des frameworks.

### Principes fondamentaux

- **Indépendance du framework** : La logique métier ne dépend pas d'Angular
- **Testabilité** : Chaque couche peut être testée indépendamment
- **Indépendance de l'UI** : L'interface peut changer sans affecter la logique
- **Indépendance de la base de données** : La source de données peut être remplacée
- **Indépendance de tout agent externe** : La logique métier ne connaît rien du monde extérieur

## 🏛️ Architecture

L'architecture est organisée en **4 couches principales**, chacune ayant des responsabilités clairement définies :

```
┌─────────────────────────────────────────────────────────┐
│                    PRESENTATION                          │
│  (Pages, Components, ViewModels, State Management)      │
└─────────────────────────────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────┐
│                   INFRASTRUCTURE                         │
│     (API, Repositories Impl, Storage, Mappers)          │
└─────────────────────────────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────┐
│                    APPLICATION                           │
│          (Use Cases, Ports, Interfaces)                 │
└─────────────────────────────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────┐
│                      DOMAIN                              │
│      (Entities, Value Objects, Interfaces)              │
└─────────────────────────────────────────────────────────┘
```

## 📚 Les couches

### 1. Domain (Cœur métier)
**Le centre de l'application - Aucune dépendance externe**

- **Entities** : Objets métier avec identité et cycle de vie
- **Value Objects** : Objets immuables définis par leurs valeurs
- **Repository Interfaces** : Contrats pour l'accès aux données

```typescript
// Exemple : User Entity
export class User {
  constructor(
    public readonly id: string,
    public readonly email: Email,
    public readonly profile: UserProfile
  ) {}
}
```

### 2. Application (Cas d'utilisation)
**Orchestration de la logique métier**

- **Use Cases** : Scénarios d'utilisation de l'application
- **Ports** : Interfaces pour les adaptateurs externes

```typescript
// Exemple : Use Case
export class GetUserByIdUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(id: string): Observable<User> {
    return this.userRepository.findById(id);
  }
}
```

### 3. Infrastructure (Détails techniques)
**Implémentation concrète des interfaces**

- **Repositories** : Implémentation des interfaces du domain
- **HTTP** : Services API et intercepteurs
- **Storage** : LocalStorage, SessionStorage, IndexedDB
- **Mappers** : Transformation entre DTOs et Entities

```typescript
// Exemple : Repository Implementation
@Injectable()
export class UserRepositoryImpl implements UserRepository {
  constructor(private http: HttpClient) {}

  findById(id: string): Observable<User> {
    return this.http.get<UserDTO>(`/users/${id}`)
      .pipe(map(dto => UserMapper.toDomain(dto)));
  }
}
```

### 4. Presentation (Interface utilisateur)
**Interaction avec l'utilisateur**

- **Pages** : Composants de niveau page (routes)
- **Components** : Composants réutilisables
- **ViewModels** : Logique de présentation
- **State** : Gestion d'état (Signals, Stores)

```typescript
// Exemple : Component avec Signals
@Component({
  selector: 'app-user-profile',
  standalone: true,
  template: `...`
})
export class UserProfileComponent {
  private getUserUseCase = inject(GetUserByIdUseCase);

  user = signal<User | null>(null);
  loading = signal(false);

  loadUser(id: string): void {
    this.loading.set(true);
    this.getUserUseCase.execute(id).subscribe(user => {
      this.user.set(user);
      this.loading.set(false);
    });
  }
}
```

## 🔄 Diagramme de dépendances

### Règle de dépendance
**Les dépendances pointent toujours vers l'intérieur (vers le Domain)**

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  ┌──────────────┐         ┌──────────────┐               │
│  │ Presentation │────────▶│Infrastructure│               │
│  └──────────────┘         └──────────────┘               │
│         │                        │                        │
│         │                        │                        │
│         ▼                        ▼                        │
│  ┌──────────────────────────────────┐                    │
│  │        Application               │                    │
│  │    (Use Cases, Ports)            │                    │
│  └──────────────────────────────────┘                    │
│                   │                                       │
│                   │                                       │
│                   ▼                                       │
│         ┌─────────────────┐                              │
│         │     Domain      │                              │
│         │  (NO DEPS !)    │                              │
│         └─────────────────┘                              │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Inversion de dépendance (DIP)

```typescript
// ✅ CORRECT : Infrastructure dépend de Domain
// Domain définit l'interface
export interface UserRepository {
  findById(id: string): Observable<User>;
}

// Infrastructure implémente l'interface
export class UserRepositoryImpl implements UserRepository {
  findById(id: string): Observable<User> { /* ... */ }
}

// ❌ INCORRECT : Domain ne doit JAMAIS importer Infrastructure
// Domain
import { HttpClient } from '@angular/common/http'; // ❌ NON !
```

## 📁 Structure du projet

```
angular-clean-architecture-demo/
├── src/
│   └── app/
│       ├── core/                    # Couches Domain + Application
│       │   ├── domain/              # Logique métier pure
│       │   │   ├── entities/        # Entités métier
│       │   │   ├── repositories/    # Interfaces repositories
│       │   │   └── value-objects/   # Objets valeur
│       │   └── application/         # Cas d'utilisation
│       │       ├── use-cases/       # Scénarios métier
│       │       └── ports/           # Interfaces pour adapters
│       ├── infrastructure/          # Implémentations techniques
│       │   ├── repositories/        # Implémentation repositories
│       │   ├── http/                # Services HTTP
│       │   ├── storage/             # Stockage local
│       │   └── mappers/             # Transformations DTO ↔ Domain
│       ├── presentation/            # Interface utilisateur
│       │   ├── pages/               # Pages (routed components)
│       │   ├── components/          # Composants réutilisables
│       │   ├── state/               # Gestion d'état
│       │   └── view-models/         # Logique de présentation
│       └── shared/                  # Code partagé
│           ├── utils/               # Utilitaires
│           └── types/               # Types TypeScript
```

## ⚙️ Principes appliqués

### SOLID

- **S**ingle Responsibility : Une classe = une responsabilité
- **O**pen/Closed : Ouvert à l'extension, fermé à la modification
- **L**iskov Substitution : Les sous-types doivent être substituables
- **I**nterface Segregation : Interfaces spécifiques plutôt que générales
- **D**ependency Inversion : Dépendre des abstractions, pas des implémentations

### Autres principes

- **Separation of Concerns** : Chaque couche a sa responsabilité
- **DRY** (Don't Repeat Yourself) : Éviter la duplication
- **KISS** (Keep It Simple, Stupid) : Simplicité avant tout
- **YAGNI** (You Aren't Gonna Need It) : N'implémentez que ce qui est nécessaire

## ✨ Avantages de cette architecture

### 1. **Maintenabilité**
- Code organisé et prévisible
- Facile à comprendre pour les nouveaux développeurs
- Changements localisés dans une seule couche

### 2. **Testabilité**
- Logique métier testable sans Angular
- Mocking facile grâce aux interfaces
- Tests unitaires rapides (pas de DOM)

```typescript
// Test d'un Use Case (pas besoin de TestBed !)
describe('GetUserByIdUseCase', () => {
  it('should return user', (done) => {
    const mockRepo: UserRepository = {
      findById: (id) => of(new User(id, /* ... */))
    };

    const useCase = new GetUserByIdUseCase(mockRepo);
    useCase.execute('123').subscribe(user => {
      expect(user.id).toBe('123');
      done();
    });
  });
});
```

### 3. **Indépendance du framework**
- La logique métier ne dépend pas d'Angular
- Migration vers un autre framework facilitée
- Réutilisation du code métier possible

### 4. **Scalabilité**
- Structure claire pour les grandes équipes
- Ajout de fonctionnalités sans régression
- Parallélisation du développement possible

### 5. **Flexibilité**
- Changement de source de données facile
- Remplacement de composants UI sans impact
- Adaptation aux nouvelles exigences

### 6. **Qualité du code**
- Respect des principes SOLID
- Couplage faible, cohésion forte
- Code plus lisible et documenté

## 🚀 Comment démarrer

### 1. Comprendre les couches

Commencez par explorer les README.md de chaque dossier dans cet ordre :

1. `src/app/core/domain/` - Le cœur métier
2. `src/app/core/application/` - Les cas d'utilisation
3. `src/app/infrastructure/` - Les implémentations
4. `src/app/presentation/` - L'interface utilisateur

### 2. Workflow de développement

Pour ajouter une nouvelle fonctionnalité :

```
1. Domain Layer
   └─> Créer l'entité ou value object
   └─> Définir l'interface du repository

2. Application Layer
   └─> Créer le use case
   └─> Définir les ports si nécessaire

3. Infrastructure Layer
   └─> Implémenter le repository
   └─> Créer les mappers DTO ↔ Domain

4. Presentation Layer
   └─> Créer le component/page
   └─> Utiliser le use case
   └─> Gérer l'état avec Signals
```

### 3. Exemple complet

Imaginons l'ajout d'une fonctionnalité "Gestion de produits" :

#### Étape 1 : Domain
```typescript
// entities/product.entity.ts
export class Product {
  constructor(
    public readonly id: string,
    public readonly name: string,
    public readonly price: Money
  ) {}
}

// repositories/product.repository.ts
export interface ProductRepository {
  findAll(): Observable<Product[]>;
  findById(id: string): Observable<Product>;
  save(product: Product): Observable<Product>;
}
```

#### Étape 2 : Application
```typescript
// use-cases/get-all-products.use-case.ts
export class GetAllProductsUseCase {
  constructor(private productRepository: ProductRepository) {}

  execute(): Observable<Product[]> {
    return this.productRepository.findAll();
  }
}
```

#### Étape 3 : Infrastructure
```typescript
// repositories/product-repository.impl.ts
@Injectable()
export class ProductRepositoryImpl implements ProductRepository {
  constructor(private http: HttpClient) {}

  findAll(): Observable<Product[]> {
    return this.http.get<ProductDTO[]>('/api/products')
      .pipe(map(dtos => dtos.map(ProductMapper.toDomain)));
  }
}
```

#### Étape 4 : Presentation
```typescript
// pages/products/products.component.ts
@Component({
  selector: 'app-products',
  standalone: true,
  template: `
    <h1>Produits</h1>
    @if (loading()) {
      <p>Chargement...</p>
    } @else {
      @for (product of products(); track product.id) {
        <div>{{ product.name }} - {{ product.price }}</div>
      }
    }
  `
})
export class ProductsComponent implements OnInit {
  private getAllProductsUseCase = inject(GetAllProductsUseCase);

  products = signal<Product[]>([]);
  loading = signal(false);

  ngOnInit(): void {
    this.loadProducts();
  }

  loadProducts(): void {
    this.loading.set(true);
    this.getAllProductsUseCase.execute().subscribe({
      next: (products) => {
        this.products.set(products);
        this.loading.set(false);
      }
    });
  }
}
```

### 4. Configuration de l'injection de dépendances

```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    // Use Cases
    GetAllProductsUseCase,

    // Repositories (Binding interface → implementation)
    {
      provide: ProductRepository,
      useClass: ProductRepositoryImpl
    }
  ]
};
```

## 📖 Bonnes pratiques

### 1. Ne jamais violer la règle de dépendance
❌ Domain/Application NE DOIT JAMAIS importer Infrastructure/Presentation

### 2. Utiliser l'injection de dépendances
✅ Toujours injecter les dépendances via le constructeur

### 3. Privilégier l'immutabilité
✅ Utiliser `readonly`, Value Objects, Signals

### 4. Tester chaque couche indépendamment
✅ Tests unitaires pour Domain/Application sans TestBed
✅ Tests d'intégration pour Infrastructure
✅ Tests de composants pour Presentation

### 5. Garder les Use Cases simples
✅ Un Use Case = Une action métier
✅ Pas de logique technique dans les Use Cases

### 6. Utiliser les Mappers
✅ Toujours transformer les DTOs en entités Domain
✅ Ne jamais exposer les DTOs aux couches supérieures

### 7. Standalone Components
✅ Utiliser les standalone components Angular
✅ Import explicite des dépendances

### 8. Signals pour l'état
✅ Préférer les Signals aux Subjects/BehaviorSubject
✅ État immutable et réactif

## 📚 Ressources

- [Clean Architecture - Uncle Bob](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Angular Documentation](https://angular.dev)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
- [Domain-Driven Design](https://martinfowler.com/tags/domain%20driven%20design.html)

## 📝 Licence

Ce projet est un exemple éducatif libre d'utilisation.

---

**Bonne exploration de la Clean Architecture avec Angular ! 🚀**
