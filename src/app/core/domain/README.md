# 💎 Domain - Logique métier pure

## 📋 Description

Le **Domain** est le **cœur absolu** de votre application. C'est la couche la plus importante qui contient toute la logique métier, les règles d'entreprise et les concepts du domaine.

> **Règle d'or** : Cette couche ne doit avoir AUCUNE dépendance vers les autres couches ou frameworks externes.

## 🏗️ Structure

```
domain/
├── entities/          # Objets métier avec identité
├── repositories/      # Interfaces pour l'accès aux données
├── value-objects/     # Objets immuables définis par leurs valeurs
└── README.md
```

## 📚 Composants du Domain

### 1. Entities (Entités)
Objets avec une **identité unique** et un **cycle de vie**

### 2. Value Objects (Objets valeur)
Objets **immuables** définis par leurs **valeurs**, pas leur identité

### 3. Repository Interfaces
**Contrats** définissant comment accéder aux entités

## ✅ Ce qu'on doit mettre

### Types de code autorisés
- ✅ Classes d'entités métier
- ✅ Value Objects
- ✅ Interfaces de repositories
- ✅ Enums métier
- ✅ Types TypeScript métier
- ✅ Logique de validation métier
- ✅ Règles d'entreprise
- ✅ Exceptions métier personnalisées

### Exemples de concepts métier
```typescript
// ✅ Entités
export class User { }
export class Order { }
export class Product { }
export class Invoice { }

// ✅ Value Objects
export class Email { }
export class Money { }
export class Address { }
export class DateRange { }

// ✅ Interfaces de repositories
export interface UserRepository { }
export interface OrderRepository { }

// ✅ Enums métier
export enum OrderStatus {
  Pending = 'PENDING',
  Confirmed = 'CONFIRMED',
  Shipped = 'SHIPPED',
  Delivered = 'DELIVERED'
}

// ✅ Exceptions métier
export class InsufficientStockError extends Error {
  constructor(productId: string, requested: number, available: number) {
    super(`Not enough stock for product ${productId}`);
  }
}
```

## ❌ Ce qu'on ne doit PAS mettre

### Dépendances interdites
- ❌ Imports Angular (`@angular/*`)
- ❌ HttpClient ou appels HTTP
- ❌ LocalStorage, SessionStorage
- ❌ DOM, Window, Document
- ❌ Router Angular
- ❌ Forms Angular
- ❌ Composants, Directives, Pipes
- ❌ Services Angular (@Injectable)
- ❌ Implémentations de repositories (→ Infrastructure)
- ❌ Logique de présentation (→ Presentation)

### Exceptions : Dépendances autorisées
- ✅ **RxJS** : Pour les Observables dans les interfaces
- ✅ **Librairies JS pures** : date-fns, lodash (si nécessaire)
- ✅ **Types TypeScript** : Pour le typage

```typescript
// ✅ CORRECT - Observable pour interface
import { Observable } from 'rxjs';

export interface UserRepository {
  findById(id: string): Observable<User>;
}

// ✅ CORRECT - Librairie JS pure
import { addDays } from 'date-fns';

export class Subscription {
  getExpirationDate(): Date {
    return addDays(this.startDate, this.durationInDays);
  }
}

// ❌ INCORRECT - Dépendance Angular
import { HttpClient } from '@angular/common/http'; // ❌ NON !
import { Injectable } from '@angular/core'; // ❌ NON !
```

## 🎯 Principes SOLID

### Single Responsibility Principle (SRP)
**Une classe = une seule raison de changer**

```typescript
// ✅ CORRECT - Responsabilité unique
export class User {
  constructor(
    public readonly id: string,
    public readonly email: Email,
    public readonly profile: UserProfile
  ) {}

  updateProfile(newProfile: UserProfile): User {
    return new User(this.id, this.email, newProfile);
  }
}

// ❌ INCORRECT - Trop de responsabilités
export class User {
  sendEmail() { }           // ❌ Responsabilité d'un service email
  saveToDatabase() { }      // ❌ Responsabilité d'un repository
  renderHTML() { }          // ❌ Responsabilité de la vue
  logActivity() { }         // ❌ Responsabilité d'un logger
}
```

