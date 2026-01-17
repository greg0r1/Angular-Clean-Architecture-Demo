# 📦 Repositories - Interfaces de persistence

## 📋 Description

Les **Repository Interfaces** définissent les **contrats** pour l'accès et la persistence des entités, **sans spécifier comment** ces opérations sont implémentées.

> **Principe clé** : Le Domain définit **QUOI** (l'interface), l'Infrastructure définit **COMMENT** (l'implémentation).

## 🎯 Responsabilité

Les repositories agissent comme une **abstraction** entre le domain et la couche de persistence, permettant :
- De manipuler des **collections d'entités** comme si c'était de la mémoire
- D'isoler le domain de la technologie de persistence
- De faciliter les tests (mocking facile)

## ✅ Ce qu'on doit mettre

### Dans ce dossier (Domain)
- ✅ **Interfaces** de repositories uniquement
- ✅ Méthodes retournant des Observables (RxJS)
- ✅ Méthodes retournant des Entités du domain
- ✅ Noms métier (pas techniques)

### Exemples
```typescript
// ✅ Interfaces de repositories
export interface UserRepository { }
export interface OrderRepository { }
export interface ProductRepository { }
export interface CustomerRepository { }
export interface InvoiceRepository { }
```

## ❌ Ce qu'on ne doit PAS mettre

### Interdictions absolues
- ❌ Implémentations concrètes (→ Infrastructure)
- ❌ HttpClient ou appels HTTP
- ❌ Code SQL ou NoSQL
- ❌ Détails de persistence (LocalStorage, IndexedDB...)
- ❌ Annotations Angular (`@Injectable`)
- ❌ Mappers ou DTOs
- ❌ Logique métier (→ Entities)

```typescript
// ❌ INCORRECT - Implémentation dans Domain
export class UserRepositoryImpl implements UserRepository {
  constructor(private http: HttpClient) {} // ❌ NON !

  findById(id: string): Observable<User> {
    return this.http.get(`/users/${id}`); // ❌ NON !
  }
}

// ✅ CORRECT - Seulement l'interface dans Domain
export interface UserRepository {
  findById(id: string): Observable<User>;
}
```

## 🏗️ Structure d'une interface de repository

### Template de base

```typescript
import { Observable } from 'rxjs';
import { Entity } from '../entities/entity';

export interface EntityRepository {
  // Lecture
  findById(id: string): Observable<Entity | null>;
  findAll(): Observable<Entity[]>;

  // Écriture
  save(entity: Entity): Observable<Entity>;
  delete(id: string): Observable<void>;
}
```

### Convention de nommage

```typescript
// ✅ CORRECT - Noms clairs et métier
export interface UserRepository {
  findById(id: string): Observable<User | null>;
  findByEmail(email: Email): Observable<User | null>;
  findActiveUsers(): Observable<User[]>;
  save(user: User): Observable<User>;
  delete(id: string): Observable<void>;
}

// ❌ INCORRECT - Noms techniques
export interface UserRepository {
  get(id: string): Observable<User>; // ❌ Trop vague
  query(sql: string): Observable<User[]>; // ❌ Détail d'implémentation
  insert(user: User): Observable<User>; // ❌ Vocabulaire SQL
  executeStoredProcedure(name: string): Observable<any>; // ❌ Détail technique
}
```

## 📖 Principes SOLID

### Single Responsibility (SRP)

Un repository = Une entité principale

```typescript
// ✅ CORRECT - Un repository par entité
export interface UserRepository {
  findById(id: string): Observable<User | null>;
  findByEmail(email: Email): Observable<User | null>;
  save(user: User): Observable<User>;
}

export interface OrderRepository {
  findById(id: string): Observable<Order | null>;
  findByCustomerId(customerId: string): Observable<Order[]>;
  save(order: Order): Observable<Order>;
}

// ❌ INCORRECT - Trop de responsabilités
export interface ApplicationRepository {
  findUser(id: string): Observable<User>;
  findOrder(id: string): Observable<Order>;
  findProduct(id: string): Observable<Product>;
  saveUser(user: User): Observable<User>;
  saveOrder(order: Order): Observable<Order>;
  // ❌ Trop d'entités différentes
}
```

### Interface Segregation (ISP)

Interfaces spécifiques plutôt que générales

```typescript
// ✅ CORRECT - Interfaces ségrégées
export interface Readable<T> {
  findById(id: string): Observable<T | null>;
  findAll(): Observable<T[]>;
}

export interface Writable<T> {
  save(entity: T): Observable<T>;
  delete(id: string): Observable<void>;
}

export interface Searchable<T> {
  search(criteria: SearchCriteria): Observable<T[]>;
}

// Composition selon les besoins
export interface UserRepository extends Readable<User>, Writable<User> {}

export interface ProductRepository extends
  Readable<Product>,
  Writable<Product>,
  Searchable<Product> {
  // Méthodes spécifiques aux produits
  findByCategory(category: string): Observable<Product[]>;
  findInStock(): Observable<Product[]>;
}

// Un use case peut dépendre uniquement de ce dont il a besoin
export class GetUserUseCase {
  constructor(private repository: Readable<User>) {} // ✅ Seulement lecture
}

// ❌ INCORRECT - Interface monolithique
export interface GenericRepository<T> {
  findById(id: string): Observable<T>;
  findAll(): Observable<T[]>;
  findOne(criteria: any): Observable<T>;
  findMany(criteria: any): Observable<T[]>;
  save(entity: T): Observable<T>;
  saveMany(entities: T[]): Observable<T[]>;
  update(id: string, data: Partial<T>): Observable<T>;
  updateMany(criteria: any, data: any): Observable<void>;
  delete(id: string): Observable<void>;
  deleteMany(criteria: any): Observable<void>;
  count(criteria?: any): Observable<number>;
  exists(id: string): Observable<boolean>;
  // ❌ Trop de méthodes, tous les clients doivent dépendre de toutes
}
```

### Dependency Inversion (DIP)

L'interface est dans le Domain, l'implémentation dans l'Infrastructure

```typescript
// ========== DOMAIN LAYER ==========
// domain/repositories/user.repository.ts
export interface UserRepository {
  findById(id: string): Observable<User | null>;
  save(user: User): Observable<User>;
}

// domain/entities/user.entity.ts
export class User {
  // Entité pure
}

// ========== APPLICATION LAYER ==========
// application/use-cases/get-user.use-case.ts
export class GetUserUseCase {
  constructor(private userRepository: UserRepository) {} // ✅ Dépend de l'interface

  execute(id: string): Observable<User> {
    return this.userRepository.findById(id).pipe(
      map(user => {
        if (!user) throw new Error('User not found');
        return user;
      })
    );
  }
}

// ========== INFRASTRUCTURE LAYER ==========
// infrastructure/repositories/user-repository.impl.ts
@Injectable()
export class UserRepositoryImpl implements UserRepository {
  constructor(private http: HttpClient) {} // Détails techniques ici

  findById(id: string): Observable<User | null> {
    return this.http.get<UserDTO>(`/api/users/${id}`).pipe(
      map(dto => UserMapper.toDomain(dto)),
      catchError(() => of(null))
    );
  }

  save(user: User): Observable<User> {
    const dto = UserMapper.toDTO(user);
    return this.http.post<UserDTO>('/api/users', dto).pipe(
      map(dto => UserMapper.toDomain(dto))
    );
  }
}

// ========== CONFIGURATION ==========
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    GetUserUseCase,
    {
      provide: UserRepository, // ✅ Interface du Domain
      useClass: UserRepositoryImpl // ✅ Implémentation de l'Infrastructure
    }
  ]
};
```

## 📚 Bonnes pratiques

### 1. Retourner des Observables

Utilisez RxJS Observable pour la gestion asynchrone.

```typescript
// ✅ CORRECT - Observables
import { Observable } from 'rxjs';

export interface ProductRepository {
  findById(id: string): Observable<Product | null>;
  findAll(): Observable<Product[]>;
  save(product: Product): Observable<Product>;
}

// ❌ INCORRECT - Promises (pas idiomatique Angular)
export interface ProductRepository {
  findById(id: string): Promise<Product | null>; // ❌ Préférer Observable
}

// ❌ INCORRECT - Synchrone (pas réaliste)
export interface ProductRepository {
  findById(id: string): Product | null; // ❌ Les opérations sont async
}
```

### 2. Retourner null pour les éléments non trouvés

```typescript
// ✅ CORRECT - null pour "non trouvé"
export interface UserRepository {
  findById(id: string): Observable<User | null>;
  findByEmail(email: Email): Observable<User | null>;
}

// Utilisation dans use case
export class GetUserUseCase {
  execute(id: string): Observable<User> {
    return this.userRepository.findById(id).pipe(
      map(user => {
        if (user === null) {
          throw new UserNotFoundError(id);
        }
        return user;
      })
    );
  }
}

// ❌ INCORRECT - Lancer une exception dans le repository
export interface UserRepository {
  findById(id: string): Observable<User>; // Que faire si non trouvé ?
}

// L'implémentation ne devrait pas lancer d'exception pour "non trouvé"
```

### 3. Méthodes de recherche spécifiques

Créez des méthodes avec des noms métier clairs.

```typescript
// ✅ CORRECT - Méthodes spécifiques et métier
export interface OrderRepository {
  findById(id: string): Observable<Order | null>;
  findByCustomerId(customerId: string): Observable<Order[]>;
  findPendingOrders(): Observable<Order[]>;
  findOrdersInDateRange(start: Date, end: Date): Observable<Order[]>;
  findRecentOrders(limit: number): Observable<Order[]>;
  save(order: Order): Observable<Order>;
}

// ❌ INCORRECT - Méthode générique avec critères vagues
export interface OrderRepository {
  find(criteria: any): Observable<Order[]>; // ❌ Trop vague
  query(params: QueryParams): Observable<Order[]>; // ❌ Pas clair
}
```

### 4. save() pour create et update

Utilisez une seule méthode `save()` plutôt que create/update séparés.

```typescript
// ✅ CORRECT - Méthode save unique
export interface UserRepository {
  save(user: User): Observable<User>;
  // L'implémentation détermine si c'est un create ou update
}

// Utilisation
const newUser = new User(generateId(), email, profile);
const savedUser = await userRepository.save(newUser); // Create

const updatedUser = existingUser.updateProfile(newProfile);
const savedUpdatedUser = await userRepository.save(updatedUser); // Update

// ❌ INCORRECT - create et update séparés
export interface UserRepository {
  create(user: User): Observable<User>;
  update(user: User): Observable<User>;
  // ❌ Comment savoir quelle méthode utiliser ?
  // ❌ L'entité doit-elle savoir si elle est nouvelle ou existante ?
}
```

### 5. Éviter les méthodes de requête complexes

Ne pas exposer la complexité de la requête dans l'interface.

```typescript
// ✅ CORRECT - Interface simple et claire
export interface ProductRepository {
  findById(id: string): Observable<Product | null>;
  findByCategory(category: string): Observable<Product[]>;
  findInPriceRange(min: Money, max: Money): Observable<Product[]>;
  search(criteria: ProductSearchCriteria): Observable<Product[]>;
}

export interface ProductSearchCriteria {
  readonly name?: string;
  readonly category?: string;
  readonly minPrice?: Money;
  readonly maxPrice?: Money;
  readonly inStock?: boolean;
}

// ❌ INCORRECT - Exposition de détails SQL
export interface ProductRepository {
  executeQuery(sql: string, params: any[]): Observable<Product[]>; // ❌ NON !
  findByJoin(table: string, condition: string): Observable<Product[]>; // ❌ NON !
  findWithRawQuery(query: QueryBuilder): Observable<Product[]>; // ❌ NON !
}
```

### 6. Pas de logique métier dans les repositories

La logique métier reste dans les entités et use cases.

```typescript
// ✅ CORRECT - Repository simple, logique dans use case
export interface OrderRepository {
  findById(id: string): Observable<Order | null>;
  save(order: Order): Observable<Order>;
}

export class CancelOrderUseCase {
  constructor(private orderRepository: OrderRepository) {}

  execute(orderId: string): Observable<Order> {
    return this.orderRepository.findById(orderId).pipe(
      map(order => {
        if (!order) throw new Error('Order not found');
        return order.cancel(); // ✅ Logique dans l'entité
      }),
      switchMap(cancelledOrder =>
        this.orderRepository.save(cancelledOrder)
      )
    );
  }
}

// ❌ INCORRECT - Logique métier dans le repository
export interface OrderRepository {
  cancelOrder(orderId: string): Observable<Order>; // ❌ Logique métier
  validateOrder(orderId: string): Observable<boolean>; // ❌ Logique métier
  calculateOrderTotal(orderId: string): Observable<number>; // ❌ Logique métier
}
```

## 📊 Exemples complets

### Exemple 1 : User Repository

```typescript
import { Observable } from 'rxjs';
import { User } from '../entities/user.entity';
import { Email } from '../value-objects/email';

export interface UserRepository {
  // Lecture par ID
  findById(id: string): Observable<User | null>;

  // Lecture par critères uniques
  findByEmail(email: Email): Observable<User | null>;

  // Lecture de collections
  findAll(): Observable<User[]>;
  findActiveUsers(): Observable<User[]>;
  findByRole(role: string): Observable<User[]>;

  // Recherche
  search(criteria: UserSearchCriteria): Observable<User[]>;

  // Écriture
  save(user: User): Observable<User>;
  delete(id: string): Observable<void>;

  // Vérifications
  existsByEmail(email: Email): Observable<boolean>;
}

export interface UserSearchCriteria {
  readonly name?: string;
  readonly email?: string;
  readonly role?: string;
  readonly isActive?: boolean;
  readonly createdAfter?: Date;
}
```

### Exemple 2 : Order Repository

```typescript
import { Observable } from 'rxjs';
import { Order } from '../entities/order.entity';
import { OrderStatus } from '../entities/order.entity';
import { DateRange } from '../value-objects/date-range';

export interface OrderRepository {
  // Lecture
  findById(id: string): Observable<Order | null>;
  findAll(): Observable<Order[]>;

  // Recherche par relations
  findByCustomerId(customerId: string): Observable<Order[]>;

  // Recherche par état
  findByStatus(status: OrderStatus): Observable<Order[]>;
  findPendingOrders(): Observable<Order[]>;

  // Recherche par date
  findByDateRange(range: DateRange): Observable<Order[]>;
  findRecentOrders(limit: number): Observable<Order[]>;

  // Statistiques simples
  countByCustomerId(customerId: string): Observable<number>;

  // Écriture
  save(order: Order): Observable<Order>;
  delete(id: string): Observable<void>;
}
```

### Exemple 3 : Product Repository avec composition

```typescript
import { Observable } from 'rxjs';
import { Product } from '../entities/product.entity';

// Interfaces de base
export interface Readable<T> {
  findById(id: string): Observable<T | null>;
  findAll(): Observable<T[]>;
}

export interface Writable<T> {
  save(entity: T): Observable<T>;
  delete(id: string): Observable<void>;
}

export interface Searchable<T> {
  search(criteria: any): Observable<T[]>;
}

// Interface spécifique au produit
export interface ProductRepository extends
  Readable<Product>,
  Writable<Product>,
  Searchable<Product> {

  // Méthodes spécifiques aux produits
  findByCategory(category: string): Observable<Product[]>;
  findInStock(): Observable<Product[]>;
  findByPriceRange(min: number, max: number): Observable<Product[]>;
  findFeaturedProducts(): Observable<Product[]>;
}

export interface ProductSearchCriteria {
  readonly name?: string;
  readonly category?: string;
  readonly minPrice?: number;
  readonly maxPrice?: number;
  readonly inStock?: boolean;
  readonly featured?: boolean;
}
```

### Exemple 4 : Repository avec pagination

```typescript
import { Observable } from 'rxjs';
import { Article } from '../entities/article.entity';

export interface PaginatedResult<T> {
  readonly items: readonly T[];
  readonly total: number;
  readonly page: number;
  readonly pageSize: number;
  readonly totalPages: number;
}

export interface ArticleRepository {
  findById(id: string): Observable<Article | null>;

  // Pagination
  findAllPaginated(page: number, pageSize: number): Observable<PaginatedResult<Article>>;

  findByAuthorPaginated(
    authorId: string,
    page: number,
    pageSize: number
  ): Observable<PaginatedResult<Article>>;

  findPublishedPaginated(
    page: number,
    pageSize: number
  ): Observable<PaginatedResult<Article>>;

  // Opérations standards
  save(article: Article): Observable<Article>;
  delete(id: string): Observable<void>;
}
```

## 🧪 Tests avec mocks

Les interfaces facilitent les tests avec des mocks.

```typescript
// Test d'un use case avec mock repository
describe('GetUserUseCase', () => {
  it('should return user when found', (done) => {
    // Mock du repository
    const mockRepository: UserRepository = {
      findById: (id) => of(new User(id, Email.create('test@test.com'), 'Test')),
      findByEmail: () => of(null),
      findAll: () => of([]),
      save: (user) => of(user),
      delete: () => of(void 0),
      existsByEmail: () => of(false),
      search: () => of([]),
      findActiveUsers: () => of([]),
      findByRole: () => of([])
    };

    const useCase = new GetUserUseCase(mockRepository);

    useCase.execute('123').subscribe(user => {
      expect(user.id).toBe('123');
      expect(user.email.getValue()).toBe('test@test.com');
      done();
    });
  });

  it('should throw error when user not found', (done) => {
    // Mock retournant null
    const mockRepository: UserRepository = {
      findById: () => of(null),
      // ... autres méthodes
    } as UserRepository;

    const useCase = new GetUserUseCase(mockRepository);

    useCase.execute('123').subscribe({
      error: (error) => {
        expect(error.message).toContain('not found');
        done();
      }
    });
  });
});
```

## 🔄 Flux de données

```
Presentation Layer
       │
       ▼
Use Case (Application Layer)
       │
       ▼
Repository Interface (Domain Layer) ◄─── Nous sommes ICI
       ▲
       │
       │ (implements)
       │
Repository Implementation (Infrastructure Layer)
       │
       ▼
HTTP / Database / Storage
```

## 🎓 Points clés

1. **Interface seulement** : Pas d'implémentation dans le Domain
2. **Contrat clair** : Définir QUOI, pas COMMENT
3. **Observables** : Utiliser RxJS pour l'asynchrone
4. **Noms métier** : Langage du domaine, pas technique
5. **Retourner null** : Pour les éléments non trouvés
6. **Méthodes spécifiques** : Éviter les méthodes génériques vagues
7. **Pas de logique métier** : Seulement l'accès aux données
8. **Testabilité** : Facile à mocker pour les tests

---

**Les repository interfaces sont le pont entre votre logique métier pure et le monde extérieur ! 📦**
