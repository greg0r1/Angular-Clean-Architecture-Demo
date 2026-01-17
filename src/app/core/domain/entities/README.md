# 🎭 Entities - Entités métier

## 📋 Description

Les **Entities** (entités) sont des **objets métier avec une identité unique** qui persiste dans le temps, même si leurs attributs changent. Elles représentent les concepts centraux de votre domaine métier.

> **Caractéristique clé** : Une entité est définie par son **identité** (ID), pas par ses attributs.

## 🆔 Identité vs Égalité

### Entité = Identité

Deux entités sont égales si elles ont **le même ID**, même si leurs attributs diffèrent.

```typescript
const user1 = new User('123', 'john@example.com', 'John');
const user2 = new User('123', 'jane@example.com', 'Jane');

// Même ID → Même entité (même si nom et email différents)
console.log(user1.id === user2.id); // true → Même utilisateur
```

### Value Object = Valeur

Deux value objects sont égaux si **toutes leurs valeurs** sont identiques.

```typescript
const email1 = Email.create('john@example.com');
const email2 = Email.create('john@example.com');

// Mêmes valeurs → Égaux (pas d'ID)
console.log(email1.equals(email2)); // true
```

## ✅ Ce qu'on doit mettre

### Types d'entités
- ✅ Objets métier avec identité (User, Order, Product, Invoice...)
- ✅ Logique métier liée à l'entité
- ✅ Méthodes de modification (retournant une nouvelle instance)
- ✅ Règles de validation métier
- ✅ Calculs métier
- ✅ Règles d'invariance