### Open/Closed Principle (OCP)
**Ouvert à l'extension, fermé à la modification**

```typescript
// ✅ CORRECT - Extension par stratégie
export interface PricingStrategy {
  calculate(basePrice: number): number;
}

export class RegularPricing implements PricingStrategy {
  calculate(basePrice: number): number {
    return basePrice;
  }
}

export class DiscountPricing implements PricingStrategy {
  constructor(private discountPercent: number) {}

  calculate(basePrice: number): number {
    return basePrice * (1 - this.discountPercent / 100);
  }
}

export class Product {
  constructor(
    public readonly id: string,
    private readonly basePrice: number,
    private readonly pricingStrategy: PricingStrategy
  ) {}

  getPrice(): number {
    return this.pricingStrategy.calculate(this.basePrice);
  }
}

// ❌ INCORRECT - Modification nécessaire pour chaque nouveau type
export class Product {
  getPrice(): number {
    if (this.type === 'regular') {
      return this.basePrice;
    } else if (this.type === 'discounted') {
      return this.basePrice * 0.9;
    } else if (this.type === 'premium') {
      return this.basePrice * 1.2;
    }
    // ❌ Besoin de modifier cette méthode pour chaque nouveau type
  }
}
```

### Liskov Substitution Principle (LSP)
**Les sous-types doivent être substituables à leur type de base**

```typescript
// ✅ CORRECT - Respecte le contrat
export abstract class Account {
  constructor(protected balance: number) {}

  abstract withdraw(amount: number): void;

  getBalance(): number {
    return this.balance;
  }
}

export class SavingsAccount extends Account {
  withdraw(amount: number): void {
    if (amount > this.balance) {
      throw new Error('Insufficient funds');
    }
    this.balance -= amount;
  }
}

export class CheckingAccount extends Account {
  withdraw(amount: number): void {
    if (amount > this.balance + this.overdraftLimit) {
      throw new Error('Exceeds overdraft limit');
    }
    this.balance -= amount;
  }
}

// ❌ INCORRECT - Viole le contrat
export class FixedDepositAccount extends Account {
  withdraw(amount: number): void {
    throw new Error('Cannot withdraw from fixed deposit'); // ❌ Viole LSP
  }
}
```

### Interface Segregation Principle (ISP)
**Interfaces spécifiques plutôt que générales**

```typescript
// ✅ CORRECT - Interfaces ségrégées
export interface Readable<T> {
  findById(id: string): Observable<T>;
  findAll(): Observable<T[]>;
}

export interface Writable<T> {
  save(entity: T): Observable<T>;
  delete(id: string): Observable<void>;
}

export interface Searchable<T> {
  search(criteria: SearchCriteria): Observable<T[]>;
}

// Le repository peut implémenter uniquement ce dont il a besoin
export interface UserRepository extends Readable<User>, Writable<User> { }

// Un use case de lecture n'a besoin que de Readable
export class GetUserUseCase {
  constructor(private repository: Readable<User>) {} // ✅ Seulement lecture
}

// ❌ INCORRECT - Interface monolithique
export interface Repository<T> {
  findById(id: string): Observable<T>;
  findAll(): Observable<T[]>;
  save(entity: T): Observable<T>;
  delete(id: string): Observable<void>;
  search(criteria: any): Observable<T[]>;
  bulkInsert(entities: T[]): Observable<void>;
  updatePartial(id: string, data: any): Observable<T>;
  count(): Observable<number>;
  exists(id: string): Observable<boolean>;
  // ❌ Trop de méthodes, tous les clients doivent dépendre de toutes
}
```

### Dependency Inversion Principle (DIP)
**Dépendre des abstractions, pas des concrétions**

```typescript
// ✅ CORRECT - Interface dans Domain, implémentation dans Infrastructure
// domain/repositories/user.repository.ts
export interface UserRepository {
  findById(id: string): Observable<User>;
  save(user: User): Observable<User>;
}

// domain/entities/user.entity.ts
export class User {
  // Logique métier pure
}

// application/use-cases/get-user.use-case.ts
export class GetUserUseCase {
  constructor(private userRepository: UserRepository) {} // ✅ Dépend de l'abstraction

  execute(id: string): Observable<User> {
    return this.userRepository.findById(id);
  }
}

// infrastructure/repositories/user-repository.impl.ts (autre couche)
export class UserRepositoryImpl implements UserRepository {
  // Implémentation concrète
}

// ❌ INCORRECT - Dépendance concrète
import { UserRepositoryImpl } from '@infrastructure/...'; // ❌ NON !

export class GetUserUseCase {
  constructor(private userRepository: UserRepositoryImpl) {} // ❌ Dépend d'une implémentation
}
```

