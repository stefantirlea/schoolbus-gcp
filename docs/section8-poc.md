# PROOF OF CONCEPT: APIGEE + FIREBASE AUTH + NESTJS

## Arhitectură POC

Pentru implementarea rapidă a unui Proof of Concept care să demonstreze arhitectura, vom folosi:

- **Apigee API Gateway** - pentru management API centralizat
- **Firebase Auth** - pentru autentificare și gestionare JWT
- **NestJS** - pentru microservicii
- **Cloud SQL** - pentru stocarea datelor

```
[Client Apps] → [Apigee API Gateway] ⟷ [Firebase Auth]
                      ↓
            [NestJS Microservicii] ↔ [PostgreSQL]
```

## Plan de Implementare POC (4-5 zile)

### 1. Setup Inițial (1 zi)

**A. Creare Proiect GCP**
```bash
# Creare proiect GCP
gcloud projects create schoolbus-poc-123 --name="SchoolBus POC"

# Activare billing
gcloud billing projects link schoolbus-poc-123 --billing-account=BILLING_ACCOUNT_ID

# Activare APIs necesare
gcloud services enable apigee.googleapis.com firebase.googleapis.com \
  container.googleapis.com cloudbuild.googleapis.com \
  secretmanager.googleapis.com --project=schoolbus-poc-123
```

**B. Setup Firebase**
```bash
# Instalare Firebase CLI
npm install -g firebase-tools

# Login și inițializare Firebase
firebase login
firebase init --project=schoolbus-poc-123 authentication
```

**C. Setup GKE Minimal**
```bash
# Creare cluster GKE pentru microservicii
gcloud container clusters create schoolbus-poc-cluster \
  --project=schoolbus-poc-123 \
  --zone=us-central1-a \
  --num-nodes=1 \
  --machine-type=e2-standard-2
```

### 2. Firebase Auth (0.5 zi)

**A. Configurare Firebase Auth**
Activăm autentificarea cu email/parolă și creăm utilizatori de test.

**B. Firebase Admin SDK Setup**
```javascript
// scripts/setup-users.js
const admin = require('firebase-admin');
const serviceAccount = require('./service-account.json');

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount)
});

async function setupUsers() {
  // Creează utilizator administrator
  const adminUser = await admin.auth().createUser({
    email: 'admin@schoolbus.com',
    password: 'adminpassword',
    displayName: 'Admin User'
  });
  
  // Setează custom claim pentru rol
  await admin.auth().setCustomUserClaims(adminUser.uid, { role: 'admin' });
  
  // Creează utilizator standard
  const standardUser = await admin.auth().createUser({
    email: 'user@schoolbus.com',
    password: 'userpassword',
    displayName: 'Standard User'
  });
  
  await admin.auth().setCustomUserClaims(standardUser.uid, { role: 'user' });
  
  console.log('Users created with role claims');
}

setupUsers();
```

### 3. Implementare Apigee (1 zi)

**A. Configurare Organizație și Environment Apigee**
Creăm organizația și environment-ul Apigee pentru POC.

**B. Crearea API Proxy**
Implementăm două proxy-uri:
1. **Firebase Auth Proxy** - pentru managementul token-urilor
2. **Demo API Proxy** - pentru accesul la microservicii

**C. Politici Apigee**
```xml
<VerifyJWT name="VF-ValidateFirebaseToken">
  <DisplayName>Verify Firebase JWT</DisplayName>
  <JwksUri>https://www.googleapis.com/service_accounts/v1/jwk/securetoken@system.gserviceaccount.com</JwksUri>
  <Audience>schoolbus-poc-123</Audience>
  <Issuer>https://securetoken.google.com/schoolbus-poc-123</Issuer>
  <Source>request.header.authorization</Source>
  <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
</VerifyJWT>

<ExtractVariables name="EV-ExtractRole">
  <DisplayName>Extract Role</DisplayName>
  <Source>jwt.decoded.claim.role</Source>
  <VariablePrefix>jwt</VariablePrefix>
  <Variable name="role">role</Variable>
</ExtractVariables>
```

### 4. Microserviciu Demo NestJS (1-2 zile)

