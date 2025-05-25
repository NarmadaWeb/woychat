# woychat
Free Chat With Your Friend

### Struktur Web
```bash
+----------------+       +----------------+
|    Frontend    |       |     Admin      |
|  (HTML, JS)    |       |     Panel      |
+----------------+       +----------------+
        |                        |
        |  HTTPS (REST/WebSockets)  |
        v                        v
+------------------------------------+
|            API Gateway             | (Nginx, Kong, Ocelot, custom Go gateway)
+------------------------------------+
        |   |   |   |   |
        |   |   |   |   +--------------------------+
        |   |   |   +--------------------------+   |
        |   |   +--------------------------+   |   |
        |   +--------------------------+   |   |   |
        v                              v   v   v   v
+----------------+  +----------------+  +----------------+  +----------------+  +----------------+
|  Auth Service  |  |  User Service  |  |  Chat Service  |  | Notif. Service |  | Media Service  |
|  (Golang App)  |  |  (Golang App)  |  |  (Golang App)  |  |  (Golang App)  |  |  (Golang App)  |
| - Login/Reg.   |  | - User Profiles|  | - Messages     |  | - Push Notif.  |  | - Upload/Down. |
| - JWT issuance |  | - Contacts     |  | - Real-time WS |  | - In-app Notif.|  | - Image Resize |
+----------------+  +----------------+  +----------------+  +----------------+  +----------------+
        |                  |                 |                      |                  |
        |                  |                 |                      |                  |
        +------------------+-----------------+----------------------+------------------+
        |                  |                 |                      |                  |
        v                  v                 v                      v                  v
+-----------------+  +-----------------+  +-----------------+  +-----------------+  +-----------------+
|   DB (Postgres) |  |   DB (Postgres) |  |   DB (Postgres) |  |   DB (Postgres) |  |Object Storage(S3)|
|    (e.g., Auth) |  |    (e.g., Users)|  |    (e.g., Chats)|  |   (e.g., Notif.)|  +-----------------+
+-----------------+  +-----------------+  +-----------------+  +-----------------+
                             |
                             +-----------------------+
                                                     |
                                                     v
+----------------------------------------------------+
|                 Message Broker (Kafka/RabbitMQ)    |
| (e.g., for user status updates, new message events)|
+----------------------------------------------------+
                             |
                             v
+----------------------------------------------------+
|                       Redis                        |
| (Caching, WS Session Management, Pub/Sub for Broker)|
+----------------------------------------------------+
```
### Structure File and folder
```bash
woychat/
├── .env
├── docker-compose.yml
├── README.md

├── services/
│   ├── api-gateway/
│   │   ├── Dockerfile
│   │   ├── main.go
│   │   ├── go.mod
│   │   ├── go.sum
│   │   ├── routes/
│   │   │   └── routes.go
│   │   ├── middlewares/
│   │   │   └── auth_proxy.go
│   │   └── config/
│   │       └── config.go
│   │       └── utils.go
│   ├── auth-service/
│   │   ├── Dockerfile
│   │   ├── main.go
│   │   ├── go.mod
│   │   ├── go.sum
│   │   ├── internal/
│   │   │   ├── auth/
│   │   │   │   ├── handler.go
│   │   │   │   ├── service.go
│   │   │   │   └── repository.go
│   │   │   └── models/
│   │   │       └── user.go
│   │   ├── pkg/
│   │   │   ├── jwt/
│   │   │   │   └── jwt.go
│   │   │   └── password/
│   │   │       └── password.go
│   │   └── config/
│   │       └── database.go
│   │       └── config.go
│   │
│   ├── user-service/
│   │   ├── Dockerfile
│   │   ├── main.go
│   │   ├── go.mod
│   │   ├── go.sum
│   │   ├── internal/
│   │   │   ├── user/
│   │   │   │   ├── handler.go
│   │   │   │   ├── service.go
│   │   │   │   └── repository.go
│   │   │   └── models/
│   │   │       └── user.
│   │   ├── pkg/
│   │   │   └── grpc_clients/
│   │   │       └── auth_client.go
│   │   │   └── utils/
│   │   │       └── common.go
│   │   ├── config/
│   │   │   └── database.go
│   │   │   └── config.go
│   │
│   ├── chat-service/
│   │   ├── Dockerfile
│   │   ├── main.go
│   │   ├── go.mod
│   │   ├── go.sum
│   │   ├── internal/
│   │   │   ├── chat/
│   │   │   │   ├── handler.go
│   │   │   │   ├── service.go
│   │   │   │   └── repository.go
│   │   │   │   └── websocket.go
│   │   │   └── models/
│   │   │       ├── message.go
│   │   │       └── chat.go
│   │   │       └── participant.go
│   │   ├── pkg/
│   │   │   ├── grpc_clients/
│   │   │   │   └── user_client.go
│   │   │   └── middleware/
│   │   │       └── auth.go
│   │   └── config/
│   │       └── database.go
│   │       └── redis.go
│   │       └── config.go
│   │
│   ├── notification-service/
│   │   ├── Dockerfile
│   │   ├── main.go
│   │   ├── go.mod
│   │   ├── go.sum
│   │   ├── internal/
│   │   │   ├── notification/
│   │   │   │   ├── handler.go
│   │   │   │   ├── service.go)
│   │   │   │   └── consumer.go
│   │   │   └── models/
│   │   │       └── notification.go
│   │   ├── pkg/
│   │   │   ├── firebase/
│   │   │   │   └── fcm.go
│   │   │   └── grpc_clients/
│   │   │       └── user_client.go
│   │   └── config/
│   │       └── broker.go
│   │       └── config.go
│   │
│   ├── media-service/
│   │   ├── Dockerfile
│   │   ├── main.go
│   │   ├── go.mod
│   │   ├── go.sum
│   │   ├── internal/
│   │   │   ├── media/
│   │   │   │   ├── handler.go
│   │   │   │   ├── service.go
│   │   │   │   └── s3.go
│   │   │   └── models/
│   │   │       └── media.go
│   │   ├── pkg/
│   │   │   └── utils/
│   │   │       └── image.go
│   │   └── config/
│   │       └── s3.go
│   │       └── config.go
│   │
│   └── proto/
│       ├── auth.proto
│       ├── user.proto
│       ├── chat.proto
│       └── media.proto
```
