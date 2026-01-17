# 🎬 Use Cases - Cas d'utilisation

## 📋 Description

Les **Use Cases** représentent les **actions métier** que l'application peut exécuter. Chaque use case correspond à **un scénario d'utilisation** spécifique de l'application.

> **Principe clé** : Un Use Case = Une action métier unique et complète.

## 🎯 Responsabilité

Un use case :
- **Orchestre** la logique métier
- **Coordonne** les entités et repositories
- **Gère** le workflow de l'action
- **Ne contient PAS** de logique métier (→ déléguer aux entités)
- **Ne contient PAS** de détails techniques (→ déléguer à l'infrastructure)

## ✅ Ce qu'on doit mettre

### Types de use cases
- ✅ Actions CRUD (Create, Read, Update, Delete)
- ✅ Workflows métier complexes
- ✅ Orchestration de plusieurs entités
- ✅ Gestion des transactions
- ✅ Coordination avec services externes (via ports)

### Exemples
```typescript
// ✅ Use cases typiques
export class CreateUserUseCase { }
export class GetUserByIdUseCase { }
export class UpdateUserProfileUseCase { }
export class DeleteUserUseCase { }
export class ActivateUserAccountUseCase { }

export class CreateOrderUseCase { }
export class ConfirmOrderUseCase { }
export class CancelOrderUseCase { }
export class ShipOrderUseCase { }

export class ProcessPaymentUseCase { }
export class RefundPaymentUseCase { }

export class SendWelcomeEmailUseCase { }
export class ResetPasswordUseCase { }
```

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Logique métier des entités (→ Domain/Entities)
- ❌ Calculs métier (→ Domain/Entities)
- ❌ Validation métier (→ Domain/Value Objects)
- ❌ Détails de persistance (→ Infrastructure)
- ❌ Appels HTTP directs (→ Infrastructure)
- ❌ Logique d'interface (→ Presentation)

## 🏗️ Structure d'un Use Case

### Template de base

```typescript
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { map, switchMap } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class ActionNameUseCase {
  constructor(
    private repository: EntityRepository,
    private service?: OptionalService
  ) {}

  execute(params: InputDTO): Observable<OutputType> {
    // 1. Validation des paramètres
    this.validateInput(params);

    // 2. Récupération des données
    return this.repository.findById(params.id).pipe(
      // 3. Logique métier (déléguer aux entités)
      map(entity => {
        if (!entity) throw new Error('Not found');
        return entity.doSomething(params);
      }),
      // 4. Persistance
      switchMap(entity => this.repository.save(entity))
    );
  }

  private validateInput(params: InputDTO): void {
    if (!params.id) {
      throw new Error('ID is required');
    }
  }
}
```

## 📖 Principes et bonnes pratiques

### 1. Responsabilité unique

Un use case = Une action métier

```typescript
// ✅ CORRECT - Actions séparées
@Injectable({ providedIn: 'root' })
export class CreateUserUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(dto: CreateUserDTO): Observable<User> {
    const user = User.create(dto);
    return this.userRepository.save(user);
  }
}

@Injectable({ providedIn: 'root' })
export class UpdateUserUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(id: string, dto: UpdateUserDTO): Observable<User> {
    return this.userRepository.findById(id).pipe(
      map(user => {
        if (!user) throw new Error('User not found');
        return user.update(dto);
      }),
      switchMap(user => this.userRepository.save(user))
    );
  }
}

// ❌ INCORRECT - Trop d'actions dans un service
@Injectable({ providedIn: 'root' })
export class UserService {
  createUser(dto: CreateUserDTO): Observable<User> { }
  getUserById(id: string): Observable<User> { }
  updateUser(id: string, dto: UpdateUserDTO): Observable<User> { }
  deleteUser(id: string): Observable<void> { }
  activateUser(id: string): Observable<User> { }
  deactivateUser(id: string): Observable<User> { }
  // ❌ Trop de responsabilités
}
```

### 2. Nommage

Convention : `[Verbe][Entité][Complément]UseCase`

```typescript
// ✅ CORRECT - Noms clairs
export class CreateUserUseCase { }
export class GetUserByIdUseCase { }
export class UpdateUserProfileUseCase { }
export class DeleteUserUseCase { }
export class ActivateUserAccountUseCase { }
export class DeactivateUserAccountUseCase { }
export class ResetUserPasswordUseCase { }

export class GetAllProductsUseCase { }
export class GetProductsByCategoryUseCase { }
export class SearchProductsUseCase { }

export class ConfirmOrderUseCase { }
export class CancelOrderUseCase { }
export class ShipOrderUseCase { }

// ❌ INCORRECT - Noms vagues
export class UserUseCase { }        // ❌ Trop vague
export class HandleUser { }         // ❌ Pas clair
export class DoSomething { }        // ❌ Que fait-il ?
export class UserManager { }        // ❌ Responsabilité floue
```

### 3. Méthode execute

Un seul point d'entrée : `execute()`

```typescript
// ✅ CORRECT - Méthode execute unique
export class GetUserByIdUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(id: string): Observable<User> {
    return this.userRepository.findById(id).pipe(
      map(user => {
        if (!user) throw new UserNotFoundError(id);
        return user;
      })
    );
  }
}

// Utilisation
this.getUserByIdUseCase.execute('123').subscribe(user => {
  console.log(user);
});

// ❌ INCORRECT - Plusieurs méthodes publiques
export class UserUseCase {
  getById(id: string): Observable<User> { }
  getByEmail(email: string): Observable<User> { }
  getAll(): Observable<User[]> { }
  // ❌ Créer des use cases séparés
}
```

### 4. DTOs pour les paramètres

Utiliser des DTOs pour les entrées complexes

```typescript
// ✅ CORRECT - DTO pour paramètres multiples
export interface CreateOrderDTO {
  readonly customerId: string;
  readonly items: OrderItemDTO[];
  readonly shippingAddress: AddressDTO;
  readonly billingAddress: AddressDTO;
}

export interface OrderItemDTO {
  readonly productId: string;
  readonly quantity: number;
}

export class CreateOrderUseCase {
  execute(dto: CreateOrderDTO): Observable<Order> {
    // Validation
    this.validate(dto);

    // Création
    const order = Order.create(dto);
    return this.orderRepository.save(order);
  }

  private validate(dto: CreateOrderDTO): void {
    if (!dto.customerId) throw new Error('Customer ID required');
    if (!dto.items.length) throw new Error('Order must have items');
  }
}

// ❌ INCORRECT - Trop de paramètres primitifs
export class CreateOrderUseCase {
  execute(
    customerId: string,
    productIds: string[],
    quantities: number[],
    shippingStreet: string,
    shippingCity: string,
    shippingPostalCode: string
    // ... ❌ Trop de paramètres
  ): Observable<Order> { }
}
```

### 5. Gestion des erreurs

Erreurs métier explicites

```typescript
// ✅ CORRECT - Exceptions métier personnalisées
export class UserNotFoundError extends Error {
  constructor(userId: string) {
    super(`User with ID ${userId} not found`);
    this.name = 'UserNotFoundError';
  }
}

export class InsufficientStockError extends Error {
  constructor(productId: string, requested: number, available: number) {
    super(`Not enough stock for product ${productId}. Requested: ${requested}, Available: ${available}`);
    this.name = 'InsufficientStockError';
  }
}

export class GetUserByIdUseCase {
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
      })
    );
  }
}

// Utilisation dans le component
this.getUserUseCase.execute(id).subscribe({
  next: (user) => console.log(user),
  error: (error) => {
    if (error instanceof UserNotFoundError) {
      this.notificationService.showError('Utilisateur introuvable');
    } else {
      this.notificationService.showError('Erreur inconnue');
    }
  }
});
```

### 6. Déléguer la logique métier aux entités

```typescript
// ✅ CORRECT - Logique dans l'entité
export class ConfirmOrderUseCase {
  execute(orderId: string): Observable<Order> {
    return this.orderRepository.findById(orderId).pipe(
      map(order => {
        if (!order) throw new OrderNotFoundError(orderId);
        return order.confirm(); // ✅ Logique dans l'entité
      }),
      switchMap(order => this.orderRepository.save(order))
    );
  }
}

// Entity Order
export class Order {
  confirm(): Order {
    // ✅ Logique métier ici
    if (this.status !== OrderStatus.Pending) {
      throw new Error('Only pending orders can be confirmed');
    }
    if (this.items.length === 0) {
      throw new Error('Cannot confirm empty order');
    }
    return new Order(/* ... */, OrderStatus.Confirmed, /* ... */);
  }
}

// ❌ INCORRECT - Logique dans le use case
export class ConfirmOrderUseCase {
  execute(orderId: string): Observable<Order> {
    return this.orderRepository.findById(orderId).pipe(
      map(order => {
        // ❌ Logique métier dans le use case
        if (order.status !== OrderStatus.Pending) {
          throw new Error('Only pending orders can be confirmed');
        }
        if (order.items.length === 0) {
          throw new Error('Cannot confirm empty order');
        }

        return new Order(/* ... */, OrderStatus.Confirmed, /* ... */);
      }),
      switchMap(order => this.orderRepository.save(order))
    );
  }
}
```

## 📊 Exemples complets

### Exemple 1 : CRUD User complet

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

// ========== EXCEPTIONS ==========
export class UserNotFoundError extends Error {
  constructor(userId: string) {
    super(`User ${userId} not found`);
    this.name = 'UserNotFoundError';
  }
}

export class EmailAlreadyExistsError extends Error {
  constructor(email: string) {
    super(`Email ${email} already exists`);
    this.name = 'EmailAlreadyExistsError';
  }
}

// ========== CREATE ==========
@Injectable({ providedIn: 'root' })
export class CreateUserUseCase {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService
  ) {}

  execute(dto: CreateUserDTO): Observable<User> {
    const email = Email.create(dto.email);

    return this.userRepository.existsByEmail(email).pipe(
      switchMap(exists => {
        if (exists) {
          return throwError(() => new EmailAlreadyExistsError(dto.email));
        }

        const password = Password.create(dto.password);
        const user = User.create(email, dto.firstName, dto.lastName, password);

        return this.userRepository.save(user);
      }),
      tap(user =>
        this.emailService.sendWelcomeEmail(user.email).subscribe()
      )
    );
  }
}