### Exemples
```typescript
// ✅ Entités typiques
export class User { }
export class Order { }
export class Product { }
export class Customer { }
export class Invoice { }
export class Subscription { }
export class Article { }
export class Comment { }
export class Transaction { }
```

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Annotations Angular (`@Injectable`, `@Component`)
- ❌ Logique de persistance (save, load...)
- ❌ Appels HTTP ou API
- ❌ Accès au localStorage
- ❌ Logique de présentation (formatage pour l'UI)
- ❌ Dépendances à des services externes
- ❌ Mutations directes (préférer l'immutabilité)

## 🏗️ Structure d'une entité

### Template de base

```typescript
export class EntityName {
  // 1. Propriétés readonly (immutabilité)
  constructor(
    public readonly id: string,              // ✅ ID obligatoire
    public readonly property1: ValueObject,  // ✅ Value objects
    public readonly property2: string,
    public readonly createdAt: Date
  ) {
    // 2. Validation dans le constructeur
    this.validate();
  }

  // 3. Méthode de validation privée
  private validate(): void {
    if (!this.id) {
      throw new Error('ID is required');
    }
    // Autres validations...
  }

  // 4. Logique métier (retourne nouvelle instance)
  updateProperty(newValue: ValueObject): EntityName {
    return new EntityName(
      this.id,
      newValue,
      this.property2,
      this.createdAt
    );
  }

  // 5. Règles métier
  canBeDeleted(): boolean {
    // Logique métier
    return true;
  }

  // 6. Calculs métier
  calculateSomething(): number {
    // Logique de calcul
    return 0;
  }
}
```

## 📖 Principes SOLID pour les entités

### Single Responsibility (SRP)

Une entité = Une responsabilité métier claire

```typescript
// ✅ CORRECT - Responsabilité : Gérer un utilisateur
export class User {
  constructor(
    public readonly id: string,
    public readonly email: Email,
    public readonly profile: UserProfile,
    public readonly status: UserStatus
  ) {}

  activate(): User {
    if (this.status === UserStatus.Active) {
      throw new Error('User already active');
    }
    return new User(this.id, this.email, this.profile, UserStatus.Active);
  }

  updateProfile(newProfile: UserProfile): User {
    return new User(this.id, this.email, newProfile, this.status);
  }

  canLogin(): boolean {
    return this.status === UserStatus.Active;
  }
}

// ❌ INCORRECT - Trop de responsabilités
export class User {
  sendEmail() { }              // ❌ Responsabilité du service email
  calculateTaxes() { }         // ❌ Responsabilité du service fiscal
  generatePDF() { }            // ❌ Responsabilité du service PDF
  encryptPassword() { }        // ❌ Responsabilité du service crypto
}
```

### Open/Closed Principle (OCP)

Entité ouverte à l'extension, fermée à la modification

```typescript
// ✅ CORRECT - Extension via composition
export enum DiscountType {
  None,
  Percentage,
  Fixed
}

export class Discount {
  constructor(
    public readonly type: DiscountType,
    public readonly value: number
  ) {}

  apply(price: Money): Money {
    switch (this.type) {
      case DiscountType.Percentage:
        return price.multiply(1 - this.value / 100);
      case DiscountType.Fixed:
        return Money.create(Math.max(0, price.amount - this.value), price.currency);
      default:
        return price;
    }
  }
}

export class Product {
  constructor(
    public readonly id: string,
    public readonly name: string,
    public readonly basePrice: Money,
    public readonly discount: Discount
  ) {}

  getPrice(): Money {
    return this.discount.apply(this.basePrice);
  }
}
```

### Liskov Substitution (LSP)

Les sous-classes doivent respecter le contrat de la classe de base

```typescript
// ✅ CORRECT
export abstract class Account {
  constructor(
    public readonly id: string,
    protected balance: number
  ) {}

  abstract canWithdraw(amount: number): boolean;

  getBalance(): number {
    return this.balance;
  }
}

export class SavingsAccount extends Account {
  canWithdraw(amount: number): boolean {
    return amount <= this.balance; // ✅ Respecte le contrat
  }
}

export class CheckingAccount extends Account {
  constructor(
    id: string,
    balance: number,
    private overdraftLimit: number
  ) {
    super(id, balance);
  }

  canWithdraw(amount: number): boolean {
    return amount <= this.balance + this.overdraftLimit; // ✅ Respecte le contrat
  }
}
```

## 📚 Bonnes pratiques

### 1. Immutabilité

Toujours retourner une nouvelle instance au lieu de muter.

```typescript
// ✅ CORRECT - Immutable
export class Order {
  constructor(
    public readonly id: string,
    public readonly items: readonly OrderItem[],
    public readonly status: OrderStatus
  ) {}

  addItem(item: OrderItem): Order {
    return new Order(
      this.id,
      [...this.items, item], // ✅ Nouveau tableau
      this.status
    );
  }

  confirm(): Order {
    return new Order(this.id, this.items, OrderStatus.Confirmed);
  }
}

// Utilisation
const order = new Order('1', [], OrderStatus.Pending);
const orderWithItem = order.addItem(item); // ✅ Nouvelle instance
const confirmedOrder = orderWithItem.confirm(); // ✅ Nouvelle instance

// ❌ INCORRECT - Mutable
export class Order {
  public id: string;
  public items: OrderItem[];
  public status: OrderStatus;

  addItem(item: OrderItem): void {
    this.items.push(item); // ❌ Mutation
  }

  confirm(): void {
    this.status = OrderStatus.Confirmed; // ❌ Mutation
  }
}
```

### 2. Validation dans le constructeur

```typescript
// ✅ CORRECT - Validation au constructeur
export class Product {
  constructor(
    public readonly id: string,
    public readonly name: string,
    public readonly price: Money,
    public readonly stock: number
  ) {
    if (!id || id.trim() === '') {
      throw new Error('Product ID is required');
    }
    if (!name || name.trim() === '') {
      throw new Error('Product name is required');
    }
    if (stock < 0) {
      throw new Error('Stock cannot be negative');
    }
  }

  // L'entité est TOUJOURS dans un état valide
  decreaseStock(quantity: number): Product {
    const newStock = this.stock - quantity;
    if (newStock < 0) {
      throw new Error('Insufficient stock');
    }
    return new Product(this.id, this.name, this.price, newStock);
  }
}

// Impossible de créer une entité invalide
const product = new Product('', 'Name', price, -5); // ❌ Lance une exception
```

### 3. Logique métier dans l'entité

La logique métier appartient à l'entité, pas à un service.

```typescript
// ✅ CORRECT - Logique métier dans l'entité
export class Invoice {
  constructor(
    public readonly id: string,
    public readonly items: readonly InvoiceItem[],
    public readonly issueDate: Date,
    public readonly dueDate: Date,
    public readonly isPaid: boolean
  ) {}

  calculateTotal(): Money {
    return this.items.reduce(
      (sum, item) => sum.add(item.getTotal()),
      Money.create(0, 'EUR')
    );
  }

  calculateTax(taxRate: number): Money {
    return this.calculateTotal().multiply(taxRate);
  }

  isOverdue(currentDate: Date): boolean {
    return !this.isPaid && currentDate > this.dueDate;
  }

  getDaysUntilDue(currentDate: Date): number {
    const diff = this.dueDate.getTime() - currentDate.getTime();
    return Math.ceil(diff / (1000 * 60 * 60 * 24));
  }

  markAsPaid(): Invoice {
    if (this.isPaid) {
      throw new Error('Invoice already paid');
    }
    return new Invoice(
      this.id,
      this.items,
      this.issueDate,
      this.dueDate,
      true
    );
  }
}

// ❌ INCORRECT - Logique métier dans un service
export class Invoice {
  public id: string;
  public items: InvoiceItem[];
  public isPaid: boolean;
  // ... juste des données
}

export class InvoiceService {
  // ❌ La logique devrait être dans l'entité
  calculateTotal(invoice: Invoice): Money { }
  isOverdue(invoice: Invoice, date: Date): boolean { }
  markAsPaid(invoice: Invoice): void { }
}
```

### 4. Utiliser des Value Objects

Encapsulez les concepts métier dans des Value Objects.

```typescript
// ✅ CORRECT - Value Objects pour concepts métier
export class User {
  constructor(
    public readonly id: string,
    public readonly email: Email,           // ✅ Value Object
    public readonly password: Password,     // ✅ Value Object
    public readonly birthDate: BirthDate,   // ✅ Value Object
    public readonly address: Address        // ✅ Value Object
  ) {}

  changeEmail(newEmail: Email): User {
    return new User(
      this.id,
      newEmail,
      this.password,
      this.birthDate,
      this.address
    );
  }

  getAge(): number {
    return this.birthDate.calculateAge();
  }
}

// ❌ INCORRECT - Primitives obsession
export class User {
  constructor(
    public readonly id: string,
    public readonly email: string,          // ❌ Primitive
    public readonly password: string,       // ❌ Primitive
    public readonly birthDay: number,       // ❌ Primitive
    public readonly birthMonth: number,     // ❌ Primitive
    public readonly birthYear: number,      // ❌ Primitive
    public readonly street: string,         // ❌ Primitive
    public readonly city: string,           // ❌ Primitive
    public readonly postalCode: string      // ❌ Primitive
  ) {}
}
```

### 5. Factory methods pour construction complexe

```typescript
// ✅ CORRECT - Factory methods
export class Order {
  private constructor(
    public readonly id: string,
    public readonly customerId: string,
    public readonly items: readonly OrderItem[],
    public readonly status: OrderStatus,
    public readonly createdAt: Date
  ) {}

  // Factory method pour créer une nouvelle commande
  static createNew(customerId: string, items: OrderItem[]): Order {
    if (items.length === 0) {
      throw new Error('Order must have at least one item');
    }
    return new Order(
      generateId(), // Génération d'ID
      customerId,
      items,
      OrderStatus.Pending,
      new Date()
    );
  }

  // Factory method pour reconstruire depuis la DB
  static fromPersistence(
    id: string,
    customerId: string,
    items: OrderItem[],
    status: OrderStatus,
    createdAt: Date
  ): Order {
    return new Order(id, customerId, items, status, createdAt);
  }
}

// Utilisation
const newOrder = Order.createNew('customer-123', [item1, item2]);
const existingOrder = Order.fromPersistence('order-1', 'customer-123', items, status, date);
```

### 6. Méthodes de requête (queries) et commandes

```typescript
export class Article {
  constructor(
    public readonly id: string,
    public readonly title: string,
    public readonly content: string,
    public readonly publishedAt: Date | null,
    public readonly tags: readonly string[]
  ) {}

  // ✅ Queries (retourne des informations)
  isPublished(): boolean {
    return this.publishedAt !== null;
  }

  isDraft(): boolean {
    return this.publishedAt === null;
  }

  hasTag(tag: string): boolean {
    return this.tags.includes(tag);
  }

  getWordCount(): number {
    return this.content.split(/\s+/).length;
  }

  // ✅ Commands (retourne une nouvelle instance)
  publish(): Article {
    if (this.isPublished()) {
      throw new Error('Article already published');
    }
    return new Article(
      this.id,
      this.title,
      this.content,
      new Date(),
      this.tags
    );
  }

  addTag(tag: string): Article {
    if (this.hasTag(tag)) {
      throw new Error('Tag already exists');
    }
    return new Article(
      this.id,
      this.title,
      this.content,
      this.publishedAt,
      [...this.tags, tag]
    );
  }
}
```

## 📊 Exemples complets

### Exemple 1 : Système de réservation

```typescript
export enum BookingStatus {
  Pending = 'PENDING',
  Confirmed = 'CONFIRMED',
  Cancelled = 'CANCELLED',
  Completed = 'COMPLETED'
}

export class Booking {
  constructor(
    public readonly id: string,
    public readonly customerId: string,
    public readonly resourceId: string,
    public readonly dateRange: DateRange,
    public readonly status: BookingStatus,
    public readonly createdAt: Date
  ) {
    this.validate();
  }

  private validate(): void {
    if (!this.id) throw new Error('Booking ID is required');
    if (!this.customerId) throw new Error('Customer ID is required');
    if (!this.resourceId) throw new Error('Resource ID is required');
  }

  static createNew(
    customerId: string,
    resourceId: string,
    dateRange: DateRange
  ): Booking {
    return new Booking(
      generateId(),
      customerId,
      resourceId,
      dateRange,
      BookingStatus.Pending,
      new Date()
    );
  }

  canBeCancelled(): boolean {
    return this.status === BookingStatus.Pending ||
           this.status === BookingStatus.Confirmed;
  }

  cancel(): Booking {
    if (!this.canBeCancelled()) {
      throw new Error(`Cannot cancel booking with status ${this.status}`);
    }
    return new Booking(
      this.id,
      this.customerId,
      this.resourceId,
      this.dateRange,
      BookingStatus.Cancelled,
      this.createdAt
    );
  }

  confirm(): Booking {
    if (this.status !== BookingStatus.Pending) {
      throw new Error('Only pending bookings can be confirmed');
    }
    return new Booking(
      this.id,
      this.customerId,
      this.resourceId,
      this.dateRange,
      BookingStatus.Confirmed,
      this.createdAt
    );
  }

  complete(): Booking {
    if (this.status !== BookingStatus.Confirmed) {
      throw new Error('Only confirmed bookings can be completed');
    }
    return new Booking(
      this.id,
      this.customerId,
      this.resourceId,
      this.dateRange,
      BookingStatus.Completed,
      this.createdAt
    );
  }

  overlaps(other: Booking): boolean {
    if (this.resourceId !== other.resourceId) {
      return false;
    }
    return this.dateRange.overlaps(other.dateRange);
  }
}
```

### Exemple 2 : Système de panier

```typescript
export class CartItem {
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

  changeQuantity(newQuantity: number): CartItem {
    if (newQuantity <= 0) {
      throw new Error('Quantity must be positive');
    }
    return new CartItem(
      this.productId,
      this.productName,
      this.price,
      newQuantity
    );
  }
}

export class Cart {
  constructor(
    public readonly id: string,
    public readonly customerId: string,
    public readonly items: readonly CartItem[]
  ) {}

  static createEmpty(customerId: string): Cart {
    return new Cart(generateId(), customerId, []);
  }

  isEmpty(): boolean {
    return this.items.length === 0;
  }

  getItemCount(): number {
    return this.items.reduce((sum, item) => sum + item.quantity, 0);
  }

  getTotal(): Money {
    if (this.isEmpty()) {
      return Money.create(0, 'EUR');
    }
    return this.items.reduce(
      (sum, item) => sum.add(item.getSubtotal()),
      Money.create(0, this.items[0].price.currency)
    );
  }

  hasProduct(productId: string): boolean {
    return this.items.some(item => item.productId === productId);
  }

  addItem(item: CartItem): Cart {
    if (this.hasProduct(item.productId)) {
      // Augmenter la quantité si le produit existe déjà
      return this.updateQuantity(item.productId,
        this.getItem(item.productId)!.quantity + item.quantity
      );
    }
    return new Cart(this.id, this.customerId, [...this.items, item]);
  }

  removeItem(productId: string): Cart {
    const newItems = this.items.filter(item => item.productId !== productId);
    return new Cart(this.id, this.customerId, newItems);
  }

  updateQuantity(productId: string, newQuantity: number): Cart {
    const newItems = this.items.map(item =>
      item.productId === productId
        ? item.changeQuantity(newQuantity)
        : item
    );
    return new Cart(this.id, this.customerId, newItems);
  }

  clear(): Cart {
    return new Cart(this.id, this.customerId, []);
  }

  private getItem(productId: string): CartItem | undefined {
    return this.items.find(item => item.productId === productId);
  }
}
```

## 🧪 Tests d'entités

Les entités doivent être testées sans aucune dépendance Angular.

```typescript
describe('Cart', () => {
  const price = Money.create(10, 'EUR');
  const item = new CartItem('1', 'Product 1', price, 2);

  it('should create empty cart', () => {
    const cart = Cart.createEmpty('customer-1');

    expect(cart.isEmpty()).toBe(true);
    expect(cart.getItemCount()).toBe(0);
  });

  it('should add item to cart', () => {
    const cart = Cart.createEmpty('customer-1');
    const cartWithItem = cart.addItem(item);

    expect(cartWithItem.isEmpty()).toBe(false);
    expect(cartWithItem.getItemCount()).toBe(2);
    expect(cartWithItem.hasProduct('1')).toBe(true);
  });

  it('should calculate total correctly', () => {
    const cart = Cart.createEmpty('customer-1').addItem(item);
    const total = cart.getTotal();

    expect(total.amount).toBe(20); // 10 * 2
  });

  it('should increase quantity when adding existing product', () => {
    const cart = Cart.createEmpty('customer-1')
      .addItem(item)
      .addItem(new CartItem('1', 'Product 1', price, 3));

    expect(cart.getItemCount()).toBe(5); // 2 + 3
  });

  it('should remove item from cart', () => {
    const cart = Cart.createEmpty('customer-1')
      .addItem(item)
      .removeItem('1');

    expect(cart.isEmpty()).toBe(true);
  });
});
```

## 🎓 Points clés

1. **Identité** : Chaque entité a un ID unique
2. **Immutabilité** : Retourner de nouvelles instances
3. **Validation** : Toujours dans un état valide
4. **Logique métier** : Contient les règles métier
5. **Value Objects** : Utiliser pour les concepts métier
6. **Testabilité** : Testable sans framework
7. **Indépendance** : Aucune dépendance technique

---

**Les entités sont le cœur de votre domaine métier. Prenez soin de les modéliser correctement ! 🎭**
