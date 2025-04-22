# ANEXE TEHNICE

## 7.1 Diagrama Arhitecturii GCP

```
                    [Client Apps]
                         │
                         ▼
                   [Cloud Load Balancer]
                         │
                         ▼
                   [Apigee Gateway] ◄───► [Firebase Auth]
                         │
                         ▼
                 [Google Kubernetes Engine]
                         │
          ┌──────────────┼───────────────┐
          │              │               │
          ▼              ▼               ▼
 [API Microservices] [Data Services] [Realtime Services]
          │              │               │
          └──────► [Cloud SQL] ◄─────────┘
                      │
                      ▼
               [Cloud Storage]
```

## 7.2 Schema Baza de Date (Simplificată)

```
+---------------+     +---------------+     +---------------+
| School        |     | Student       |     | User          |
+---------------+     +---------------+     +---------------+
| id            |     | id            |     | id            |
| name          |     | school_id     |◄────| email         |
| address       |     | first_name    |     | password_hash |
| city          |     | last_name     |     | role          |
| state         |     | grade         |     | first_name    |
| postal_code   |     | guardian_name |     | last_name     |
| country       |     | guardian_phone|     | created_at    |
| phone         |     | created_at    |     | updated_at    |
| email         |     | updated_at    |     +---------------+
| type          |     +---------------+
| geo_lat       |
| geo_lng       |
| active        |◄────┐
| created_at    |     │
| updated_at    |     │
+---------------+     │
                      │
+---------------+     │                     +---------------+
| Bus           |     │                     | RouteStop     |
+---------------+     │                     | id            |
+---------------+     │                     | route_id      |◄─┐
| id            |     │                     | stop_number   |  │
| name          |     │                     | address       |  │
| license_plate |     │                     | latitude      |  │
| make          |     │                     | longitude     |  │
| model         |     │                     | arrival_time  |  │
| capacity      |     │                     | created_at    |  │
| driver_name   |     │                     | updated_at    |  │
| driver_phone  |     │                     |             +---------------+
| active        |     │                     |             | Route         |
+---------------+     │                     |             +---------------+
       ▲              │                     |             | id            |
       │              │                     |             | name          |
       │              │                     |             | description   |
+---------------+     │                     |             | school_id     |
| VehiclePosition|    │                     |             | start_time    |
+---------------+     │                     |             | end_time      |
| id            |     │                     |             | active        |
| bus_id        |◄────┘                     |             | created_at    |
| latitude      |                           |             | updated_at    |
| longitude     |                           |             +---------------+
| heading       |                           |
| speed         |                           |
| timestamp     |                           |
| created_at    |                           |
| updated_at    |                           |
+---------------+                           +---------------+
```

## 7.3 API Endpoints (Exemple)

**Bus Management Service:**
- GET    /api/buses                - Get all buses
- GET    /api/buses/{id}           - Get specific bus
- POST   /api/buses                - Create new bus
- PUT    /api/buses/{id}           - Update bus
- DELETE /api/buses/{id}           - Delete bus
- GET    /api/buses/{id}/position  - Get last known position
- GET    /api/buses/{id}/route     - Get current route
- GET    /api/buses/{id}/schedule  - Get bus schedule

**School & Student Management:**
- GET    /api/schools                    - Get all schools
- GET    /api/schools/{id}               - Get specific school
- POST   /api/schools                    - Create school
- GET    /api/schools/{id}/students      - Get students of school
- POST   /api/students                   - Create student
- GET    /api/students/{id}              - Get specific student
- PUT    /api/students/{id}              - Update student
- DELETE /api/students/{id}              - Delete student

## 7.4 Implementare Firebase Auth

```javascript
// Configurare Firebase
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "schoolbus-app.firebaseapp.com",
  projectId: "schoolbus-app",
  storageBucket: "schoolbus-app.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef123456"
};

// Inițializare Firebase
firebase.initializeApp(firebaseConfig);

// Autentificare utilizator
const signIn = async (email, password) => {
  try {
    const userCredential = await firebase.auth().signInWithEmailAndPassword(email, password);
    const user = userCredential.user;
    const token = await user.getIdToken();
    
    // Token-ul poate fi folosit pentru autorizare API
    return { user, token };
  } catch (error) {
    console.error("Error signing in:", error);
    throw error;
  }
};
```

## 7.5 Real-time Tracking Implementation

```typescript
// NestJS WebSocket Gateway
@WebSocketGateway({ namespace: 'tracking' })
export class TrackingGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;
  
  private logger = new Logger('TrackingGateway');
  
  handleConnection(client: Socket, ...args: any[]) {
    this.logger.log(`Client connected: ${client.id}`);
  }
  
  handleDisconnect(client: Socket) {
    this.logger.log(`Client disconnected: ${client.id}`);
  }
  
  @SubscribeMessage('subscribeToVehicle')
  handleSubscribeToVehicle(client: Socket, vehicleId: string) {
    client.join(`vehicle:${vehicleId}`);
    return { event: 'subscribed', data: { vehicleId } };
  }
  
  // Trimite update-uri de poziție către clienți
  sendPositionUpdate(vehicleId: string, position: VehiclePosition) {
    this.server.to(`vehicle:${vehicleId}`).emit('positionUpdate', position);
  }
}
```