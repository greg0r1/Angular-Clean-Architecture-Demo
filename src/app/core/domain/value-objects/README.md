# 💎 Value Objects - Objets valeur

## 📋 Description

Les **Value Objects** sont des objets **immuables** définis par leurs **valeurs**, pas par une identité. Deux value objects avec les mêmes valeurs sont considérés comme égaux.

> **Caractéristique clé** : Un Value Object est défini par **CE QU'IL EST** (ses valeurs), pas par **QUI IL EST** (identité).

## 🆚 Value Object vs Entity

### Value Object
- ✅ **Pas d'identité** (pas d'ID)
- ✅ **Immuable** (ne change jamais)
- ✅ Égalité basée sur les **valeurs**
- ✅ Peut être **remplacé** librement

```typescript
const email1 = Email.create('john@example.com');
const email2 = Email.create('john@example.com');

email1.equals(email2); // true - mêmes valeurs = égaux
```

### Entity
- ✅ **A une identité** (ID unique)
- ✅ **Mutable** (peut changer)
- ✅ Égalité basée sur **l'identité**
- ✅ Suit un **cycle de vie**

```typescript
const user1 = new User('123', email, 'John');
const user2 = new User('123', email, 'Jane');

user1.id === user2.id; // true - même ID = même entité
// (même si nom différent)
```

## ✅ Ce qu'on doit mettre

### Types de Value Objects
- ✅ Concepts métier simples (Email, Money, Address...)
- ✅ Mesures et quantités (Weight, Distance, Temperature...)
- ✅ Plages et intervalles (DateRange, PriceRange...)
- ✅ Identifiants typés (UserId, OrderId...)
- ✅ Formats standardisés (PhoneNumber, ISBN, IBAN...)
- ✅ Règles de validation encapsulées

### Exemples
```typescript
// ✅ Value Objects typiques
export class Email { }
export class Money { }
export class Address { }
export class PhoneNumber { }
export class DateRange { }
export class Quantity { }
export class Percentage { }
export class Password { }
export class URL { }
export class Color { }
```

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Objets avec identité (→ Entities)
- ❌ Objets mutables
- ❌ Logique de persistance
- ❌ Dépendances à Angular
- ❌ Appels HTTP
- ❌ Accès au DOM

## 🏗️ Structure d'un Value Object

### Template de base

```typescript
export class ValueObjectName {
  // 1. Constructeur PRIVÉ
  private constructor(
    public readonly value1: string,
    public readonly value2: number
  ) {}

  // 2. Factory method statique avec validation
  static create(value1: string, value2: number): ValueObjectName {
    // Validation
    if (!value1) {
      throw new Error('Value1 is required');
    }
    if (value2 < 0) {
      throw new Error('Value2 must be positive');
    }

    return new ValueObjectName(value1, value2);
  }

  // 3. Méthode equals
  equals(other: ValueObjectName): boolean {
    return this.value1 === other.value1 &&
           this.value2 === other.value2;
  }

  // 4. Méthodes métier (retournent de nouvelles instances)
  transform(): ValueObjectName {
    return ValueObjectName.create(/* nouveaux values */);
  }

  // 5. Getters si nécessaire
  getValue(): string {
    return this.value1;
  }
}
```

## 📖 Caractéristiques essentielles

### 1. Immutabilité

Les Value Objects ne changent JAMAIS. Toute "modification" crée une nouvelle instance.

```typescript
// ✅ CORRECT - Immutable
export class Money {
  private constructor(
    public readonly amount: number,
    public readonly currency: string
  ) {}

  static create(amount: number, currency: string): Money {
    if (amount < 0) throw new Error('Amount cannot be negative');
    if (!currency) throw new Error('Currency is required');
    return new Money(amount, currency);
  }

  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('Cannot add different currencies');
    }
    // ✅ Retourne une NOUVELLE instance
    return Money.create(this.amount + other.amount, this.currency);
  }

  multiply(factor: number): Money {
    // ✅ Retourne une NOUVELLE instance
    return Money.create(this.amount * factor, this.currency);
  }

  equals(other: Money): boolean {
    return this.amount === other.amount &&
           this.currency === other.currency;
  }
}

// Utilisation
const price = Money.create(10, 'EUR');
const discountedPrice = price.multiply(0.9); // ✅ Nouvelle instance

console.log(price.amount); // 10 - original inchangé
console.log(discountedPrice.amount); // 9

// ❌ INCORRECT - Mutable
export class Money {
  public amount: number;
  public currency: string;

  add(other: Money): void {
    this.amount += other.amount; // ❌ Mutation !
  }
}
```

