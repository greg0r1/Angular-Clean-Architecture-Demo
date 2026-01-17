# 🎯 Application - Cas d'utilisation

## 📋 Description

La couche **Application** orchestre la **logique métier** en coordonnant les entités du Domain pour réaliser des **cas d'utilisation** (use cases) spécifiques de l'application.

> **Principe clé** : La couche Application décrit **CE QUE** fait l'application, sans se soucier des détails techniques du **COMMENT**.

## 🏗️ Structure

```
application/
├── use-cases/     # Scénarios d'utilisation de l'application
├── ports/         # Interfaces pour les services externes
└── README.md
```

## 📚 Composants

### 1. Use Cases (Cas d'utilisation)
Les **scénarios métier** que l'application peut exécuter.

Exemples :
- `CreateUserUseCase`
- `UpdateOrderStatusUseCase`
- `CalculateInvoiceTotalUseCase`
- `SendWelcomeEmailUseCase`

### 2. Ports (Interfaces)
Les **contrats** pour les services externes dont les use cases ont besoin.

Exemples :
- `EmailService` (interface)
- `NotificationService` (interface)
- `FileStorageService` (interface)

## 🔄 Flux de données

```
Presentation (Component)
       │
       │ injecte & appelle
       ▼
Use Case (Application) ◄─── Nous sommes ICI
       │
       │ utilise
       ▼
Repository Interface (Domain)
       ▲
       │ implémente
       │
Repository Impl (Infrastructure)
```

## ✅ Ce qu'on doit mettre

### Dans la couche Application
- ✅ Use Cases (un par action métier)
- ✅ Orchestration de plusieurs entités
- ✅ Coordination avec les repositories
- ✅ Interfaces de ports (services externes)
- ✅ DTOs d'application si nécessaire
- ✅ Gestion des transactions
- ✅ Logique de workflow

### Exemples
```typescript
// ✅ Use Cases
export class CreateUserUseCase { }
export class GetUserByIdUseCase { }
export class UpdateUserProfileUseCase { }
export class DeleteUserUseCase { }
export class ActivateUserUseCase { }

// ✅ Ports (interfaces)
export interface EmailService {
  sendEmail(to: string, subject: string, body: string): Observable<void>;
}

export interface NotificationService {
  notify(userId: string, message: string): Observable<void>;
}
```

## ❌ Ce qu'on ne doit PAS mettre

### Interdictions
- ❌ Logique métier des entités (→ Domain/Entities)
- ❌ Détails de persistance (→ Infrastructure)
- ❌ Logique de présentation (→ Presentation)
- ❌ Appels HTTP directs (→ Infrastructure)
- ❌ Accès au localStorage (→ Infrastructure)
- ❌ Manipulation du DOM (→ Presentation)
- ❌ Implémentation des ports (→ Infrastructure)

```typescript
// ❌ INCORRECT - Logique métier dans use case
export class CreateOrderUseCase {
  execute(data: CreateOrderDTO): Observable<Order> {
    // ❌ Logique métier qui devrait être dans l'entité Order
    const total = data.items.reduce((sum, item) =>
      sum + item.price * item.quantity, 0
    );

    if (total < 0) {
      throw new Error('Invalid total');
    }
    // ...
  }
}

// ✅ CORRECT - Déléguer à l'entité
export class CreateOrderUseCase {
  execute(data: CreateOrderDTO): Observable<Order> {
    const order = Order.create(data); // ✅ Logique dans l'entité
    return this.orderRepository.save(order);
  }
}
```

## 📖 Principes SOLID

### Single Responsibility (SRP)

Un use case = Une action métier unique

```typescript
// ✅ CORRECT - Un use case par action
export class CreateUserUseCase {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService
  ) {}

  execute(dto: CreateUserDTO): Observable<User> {
    const user = User.create(dto);
    return this.userRepository.save(user).pipe(
      tap(savedUser =>
        this.emailService.sendWelcomeEmail(savedUser.email)
      )
    );
  }
}

export class UpdateUserProfileUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(userId: string, dto: UpdateProfileDTO): Observable<User> {
    return this.userRepository.findById(userId).pipe(
      map(user => {
        if (!user) throw new Error('User not found');
        return user.updateProfile(dto);
      }),
      switchMap(user => this.userRepository.save(user))
    );
  }
}

// ❌ INCORRECT - Trop de responsabilités
export class UserUseCase {
  createUser() { }
  updateUser() { }
  deleteUser() { }
  activateUser() { }
  deactivateUser() { }
  resetPassword() { }
  sendWelcomeEmail() { }
  // ❌ Trop d'actions dans un seul use case
}
```

### Dependency Inversion (DIP)

Le use case dépend des interfaces, pas des implémentations