**A. Structură Proiect NestJS**
Cream un proiect NestJS pentru gestionarea școlilor și autobuzelor.

**B. Implementare Entități și Repository**
```typescript
// src/schools/entities/school.entity.ts
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class School {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @Column()
  address: string;

  @Column({ nullable: true })
  phone: string;

  @Column({ default: true })
  isActive: boolean;
  
  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

**C. Implementare Controllers și Services**
```typescript
// src/schools/schools.controller.ts
import { Controller, Get, Post, Body, Param, UseGuards } from '@nestjs/common';
import { SchoolsService } from './schools.service';
import { CreateSchoolDto } from './dto/create-school.dto';
import { JwtAuthGuard } from '../auth/jwt-auth.guard';
import { RolesGuard } from '../auth/roles.guard';
import { Roles } from '../auth/roles.decorator';

@Controller('schools')
@UseGuards(JwtAuthGuard, RolesGuard)
export class SchoolsController {
  constructor(private readonly schoolsService: SchoolsService) {}

  @Get()
  findAll() {
    return this.schoolsService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.schoolsService.findOne(id);
  }

  @Post()
  @Roles('admin')
  create(@Body() createSchoolDto: CreateSchoolDto) {
    return this.schoolsService.create(createSchoolDto);
  }
}
```

### 5. Frontend Demo (1 zi)

**A. Componente Login cu Firebase**
```javascript
// Login.jsx
import { useState } from 'react';
import { getAuth, signInWithEmailAndPassword } from "firebase/auth";
import { initializeApp } from "firebase/app";

const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "schoolbus-poc-123.firebaseapp.com",
  projectId: "schoolbus-poc-123",
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);

function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleLogin = async (e) => {
    e.preventDefault();
    try {
      const userCredential = await signInWithEmailAndPassword(auth, email, password);
      const idToken = await userCredential.user.getIdToken();
      localStorage.setItem('idToken', idToken);
      window.location.href = '/dashboard';
    } catch (error) {
      console.error("Error:", error);
      alert("Authentication failed");
    }
  };

  return (
    <div>
      <h1>Login - SchoolBus POC</h1>
      <form onSubmit={handleLogin}>
        {/* Form fields */}
        <button type="submit">Login</button>
      </form>
    </div>
  );
}
```

**B. API Client**
```javascript
// api/client.js
const API_URL = 'https://api.example.com';

export async function fetchWithAuth(endpoint, options = {}) {
  const idToken = localStorage.getItem('idToken');
  
  if (!idToken) {
    window.location.href = '/login';
    return;
  }
  
  try {
    const response = await fetch(`${API_URL}${endpoint}`, {
      ...options,
      headers: {
        ...options.headers,
        'Authorization': `Bearer ${idToken}`,
        'Content-Type': 'application/json',
      },
    });
    
    if (response.status === 401) {
      localStorage.removeItem('idToken');
      window.location.href = '/login';
      return;
    }
    
    return await response.json();
  } catch (error) {
    console.error('API Error:', error);
    throw error;
  }
}
```

## Avantajele Abordării POC

1. **Timp Redus de Implementare** - 4-5 zile pentru un POC funcțional
2. **Utilizare Servicii Managed** - Firebase Auth și Apigee reduc complexitatea
3. **GCP Native** - Toate componentele sunt servicii native GCP
4. **Consistență** - Utilizarea aceleiași platforme pentru toate serviciile
5. **Performanță** - Optimizare automată prin servicii managed
6. **Securitate** - Integrare nativă între serviciile GCP

## Integrare cu Arhitectura Finală

Acest POC demonstrează conceptele cheie ale arhitecturii și poate fi extins pentru implementarea completă:

1. **Extensie Microservicii** - Adăugare microservicii pentru restul entităților
2. **Implementare Tracking** - Adăugare servicii real-time pentru tracking
3. **Mobile App** - Dezvoltare React Native pentru mobile
4. **Admin Dashboard** - Implementare dashboard complet

POC-ul validează arhitectura bazată pe GCP și oferă fundament pentru implementarea completă conform planului în 2 faze.