### 2. Constructeur privé + Factory method

Le constructeur est privé, la création se fait via une factory method statique.

```typescript
// ✅ CORRECT - Constructeur privé
export class Email {
  private constructor(private readonly value: string) {}

  static create(email: string): Email {
    // Validation au moment de la création
    if (!email || email.trim() === '') {
      throw new Error('Email cannot be empty');
    }
    if (!this.isValid(email)) {
      throw new Error('Invalid email format');
    }

    // Normalisation
    const normalized = email.toLowerCase().trim();
    return new Email(normalized);
  }

  private static isValid(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }

  getValue(): string {
    return this.value;
  }

  equals(other: Email): boolean {
    return this.value === other.value;
  }
}

// Utilisation
const email = Email.create('USER@EXAMPLE.COM');
console.log(email.getValue()); // 'user@example.com' - normalisé

// Impossible de créer un email invalide
const invalid = Email.create('invalid'); // ❌ Lance une exception

// ❌ INCORRECT - Constructeur public
export class Email {
  constructor(public value: string) {} // ❌ Pas de validation
}

const invalid = new Email(''); // ❌ Email invalide créé !
```

### 3. Méthode equals

Comparaison basée sur les valeurs, pas sur la référence.

```typescript
// ✅ CORRECT - Méthode equals
export class Address {
  private constructor(
    public readonly street: string,
    public readonly city: string,
    public readonly postalCode: string,
    public readonly country: string
  ) {}

  static create(street: string, city: string, postalCode: string, country: string): Address {
    // Validation...
    return new Address(street, city, postalCode, country);
  }

  equals(other: Address): boolean {
    return this.street === other.street &&
           this.city === other.city &&
           this.postalCode === other.postalCode &&
           this.country === other.country;
  }

  getFullAddress(): string {
    return `${this.street}, ${this.postalCode} ${this.city}, ${this.country}`;
  }
}

// Utilisation
const addr1 = Address.create('1 rue de la Paix', 'Paris', '75001', 'France');
const addr2 = Address.create('1 rue de la Paix', 'Paris', '75001', 'France');

console.log(addr1 === addr2); // false - références différentes
console.log(addr1.equals(addr2)); // true - mêmes valeurs

// ❌ INCORRECT - Comparaison de référence
if (addr1 === addr2) { } // ❌ Toujours false
```

### 4. Validation complète

Toute la validation est encapsulée dans le Value Object.

```typescript
// ✅ CORRECT - Validation encapsulée
export class PhoneNumber {
  private constructor(private readonly value: string) {}

  static create(phone: string): PhoneNumber {
    if (!phone) {
      throw new Error('Phone number is required');
    }

    // Normalisation
    const normalized = phone.replace(/\s+/g, '');

    // Validation
    if (normalized.length < 10) {
      throw new Error('Phone number too short');
    }
    if (!/^\+?\d+$/.test(normalized)) {
      throw new Error('Phone number must contain only digits and optional +');
    }

    return new PhoneNumber(normalized);
  }

  getValue(): string {
    return this.value;
  }

  getFormatted(): string {
    // Format français : +33 6 12 34 56 78
    if (this.value.startsWith('+33')) {
      const digits = this.value.slice(3);
      return `+33 ${digits.charAt(0)} ${digits.slice(1, 3)} ${digits.slice(3, 5)} ${digits.slice(5, 7)} ${digits.slice(7)}`;
    }
    return this.value;
  }

  equals(other: PhoneNumber): boolean {
    return this.value === other.value;
  }
}

// Utilisation - Impossible de créer un numéro invalide
const phone = PhoneNumber.create('+33 6 12 34 56 78'); // ✅ OK
const invalid = PhoneNumber.create('abc'); // ❌ Lance une exception
```

## 📚 Bonnes pratiques

### 1. Encapsuler les primitives

Ne pas utiliser des primitives pour les concepts métier.