## 📖 Bonnes pratiques

### 1. Immutabilité

Les entités et value objects doivent être **immuables** autant que possible.

```typescript
// ✅ CORRECT - Immutable
export class User {
  constructor(
    public readonly id: string,
    public readonly email: Email,
    public readonly name: string
  ) {}

  // Retourne une nouvelle instance
  changeName(newName: string): User {
    return new User(this.id, this.email, newName);
  }

  changeEmail(newEmail: Email): User {
    return new User(this.id, newEmail, this.name);
  }
}

// Utilisation
const user = new User('1', email, 'John');
const updatedUser = user.changeName('Jane'); // ✅ Nouvelle instance
console.log(user.name); // 'John' - original inchangé
console.log(updatedUser.name); // 'Jane'

// ❌ INCORRECT - Mutable
export class User {
  public id: string;
  public email: string;
  public name: string;

  changeName(newName: string): void {
    this.name = newName; // ❌ Mutation
  }
}
```

### 2. Validation dans les constructeurs ou factory methods

```typescript
// ✅ CORRECT - Validation dans factory method
export class Email {
  private constructor(private readonly value: string) {}

  static create(email: string): Email {
    if (!email || email.trim() === '') {
      throw new Error('Email cannot be empty');
    }
    if (!email.includes('@')) {
      throw new Error('Invalid email format');
    }
    if (email.length > 255) {
      throw new Error('Email too long');
    }
    return new Email(email.toLowerCase().trim());
  }

  getValue(): string {
    return this.value;
  }

  equals(other: Email): boolean {
    return this.value === other.value;
  }
}

// Utilisation
const email = Email.create('user@example.com'); // ✅ Valide
const invalid = Email.create('invalid'); // ❌ Lance une exception

// ✅ CORRECT - Validation dans le constructeur
export class Money {
  constructor(
    public readonly amount: number,
    public readonly currency: string
  ) {
    if (amount < 0) {
      throw new Error('Amount cannot be negative');
    }
    if (!currency || currency.length !== 3) {
      throw new Error('Invalid currency code');
    }
  }

  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('Cannot add different currencies');
    }
    return new Money(this.amount + other.amount, this.currency);
  }
}
```

### 3. Logique métier dans les entités

Les entités doivent contenir leur propre logique métier.

```typescript
// ✅ CORRECT - Logique métier dans l'entité
export class Order {
  constructor(
    public readonly id: string,
    public readonly items: OrderItem[],
    public readonly status: OrderStatus,
    public readonly createdAt: Date
  ) {}

  calculateTotal(): Money {
    return this.items.reduce(
      (total, item) => total.add(item.getSubtotal()),
      new Money(0, 'EUR')
    );
  }

  canBeCancelled(): boolean {
    return this.status === OrderStatus.Pending ||
           this.status === OrderStatus.Confirmed;
  }

  cancel(): Order {
    if (!this.canBeCancelled()) {
      throw new Error('Order cannot be cancelled in current status');
    }
    return new Order(this.id, this.items, OrderStatus.Cancelled, this.createdAt);
  }

  confirm(): Order {
    if (this.status !== OrderStatus.Pending) {
      throw new Error('Only pending orders can be confirmed');
    }
    if (this.items.length === 0) {
      throw new Error('Cannot confirm empty order');
    }
    return new Order(this.id, this.items, OrderStatus.Confirmed, this.createdAt);
  }
}

// ❌ INCORRECT - Logique métier dans un service
export class Order {
  public id: string;
  public items: OrderItem[];
  public status: OrderStatus;
}

export class OrderService {
  // ❌ La logique métier devrait être dans l'entité
  calculateTotal(order: Order): Money { /* ... */ }
  canBeCancelled(order: Order): boolean { /* ... */ }
  cancel(order: Order): void { /* ... */ }
}
```