```typescript
// ✅ CORRECT - Dépendances d'interfaces
export class SendWelcomeEmailUseCase {
  constructor(
    private userRepository: UserRepository,      // Interface du Domain
    private emailService: EmailService           // Interface (port)
  ) {}

  execute(userId: string): Observable<void> {
    return this.userRepository.findById(userId).pipe(
      switchMap(user => {
        if (!user) throw new Error('User not found');
        return this.emailService.sendWelcomeEmail(user.email);
      })
    );
  }
}

// ❌ INCORRECT - Dépendances concrètes
import { UserRepositoryImpl } from '@infrastructure/...'; // ❌
import { GmailService } from '@infrastructure/...'; // ❌

export class SendWelcomeEmailUseCase {
  constructor(
    private userRepository: UserRepositoryImpl, // ❌ Implémentation
    private emailService: GmailService          // ❌ Implémentation
  ) {}
}
```

## 🛡️ Bonnes pratiques

### 1. Structure d'un Use Case

```typescript
// Template de base
export class ActionNameUseCase {
  // 1. Injection des dépendances (interfaces)
  constructor(
    private repository: EntityRepository,
    private service?: OptionalPort
  ) {}

  // 2. Méthode execute avec paramètres typés
  execute(params: InputDTO): Observable<OutputType> {
    // 3. Orchestration
    return this.repository.findById(params.id).pipe(
      // 4. Logique métier (déléguer aux entités)
      map(entity => entity.doSomething(params)),
      // 5. Persistance
      switchMap(entity => this.repository.save(entity)),
      // 6. Effets de bord optionnels
      tap(result => this.service?.notify(result))
    );
  }
}
```

### 2. Nommage des Use Cases

```typescript
// ✅ CORRECT - Noms clairs et métier
export class CreateUserUseCase { }
export class GetUserByIdUseCase { }
export class UpdateUserProfileUseCase { }
export class DeleteUserUseCase { }
export class ActivateUserAccountUseCase { }
export class DeactivateUserAccountUseCase { }
export class ResetUserPasswordUseCase { }

// Convention : [Verbe][Entité][Complément]UseCase

// ❌ INCORRECT - Noms vagues ou techniques
export class UserService { }         // ❌ Trop vague
export class UserCRUD { }            // ❌ Trop technique
export class HandleUser { }          // ❌ Pas clair
export class UserManager { }         // ❌ Que fait-il ?
```

### 3. Gestion des erreurs

```typescript
// ✅ CORRECT - Gestion explicite des erreurs
export class GetUserByIdUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(id: string): Observable<User> {
    if (!id) {
      return throwError(() => new Error('User ID is required'));
    }

    return this.userRepository.findById(id).pipe(
      map(user => {
        if (!user) {
          throw new UserNotFoundError(id);
        }
        return user;
      }),
      catchError(error => {
        // Log ou traitement spécifique
        return throwError(() => error);
      })
    );
  }
}

// Exceptions métier personnalisées
export class UserNotFoundError extends Error {
  constructor(userId: string) {
    super(`User with ID ${userId} not found`);
    this.name = 'UserNotFoundError';
  }
}
```

### 4. Use Cases composés

Un use case peut en appeler d'autres.

```typescript
// ✅ CORRECT - Composition de use cases
export class RegisterUserUseCase {
  constructor(
    private createUserUseCase: CreateUserUseCase,
    private sendWelcomeEmailUseCase: SendWelcomeEmailUseCase,
    private notificationService: NotificationService
  ) {}

  execute(dto: RegisterUserDTO): Observable<User> {
    return this.createUserUseCase.execute(dto).pipe(
      tap(user => {
        // Effets de bord parallèles
        this.sendWelcomeEmailUseCase.execute(user.id).subscribe();
        this.notificationService.notifyAdmins('New user registered').subscribe();
      })
    );
  }
}
```

### 5. DTOs d'application

Utilisez des DTOs pour les entrées/sorties des use cases.

```typescript
// ✅ CORRECT - DTOs typés
export interface CreateUserDTO {
  readonly email: string;
  readonly firstName: string;
  readonly lastName: string;
  readonly password: string;
}

export interface UpdateProfileDTO {
  readonly firstName?: string;
  readonly lastName?: string;
  readonly bio?: string;
}

export class CreateUserUseCase {
  execute(dto: CreateUserDTO): Observable<User> {
    // Validation du DTO
    this.validate(dto);

    // Création de l'entité
    const email = Email.create(dto.email);
    const password = Password.create(dto.password);
    const user = User.create(email, dto.firstName, dto.lastName, password);

    return this.userRepository.save(user);
  }

  private validate(dto: CreateUserDTO): void {
    if (!dto.email || !dto.password) {
      throw new Error('Email and password are required');
    }
  }
}

// ❌ INCORRECT - Trop de paramètres primitifs
export class CreateUserUseCase {
  execute(
    email: string,
    firstName: string,
    lastName: string,
    password: string,
    age: number,
    city: string
    // ... ❌ Trop de paramètres
  ): Observable<User> { }
}
```