// ========== READ ==========
@Injectable({ providedIn: 'root' })
export class GetUserByIdUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(id: string): Observable<User> {
    if (!id) {
      return throwError(() => new Error('User ID is required'));
    }

    return this.userRepository.findById(id).pipe(
      map(user => {
        if (!user) throw new UserNotFoundError(id);
        return user;
      })
    );
  }
}

@Injectable({ providedIn: 'root' })
export class GetAllUsersUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(): Observable<User[]> {
    return this.userRepository.findAll();
  }
}

@Injectable({ providedIn: 'root' })
export class GetUserByEmailUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(email: string): Observable<User> {
    const emailVO = Email.create(email);

    return this.userRepository.findByEmail(emailVO).pipe(
      map(user => {
        if (!user) throw new UserNotFoundError(email);
        return user;
      })
    );
  }
}

// ========== UPDATE ==========
@Injectable({ providedIn: 'root' })
export class UpdateUserUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(id: string, dto: UpdateUserDTO): Observable<User> {
    return this.userRepository.findById(id).pipe(
      map(user => {
        if (!user) throw new UserNotFoundError(id);
        return user.update(dto.firstName, dto.lastName);
      }),
      switchMap(user => this.userRepository.save(user))
    );
  }
}

// ========== DELETE ==========
@Injectable({ providedIn: 'root' })
export class DeleteUserUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(id: string): Observable<void> {
    return this.userRepository.findById(id).pipe(
      switchMap(user => {
        if (!user) throw new UserNotFoundError(id);

        if (!user.canBeDeleted()) {
          throw new Error('User cannot be deleted');
        }

        return this.userRepository.delete(id);
      })
    );
  }
}