### 4. Value Objects pour concepts métier

Encapsulez les concepts métier dans des Value Objects.

```typescript
// ✅ CORRECT - Value Objects
export class DateRange {
  private constructor(
    public readonly start: Date,
    public readonly end: Date
  ) {}

  static create(start: Date, end: Date): DateRange {
    if (start >= end) {
      throw new Error('Start date must be before end date');
    }
    return new DateRange(start, end);
  }

  contains(date: Date): boolean {
    return date >= this.start && date <= this.end;
  }

  overlaps(other: DateRange): boolean {
    return this.start <= other.end && this.end >= other.start;
  }

  getDurationInDays(): number {
    return Math.ceil((this.end.getTime() - this.start.getTime()) / (1000 * 60 * 60 * 24));
  }
}

export class Address {
  private constructor(
    public readonly street: string,
    public readonly city: string,
    public readonly postalCode: string,
    public readonly country: string
  ) {}

  static create(street: string, city: string, postalCode: string, country: string): Address {
    if (!street || !city || !postalCode || !country) {
      throw new Error('All address fields are required');
    }
    return new Address(street.trim(), city.trim(), postalCode.trim(), country.trim());
  }

  getFullAddress(): string {
    return `${this.street}, ${this.postalCode} ${this.city}, ${this.country}`;
  }

  equals(other: Address): boolean {
    return this.street === other.street &&
           this.city === other.city &&
           this.postalCode === other.postalCode &&
           this.country === other.country;
  }
}

// ❌ INCORRECT - Primitives au lieu de Value Objects
export class User {
  constructor(
    public id: string,
    public email: string,           // ❌ Devrait être Email
    public street: string,          // ❌ Devrait être Address
    public city: string,
    public postalCode: string,
    public country: string
  ) {}
}
```

### 5. Interfaces de repositories minimalistes

```typescript
// ✅ CORRECT - Interface simple et claire
export interface UserRepository {
  findById(id: string): Observable<User | null>;
  findByEmail(email: Email): Observable<User | null>;
  save(user: User): Observable<User>;
  delete(id: string): Observable<void>;
}

// ✅ CORRECT - Interface spécialisée
export interface ProductRepository {
  findById(id: string): Observable<Product | null>;
  findByCategory(category: string): Observable<Product[]>;
  search(criteria: ProductSearchCriteria): Observable<Product[]>;
  save(product: Product): Observable<Product>;
}

// ❌ INCORRECT - Interface trop générique ou complexe
export interface Repository<T> {
  find(query: any): Observable<T[]>;  // ❌ Trop vague
  execute(sql: string): Observable<any>; // ❌ Détail d'implémentation
}
```

## 📊 Exemple complet : Système de commandes

