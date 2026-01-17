# 📦 Repositories - Implémentations

## 📋 Description

Les **Repository Implementations** sont les **implémentations concrètes** des interfaces de repositories définies dans le Domain. Elles gèrent l'accès réel aux données (API, BDD, Storage...).

> **Principe clé** : Implémenter les contrats du Domain avec les technologies concrètes.

## ✅ Ce qu'on doit mettre

- ✅ Implémentations des interfaces du Domain
- ✅ Appels HTTP (HttpClient)
- ✅ Gestion du cache
- ✅ Gestion des erreurs HTTP
- ✅ Transformations DTO → Domain (via Mappers)
- ✅ Annotations @Injectable

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Logique métier (→ Domain/Entities)
- ❌ Logique de cas d'utilisation (→ Application)
- ❌ Logique de présentation (→ Presentation)

## 🏗️ Structure type

```typescript
// Template de repository implementation
@Injectable({ providedIn: 'root' })
export class EntityRepositoryImpl implements EntityRepository {
  private readonly apiUrl = '/api/entities';

  constructor(
    private http: HttpClient,
    private mapper: EntityMapper
  ) {}

  findById(id: string): Observable<Entity | null> {
    return this.http.get<EntityDTO>(`${this.apiUrl}/${id}`).pipe(
      map(dto => this.mapper.toDomain(dto)),
      catchError(this.handleError)
    );
  }

  findAll(): Observable<Entity[]> {
    return this.http.get<EntityDTO[]>(this.apiUrl).pipe(
      map(dtos => dtos.map(dto => this.mapper.toDomain(dto)))
    );
  }

  save(entity: Entity): Observable<Entity> {
    const dto = this.mapper.toDTO(entity);

    if (this.isNew(entity)) {
      return this.create(dto);
    } else {
      return this.update(entity.id, dto);
    }
  }

  delete(id: string): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }

  private create(dto: EntityDTO): Observable<Entity> {
    return this.http.post<EntityDTO>(this.apiUrl, dto).pipe(
      map(dto => this.mapper.toDomain(dto))
    );
  }

  private update(id: string, dto: EntityDTO): Observable<Entity> {
    return this.http.put<EntityDTO>(`${this.apiUrl}/${id}`, dto).pipe(
      map(dto => this.mapper.toDomain(dto))
    );
  }

  private handleError(error: HttpErrorResponse): Observable<never> {
    if (error.status === 404) {
      return of(null) as any;
    }
    return throwError(() => error);
  }

  private isNew(entity: Entity): boolean {
    // Logique pour déterminer si l'entité est nouvelle
    return !entity.id || entity.id.startsWith('temp-');
  }
}
```

## 📊 Exemples complets

### User Repository avec cache

```typescript
@Injectable({ providedIn: 'root' })
export class UserRepositoryImpl implements UserRepository {
  private readonly apiUrl = '/api/users';
  private cache = new Map<string, User>();
  private cacheExpiry = new Map<string, number>();
  private readonly CACHE_TTL = 5 * 60 * 1000; // 5 minutes

  constructor(
    private http: HttpClient,
    private mapper: UserMapper
  ) {}

  findById(id: string): Observable<User | null> {
    // Vérifier le cache
    const cached = this.getFromCache(id);
    if (cached) {
      return of(cached);
    }

    return this.http.get<UserDTO>(`${this.apiUrl}/${id}`).pipe(
      map(dto => this.mapper.toDomain(dto)),
      tap(user => this.addToCache(id, user)),
      catchError(error => {
        if (error.status === 404) return of(null);
        return throwError(() => error);
      })
    );
  }

  findByEmail(email: Email): Observable<User | null> {
    return this.http.get<UserDTO[]>(`${this.apiUrl}?email=${email.getValue()}`).pipe(
      map(dtos => dtos.length > 0 ? this.mapper.toDomain(dtos[0]) : null)
    );
  }

  findAll(): Observable<User[]> {
    return this.http.get<UserDTO[]>(this.apiUrl).pipe(
      map(dtos => dtos.map(dto => this.mapper.toDomain(dto)))
    );
  }

  save(user: User): Observable<User> {
    const dto = this.mapper.toDTO(user);
    const request = user.id
      ? this.http.put<UserDTO>(`${this.apiUrl}/${user.id}`, dto)
      : this.http.post<UserDTO>(this.apiUrl, dto);

    return request.pipe(
      map(dto => this.mapper.toDomain(dto)),
      tap(savedUser => {
        this.addToCache(savedUser.id, savedUser);
      })
    );
  }

  delete(id: string): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`).pipe(
      tap(() => this.removeFromCache(id))
    );
  }

  existsByEmail(email: Email): Observable<boolean> {
    return this.http.get<boolean>(`${this.apiUrl}/exists?email=${email.getValue()}`);
  }

  private getFromCache(id: string): User | null {
    const expiry = this.cacheExpiry.get(id);
    if (expiry && expiry > Date.now()) {
      return this.cache.get(id) || null;
    }
    this.removeFromCache(id);
    return null;
  }

  private addToCache(id: string, user: User): void {
    this.cache.set(id, user);
    this.cacheExpiry.set(id, Date.now() + this.CACHE_TTL);
  }

  private removeFromCache(id: string): void {
    this.cache.delete(id);
    this.cacheExpiry.delete(id);
  }

  clearCache(): void {
    this.cache.clear();
    this.cacheExpiry.clear();
  }
}
```

### Order Repository avec pagination

```typescript
@Injectable({ providedIn: 'root' })
export class OrderRepositoryImpl implements OrderRepository {
  private readonly apiUrl = '/api/orders';