```typescript
// ✅ CORRECT - Value Objects au lieu de primitives
export class User {
  constructor(
    public readonly id: string,
    public readonly email: Email,        // ✅ Value Object
    public readonly age: Age,            // ✅ Value Object
    public readonly salary: Money        // ✅ Value Object
  ) {}
}

// ❌ INCORRECT - Primitive obsession
export class User {
  constructor(
    public readonly id: string,
    public readonly email: string,       // ❌ Primitive
    public readonly age: number,         // ❌ Primitive
    public readonly salary: number       // ❌ Primitive
  ) {}
  // Où mettre la validation de l'email ?
  // Comment empêcher un âge négatif ?
  // Comment gérer la devise du salaire ?
}
```

### 2. Opérations métier dans le Value Object

```typescript
// ✅ CORRECT - Opérations métier encapsulées
export class DateRange {
  private constructor(
    public readonly start: Date,
    public readonly end: Date
  ) {}

  static create(start: Date, end: Date): DateRange {
    if (start >= end) {
      throw new Error('Start must be before end');
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
    const diff = this.end.getTime() - this.start.getTime();
    return Math.ceil(diff / (1000 * 60 * 60 * 24));
  }

  extend(days: number): DateRange {
    const newEnd = new Date(this.end);
    newEnd.setDate(newEnd.getDate() + days);
    return DateRange.create(this.start, newEnd);
  }

  equals(other: DateRange): boolean {
    return this.start.getTime() === other.start.getTime() &&
           this.end.getTime() === other.end.getTime();
  }
}

// Utilisation
const range = DateRange.create(new Date('2024-01-01'), new Date('2024-01-31'));
console.log(range.getDurationInDays()); // 30
console.log(range.contains(new Date('2024-01-15'))); // true

const extended = range.extend(7); // Nouvelle instance avec 7 jours de plus
```

### 3. Value Objects composés

Les Value Objects peuvent contenir d'autres Value Objects.

```typescript
// ✅ CORRECT - Composition de Value Objects
export class FullName {
  private constructor(
    public readonly firstName: string,
    public readonly lastName: string
  ) {}

  static create(firstName: string, lastName: string): FullName {
    if (!firstName || !lastName) {
      throw new Error('First name and last name are required');
    }
    return new FullName(
      firstName.trim(),
      lastName.trim()
    );
  }

  getFullName(): string {
    return `${this.firstName} ${this.lastName}`;
  }

  getInitials(): string {
    return `${this.firstName[0]}${this.lastName[0]}`.toUpperCase();
  }

  equals(other: FullName): boolean {
    return this.firstName === other.firstName &&
           this.lastName === other.lastName;
  }
}

export class ContactInfo {
  private constructor(
    public readonly email: Email,
    public readonly phone: PhoneNumber
  ) {}

  static create(email: Email, phone: PhoneNumber): ContactInfo {
    return new ContactInfo(email, phone);
  }

  equals(other: ContactInfo): boolean {
    return this.email.equals(other.email) &&
           this.phone.equals(other.phone);
  }
}

export class Person {
  constructor(
    public readonly id: string,
    public readonly name: FullName,        // ✅ Value Object composé
    public readonly contact: ContactInfo   // ✅ Value Object composé
  ) {}
}
```

### 4. Méthodes utilitaires

```typescript
// ✅ CORRECT - Méthodes utilitaires dans le Value Object
export class Percentage {
  private constructor(public readonly value: number) {}

  static create(value: number): Percentage {
    if (value < 0 || value > 100) {
      throw new Error('Percentage must be between 0 and 100');
    }
    return new Percentage(value);
  }

  static fromDecimal(decimal: number): Percentage {
    return Percentage.create(decimal * 100);
  }

  toDecimal(): number {
    return this.value / 100;
  }

  applyTo(amount: number): number {
    return amount * this.toDecimal();
  }

  add(other: Percentage): Percentage {
    return Percentage.create(Math.min(100, this.value + other.value));
  }

  equals(other: Percentage): boolean {
    return this.value === other.value;
  }
}

// Utilisation
const discount = Percentage.create(20); // 20%
const price = 100;
const discountedPrice = price - discount.applyTo(price); // 80
```

## 📊 Exemples complets

### Exemple 1 : Money