// ========== MÉTIER SPÉCIFIQUE ==========
@Injectable({ providedIn: 'root' })
export class ActivateUserUseCase {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService
  ) {}

  execute(id: string): Observable<User> {
    return this.userRepository.findById(id).pipe(
      map(user => {
        if (!user) throw new UserNotFoundError(id);
        return user.activate(); // Logique dans l'entité
      }),
      switchMap(user => this.userRepository.save(user)),
      tap(user =>
        this.emailService.sendActivationConfirmation(user.email).subscribe()
      )
    );
  }
}
```

### Exemple 2 : Workflow de commande

```typescript
// ========== CREATE ORDER ==========
@Injectable({ providedIn: 'root' })
export class CreateOrderUseCase {
  constructor(
    private orderRepository: OrderRepository,
    private productRepository: ProductRepository,
    private customerRepository: CustomerRepository
  ) {}

  execute(dto: CreateOrderDTO): Observable<Order> {
    // Vérifier que le client existe
    return this.customerRepository.findById(dto.customerId).pipe(
      switchMap(customer => {
        if (!customer) {
          return throwError(() => new Error('Customer not found'));
        }
        if (!customer.isActive) {
          return throwError(() => new Error('Customer is inactive'));
        }

        // Récupérer tous les produits
        return forkJoin(
          dto.items.map(item =>
            this.productRepository.findById(item.productId)
          )
        );
      }),
      map(products => {
        // Vérifier que tous les produits existent
        if (products.some(p => !p)) {
          throw new Error('Some products not found');
        }

        // Créer les OrderItems avec vérification du stock
        const orderItems = dto.items.map((itemDTO, index) => {
          const product = products[index]!;

          if (!product.hasEnoughStock(itemDTO.quantity)) {
            throw new InsufficientStockError(
              product.id,
              itemDTO.quantity,
              product.stock
            );
          }

          return OrderItem.create(
            product.id,
            product.name,
            product.price,
            itemDTO.quantity
          );
        });

        // Créer la commande
        return Order.createNew(dto.customerId, orderItems);
      }),
      switchMap(order => this.orderRepository.save(order))
    );
  }
}