  constructor(
    private http: HttpClient,
    private mapper: OrderMapper
  ) {}

  findById(id: string): Observable<Order | null> {
    return this.http.get<OrderDTO>(`${this.apiUrl}/${id}`).pipe(
      map(dto => this.mapper.toDomain(dto)),
      catchError(error => error.status === 404 ? of(null) : throwError(() => error))
    );
  }

  findByCustomerId(customerId: string): Observable<Order[]> {
    return this.http.get<OrderDTO[]>(`${this.apiUrl}?customerId=${customerId}`).pipe(
      map(dtos => dtos.map(dto => this.mapper.toDomain(dto)))
    );
  }

  findByStatus(status: OrderStatus): Observable<Order[]> {
    return this.http.get<OrderDTO[]>(`${this.apiUrl}?status=${status}`).pipe(
      map(dtos => dtos.map(dto => this.mapper.toDomain(dto)))
    );
  }

  findPendingOrders(): Observable<Order[]> {
    return this.findByStatus(OrderStatus.Pending);
  }

  findByDateRange(range: DateRange): Observable<Order[]> {
    const params = {
      startDate: range.start.toISOString(),
      endDate: range.end.toISOString()
    };

    return this.http.get<OrderDTO[]>(this.apiUrl, { params }).pipe(
      map(dtos => dtos.map(dto => this.mapper.toDomain(dto)))
    );
  }

  findRecentOrders(limit: number): Observable<Order[]> {
    return this.http.get<OrderDTO[]>(`${this.apiUrl}?limit=${limit}&sort=-createdAt`).pipe(
      map(dtos => dtos.map(dto => this.mapper.toDomain(dto)))
    );
  }

  save(order: Order): Observable<Order> {
    const dto = this.mapper.toDTO(order);

    const request = order.id
      ? this.http.put<OrderDTO>(`${this.apiUrl}/${order.id}`, dto)
      : this.http.post<OrderDTO>(this.apiUrl, dto);

    return request.pipe(
      map(dto => this.mapper.toDomain(dto))
    );
  }

  delete(id: string): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }

  countByCustomerId(customerId: string): Observable<number> {
    return this.http.get<{ count: number }>(`${this.apiUrl}/count?customerId=${customerId}`).pipe(
      map(response => response.count)
    );
  }
}
```

## 🧪 Tests

```typescript
describe('UserRepositoryImpl', () => {
  let repository: UserRepositoryImpl;
  let httpMock: HttpTestingController;
  let mapper: UserMapper;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserRepositoryImpl, UserMapper]
    });

    repository = TestBed.inject(UserRepositoryImpl);
    httpMock = TestBed.inject(HttpTestingController);
    mapper = TestBed.inject(UserMapper);
  });

  afterEach(() => {
    httpMock.verify();
  });

  it('should find user by id', (done) => {
    const mockDTO: UserDTO = {
      id: '123',
      email: 'test@test.com',
      firstName: 'John',
      lastName: 'Doe'
    };

    repository.findById('123').subscribe(user => {
      expect(user).toBeDefined();
      expect(user!.id).toBe('123');
      expect(user!.email.getValue()).toBe('test@test.com');
      done();
    });

    const req = httpMock.expectOne('/api/users/123');
    expect(req.request.method).toBe('GET');
    req.flush(mockDTO);
  });

  it('should return null for non-existing user', (done) => {
    repository.findById('999').subscribe(user => {
      expect(user).toBeNull();
      done();
    });

    const req = httpMock.expectOne('/api/users/999');
    req.flush(null, { status: 404, statusText: 'Not Found' });
  });

  it('should save new user', (done) => {
    const user = new User('', Email.create('new@test.com'), 'New', 'User');
    const mockDTO: UserDTO = {
      id: '456',
      email: 'new@test.com',
      firstName: 'New',
      lastName: 'User'
    };

    repository.save(user).subscribe(savedUser => {
      expect(savedUser.id).toBe('456');
      done();
    });

    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('POST');
    req.flush(mockDTO);
  });
});
```

## 🎓 Points clés

1. **Implémente** : Interfaces du Domain
2. **HttpClient** : Pour les appels API
3. **Mappers** : Transformation DTO ↔ Domain
4. **Gestion d'erreurs** : 404 → null, autres → throwError
5. **Cache** : Si pertinent pour performance
6. **@Injectable** : Service Angular
7. **Tests** : Avec HttpClientTestingModule

---

**Les repository implementations sont le pont entre votre logique et les données ! 📦**
