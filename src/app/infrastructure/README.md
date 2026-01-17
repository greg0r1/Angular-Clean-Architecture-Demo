# 🔧 Infrastructure - Implémentations techniques

## 📋 Description

La couche **Infrastructure** contient toutes les **implémentations concrètes** des interfaces définies dans les couches Domain et Application. C'est ici que se trouvent les détails techniques.

> **Principe clé** : L'Infrastructure **implémente** les contrats définis par le Domain et l'Application.

## 🏗️ Structure

```
infrastructure/
├── repositories/     # Implémentations des repositories
├── http/            # Services HTTP et intercepteurs
├── storage/         # LocalStorage, SessionStorage, IndexedDB
├── mappers/         # Transformation DTOs ↔ Domain
└── README.md
```

## 🔄 Flux de dépendances

```
Domain (Interfaces)
   ▲
   │ implémente
   │
Infrastructure (Implémentations) ◄─── Nous sommes ICI
   │
   │ utilise
   ▼
Services externes (API, BDD, Storage...)
```

## ✅ Ce qu'on doit mettre

### Dans Infrastructure
- ✅ Implémentations des repository interfaces
- ✅ Services HTTP (HttpClient)
- ✅ Intercepteurs HTTP
- ✅ Services de storage (LocalStorage, etc.)
- ✅ Implémentations des ports (Email, Notification...)
- ✅ Mappers (DTO ↔ Domain)
- ✅ Adaptateurs pour services tiers
- ✅ Configuration technique
- ✅ Guards, Resolvers

### Exemples
```typescript
// ✅ Implémentations
@Injectable({ providedIn: 'root' })
export class UserRepositoryImpl implements UserRepository { }

@Injectable({ providedIn: 'root' })
export class EmailServiceImpl implements EmailService { }

@Injectable({ providedIn: 'root' })
export class LocalStorageService { }

export class UserMapper {
  static toDomain(dto: UserDTO): User { }
  static toDTO(user: User): UserDTO { }
}
```

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Logique métier (→ Domain)
- ❌ Use cases (→ Application)
- ❌ Components (→ Presentation)
- ❌ Logique de présentation (→ Presentation)

## 📖 Principes SOLID

### Dependency Inversion (DIP)

L'Infrastructure dépend du Domain, pas l'inverse.

```typescript
// ✅ CORRECT - Infrastructure implémente interface du Domain
// domain/repositories/user.repository.ts
export interface UserRepository {
  findById(id: string): Observable<User>;
}

// infrastructure/repositories/user-repository.impl.ts
@Injectable({ providedIn: 'root' })
export class UserRepositoryImpl implements UserRepository {
  constructor(private http: HttpClient) {}

  findById(id: string): Observable<User> {
    return this.http.get<UserDTO>(`/api/users/${id}`).pipe(
      map(dto => UserMapper.toDomain(dto))
    );
  }
}

// ❌ INCORRECT - Domain dépendrait d'Infrastructure
// domain/repositories/user.repository.ts
import { HttpClient } from '@angular/common/http'; // ❌ NON !

export class UserRepository {
  constructor(private http: HttpClient) {} // ❌ NON !
}
```

### Single Responsibility (SRP)

Séparer les responsabilités techniques.

```typescript
// ✅ CORRECT - Responsabilités séparées
@Injectable({ providedIn: 'root' })
export class UserRepositoryImpl implements UserRepository {
  constructor(
    private http: HttpClient,
    private mapper: UserMapper
  ) {}

  findById(id: string): Observable<User> {
    return this.http.get<UserDTO>(`/api/users/${id}`).pipe(
      map(dto => this.mapper.toDomain(dto))
    );
  }
}

// Mapper séparé
@Injectable({ providedIn: 'root' })
export class UserMapper {
  toDomain(dto: UserDTO): User { }
  toDTO(user: User): UserDTO { }
}

// ❌ INCORRECT - Trop de responsabilités
@Injectable({ providedIn: 'root' })
export class UserService {
  fetchUser() { }
  saveToLocalStorage() { }
  sendEmail() { }
  generatePDF() { }
  // ❌ Trop de choses différentes
}
```

## 🛡️ Bonnes pratiques

### 1. Implémenter les interfaces du Domain

```typescript
// ✅ CORRECT
@Injectable({ providedIn: 'root' })
export class OrderRepositoryImpl implements OrderRepository {
  constructor(
    private http: HttpClient,
    private mapper: OrderMapper
  ) {}

  findById(id: string): Observable<Order | null> {
    return this.http.get<OrderDTO>(`/api/orders/${id}`).pipe(
      map(dto => this.mapper.toDomain(dto)),
      catchError(() => of(null))
    );
  }

  save(order: Order): Observable<Order> {
    const dto = this.mapper.toDTO(order);
    return this.http.post<OrderDTO>('/api/orders', dto).pipe(
      map(dto => this.mapper.toDomain(dto))
    );
  }
}
```

### 2. Utiliser des Mappers

Séparer la transformation DTO ↔ Domain.