## 📊 Exemples complets

### Exemple 1 : CRUD User

```typescript
// ========== DTOs ==========
export interface CreateUserDTO {
  readonly email: string;
  readonly firstName: string;
  readonly lastName: string;
  readonly password: string;
}

export interface UpdateUserDTO {
  readonly firstName?: string;
  readonly lastName?: string;
}

// ========== USE CASES ==========

// Create
export class CreateUserUseCase {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService
  ) {}

  execute(dto: CreateUserDTO): Observable<User> {
    // Vérifier si l'email existe déjà
    const email = Email.create(dto.email);

    return this.userRepository.existsByEmail(email).pipe(
      switchMap(exists => {
        if (exists) {
          return throwError(() => new Error('Email already exists'));
        }

        // Créer l'entité
        const password = Password.create(dto.password);
        const user = User.create(email, dto.firstName, dto.lastName, password);

        // Sauvegarder
        return this.userRepository.save(user);
      }),
      // Envoyer email de bienvenue
      tap(user =>
        this.emailService.sendWelcomeEmail(user.email).subscribe()
      )
    );
  }
}

// Read
export class GetUserByIdUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(id: string): Observable<User> {
    return this.userRepository.findById(id).pipe(
      map(user => {
        if (!user) {
          throw new UserNotFoundError(id);
        }
        return user;
      })
    );
  }
}

// Update
export class UpdateUserUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(id: string, dto: UpdateUserDTO): Observable<User> {
    return this.userRepository.findById(id).pipe(
      map(user => {
        if (!user) throw new UserNotFoundError(id);

        // La logique de mise à jour est dans l'entité
        return user.update(dto.firstName, dto.lastName);
      }),
      switchMap(updatedUser =>
        this.userRepository.save(updatedUser)
      )
    );
  }
}

// Delete
export class DeleteUserUseCase {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService
  ) {}

  execute(id: string): Observable<void> {
    return this.userRepository.findById(id).pipe(
      switchMap(user => {
        if (!user) throw new UserNotFoundError(id);

        // Vérifier si l'utilisateur peut être supprimé
        if (!user.canBeDeleted()) {
          throw new Error('User cannot be deleted');
        }

        return this.userRepository.delete(id);
      }),
      tap(() =>
        this.emailService.sendAccountDeletedEmail(id).subscribe()
      )
    );
  }
}
```

### Exemple 2 : Workflow de commande

```typescript
// ========== USE CASES ==========

export class CreateOrderUseCase {
  constructor(
    private orderRepository: OrderRepository,
    private productRepository: ProductRepository
  ) {}

  execute(dto: CreateOrderDTO): Observable<Order> {
    // Récupérer les produits pour vérifier le stock
    return forkJoin(
      dto.items.map(item =>
        this.productRepository.findById(item.productId)
      )
    ).pipe(
      map(products => {
        // Vérifier que tous les produits existent
        if (products.some(p => !p)) {
          throw new Error('Some products not found');
        }

        // Créer les OrderItems avec les produits
        const orderItems = dto.items.map((item, index) => {
          const product = products[index]!;

          // Vérifier le stock (logique dans l'entité Product)
          if (!product.hasEnoughStock(item.quantity)) {
            throw new InsufficientStockError(product.id);
          }

          return OrderItem.create(
            product.id,
            product.name,
            product.price,
            item.quantity
          );
        });

        // Créer la commande (logique dans l'entité Order)
        return Order.createNew(dto.customerId, orderItems);
      }),
      switchMap(order => this.orderRepository.save(order))
    );
  }
}

export class ConfirmOrderUseCase {
  constructor(
    private orderRepository: OrderRepository,
    private productRepository: ProductRepository,
    private emailService: EmailService
  ) {}

  execute(orderId: string): Observable<Order> {
    return this.orderRepository.findById(orderId).pipe(
      switchMap(order => {
        if (!order) throw new OrderNotFoundError(orderId);

        // Confirmer la commande (logique dans l'entité)
        const confirmedOrder = order.confirm();

        // Décrémenter les stocks
        const stockUpdates = confirmedOrder.items.map(item =>
          this.productRepository.findById(item.productId).pipe(
            map(product => {
              if (!product) throw new Error('Product not found');
              return product.decreaseStock(item.quantity);
            }),
            switchMap(product => this.productRepository.save(product))
          )
        );

        return forkJoin(stockUpdates).pipe(
          mapTo(confirmedOrder)
        );
      }),
      switchMap(order => this.orderRepository.save(order)),
      tap(order =>
        this.emailService.sendOrderConfirmation(order).subscribe()
      )
    );
  }
}

export class CancelOrderUseCase {
  constructor(
    private orderRepository: OrderRepository,
    private productRepository: ProductRepository,
    private emailService: EmailService
  ) {}

  execute(orderId: string, reason: string): Observable<Order> {
    return this.orderRepository.findById(orderId).pipe(
      switchMap(order => {
        if (!order) throw new OrderNotFoundError(orderId);

        // Annuler la commande (logique dans l'entité)
        const cancelledOrder = order.cancel();

        // Si la commande était confirmée, remettre le stock
        if (order.status === OrderStatus.Confirmed) {
          const stockRestores = order.items.map(item =>
            this.productRepository.findById(item.productId).pipe(
              map(product => {
                if (!product) throw new Error('Product not found');
                return product.increaseStock(item.quantity);
              }),
              switchMap(product => this.productRepository.save(product))
            )
          );

          return forkJoin(stockRestores).pipe(
            mapTo(cancelledOrder)
          );
        }

        return of(cancelledOrder);
      }),
      switchMap(order => this.orderRepository.save(order)),
      tap(order =>
        this.emailService.sendOrderCancellation(order, reason).subscribe()
      )
    );
  }
}
```

