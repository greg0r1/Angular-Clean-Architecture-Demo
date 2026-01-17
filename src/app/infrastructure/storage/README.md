# 💾 Storage - Stockage local

## 📋 Description

Le dossier **Storage** contient les services de stockage local (LocalStorage, SessionStorage, IndexedDB, etc.).

## ✅ Ce qu'on doit mettre

- ✅ Services LocalStorage
- ✅ Services SessionStorage
- ✅ Services IndexedDB
- ✅ Services de cache
- ✅ Sérialisation/Désérialisation
- ✅ Gestion des erreurs de storage

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Logique métier (→ Domain)
- ❌ Appels HTTP (→ http/)
- ❌ Repositories (→ repositories/)

## 📊 Exemples

### LocalStorage Service

```typescript
// local-storage.service.ts
@Injectable({ providedIn: 'root' })
export class LocalStorageService {
  set<T>(key: string, value: T): void {
    try {
      const serialized = JSON.stringify(value);
      localStorage.setItem(key, serialized);
    } catch (error) {
      console.error('Error saving to localStorage', error);
    }
  }

  get<T>(key: string): T | null {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : null;
    } catch (error) {
      console.error('Error reading from localStorage', error);
      return null;
    }
  }

  remove(key: string): void {
    try {
      localStorage.removeItem(key);
    } catch (error) {
      console.error('Error removing from localStorage', error);
    }
  }

  clear(): void {
    try {
      localStorage.clear();
    } catch (error) {
      console.error('Error clearing localStorage', error);
    }
  }

  has(key: string): boolean {
    return localStorage.getItem(key) !== null;
  }

  keys(): string[] {
    return Object.keys(localStorage);
  }
}
```

### Service avec expiration

```typescript
// storage-with-expiry.service.ts
interface StorageItem<T> {
  value: T;
  expiry: number;
}

@Injectable({ providedIn: 'root' })
export class StorageWithExpiryService {
  constructor(private storage: LocalStorageService) {}

  set<T>(key: string, value: T, ttlMs: number): void {
    const item: StorageItem<T> = {
      value,
      expiry: Date.now() + ttlMs
    };
    this.storage.set(key, item);
  }

  get<T>(key: string): T | null {
    const item = this.storage.get<StorageItem<T>>(key);

    if (!item) {
      return null;
    }

    // Vérifier l'expiration
    if (Date.now() > item.expiry) {
      this.storage.remove(key);
      return null;
    }

    return item.value;
  }

  remove(key: string): void {
    this.storage.remove(key);
  }
}
```

### SessionStorage Service

```typescript
// session-storage.service.ts
@Injectable({ providedIn: 'root' })
export class SessionStorageService {
  set<T>(key: string, value: T): void {
    try {
      sessionStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('Error saving to sessionStorage', error);
    }
  }

  get<T>(key: string): T | null {
    try {
      const item = sessionStorage.getItem(key);
      return item ? JSON.parse(item) : null;
    } catch (error) {
      console.error('Error reading from sessionStorage', error);
      return null;
    }
  }

  remove(key: string): void {
    sessionStorage.removeItem(key);
  }

  clear(): void {
    sessionStorage.clear();
  }
}
```

### Auth Storage Service (exemple d'utilisation)

```typescript
// auth-storage.service.ts
@Injectable({ providedIn: 'root' })
export class AuthStorageService {
  private readonly TOKEN_KEY = 'auth_token';
  private readonly USER_KEY = 'current_user';
  private readonly REFRESH_TOKEN_KEY = 'refresh_token';

  constructor(private storage: LocalStorageService) {}

  saveToken(token: string): void {
    this.storage.set(this.TOKEN_KEY, token);
  }

  getToken(): string | null {
    return this.storage.get<string>(this.TOKEN_KEY);
  }

  removeToken(): void {
    this.storage.remove(this.TOKEN_KEY);
  }

  saveRefreshToken(token: string): void {
    this.storage.set(this.REFRESH_TOKEN_KEY, token);
  }

  getRefreshToken(): string | null {
    return this.storage.get<string>(this.REFRESH_TOKEN_KEY);
  }

  saveUser(user: any): void {
    this.storage.set(this.USER_KEY, user);
  }

  getUser(): any {
    return this.storage.get(this.USER_KEY);
  }

  clearAuth(): void {
    this.removeToken();
    this.storage.remove(this.REFRESH_TOKEN_KEY);
    this.storage.remove(this.USER_KEY);
  }

  isAuthenticated(): boolean {
    return this.getToken() !== null;
  }
}
```

### IndexedDB Service (exemple simple)

```typescript
// indexed-db.service.ts
@Injectable({ providedIn: 'root' })
export class IndexedDBService {
  private dbName = 'myAppDB';
  private dbVersion = 1;
  private db?: IDBDatabase;

  async init(): Promise<void> {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, this.dbVersion);

      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve();
      };

      request.onupgradeneeded = (event) => {
        const db = (event.target as IDBOpenDBRequest).result;
        if (!db.objectStoreNames.contains('items')) {
          db.createObjectStore('items', { keyPath: 'id' });
        }
      };
    });
  }

  async save<T extends { id: string }>(storeName: string, item: T): Promise<void> {
    if (!this.db) await this.init();

    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction([storeName], 'readwrite');
      const store = transaction.objectStore(storeName);
      const request = store.put(item);

      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }

  async get<T>(storeName: string, id: string): Promise<T | null> {
    if (!this.db) await this.init();

    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction([storeName], 'readonly');
      const store = transaction.objectStore(storeName);
      const request = store.get(id);

      request.onsuccess = () => resolve(request.result || null);
      request.onerror = () => reject(request.error);
    });
  }

  async getAll<T>(storeName: string): Promise<T[]> {
    if (!this.db) await this.init();

    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction([storeName], 'readonly');
      const store = transaction.objectStore(storeName);
      const request = store.getAll();

      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }

  async delete(storeName: string, id: string): Promise<void> {
    if (!this.db) await this.init();

    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction([storeName], 'readwrite');
      const store = transaction.objectStore(storeName);
      const request = store.delete(id);

      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
}
```

## 🧪 Tests

```typescript
describe('LocalStorageService', () => {
  let service: LocalStorageService;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [LocalStorageService]
    });
    service = TestBed.inject(LocalStorageService);
    localStorage.clear();
  });

  it('should save and retrieve value', () => {
    service.set('test', { foo: 'bar' });
    const value = service.get<{ foo: string }>('test');

    expect(value).toEqual({ foo: 'bar' });
  });

  it('should return null for non-existent key', () => {
    const value = service.get('nonexistent');

    expect(value).toBeNull();
  });

  it('should remove value', () => {
    service.set('test', 'value');
    service.remove('test');
    const value = service.get('test');

    expect(value).toBeNull();
  });

  it('should clear all values', () => {
    service.set('key1', 'value1');
    service.set('key2', 'value2');
    service.clear();

    expect(service.get('key1')).toBeNull();
    expect(service.get('key2')).toBeNull();
  });
});
```

## 🎓 Points clés

1. **Sérialisation** : JSON.stringify/parse automatique
2. **Gestion d'erreurs** : Try/catch pour quota exceeded
3. **Typage** : Génériques pour type-safety
4. **Expiration** : Pour les données temporaires
5. **SessionStorage** : Données de session
6. **IndexedDB** : Grandes quantités de données
7. **Auth** : Service dédié pour tokens

---

**Le storage local permet de persister les données côté client ! 💾**