```typescript
// ========== VALUE OBJECTS ==========

// value-objects/money.ts
export class Money {
  private constructor(
    public readonly amount: number,
    public readonly currency: string
  ) {}

  static create(amount: number, currency: string): Money {
    if (amount < 0) {
      throw new Error('Amount cannot be negative');
    }
    return new Money(amount, currency);
  }

  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('Cannot add different currencies');
    }
    return Money.create(this.amount + other.amount, this.currency);
  }

  multiply(factor: number): Money {
    return Money.create(this.amount * factor, this.currency);
  }

  equals(other: Money): boolean {
    return this.amount === other.amount && this.currency === other.currency;
  }
}

// value-objects/email.ts
export class Email {
  private constructor(private readonly value: string) {}

  static create(email: string): Email {
    if (!email.includes('@')) {
      throw new Error('Invalid email');
    }
    return new Email(email.toLowerCase());
  }

  getValue(): string {
    return this.value;
  }
}

// ========== ENTITIES ==========

// entities/order-item.ts
export class OrderItem {
  constructor(
    public readonly productId: string,
    public readonly productName: string,
    public readonly price: Money,
    public readonly quantity: number
  ) {
    if (quantity <= 0) {
      throw new Error('Quantity must be positive');
    }
  }

  getSubtotal(): Money {
    return this.price.multiply(this.quantity);
  }
}

// entities/order.ts
export enum OrderStatus {
  Pending = 'PENDING',
  Confirmed = 'CONFIRMED',
  Shipped = 'SHIPPED',
  Delivered = 'DELIVERED',
  Cancelled = 'CANCELLED'
}

export class Order {
  constructor(
    public readonly id: string,
    public readonly customerId: string,
    public readonly items: readonly OrderItem[],
    public readonly status: OrderStatus,
    public readonly createdAt: Date
  ) {
    if (items.length === 0) {
      throw new Error('Order must have at least one item');
    }
  }

  calculateTotal(): Money {
    return this.items.reduce(
      (total, item) => total.add(item.getSubtotal()),
      Money.create(0, this.items[0].price.currency)
    );
  }

  canBeCancelled(): boolean {
    return this.status === OrderStatus.Pending ||
           this.status === OrderStatus.Confirmed;
  }

  cancel(): Order {
    if (!this.canBeCancelled()) {
      throw new Error(`Cannot cancel order with status ${this.status}`);
    }
    return new Order(
      this.id,
      this.customerId,
      this.items,
      OrderStatus.Cancelled,
      this.createdAt
    );
  }

  confirm(): Order {
    if (this.status !== OrderStatus.Pending) {
      throw new Error('Only pending orders can be confirmed');
    }
    return new Order(
      this.id,
      this.customerId,
      this.items,
      OrderStatus.Confirmed,
      this.createdAt
    );
  }

  ship(): Order {
    if (this.status !== OrderStatus.Confirmed) {
      throw new Error('Only confirmed orders can be shipped');
    }
    return new Order(
      this.id,
      this.customerId,
      this.items,
      OrderStatus.Shipped,
      this.createdAt
    );
  }
}

// entities/customer.ts
export class Customer {
  constructor(
    public readonly id: string,
    public readonly email: Email,
    public readonly name: string,
    public readonly isActive: boolean
  ) {}

  deactivate(): Customer {
    return new Customer(this.id, this.email, this.name, false);
  }

  activate(): Customer {
    return new Customer(this.id, this.email, this.name, true);
  }
}

// ========== REPOSITORY INTERFACES ==========

// repositories/order.repository.ts
export interface OrderRepository {
  findById(id: string): Observable<Order | null>;
  findByCustomerId(customerId: string): Observable<Order[]>;
  save(order: Order): Observable<Order>;
  delete(id: string): Observable<void>;
}

// repositories/customer.repository.ts
export interface CustomerRepository {
  findById(id: string): Observable<Customer | null>;
  findByEmail(email: Email): Observable<Customer | null>;
  save(customer: Customer): Observable<Customer>;
}
```

## 🧪 Tests du Domain

Le Domain doit être testable **sans aucune dépendance Angular**.

```typescript
// order.entity.spec.ts
describe('Order', () => {
  const money = Money.create(10, 'EUR');
  const item = new OrderItem('1', 'Product', money, 2);

  describe('calculateTotal', () => {
    it('should calculate total correctly', () => {
      const order = new Order('1', 'customer-1', [item], OrderStatus.Pending, new Date());
      const total = order.calculateTotal();

      expect(total.amount).toBe(20);
      expect(total.currency).toBe('EUR');
    });
  });

  describe('cancel', () => {
    it('should cancel pending order', () => {
      const order = new Order('1', 'customer-1', [item], OrderStatus.Pending, new Date());
      const cancelled = order.cancel();

      expect(cancelled.status).toBe(OrderStatus.Cancelled);
    });

    it('should throw error when cancelling shipped order', () => {
      const order = new Order('1', 'customer-1', [item], OrderStatus.Shipped, new Date());

      expect(() => order.cancel()).toThrow();
    });
  });
});
```

## 🎓 Points clés à retenir

1. **Aucune dépendance externe** : Le Domain ne doit dépendre de rien
2. **Logique métier** : Toute la logique métier doit être ici
3. **Immutabilité** : Privilégiez les objets immuables
4. **Validation** : Validez dans les constructeurs/factory methods
5. **Value Objects** : Encapsulez les concepts métier
6. **Testabilité** : Doit être testable sans framework
7. **Expressivité** : Le code doit refléter le langage métier

---

**Le Domain est le cœur de votre application. C'est ici que la vraie valeur métier réside ! 💎**