// ========== CONFIRM ORDER ==========
@Injectable({ providedIn: 'root' })
export class ConfirmOrderUseCase {
  constructor(
    private orderRepository: OrderRepository,
    private productRepository: ProductRepository,
    private emailService: EmailService,
    private notificationService: NotificationService
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

        return forkJoin(stockUpdates).pipe(mapTo(confirmedOrder));
      }),
      switchMap(order => this.orderRepository.save(order)),
      tap(order => {
        // Effets de bord (parallèles)
        this.emailService.sendOrderConfirmation(order).subscribe();
        this.notificationService.notifyWarehouse(order).subscribe();
      })
    );
  }
}

// ========== CANCEL ORDER ==========
@Injectable({ providedIn: 'root' })
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

        // Annuler (logique dans l'entité)
        const cancelledOrder = order.cancel();

        // Si confirmée, remettre le stock
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

          return forkJoin(stockRestores).pipe(mapTo(cancelledOrder));
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

### Exemple 3 : Use case avec composition

```typescript
// ========== USE CASE COMPOSÉ ==========
@Injectable({ providedIn: 'root' })
export class RegisterUserUseCase {
  constructor(
    private createUserUseCase: CreateUserUseCase,
    private sendWelcomeEmailUseCase: SendWelcomeEmailUseCase,
    private createUserProfileUseCase: CreateUserProfileUseCase,
    private notificationService: NotificationService
  ) {}

  execute(dto: RegisterUserDTO): Observable<User> {
    return this.createUserUseCase.execute(dto).pipe(
      switchMap(user => {
        // Créer le profil par défaut
        return this.createUserProfileUseCase.execute(user.id).pipe(
          mapTo(user)
        );
      }),
      tap(user => {
        // Effets de bord parallèles
        this.sendWelcomeEmailUseCase.execute(user.id).subscribe();
        this.notificationService.notifyAdmins('New user registered', user).subscribe();
      })
    );
  }
}
```

## 🧪 Tests de Use Cases

```typescript
describe('CreateUserUseCase', () => {
  let useCase: CreateUserUseCase;
  let mockRepository: jest.Mocked<UserRepository>;
  let mockEmailService: jest.Mocked<EmailService>;

  beforeEach(() => {
    mockRepository = {
      save: jest.fn(),
      existsByEmail: jest.fn(),
      findById: jest.fn()
    } as any;

    mockEmailService = {
      sendWelcomeEmail: jest.fn(() => of(void 0))
    } as any;

    useCase = new CreateUserUseCase(mockRepository, mockEmailService);
  });

  it('should create user successfully', (done) => {
    const dto: CreateUserDTO = {
      email: 'test@test.com',
      firstName: 'John',
      lastName: 'Doe',
      password: 'Password123!'
    };

    mockRepository.existsByEmail.mockReturnValue(of(false));
    mockRepository.save.mockImplementation(user => of(user));

    useCase.execute(dto).subscribe({
      next: (user) => {
        expect(user).toBeDefined();
        expect(mockRepository.save).toHaveBeenCalled();
        expect(mockEmailService.sendWelcomeEmail).toHaveBeenCalled();
        done();
      }
    });
  });

  it('should throw error if email exists', (done) => {
    const dto: CreateUserDTO = {
      email: 'test@test.com',
      firstName: 'John',
      lastName: 'Doe',
      password: 'Password123!'
    };

    mockRepository.existsByEmail.mockReturnValue(of(true));

    useCase.execute(dto).subscribe({
      error: (error) => {
        expect(error).toBeInstanceOf(EmailAlreadyExistsError);
        expect(mockRepository.save).not.toHaveBeenCalled();
        done();
      }
    });
  });
});
```

## 🎓 Points clés

1. **Un use case = Une action** : Responsabilité unique
2. **Méthode execute()** : Point d'entrée unique
3. **DTOs** : Pour paramètres complexes
4. **Déléguer** : Logique métier aux entités
5. **Orchestrer** : Coordonner repositories et services
6. **Erreurs métier** : Exceptions personnalisées
7. **Observable** : Retourner des Observables
8. **Testabilité** : Facile avec mocks

---

**Les use cases sont vos scénarios métier en code ! 🎬**