```typescript
export class Money {
  private constructor(
    public readonly amount: number,
    public readonly currency: string
  ) {}

  static create(amount: number, currency: string): Money {
    if (amount < 0) {
      throw new Error('Amount cannot be negative');
    }
    if (!currency || currency.length !== 3) {
      throw new Error('Invalid currency code');
    }
    return new Money(
      Math.round(amount * 100) / 100, // Arrondi à 2 décimales
      currency.toUpperCase()
    );
  }

  static zero(currency: string): Money {
    return Money.create(0, currency);
  }

  add(other: Money): Money {
    this.assertSameCurrency(other);
    return Money.create(this.amount + other.amount, this.currency);
  }

  subtract(other: Money): Money {
    this.assertSameCurrency(other);
    return Money.create(this.amount - other.amount, this.currency);
  }

  multiply(factor: number): Money {
    return Money.create(this.amount * factor, this.currency);
  }

  divide(divisor: number): Money {
    if (divisor === 0) {
      throw new Error('Cannot divide by zero');
    }
    return Money.create(this.amount / divisor, this.currency);
  }

  isGreaterThan(other: Money): boolean {
    this.assertSameCurrency(other);
    return this.amount > other.amount;
  }

  isLessThan(other: Money): boolean {
    this.assertSameCurrency(other);
    return this.amount < other.amount;
  }

  equals(other: Money): boolean {
    return this.amount === other.amount &&
           this.currency === other.currency;
  }

  format(): string {
    return `${this.amount.toFixed(2)} ${this.currency}`;
  }

  private assertSameCurrency(other: Money): void {
    if (this.currency !== other.currency) {
      throw new Error(
        `Cannot operate on different currencies: ${this.currency} and ${other.currency}`
      );
    }
  }
}

// Utilisation
const price1 = Money.create(10.50, 'EUR');
const price2 = Money.create(5.25, 'EUR');

const total = price1.add(price2); // 15.75 EUR
const discounted = total.multiply(0.9); // 14.175 EUR (arrondi à 14.18)

console.log(discounted.format()); // "14.18 EUR"
```

### Exemple 2 : Email avec validation complète

```typescript
export class Email {
  private constructor(private readonly value: string) {}

  static create(email: string): Email {
    if (!email) {
      throw new Error('Email is required');
    }

    const normalized = email.toLowerCase().trim();

    if (!this.isValid(normalized)) {
      throw new Error('Invalid email format');
    }

    if (this.isDisposable(normalized)) {
      throw new Error('Disposable email addresses are not allowed');
    }

    return new Email(normalized);
  }

  private static isValid(email: string): boolean {
    // RFC 5322 simplifié
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return regex.test(email) && email.length <= 255;
  }

  private static isDisposable(email: string): boolean {
    const disposableDomains = ['tempmail.com', '10minutemail.com'];
    const domain = email.split('@')[1];
    return disposableDomains.includes(domain);
  }

  getValue(): string {
    return this.value;
  }

  getDomain(): string {
    return this.value.split('@')[1];
  }

  getLocalPart(): string {
    return this.value.split('@')[0];
  }

  isSameDomain(other: Email): boolean {
    return this.getDomain() === other.getDomain();
  }

  equals(other: Email): boolean {
    return this.value === other.value;
  }

  toString(): string {
    return this.value;
  }
}
```

### Exemple 3 : Address complet

```typescript
export class PostalCode {
  private constructor(private readonly value: string) {}

  static create(code: string, country: string): PostalCode {
    const normalized = code.replace(/\s+/g, '').toUpperCase();

    switch (country.toUpperCase()) {
      case 'FR':
        if (!/^\d{5}$/.test(normalized)) {
          throw new Error('Invalid French postal code');
        }
        break;
      case 'US':
        if (!/^\d{5}(-\d{4})?$/.test(normalized)) {
          throw new Error('Invalid US postal code');
        }
        break;
      default:
        if (normalized.length < 3 || normalized.length > 10) {
          throw new Error('Invalid postal code');
        }
    }

    return new PostalCode(normalized);
  }

  getValue(): string {
    return this.value;
  }

  equals(other: PostalCode): boolean {
    return this.value === other.value;
  }
}

export class Address {
  private constructor(
    public readonly street: string,
    public readonly city: string,
    public readonly postalCode: PostalCode,
    public readonly country: string
  ) {}

  static create(
    street: string,
    city: string,
    postalCode: string,
    country: string
  ): Address {
    if (!street || !city || !country) {
      throw new Error('All address fields are required');
    }

    const code = PostalCode.create(postalCode, country);

    return new Address(
      street.trim(),
      city.trim(),
      code,
      country.trim().toUpperCase()
    );
  }

  getFullAddress(): string {
    return `${this.street}, ${this.postalCode.getValue()} ${this.city}, ${this.country}`;
  }

  getFormattedAddress(): string[] {
    return [
      this.street,
      `${this.postalCode.getValue()} ${this.city}`,
      this.country
    ];
  }

  isSameCity(other: Address): boolean {
    return this.city === other.city &&
           this.postalCode.equals(other.postalCode);
  }

  equals(other: Address): boolean {
    return this.street === other.street &&
           this.city === other.city &&
           this.postalCode.equals(other.postalCode) &&
           this.country === other.country;
  }
}
```

