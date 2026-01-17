# 🔄 Mappers - Transformations DTO ↔ Domain

## 📋 Description

Les **Mappers** transforment les données entre les **DTOs** (Data Transfer Objects) de l'API et les **entités du Domain**.

> **Principe clé** : Séparer la structure des données API de la structure du Domain.

## 🎯 Responsabilité

- ✅ Transformation DTO → Domain (de l'API vers le modèle métier)
- ✅ Transformation Domain → DTO (du modèle métier vers l'API)
- ✅ Validation des données entrantes
- ✅ Création des Value Objects
- ✅ Gestion des champs nullables

## ✅ Ce qu'on doit mettre

- ✅ Classes de mappers avec méthodes statiques
- ✅ Logique de transformation
- ✅ Création de Value Objects
- ✅ Gestion des cas particuliers (null, undefined)
- ✅ Validation des DTOs

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Logique métier (→ Domain/Entities)
- ❌ Appels HTTP (→ repositories/)
- ❌ Logique de cas d'utilisation (→ Application)

## 🏗️ Structure type

```typescript
// Template de mapper
@Injectable({ providedIn: 'root' })
export class EntityMapper {
  // DTO → Domain
  toDomain(dto: EntityDTO): Entity {
    return new Entity(
      dto.id,
      this.mapValueObject(dto.field),
      dto.otherField
    );
  }

  // Domain → DTO
  toDTO(entity: Entity): EntityDTO {
    return {
      id: entity.id,
      field: entity.valueObject.getValue(),
      otherField: entity.otherField
    };
  }

  // Array DTO → Array Domain
  toDomainList(dtos: EntityDTO[]): Entity[] {
    return dtos.map(dto => this.toDomain(dto));
  }

  // Array Domain → Array DTO
  toDTOList(entities: Entity[]): EntityDTO[] {
    return entities.map(entity => this.toDTO(entity));
  }

  private mapValueObject(value: string): ValueObject {
    return ValueObject.create(value);
  }
}
```

## 📊 Exemples complets

### User Mapper

```typescript
// DTOs
export interface UserDTO {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  birthDate: string;
  address?: AddressDTO;
  createdAt: string;
  updatedAt: string;
}

export interface AddressDTO {
  street: string;
  city: string;
  postalCode: string;
  country: string;
}

// Mapper
@Injectable({ providedIn: 'root' })
export class UserMapper {
  toDomain(dto: UserDTO): User {
    return new User(
      dto.id,
      Email.create(dto.email),
      FullName.create(dto.firstName, dto.lastName),
      BirthDate.create(new Date(dto.birthDate)),
      dto.address ? this.mapAddress(dto.address) : null,
      new Date(dto.createdAt)
    );
  }

  toDTO(user: User): UserDTO {
    return {
      id: user.id,
      email: user.email.getValue(),
      firstName: user.name.firstName,
      lastName: user.name.lastName,
      birthDate: user.birthDate.getValue().toISOString(),
      address: user.address ? this.addressToDTO(user.address) : undefined,
      createdAt: user.createdAt.toISOString(),
      updatedAt: new Date().toISOString()
    };
  }

  toDomainList(dtos: UserDTO[]): User[] {
    return dtos.map(dto => this.toDomain(dto));
  }

  private mapAddress(dto: AddressDTO): Address {
    return Address.create(
      dto.street,
      dto.city,
      dto.postalCode,
      dto.country
    );
  }

  private addressToDTO(address: Address): AddressDTO {
    return {
      street: address.street,
      city: address.city,
      postalCode: address.postalCode.getValue(),
      country: address.country
    };
  }
}
```

### Order Mapper avec relations

```typescript
// DTOs
export interface OrderDTO {
  id: string;
  customerId: string;
  items: OrderItemDTO[];
  status: string;
  total: number;
  currency: string;
  createdAt: string;
}

export interface OrderItemDTO {
  productId: string;
  productName: string;
  price: number;
  currency: string;
  quantity: number;
}

// Mapper
@Injectable({ providedIn: 'root' })
export class OrderMapper {
  toDomain(dto: OrderDTO): Order {
    const items = dto.items.map(itemDTO => this.mapOrderItem(itemDTO));

    return new Order(
      dto.id,
      dto.customerId,
      items,
      this.mapStatus(dto.status),
      new Date(dto.createdAt)
    );
  }

  toDTO(order: Order): OrderDTO {
    return {
      id: order.id,
      customerId: order.customerId,
      items: order.items.map(item => this.orderItemToDTO(item)),
      status: order.status,
      total: order.calculateTotal().amount,
      currency: order.calculateTotal().currency,
      createdAt: order.createdAt.toISOString()
    };
  }

  private mapOrderItem(dto: OrderItemDTO): OrderItem {
    return new OrderItem(
      dto.productId,
      dto.productName,
      Money.create(dto.price, dto.currency),
      dto.quantity
    );
  }

  private orderItemToDTO(item: OrderItem): OrderItemDTO {
    return {
      productId: item.productId,
      productName: item.productName,
      price: item.price.amount,
      currency: item.price.currency,
      quantity: item.quantity
    };
  }

  private mapStatus(status: string): OrderStatus {
    switch (status.toUpperCase()) {
      case 'PENDING': return OrderStatus.Pending;
      case 'CONFIRMED': return OrderStatus.Confirmed;
      case 'SHIPPED': return OrderStatus.Shipped;
      case 'DELIVERED': return OrderStatus.Delivered;
      case 'CANCELLED': return OrderStatus.Cancelled;
      default: throw new Error(`Unknown order status: ${status}`);
    }
  }
}
```

### Mapper avec validation

```typescript
@Injectable({ providedIn: 'root' })
export class ProductMapper {
  toDomain(dto: ProductDTO): Product {
    // Validation du DTO
    this.validateDTO(dto);

    return new Product(
      dto.id,
      dto.name,
      Money.create(dto.price, dto.currency),
      dto.stock,
      dto.description,
      dto.imageUrl ? new URL(dto.imageUrl) : null
    );
  }

  toDTO(product: Product): ProductDTO {
    return {
      id: product.id,
      name: product.name,
      price: product.price.amount,
      currency: product.price.currency,
      stock: product.stock,
      description: product.description,
      imageUrl: product.imageUrl?.toString()
    };
  }

  private validateDTO(dto: ProductDTO): void {
    const errors: string[] = [];

    if (!dto.id) {
      errors.push('Product ID is required');
    }

    if (!dto.name || dto.name.trim() === '') {
      errors.push('Product name is required');
    }

    if (dto.price < 0) {
      errors.push('Product price cannot be negative');
    }

    if (!dto.currency || dto.currency.length !== 3) {
      errors.push('Invalid currency code');
    }

    if (dto.stock < 0) {
      errors.push('Product stock cannot be negative');
    }

    if (errors.length > 0) {
      throw new Error(`Invalid ProductDTO: ${errors.join(', ')}`);
    }
  }
}
```

### Mapper avec champs optionnels

```typescript
@Injectable({ providedIn: 'root' })
export class ArticleMapper {
  toDomain(dto: ArticleDTO): Article {
    return new Article(
      dto.id,
      dto.title,
      dto.content,
      dto.authorId,
      dto.publishedAt ? new Date(dto.publishedAt) : null,
      dto.tags || [],
      dto.metadata ? this.mapMetadata(dto.metadata) : null
    );
  }

  toDTO(article: Article): ArticleDTO {
    return {
      id: article.id,
      title: article.title,
      content: article.content,
      authorId: article.authorId,
      publishedAt: article.publishedAt?.toISOString(),
      tags: article.tags,
      metadata: article.metadata ? this.metadataToDTO(article.metadata) : undefined
    };
  }

  private mapMetadata(dto: MetadataDTO): ArticleMetadata {
    return new ArticleMetadata(
      dto.readTime,
      dto.category,
      dto.featured
    );
  }

  private metadataToDTO(metadata: ArticleMetadata): MetadataDTO {
    return {
      readTime: metadata.readTime,
      category: metadata.category,
      featured: metadata.featured
    };
  }
}
```

### Mapper avec dates

```typescript
@Injectable({ providedIn: 'root' })
export class BookingMapper {
  toDomain(dto: BookingDTO): Booking {
    return new Booking(
      dto.id,
      dto.userId,
      dto.resourceId,
      DateRange.create(
        this.parseDate(dto.startDate),
        this.parseDate(dto.endDate)
      ),
      this.mapStatus(dto.status),
      this.parseDate(dto.createdAt)
    );
  }

  toDTO(booking: Booking): BookingDTO {
    return {
      id: booking.id,
      userId: booking.userId,
      resourceId: booking.resourceId,
      startDate: this.formatDate(booking.dateRange.start),
      endDate: this.formatDate(booking.dateRange.end),
      status: booking.status,
      createdAt: this.formatDate(booking.createdAt)
    };
  }

  private parseDate(dateString: string): Date {
    const date = new Date(dateString);
    if (isNaN(date.getTime())) {
      throw new Error(`Invalid date: ${dateString}`);
    }
    return date;
  }

  private formatDate(date: Date): string {
    return date.toISOString();
  }

  private mapStatus(status: string): BookingStatus {
    const statusMap: Record<string, BookingStatus> = {
      'PENDING': BookingStatus.Pending,
      'CONFIRMED': BookingStatus.Confirmed,
      'CANCELLED': BookingStatus.Cancelled,
      'COMPLETED': BookingStatus.Completed
    };

    const mapped = statusMap[status.toUpperCase()];
    if (!mapped) {
      throw new Error(`Unknown booking status: ${status}`);
    }

    return mapped;
  }
}
```

## 🧪 Tests

```typescript
describe('UserMapper', () => {
  let mapper: UserMapper;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [UserMapper]
    });
    mapper = TestBed.inject(UserMapper);
  });

  describe('toDomain', () => {
    it('should map DTO to User entity', () => {
      const dto: UserDTO = {
        id: '123',
        email: 'test@test.com',
        firstName: 'John',
        lastName: 'Doe',
        birthDate: '1990-01-01',
        createdAt: '2024-01-01T00:00:00Z',
        updatedAt: '2024-01-01T00:00:00Z'
      };

      const user = mapper.toDomain(dto);

      expect(user.id).toBe('123');
      expect(user.email.getValue()).toBe('test@test.com');
      expect(user.name.firstName).toBe('John');
      expect(user.name.lastName).toBe('Doe');
    });

    it('should handle optional address', () => {
      const dtoWithAddress: UserDTO = {
        id: '123',
        email: 'test@test.com',
        firstName: 'John',
        lastName: 'Doe',
        birthDate: '1990-01-01',
        address: {
          street: '1 rue de la Paix',
          city: 'Paris',
          postalCode: '75001',
          country: 'France'
        },
        createdAt: '2024-01-01T00:00:00Z',
        updatedAt: '2024-01-01T00:00:00Z'
      };

      const user = mapper.toDomain(dtoWithAddress);

      expect(user.address).toBeDefined();
      expect(user.address!.city).toBe('Paris');
    });
  });

  describe('toDTO', () => {
    it('should map User entity to DTO', () => {
      const user = new User(
        '123',
        Email.create('test@test.com'),
        FullName.create('John', 'Doe'),
        BirthDate.create(new Date('1990-01-01')),
        null,
        new Date('2024-01-01')
      );

      const dto = mapper.toDTO(user);

      expect(dto.id).toBe('123');
      expect(dto.email).toBe('test@test.com');
      expect(dto.firstName).toBe('John');
      expect(dto.lastName).toBe('Doe');
    });
  });
});
```

## 🎓 Points clés

1. **Séparation** : DTO ≠ Domain
2. **Value Objects** : Créer lors du mapping
3. **Validation** : Valider les DTOs entrants
4. **Nullable** : Gérer les champs optionnels
5. **Dates** : Parser/formater correctement
6. **Enums** : Mapper strings → enums
7. **Tests** : Tester les deux directions
8. **@Injectable** : Pour injection de dépendances

---

**Les mappers gardent votre Domain pur en isolant les formats externes ! 🔄**
