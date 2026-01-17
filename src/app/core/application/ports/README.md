# 🔌 Ports - Interfaces pour services externes

## 📋 Description

Les **Ports** sont des **interfaces** qui définissent les contrats pour les services externes dont l'application a besoin, sans spécifier leur implémentation.

> **Principe clé** : Un port définit **QUOI** (le contrat), l'adaptateur (Infrastructure) définit **COMMENT** (l'implémentation).

## 🎯 Origine du concept

Le terme "Port" vient de l'**architecture hexagonale** (Ports & Adapters) :
- **Port** : Interface côté application (ce que l'application attend)
- **Adapter** : Implémentation côté infrastructure (comment c'est fait)

## 🔄 Flux de dépendance

```
Use Case (Application)
       │
       │ dépend de (interface)
       ▼
Port (Application) ◄─── Nous sommes ICI
       ▲
       │ implémente
       │
Adapter (Infrastructure)
       │
       ▼
Service Externe (Email, SMS, Storage...)
```

## ✅ Ce qu'on doit mettre

### Types de ports
- ✅ Interfaces pour services externes
- ✅ Interfaces pour notifications (Email, SMS, Push...)
- ✅ Interfaces pour stockage (File, Cloud...)
- ✅ Interfaces pour services tiers (Payment, Maps...)
- ✅ Interfaces pour génération (PDF, Export...)
- ✅ Interfaces pour authentification externe

### Exemples
```typescript
// ✅ Ports typiques
export interface EmailService {
  sendEmail(to: Email, subject: string, body: string): Observable<void>;
}

export interface NotificationService {
  send(userId: string, message: string): Observable<void>;
}

export interface FileStorageService {
  upload(file: File, path: string): Observable<string>;
  download(path: string): Observable<Blob>;
}

export interface PaymentService {
  processPayment(amount: Money, cardToken: string): Observable<PaymentResult>;
}

export interface PdfGeneratorService {
  generateInvoice(invoice: Invoice): Observable<Blob>;
}
```

## ❌ Ce qu'on ne doit PAS mettre

### Interdictions
- ❌ Implémentations concrètes (→ Infrastructure)
- ❌ Imports de librairies externes
- ❌ Détails techniques (SMTP, AWS SDK...)
- ❌ Configuration (URLs, clés API...)
- ❌ Annotations Angular (@Injectable)

```typescript
// ❌ INCORRECT - Implémentation dans Application
import { HttpClient } from '@angular/common/http'; // ❌ NON !

@Injectable() // ❌ Pas dans Application !
export class GmailService implements EmailService {
  constructor(private http: HttpClient) {} // ❌ Détails techniques

  sendEmail(to: Email, subject: string, body: string): Observable<void> {
    return this.http.post('https://gmail.api...', { }); // ❌ Implémentation
  }
}

// ✅ CORRECT - Seulement l'interface dans Application
export interface EmailService {
  sendEmail(to: Email, subject: string, body: string): Observable<void>;
}
```

## 🏗️ Structure d'un Port

### Template de base

```typescript
import { Observable } from 'rxjs';
import { DomainType } from '../../domain/...';

export interface ServiceName {
  // Méthodes métier avec types du Domain
  methodName(param: DomainType): Observable<ResultType>;
}

// Token d'injection (optionnel mais recommandé)
export const SERVICE_NAME = new InjectionToken<ServiceName>('ServiceName');
```

## 📖 Bonnes pratiques

### 1. Noms métier (pas techniques)

```typescript
// ✅ CORRECT - Noms métier
export interface EmailService {
  sendWelcomeEmail(to: Email): Observable<void>;
  sendPasswordResetEmail(to: Email, token: string): Observable<void>;
  sendOrderConfirmation(order: Order): Observable<void>;
}

export interface NotificationService {
  notifyUser(userId: string, message: string): Observable<void>;
  notifyAdmins(message: string): Observable<void>;
}

// ❌ INCORRECT - Noms techniques
export interface SmtpService { // ❌ Détail technique
  sendViaSMTP(config: SmtpConfig): Observable<void>;
}

export interface AwsS3Service { // ❌ Détail d'implémentation
  putObject(bucket: string, key: string): Observable<void>;
}
```

### 2. Utiliser les types du Domain

Les ports doivent utiliser les entités et value objects du Domain.

```typescript
// ✅ CORRECT - Types du Domain
import { Email } from '../../domain/value-objects/email';
import { Order } from '../../domain/entities/order.entity';
import { Invoice } from '../../domain/entities/invoice.entity';

export interface EmailService {
  sendEmail(to: Email, subject: string, body: string): Observable<void>;
  sendOrderConfirmation(order: Order): Observable<void>;
}

export interface PdfGeneratorService {
  generateInvoice(invoice: Invoice): Observable<Blob>;
  generateReport(data: ReportData): Observable<Blob>;
}

// ❌ INCORRECT - Types primitifs ou techniques
export interface EmailService {
  sendEmail(to: string, subject: string, body: string): Observable<void>; // ❌ string au lieu de Email
}

export interface PdfGeneratorService {
  generate(html: string): Observable<Blob>; // ❌ Perd le type métier
}
```

### 3. Retourner des Observables

```typescript
// ✅ CORRECT - Observables
export interface FileStorageService {
  upload(file: File, path: string): Observable<string>;
  download(path: string): Observable<Blob>;
  delete(path: string): Observable<void>;
  exists(path: string): Observable<boolean>;
}

// ❌ INCORRECT - Promises (pas idiomatique Angular)
export interface FileStorageService {
  upload(file: File, path: string): Promise<string>; // ❌ Préférer Observable
}
```

### 4. Méthodes spécifiques au métier

```typescript
// ✅ CORRECT - Méthodes métier spécifiques
export interface EmailService {
  sendWelcomeEmail(to: Email): Observable<void>;
  sendPasswordResetEmail(to: Email, resetToken: string): Observable<void>;
  sendOrderConfirmation(order: Order): Observable<void>;
  sendShippingNotification(order: Order, trackingNumber: string): Observable<void>;
  sendInvoice(invoice: Invoice, to: Email): Observable<void>;
}

// ❌ INCORRECT - Méthode générique
export interface EmailService {
  send(options: any): Observable<void>; // ❌ Trop vague
}
```

### 5. Interface Segregation

Séparer les interfaces selon les responsabilités.

```typescript
// ✅ CORRECT - Interfaces séparées
export interface EmailNotificationService {
  sendEmail(to: Email, subject: string, body: string): Observable<void>;
}

export interface SmsNotificationService {
  sendSms(to: PhoneNumber, message: string): Observable<void>;
}

export interface PushNotificationService {
  sendPush(userId: string, title: string, body: string): Observable<void>;
}

// Le use case choisit ce dont il a besoin
export class SendWelcomeNotificationUseCase {
  constructor(
    private emailService: EmailNotificationService, // Seulement email
    private pushService: PushNotificationService    // Seulement push
  ) {}
}

// ❌ INCORRECT - Interface monolithique
export interface NotificationService {
  sendEmail(...): Observable<void>;
  sendSms(...): Observable<void>;
  sendPush(...): Observable<void>;
  sendSlack(...): Observable<void>;
  sendWebhook(...): Observable<void>;
  // ❌ Tous les clients doivent dépendre de toutes les méthodes
}
```

## 📊 Exemples complets

### Exemple 1 : Email Service

```typescript
import { Observable } from 'rxjs';
import { Email } from '../../domain/value-objects/email';
import { Order } from '../../domain/entities/order.entity';
import { Invoice } from '../../domain/entities/invoice.entity';
import { User } from '../../domain/entities/user.entity';

export interface EmailService {
  // Emails utilisateur
  sendWelcomeEmail(to: Email): Observable<void>;
  sendAccountActivationEmail(to: Email, activationLink: string): Observable<void>;
  sendPasswordResetEmail(to: Email, resetToken: string): Observable<void>;
  sendPasswordChangedConfirmation(to: Email): Observable<void>;

  // Emails de commande
  sendOrderConfirmation(order: Order): Observable<void>;
  sendOrderCancellation(order: Order, reason: string): Observable<void>;
  sendShippingNotification(order: Order, trackingNumber: string): Observable<void>;

  // Emails de facturation
  sendInvoice(invoice: Invoice, to: Email): Observable<void>;
  sendPaymentReceived(invoice: Invoice, to: Email): Observable<void>;

  // Email générique (pour cas spéciaux)
  sendCustomEmail(to: Email, subject: string, htmlBody: string): Observable<void>;
}

// Token d'injection (optionnel)
import { InjectionToken } from '@angular/core';

export const EMAIL_SERVICE = new InjectionToken<EmailService>('EmailService');
```

### Exemple 2 : File Storage Service

```typescript
import { Observable } from 'rxjs';

export interface FileMetadata {
  readonly path: string;
  readonly size: number;
  readonly mimeType: string;
  readonly uploadedAt: Date;
}

export interface FileStorageService {
  // Upload
  upload(file: File, path: string): Observable<string>; // Retourne URL

  // Download
  download(path: string): Observable<Blob>;
  getDownloadUrl(path: string): Observable<string>;

  // Métadonnées
  getMetadata(path: string): Observable<FileMetadata>;
  exists(path: string): Observable<boolean>;

  // Suppression
  delete(path: string): Observable<void>;
  deleteMultiple(paths: string[]): Observable<void>;

  // Listage
  listFiles(directory: string): Observable<FileMetadata[]>;
}

export const FILE_STORAGE_SERVICE = new InjectionToken<FileStorageService>(
  'FileStorageService'
);
```

### Exemple 3 : Payment Service

```typescript
import { Observable } from 'rxjs';
import { Money } from '../../domain/value-objects/money';

export interface PaymentMethod {
  readonly type: 'card' | 'bank' | 'paypal';
  readonly token: string;
}

export interface PaymentResult {
  readonly success: boolean;
  readonly transactionId: string;
  readonly processedAt: Date;
  readonly amount: Money;
  readonly errorMessage?: string;
}

export interface RefundResult {
  readonly success: boolean;
  readonly refundId: string;
  readonly processedAt: Date;
  readonly amount: Money;
}

export interface PaymentService {
  // Paiement
  processPayment(
    amount: Money,
    method: PaymentMethod,
    orderId: string
  ): Observable<PaymentResult>;

  // Remboursement
  refundPayment(
    transactionId: string,
    amount: Money,
    reason: string
  ): Observable<RefundResult>;

  // Vérification
  verifyTransaction(transactionId: string): Observable<PaymentResult>;

  // Tokenisation (pour sauvegarder les cartes)
  tokenizeCard(cardNumber: string, expiryDate: string, cvv: string): Observable<string>;
}

export const PAYMENT_SERVICE = new InjectionToken<PaymentService>(
  'PaymentService'
);
```

### Exemple 4 : Notification Service

```typescript
import { Observable } from 'rxjs';
import { Email } from '../../domain/value-objects/email';
import { PhoneNumber } from '../../domain/value-objects/phone-number';

export enum NotificationPriority {
  Low = 'low',
  Normal = 'normal',
  High = 'high',
  Critical = 'critical'
}

export interface Notification {
  readonly userId: string;
  readonly title: string;
  readonly body: string;
  readonly priority: NotificationPriority;
  readonly data?: Record<string, any>;
}

export interface NotificationService {
  // Notification push
  sendPushNotification(notification: Notification): Observable<void>;

  // Notification par email
  sendEmailNotification(to: Email, subject: string, body: string): Observable<void>;

  // Notification par SMS
  sendSmsNotification(to: PhoneNumber, message: string): Observable<void>;

  // Notification in-app
  sendInAppNotification(userId: string, message: string): Observable<void>;

  // Notification multiple (tous les canaux)
  sendMultiChannelNotification(
    userId: string,
    title: string,
    body: string
  ): Observable<void>;
}

export const NOTIFICATION_SERVICE = new InjectionToken<NotificationService>(
  'NotificationService'
);
```

### Exemple 5 : PDF Generator Service

```typescript
import { Observable } from 'rxjs';
import { Invoice } from '../../domain/entities/invoice.entity';
import { Order } from '../../domain/entities/order.entity';

export interface PdfOptions {
  readonly format?: 'A4' | 'Letter';
  readonly orientation?: 'portrait' | 'landscape';
  readonly margin?: {
    top: number;
    right: number;
    bottom: number;
    left: number;
  };
}

export interface PdfGeneratorService {
  // Génération de factures
  generateInvoice(invoice: Invoice, options?: PdfOptions): Observable<Blob>;

  // Génération de bons de commande
  generateOrderReceipt(order: Order, options?: PdfOptions): Observable<Blob>;

  // Génération de rapports
  generateReport(
    title: string,
    data: any[],
    options?: PdfOptions
  ): Observable<Blob>;

  // Génération depuis HTML
  generateFromHtml(html: string, options?: PdfOptions): Observable<Blob>;
}

export const PDF_GENERATOR_SERVICE = new InjectionToken<PdfGeneratorService>(
  'PdfGeneratorService'
);
```

### Exemple 6 : Logging Service

```typescript
import { Observable } from 'rxjs';

export enum LogLevel {
  Debug = 'debug',
  Info = 'info',
  Warning = 'warning',
  Error = 'error',
  Critical = 'critical'
}

export interface LogEntry {
  readonly level: LogLevel;
  readonly message: string;
  readonly timestamp: Date;
  readonly userId?: string;
  readonly context?: Record<string, any>;
  readonly error?: Error;
}

export interface LoggingService {
  // Logging simple
  debug(message: string, context?: Record<string, any>): void;
  info(message: string, context?: Record<string, any>): void;
  warning(message: string, context?: Record<string, any>): void;
  error(message: string, error?: Error, context?: Record<string, any>): void;
  critical(message: string, error?: Error, context?: Record<string, any>): void;

  // Logging avec Observable (pour tracking)
  logAction(action: string, userId: string, data?: any): Observable<void>;

  // Récupération des logs
  getLogs(filters?: LogFilters): Observable<LogEntry[]>;
}

export interface LogFilters {
  readonly level?: LogLevel;
  readonly userId?: string;
  readonly startDate?: Date;
  readonly endDate?: Date;
}

export const LOGGING_SERVICE = new InjectionToken<LoggingService>(
  'LoggingService'
);
```

## 🔧 Utilisation dans les Use Cases

```typescript
// Use case utilisant plusieurs ports
@Injectable({ providedIn: 'root' })
export class ProcessOrderUseCase {
  constructor(
    private orderRepository: OrderRepository,
    private paymentService: PaymentService,      // Port
    private emailService: EmailService,          // Port
    private notificationService: NotificationService, // Port
    private loggingService: LoggingService       // Port
  ) {}

  execute(
    orderId: string,
    paymentMethod: PaymentMethod
  ): Observable<Order> {
    return this.orderRepository.findById(orderId).pipe(
      switchMap(order => {
        if (!order) throw new OrderNotFoundError(orderId);

        const total = order.calculateTotal();

        // Traiter le paiement
        return this.paymentService.processPayment(
          total,
          paymentMethod,
          orderId
        ).pipe(
          map(result => {
            if (!result.success) {
              throw new PaymentFailedError(result.errorMessage);
            }
            return order.markAsPaid(result.transactionId);
          })
        );
      }),
      switchMap(order => this.orderRepository.save(order)),
      tap(order => {
        // Effets de bord
        this.emailService.sendOrderConfirmation(order).subscribe();
        this.notificationService.sendPushNotification({
          userId: order.customerId,
          title: 'Commande confirmée',
          body: `Votre commande ${order.id} a été confirmée`,
          priority: NotificationPriority.Normal
        }).subscribe();
        this.loggingService.info('Order processed', { orderId: order.id });
      })
    );
  }
}
```

## 🧪 Tests avec mocks de ports

```typescript
describe('ProcessOrderUseCase', () => {
  let useCase: ProcessOrderUseCase;
  let mockPaymentService: jest.Mocked<PaymentService>;
  let mockEmailService: jest.Mocked<EmailService>;

  beforeEach(() => {
    mockPaymentService = {
      processPayment: jest.fn(),
      refundPayment: jest.fn(),
      verifyTransaction: jest.fn(),
      tokenizeCard: jest.fn()
    };

    mockEmailService = {
      sendOrderConfirmation: jest.fn(() => of(void 0)),
      sendWelcomeEmail: jest.fn(() => of(void 0))
    } as any;

    useCase = new ProcessOrderUseCase(
      mockOrderRepository,
      mockPaymentService,
      mockEmailService,
      mockNotificationService,
      mockLoggingService
    );
  });

  it('should process order successfully', (done) => {
    const paymentResult: PaymentResult = {
      success: true,
      transactionId: 'txn-123',
      processedAt: new Date(),
      amount: Money.create(100, 'EUR')
    };

    mockPaymentService.processPayment.mockReturnValue(of(paymentResult));
    mockOrderRepository.findById.mockReturnValue(of(mockOrder));
    mockOrderRepository.save.mockImplementation(order => of(order));

    useCase.execute('order-1', mockPaymentMethod).subscribe({
      next: (order) => {
        expect(order).toBeDefined();
        expect(mockPaymentService.processPayment).toHaveBeenCalled();
        expect(mockEmailService.sendOrderConfirmation).toHaveBeenCalled();
        done();
      }
    });
  });
});
```

## 🎯 Configuration avec Injection Tokens

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { EMAIL_SERVICE, EmailService } from './core/application/ports/email.service';
import { EmailServiceImpl } from './infrastructure/email/email-service.impl';

export const appConfig: ApplicationConfig = {
  providers: [
    // Binding port → adapter
    {
      provide: EMAIL_SERVICE,
      useClass: EmailServiceImpl
    },
    {
      provide: PAYMENT_SERVICE,
      useClass: StripePaymentServiceImpl
    },
    {
      provide: FILE_STORAGE_SERVICE,
      useClass: AwsS3StorageServiceImpl
    }
  ]
};

// Utilisation dans un use case
export class SendEmailUseCase {
  constructor(
    @Inject(EMAIL_SERVICE) private emailService: EmailService
  ) {}
}
```

## 🎓 Points clés

1. **Interface seulement** : Pas d'implémentation dans Application
2. **Noms métier** : Pas de détails techniques
3. **Types du Domain** : Utiliser entités et value objects
4. **Observables** : Pour l'asynchrone
5. **Interface Segregation** : Séparer les responsabilités
6. **Injection Tokens** : Pour le binding explicite
7. **Testabilité** : Facile à mocker
8. **Inversion de dépendance** : Application définit, Infrastructure implémente

---

**Les ports sont vos contrats avec le monde extérieur ! 🔌**