### Exemple 4 : Quantity avec unités

```typescript
export enum Unit {
  Kilogram = 'kg',
  Gram = 'g',
  Liter = 'L',
  Milliliter = 'mL',
  Piece = 'pcs'
}

export class Quantity {
  private constructor(
    public readonly value: number,
    public readonly unit: Unit
  ) {}

  static create(value: number, unit: Unit): Quantity {
    if (value < 0) {
      throw new Error('Quantity cannot be negative');
    }
    return new Quantity(value, unit);
  }

  static zero(unit: Unit): Quantity {
    return Quantity.create(0, unit);
  }

  add(other: Quantity): Quantity {
    this.assertSameUnit(other);
    return Quantity.create(this.value + other.value, this.unit);
  }

  subtract(other: Quantity): Quantity {
    this.assertSameUnit(other);
    if (this.value < other.value) {
      throw new Error('Result would be negative');
    }
    return Quantity.create(this.value - other.value, this.unit);
  }

  multiply(factor: number): Quantity {
    return Quantity.create(this.value * factor, this.unit);
  }

  isGreaterThan(other: Quantity): boolean {
    this.assertSameUnit(other);
    return this.value > other.value;
  }

  equals(other: Quantity): boolean {
    return this.value === other.value && this.unit === other.unit;
  }

  format(): string {
    return `${this.value} ${this.unit}`;
  }

  private assertSameUnit(other: Quantity): void {
    if (this.unit !== other.unit) {
      throw new Error(`Cannot operate on different units: ${this.unit} and ${other.unit}`);
    }
  }
}
```

## 🧪 Tests de Value Objects

```typescript
describe('Money', () => {
  describe('create', () => {
    it('should create valid money', () => {
      const money = Money.create(10.50, 'EUR');

      expect(money.amount).toBe(10.50);
      expect(money.currency).toBe('EUR');
    });

    it('should throw error for negative amount', () => {
      expect(() => Money.create(-10, 'EUR')).toThrow();
    });

    it('should normalize currency to uppercase', () => {
      const money = Money.create(10, 'eur');

      expect(money.currency).toBe('EUR');
    });
  });

  describe('add', () => {
    it('should add money with same currency', () => {
      const m1 = Money.create(10, 'EUR');
      const m2 = Money.create(5, 'EUR');

      const result = m1.add(m2);

      expect(result.amount).toBe(15);
      expect(result.currency).toBe('EUR');
    });

    it('should throw error for different currencies', () => {
      const m1 = Money.create(10, 'EUR');
      const m2 = Money.create(5, 'USD');

      expect(() => m1.add(m2)).toThrow();
    });
  });

  describe('equals', () => {
    it('should be equal with same values', () => {
      const m1 = Money.create(10, 'EUR');
      const m2 = Money.create(10, 'EUR');

      expect(m1.equals(m2)).toBe(true);
    });

    it('should not be equal with different amounts', () => {
      const m1 = Money.create(10, 'EUR');
      const m2 = Money.create(20, 'EUR');

      expect(m1.equals(m2)).toBe(false);
    });
  });
});
```

## 🎓 Points clés

1. **Immutabilité** : Ne jamais muter, toujours retourner une nouvelle instance
2. **Constructeur privé** : Utiliser des factory methods
3. **Validation** : Toujours valider dans le factory method
4. **equals()** : Comparaison basée sur les valeurs
5. **Pas d'identité** : Pas d'ID, défini par les valeurs
6. **Encapsulation** : Encapsuler les primitives
7. **Opérations métier** : Logique métier dans le Value Object

---

**Les Value Objects renforcent votre modèle de domaine et réduisent les bugs ! 💎**