### Exemple 3 : Use Case avec port

```typescript
// ========== PORT (Interface) ==========
export interface EmailService {
  sendEmail(to: Email, subject: string, body: string): Observable<void>;
  sendWelcomeEmail(to: Email): Observable<void>;
  sendPasswordResetEmail(to: Email, token: string): Observable<void>;
}

// ========== USE CASE ==========
export class RequestPasswordResetUseCase {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService, // Port
    private tokenService: TokenService  // Port
  ) {}

  execute(email: string): Observable<void> {
    const emailVO = Email.create(email);

    return this.userRepository.findByEmail(emailVO).pipe(
      switchMap(user => {
        if (!user) {
          // Pour des raisons de sécurité, ne pas révéler si l'email existe
          return of(void 0);
        }

        // Générer un token de réinitialisation
        return this.tokenService.generateResetToken(user.id).pipe(
          switchMap(token =>
            this.emailService.sendPasswordResetEmail(user.email, token)
          )
        );
      })
    );
  }
}
```

## 🧪 Tests de Use Cases

Les use cases sont faciles à tester avec des mocks.

```typescript
describe('CreateUserUseCase', () => {
  let useCase: CreateUserUseCase;
  let mockUserRepository: jest.Mocked<UserRepository>;
  let mockEmailService: jest.Mocked<EmailService>;

  beforeEach(() => {
    mockUserRepository = {
      save: jest.fn(),
      existsByEmail: jest.fn(),
      findById: jest.fn(),
      findByEmail: jest.fn()
    } as any;

    mockEmailService = {
      sendWelcomeEmail: jest.fn(() => of(void 0))
    } as any;

    useCase = new CreateUserUseCase(mockUserRepository, mockEmailService);
  });

  it('should create user successfully', (done) => {
    const dto: CreateUserDTO = {
      email: 'test@test.com',
      firstName: 'John',
      lastName: 'Doe',
      password: 'Password123!'
    };

    mockUserRepository.existsByEmail.mockReturnValue(of(false));
    mockUserRepository.save.mockImplementation(user => of(user));

    useCase.execute(dto).subscribe(user => {
      expect(user).toBeDefined();
      expect(mockUserRepository.save).toHaveBeenCalled();
      expect(mockEmailService.sendWelcomeEmail).toHaveBeenCalled();
      done();
    });
  });

  it('should throw error if email already exists', (done) => {
    const dto: CreateUserDTO = {
      email: 'test@test.com',
      firstName: 'John',
      lastName: 'Doe',
      password: 'Password123!'
    };

    mockUserRepository.existsByEmail.mockReturnValue(of(true));

    useCase.execute(dto).subscribe({
      error: (error) => {
        expect(error.message).toContain('already exists');
        expect(mockUserRepository.save).not.toHaveBeenCalled();
        done();
      }
    });
  });
});
```

## 🎓 Points clés

1. **Un use case = Une action** : Responsabilité unique
2. **Orchestration** : Coordonne les entités et repositories
3. **Pas de logique métier** : Déléguer aux entités
4. **Interfaces** : Dépendre des abstractions (repositories, ports)
5. **DTOs** : Entrées/sorties typées
6. **Gestion des erreurs** : Explicite et métier
7. **Testabilité** : Facile avec mocks
8. **Observable** : Utiliser RxJS

---

**Les use cases sont le cœur fonctionnel de votre application ! 🎯**
