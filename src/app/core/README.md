# 🎯 Core - Cœur de l'application

## 📋 Description

Le dossier `core` contient **le cœur de votre application** : la logique métier pure et les cas d'utilisation. C'est la partie la plus importante et la plus stable de votre application.

## 🏗️ Structure

```
core/
├── domain/           # Logique métier pure (entités, interfaces)
│   ├── entities/
│   ├── repositories/
│   └── value-objects/
└── application/      # Cas d'utilisation et orchestration
    ├── use-cases/
    └── ports/
```

## 📚 Sous-dossiers

### 1. Domain (Domaine métier)
**La couche la plus interne - AUCUNE dépendance externe**

Contient :
- Les **entités métier** (objets avec identité)
- Les **value objects** (objets immuables)
- Les **interfaces de repositories**
- La **logique métier pure**

### 2. Application (Cas d'utilisation)
**Orchestration de la logique métier**

Contient :
- Les **use cases** (scénarios d'utilisation)
- Les **ports** (interfaces pour les adaptateurs)
- La **coordination** entre entités

## ✅ Ce qu'on doit mettre

### Dans Domain
- Entités avec leur logique métier
- Value Objects immuables
- Interfaces de repositories
- Règles métier pures
- Validations métier

### Dans Application
- Use Cases (un par action métier)
- Interfaces de ports (pour services externes)
- DTOs d'application si nécessaire
- Orchestration de plusieurs entités

## ❌ Ce qu'on ne doit PAS mettre

### À éviter absolument
- ❌ Imports d'Angular (`@angular/*`)
- ❌ Appels HTTP directs
- ❌ Accès au localStorage
- ❌ Logique d'interface utilisateur
- ❌ Dépendances à des librairies externes (sauf utilitaires JS purs)
- ❌ Code spécifique à un framework

### Exceptions autorisées
- ✅ RxJS (pour les Observables)
- ✅ Librairies JavaScript pures (lodash, date-fns)
- ✅ Types TypeScript

## 🎯 Principe clé : Indépendance

```typescript
// ✅ CORRECT - Aucune dépendance framework
export class User {
  constructor(
    public readonly id: string,
    public readonly email: Email,
    public readonly createdAt: Date
  ) {
    this.validate();
  }

  private validate(): void {
    if (!this.id) {
      throw new Error('User ID is required');
    }
  }

  updateEmail(newEmail: Email): User {
    return new User(this.id, newEmail, this.createdAt);
  }
}

// ❌ INCORRECT - Dépendance à Angular
import { Injectable } from '@angular/core'; // ❌ NON !

@Injectable() // ❌ Pas d'annotations Angular dans Domain !
export class User { }
```

## 🔄 Flux de données

```
Presentation Layer
       ↓
  Use Case (Application)
       ↓
  Repository Interface (Domain)
       ↑
  Repository Impl (Infrastructure)
```

## 📖 Principes SOLID appliqués

### Single Responsibility (SRP)
- Chaque entité a une seule raison de changer
- Chaque use case représente une seule action métier

```typescript
// ✅ CORRECT - Une responsabilité
export class CreateUserUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(data: CreateUserDTO): Observable<User> {
    const user = new User(/* ... */);
    return this.userRepository.save(user);
  }
}

// ❌ INCORRECT - Trop de responsabilités
export class UserService {
  createUser() { }
  deleteUser() { }
  sendEmail() { }  // ❌ Pas la responsabilité de ce service
  generatePDF() { } // ❌ Pas la responsabilité de ce service
}
```

### Open/Closed (OCP)
- Ouvert à l'extension, fermé à la modification

```typescript
// ✅ CORRECT - Extension par héritage
export abstract class Notification {
  abstract send(message: string): void;
}

export class EmailNotification extends Notification {
  send(message: string): void { /* ... */ }
}

export class SMSNotification extends Notification {
  send(message: string): void { /* ... */ }
}
```

### Liskov Substitution (LSP)
- Les sous-types doivent être substituables

```typescript
// ✅ CORRECT
export interface Repository<T> {
  findById(id: string): Observable<T>;
  save(entity: T): Observable<T>;
}

export class UserRepository implements Repository<User> {
  // Respecte le contrat de Repository
  findById(id: string): Observable<User> { /* ... */ }
  save(user: User): Observable<User> { /* ... */ }
}
```

### Interface Segregation (ISP)
- Interfaces spécifiques plutôt que générales

```typescript
// ✅ CORRECT - Interfaces séparées
export interface Readable<T> {
  findById(id: string): Observable<T>;
  findAll(): Observable<T[]>;
}

export interface Writable<T> {
  save(entity: T): Observable<T>;
  delete(id: string): Observable<void>;
}

// Le client choisit ce dont il a besoin
export class GetUserUseCase {
  constructor(private repository: Readable<User>) {} // Seulement lecture
}

// ❌ INCORRECT - Interface trop large
export interface Repository<T> {
  findById(id: string): Observable<T>;
  findAll(): Observable<T[]>;
  save(entity: T): Observable<T>;
  delete(id: string): Observable<void>;
  updatePartial(id: string, data: Partial<T>): Observable<T>;
  bulkInsert(entities: T[]): Observable<T[]>;
  // ... 20 autres méthodes que le client n'utilise pas
}
```

### Dependency Inversion (DIP)
- Dépendre des abstractions, pas des implémentations

```typescript
// ✅ CORRECT - Use Case dépend de l'interface
export class CreateUserUseCase {
  constructor(private userRepository: UserRepository) {} // Interface
}

// L'implémentation sera fournie par l'Infrastructure
// via l'injection de dépendances Angular

// ❌ INCORRECT - Dépendance concrète
import { UserRepositoryImpl } from '@infrastructure/...'; // ❌ NON !

export class CreateUserUseCase {
  constructor(private userRepository: UserRepositoryImpl) {} // ❌
}
```

## 🛡️ Bonnes pratiques

### 1. Gardez le Domain pur
```typescript
// ✅ CORRECT - Logique métier pure
export class Order {
  calculateTotal(): number {
    return this.items.reduce((sum, item) =>
      sum + item.price * item.quantity, 0
    );
  }

  canBeCancelled(): boolean {
    return this.status === OrderStatus.Pending;
  }
}

// ❌ INCORRECT - Logique technique
export class Order {
  async saveToDatabase(): Promise<void> { // ❌ Logique infrastructure
    // ...
  }

  showNotification(): void { // ❌ Logique présentation
    // ...
  }
}
```

### 2. Utilisez l'immutabilité
```typescript
// ✅ CORRECT - Immutable
export class User {
  constructor(
    public readonly id: string,
    public readonly name: string
  ) {}

  changeName(newName: string): User {
    return new User(this.id, newName); // Nouvelle instance
  }
}

// ❌ INCORRECT - Mutable
export class User {
  public id: string;
  public name: string;

  changeName(newName: string): void {
    this.name = newName; // ❌ Mutation
  }
}
```

### 3. Use Cases simples et focalisés
```typescript
// ✅ CORRECT - Un use case = une action
export class ActivateUserUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(userId: string): Observable<User> {
    return this.userRepository.findById(userId).pipe(
      map(user => user.activate()),
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
  // ... 20 autres méthodes
}
```

### 4. Validation dans les entités
```typescript
// ✅ CORRECT - Validation métier dans l'entité
export class Email {
  private constructor(private readonly value: string) {}

  static create(email: string): Email {
    if (!email.includes('@')) {
      throw new Error('Invalid email format');
    }
    return new Email(email);
  }

  getValue(): string {
    return this.value;
  }
}

// Utilisation
const email = Email.create('user@example.com'); // ✅ Valide
const invalid = Email.create('invalid'); // ❌ Lance une erreur
```

### 5. Testabilité
```typescript
// ✅ Facile à tester - Pas de dépendances Angular
describe('CreateUserUseCase', () => {
  it('should create a user', (done) => {
    const mockRepository: UserRepository = {
      save: (user) => of(user),
      findById: (id) => of(null as any)
    };

    const useCase = new CreateUserUseCase(mockRepository);

    useCase.execute({ email: 'test@test.com' }).subscribe(user => {
      expect(user).toBeDefined();
      done();
    });
  });
});
```

## 📊 Exemple complet

```typescript
// ========== DOMAIN ==========

// entities/user.entity.ts
export class User {
  constructor(
    public readonly id: string,
    public readonly email: Email,
    public readonly status: UserStatus,
    public readonly createdAt: Date
  ) {}

  activate(): User {
    if (this.status === UserStatus.Active) {
      throw new Error('User is already active');
    }
    return new User(this.id, this.email, UserStatus.Active, this.createdAt);
  }

  deactivate(): User {
    return new User(this.id, this.email, UserStatus.Inactive, this.createdAt);
  }
}

// value-objects/email.ts
export class Email {
  private constructor(private readonly value: string) {}

  static create(email: string): Email {
    if (!this.isValid(email)) {
      throw new Error('Invalid email');
    }
    return new Email(email);
  }

  private static isValid(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }

  getValue(): string {
    return this.value;
  }
}

// repositories/user.repository.ts
export interface UserRepository {
  findById(id: string): Observable<User>;
  save(user: User): Observable<User>;
}

// ========== APPLICATION ==========

// use-cases/activate-user.use-case.ts
export class ActivateUserUseCase {
  constructor(private userRepository: UserRepository) {}

  execute(userId: string): Observable<User> {
    return this.userRepository.findById(userId).pipe(
      map(user => user.activate()),
      switchMap(user => this.userRepository.save(user))
    );
  }
}
```

## 🎓 Ressources

- [Clean Architecture - Uncle Bob](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Domain-Driven Design - Eric Evans](https://www.domainlanguage.com/ddd/)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)

---

**Le Core est le cœur stable de votre application. Investissez du temps pour le concevoir correctement ! 💎**