```typescript
// ✅ CORRECT - Mapper dédié
@Injectable({ providedIn: 'root' })
export class UserMapper {
  toDomain(dto: UserDTO): User {
    return new User(
      dto.id,
      Email.create(dto.email),
      new UserProfile(dto.firstName, dto.lastName)
    );
  }

  toDTO(user: User): UserDTO {
    return {
      id: user.id,
      email: user.email.getValue(),
      firstName: user.profile.firstName,
      lastName: user.profile.lastName
    };
  }
}

// ❌ INCORRECT - Transformation dans le repository
export class UserRepositoryImpl {
  findById(id: string): Observable<User> {
    return this.http.get<UserDTO>(`/api/users/${id}`).pipe(
      map(dto => new User(dto.id, Email.create(dto.email), ...)) // ❌ Logique ici
    );
  }
}
```

### 3. Gestion des erreurs HTTP

```typescript
// ✅ CORRECT - Gestion d'erreurs appropriée
@Injectable({ providedIn: 'root' })
export class ProductRepositoryImpl implements ProductRepository {
  constructor(private http: HttpClient) {}

  findById(id: string): Observable<Product | null> {
    return this.http.get<ProductDTO>(`/api/products/${id}`).pipe(
      map(dto => ProductMapper.toDomain(dto)),
      catchError(error => {
        if (error.status === 404) {
          return of(null); // ✅ Retourne null pour "non trouvé"
        }
        return throwError(() => error); // ✅ Propage les autres erreurs
      })
    );
  }
}
```

### 4. Intercepteurs HTTP

```typescript
// ✅ CORRECT - Intercepteur pour l'authentification
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(private authService: AuthService) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = this.authService.getToken();

    if (token) {
      req = req.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`
        }
      });
    }

    return next.handle(req);
  }
}

// Configuration
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptorFn])
    )
  ]
};
```

## 📊 Exemples complets

### Repository avec cache

```typescript
@Injectable({ providedIn: 'root' })
export class UserRepositoryImpl implements UserRepository {
  private cache = new Map<string, User>();

  constructor(
    private http: HttpClient,
    private mapper: UserMapper
  ) {}

  findById(id: string): Observable<User | null> {
    // Vérifier le cache
    if (this.cache.has(id)) {
      return of(this.cache.get(id)!);
    }

    return this.http.get<UserDTO>(`/api/users/${id}`).pipe(
      map(dto => this.mapper.toDomain(dto)),
      tap(user => this.cache.set(id, user)),
      catchError(error => {
        if (error.status === 404) return of(null);
        return throwError(() => error);
      })
    );
  }

  save(user: User): Observable<User> {
    const dto = this.mapper.toDTO(user);

    return this.http.post<UserDTO>('/api/users', dto).pipe(
      map(dto => this.mapper.toDomain(dto)),
      tap(savedUser => {
        this.cache.set(savedUser.id, savedUser);
      })
    );
  }

  clearCache(): void {
    this.cache.clear();
  }
}
```

### Implementation de port Email

```typescript
// Application - Port (Interface)
export interface EmailService {
  sendEmail(to: Email, subject: string, body: string): Observable<void>;
}

// Infrastructure - Adapter (Implémentation)
@Injectable({ providedIn: 'root' })
export class EmailServiceImpl implements EmailService {
  constructor(private http: HttpClient) {}

  sendEmail(to: Email, subject: string, body: string): Observable<void> {
    return this.http.post<void>('/api/emails/send', {
      to: to.getValue(),
      subject,
      body
    });
  }

  // Méthode privée pour préparer le template
  private prepareEmailTemplate(template: string, data: any): string {
    return template.replace(/{{(\w+)}}/g, (_, key) => data[key] || '');
  }
}
```

## 🧪 Tests d'Infrastructure

```typescript
describe('UserRepositoryImpl', () => {
  let repository: UserRepositoryImpl;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserRepositoryImpl, UserMapper]
    });

    repository = TestBed.inject(UserRepositoryImpl);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify();
  });

  it('should fetch user by id', (done) => {
    const mockDTO: UserDTO = {
      id: '123',
      email: 'test@test.com',
      firstName: 'John',
      lastName: 'Doe'
    };

    repository.findById('123').subscribe(user => {
      expect(user).toBeDefined();
      expect(user!.id).toBe('123');
      done();
    });

    const req = httpMock.expectOne('/api/users/123');
    expect(req.request.method).toBe('GET');
    req.flush(mockDTO);
  });

  it('should return null for 404', (done) => {
    repository.findById('999').subscribe(user => {
      expect(user).toBeNull();
      done();
    });

    const req = httpMock.expectOne('/api/users/999');
    req.flush(null, { status: 404, statusText: 'Not Found' });
  });
});
```

## 🎓 Points clés

1. **Implémente** : Les interfaces du Domain et Application
2. **Détails techniques** : HTTP, Storage, APIs externes
3. **Mappers** : Transformation DTO ↔ Domain
4. **Gestion d'erreurs** : HTTP, réseau, parsing
5. **@Injectable** : Services Angular
6. **Testabilité** : Avec HttpClientTestingModule
7. **Intercepteurs** : Auth, logging, erreurs
8. **Cache** : Si pertinent pour la performance

---

**L'Infrastructure connecte votre logique métier au monde réel ! 🔧**
