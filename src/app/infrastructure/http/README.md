# 🌐 HTTP - Services et intercepteurs

## 📋 Description

Le dossier **HTTP** contient les services HTTP génériques, les intercepteurs, et les configurations réseau pour l'application.

## ✅ Ce qu'on doit mettre

- ✅ Intercepteurs HTTP (Auth, Logging, Errors...)
- ✅ Services HTTP génériques
- ✅ Configuration d'API
- ✅ Gestion d'erreurs centralisée
- ✅ Retry logic
- ✅ Request/Response transformations

## ❌ Ce qu'on ne doit PAS mettre

- ❌ Repositories (→ infrastructure/repositories)
- ❌ Logique métier (→ Domain)
- ❌ Use cases (→ Application)

## 📊 Exemples

### Intercepteur d'authentification

```typescript
// auth.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token) {
    req = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
    });
  }

  return next(req);
};
```

### Intercepteur de gestion d'erreurs

```typescript
// error.interceptor.ts
import { HttpInterceptorFn, HttpErrorResponse } from '@angular/common/http';
import { inject } from '@angular/core';
import { catchError, throwError } from 'rxjs';

export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const notificationService = inject(NotificationService);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      let errorMessage = 'Une erreur est survenue';

      if (error.error instanceof ErrorEvent) {
        // Erreur côté client
        errorMessage = `Erreur: ${error.error.message}`;
      } else {
        // Erreur côté serveur
        switch (error.status) {
          case 400:
            errorMessage = 'Requête invalide';
            break;
          case 401:
            errorMessage = 'Non autorisé';
            break;
          case 403:
            errorMessage = 'Accès interdit';
            break;
          case 404:
            errorMessage = 'Ressource non trouvée';
            break;
          case 500:
            errorMessage = 'Erreur serveur';
            break;
          default:
            errorMessage = `Erreur ${error.status}: ${error.message}`;
        }
      }

      notificationService.showError(errorMessage);
      return throwError(() => error);
    })
  );
};
```

### Intercepteur de logging

```typescript
// logging.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { tap, catchError } from 'rxjs/operators';

export const loggingInterceptor: HttpInterceptorFn = (req, next) => {
  const logger = inject(LoggingService);
  const started = Date.now();

  return next(req).pipe(
    tap({
      next: (event) => {
        if (event.type === HttpEventType.Response) {
          const elapsed = Date.now() - started;
          logger.info(`${req.method} ${req.url} - ${event.status} (${elapsed}ms)`);
        }
      },
      error: (error) => {
        const elapsed = Date.now() - started;
        logger.error(`${req.method} ${req.url} - Error (${elapsed}ms)`, error);
      }
    })
  );
};
```

### Service API de base

```typescript
// api.service.ts
@Injectable({ providedIn: 'root' })
export class ApiService {
  private readonly baseUrl = environment.apiUrl;

  constructor(private http: HttpClient) {}

  get<T>(endpoint: string, params?: HttpParams): Observable<T> {
    return this.http.get<T>(`${this.baseUrl}${endpoint}`, { params });
  }

  post<T>(endpoint: string, body: any): Observable<T> {
    return this.http.post<T>(`${this.baseUrl}${endpoint}`, body);
  }

  put<T>(endpoint: string, body: any): Observable<T> {
    return this.http.put<T>(`${this.baseUrl}${endpoint}`, body);
  }

  delete<T>(endpoint: string): Observable<T> {
    return this.http.delete<T>(`${this.baseUrl}${endpoint}`);
  }

  patch<T>(endpoint: string, body: any): Observable<T> {
    return this.http.patch<T>(`${this.baseUrl}${endpoint}`, body);
  }
}
```

### Intercepteur avec retry

```typescript
// retry.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';
import { retry, timer } from 'rxjs';

export const retryInterceptor: HttpInterceptorFn = (req, next) => {
  // Retry 3 fois avec délai exponentiel pour les requêtes GET
  if (req.method === 'GET') {
    return next(req).pipe(
      retry({
        count: 3,
        delay: (error, retryCount) => {
          // Délai exponentiel: 1s, 2s, 4s
          const delayMs = Math.pow(2, retryCount - 1) * 1000;
          console.log(`Retry ${retryCount} after ${delayMs}ms`);
          return timer(delayMs);
        }
      })
    );
  }

  return next(req);
};
```

### Configuration des intercepteurs

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { authInterceptor } from './infrastructure/http/auth.interceptor';
import { errorInterceptor } from './infrastructure/http/error.interceptor';
import { loggingInterceptor } from './infrastructure/http/logging.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([
        loggingInterceptor,
        authInterceptor,
        errorInterceptor
      ])
    )
  ]
};
```

## 🎓 Points clés

1. **Intercepteurs** : Logique transversale HTTP
2. **Ordre** : Important pour les intercepteurs (logging → auth → error)
3. **Retry** : Pour les requêtes idempotentes (GET)
4. **Logging** : Tracer les requêtes
5. **Auth** : Ajouter tokens automatiquement
6. **Erreurs** : Gestion centralisée

---

**Les intercepteurs HTTP permettent une gestion transversale élégante ! 🌐**
