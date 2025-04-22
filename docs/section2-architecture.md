# ARHITECTURĂ TEHNICĂ

## 2.1 Prezentare Generală

Arhitectura SchoolBus este concepută ca o platformă cloud-native bazată pe microservicii, implementată integral pe Google Cloud Platform, folosind Kubernetes Engine pentru orchestrare și Apigee pentru API management.

**Principii arhitecturale:**
- **Cloud-Native**: Utilizare completă a serviciilor Google Cloud
- **Microservicii**: Dezvoltare și scalare independentă a componentelor
- **API-First**: Toate funcționalitățile expuse prin API-uri REST documentate
- **Infrastructure-as-Code**: Întreaga infrastructură definită și versionată în Terraform
- **Security-by-Design**: Securitate integrată în toate etapele, nu adăugată ulterior

## 2.2 Arhitectura Cloud și Infrastructură

### Google Cloud Platform (GCP)
- **Google Kubernetes Engine (GKE)**:
  - Auto-scaling node pools (2-6 noduri)
  - Regional cluster pentru disponibilitate înaltă
  - Automatic node upgrades și auto-repair
- **Cloud SQL**: 
  - PostgreSQL pentru stocarea datelor
  - High availability și automatic failover
  - Backup automat
- **Cloud Storage**:
  - Multi-regional pentru documente și fișiere
  - Lifecycle policies pentru optimizare costuri
- **Cloud Functions**:
  - Pentru procesare evenimente și integrări
  - Serverless pentru costuri optimizate

### Networking & Security
- **Cloud Load Balancing**:
  - HTTPS load balancer pentru aplicații
  - Health checks pentru failover automat
- **VPC & Security**:
  - Configurare network policies pentru izolare servicii
  - Private clusters pentru GKE
  - Cloud Armor pentru protecție DDoS și WAF
  - Cloud IAM pentru autorizare granulară
  - Secret Manager pentru credentials și secrets

## 2.3 Arhitectura Aplicației

### Backend Microservicii NestJS
- **Apigee API Gateway**:
  - Management centralizat al API-urilor
  - Autentificare și autorizare
  - Rate limiting și monitoring
  - Versioning și documentare API
- **Identity & Access Management**:
  - Firebase Authentication
  - Role-Based Access Control (RBAC)
  - JWT pentru tokenuri securizate
- **School Management Service**:
  - NestJS cu TypeORM
  - CRUD pentru școli
  - Management utilizatori și roluri
- **Student Management Service**:
  - NestJS cu TypeORM
  - CRUD pentru elevi
  - Asociere cu școli și rute
- **Bus Management Service**:
  - NestJS cu TypeORM
  - CRUD pentru flota de autobuze
  - Management șoferi și vehicule
- **Route Planning Service**:
  - NestJS cu TypeORM
  - Planificare și optimizare rute
  - Integrare cu Google Maps Platform
  - Calculare timpi de sosire estimați
- **Live Tracking Service**:
  - NestJS cu Socket.IO pentru real-time
  - Procesare date GPS în timp real
  - Algoritmi pentru tracking și ETA updates
- **Notification Service**:
  - Cloud Functions pentru procesare evenimente
  - Firebase Cloud Messaging pentru notificări push
  - SendGrid pentru email notifications

### Frontend Components
- **Web Application**:
  - Next.js (React) cu SSR
  - Material UI pentru design consistent
  - Progressive Web App capabilities
  - Apollo Client pentru GraphQL (opțional)
- **Mobile App**:
  - React Native (iOS & Android)
  - Offline mode cu sincronizare
  - Background tracking
  - Push notifications

## 2.4 Securitate

### Authentication & Authorization
- **Firebase Authentication**:
  - Multiple metode de autentificare (email/password, social)
  - JWT pentru stocarea și verificarea tokenurilor
  - MFA pentru conturi administrative
- **Google Cloud IAM**:
  - Service accounts pentru microservicii
  - Permisiuni granulare pentru resurse cloud
  - Auditing și monitoring

### Data Security
- **Encryption**:
  - Encryption in transit (TLS 1.3)
  - Encryption at rest pentru toate datele
  - Column-level encryption pentru date sensibile
- **Network Security**:
  - Private GKE clusters
  - VPC Service Controls
  - Firewall rules pentru limitarea traficului

## 2.5 Data Management

### Database Strategy
- **Cloud SQL for PostgreSQL**:
  - Instanță primary pentru producție
  - Read replicas pentru scalare
  - Point-in-time recovery
- **Cloud Firestore (opțional)**:
  - Pentru date real-time (locații vehicule)
  - Queries în timp real pentru tracking

### Entități Core în Modelul de Date:
- School: Informații despre școală
- Student: Elevi înregistrați
- Bus: Autobuze în flotă
- Route: Rute de transport predefinite
- VehiclePosition: Date real-time tracking
- User: Utilizatori sistem cu roluri

## 2.6 Integrări

1. **Google Maps Platform**:
   - Directions API pentru rutare
   - Geocoding API pentru adrese
   - Distance Matrix API pentru calcul timp
   - Maps Javascript API pentru vizualizare

2. **Notification Services**:
   - Firebase Cloud Messaging
   - SendGrid pentru email
   - Twilio pentru SMS (opțional)

3. **External Systems**:
   - API-uri pentru integrare cu sisteme existente școlare
   - Import/export date