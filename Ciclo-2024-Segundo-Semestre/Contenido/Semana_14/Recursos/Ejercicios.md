# Ejercicios de Sistemas Distribuidos

## 1. Comunicación entre procesos con gRPC

### Ejercicio 1.1: Servicio de Gestión de Usuarios (Go)
**Objetivo**: Crear un servicio gRPC básico para gestionar usuarios.

**Solución**:

1. Definir el protocolo (users.proto):
```protobuf
syntax = "proto3";
package users;
option go_package = "./proto";

service UserService {
  rpc CreateUser (CreateUserRequest) returns (UserResponse);
  rpc GetUser (GetUserRequest) returns (UserResponse);
}

message User {
  string id = 1;
  string name = 2;
  string email = 3;
}

message CreateUserRequest {
  string name = 1;
  string email = 2;
}

message GetUserRequest {
  string id = 1;
}

message UserResponse {
  User user = 1;
}
```

2. Implementar el servidor (server.go):
```go
package main

import (
    "context"
    "log"
    "net"
    pb "myapp/proto"
    "google.golang.org/grpc"
)

type server struct {
    pb.UnimplementedUserServiceServer
    users map[string]*pb.User
}

func (s *server) CreateUser(ctx context.Context, req *pb.CreateUserRequest) (*pb.UserResponse, error) {
    user := &pb.User{
        Id:    generateID(), // Implementar función de generación de ID
        Name:  req.Name,
        Email: req.Email,
    }
    s.users[user.Id] = user
    return &pb.UserResponse{User: user}, nil
}

func (s *server) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.UserResponse, error) {
    if user, exists := s.users[req.Id]; exists {
        return &pb.UserResponse{User: user}, nil
    }
    return nil, status.Error(codes.NotFound, "usuario no encontrado")
}

func main() {
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }
    s := grpc.NewServer()
    pb.RegisterUserServiceServer(s, &server{
        users: make(map[string]*pb.User),
    })
    log.Printf("server listening at %v", lis.Addr())
    if err := s.Serve(lis); err != nil {
        log.Fatalf("failed to serve: %v", err)
    }
}
```

3. Implementar el cliente (client.go):
```go
package main

import (
    "context"
    "log"
    "time"
    pb "myapp/proto"
    "google.golang.org/grpc"
)

func main() {
    conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure())
    if err != nil {
        log.Fatalf("did not connect: %v", err)
    }
    defer conn.Close()
    c := pb.NewUserServiceClient(conn)

    ctx, cancel := context.WithTimeout(context.Background(), time.Second)
    defer cancel()

    // Crear usuario
    r, err := c.CreateUser(ctx, &pb.CreateUserRequest{
        Name:  "Juan Pérez",
        Email: "juan@ejemplo.com",
    })
    if err != nil {
        log.Fatalf("could not create user: %v", err)
    }
    log.Printf("Usuario creado: %v", r.GetUser())

    // Obtener usuario
    user, err := c.GetUser(ctx, &pb.GetUserRequest{Id: r.GetUser().Id})
    if err != nil {
        log.Fatalf("could not get user: %v", err)
    }
    log.Printf("Usuario encontrado: %v", user.GetUser())
}
```

### Ejercicio 1.2: Servicio de Mensajería (Rust)
**Objetivo**: Implementar un servicio de mensajería en tiempo real usando gRPC en Rust.

**Solución**:

1. Definir el protocolo (messages.proto):
```protobuf
syntax = "proto3";
package messages;

service MessageService {
  rpc SendMessage (Message) returns (MessageResponse);
  rpc SubscribeToMessages (SubscribeRequest) returns (stream Message);
}

message Message {
  string id = 1;
  string content = 2;
  string sender = 3;
  int64 timestamp = 4;
}

message MessageResponse {
  bool success = 1;
  string message_id = 2;
}

message SubscribeRequest {
  string user_id = 1;
}
```

2. Implementar el servidor (main.rs):
```rust
use tonic::{transport::Server, Request, Response, Status};
use messages::message_service_server::{MessageService, MessageServiceServer};
use messages::{Message, MessageResponse, SubscribeRequest};
use tokio::sync::broadcast;
use tokio_stream::wrappers::BroadcastStream;
use std::time::{SystemTime, UNIX_EPOCH};

pub mod messages {
    tonic::include_proto!("messages");
}

#[derive(Debug)]
struct MessageServiceImpl {
    tx: broadcast::Sender<Message>,
}

#[tonic::async_trait]
impl MessageService for MessageServiceImpl {
    async fn send_message(
        &self,
        request: Request<Message>,
    ) -> Result<Response<MessageResponse>, Status> {
        let msg = request.into_inner();
        let msg_id = uuid::Uuid::new_v4().to_string();
        
        let message = Message {
            id: msg_id.clone(),
            content: msg.content,
            sender: msg.sender,
            timestamp: SystemTime::now()
                .duration_since(UNIX_EPOCH)
                .unwrap()
                .as_secs() as i64,
        };

        if let Err(e) = self.tx.send(message) {
            return Err(Status::internal(format!("Error al enviar mensaje: {}", e)));
        }

        Ok(Response::new(MessageResponse {
            success: true,
            message_id: msg_id,
        }))
    }

    type SubscribeToMessagesStream = BroadcastStream<Message>;

    async fn subscribe_to_messages(
        &self,
        request: Request<SubscribeRequest>,
    ) -> Result<Response<Self::SubscribeToMessagesStream>, Status> {
        let rx = self.tx.subscribe();
        Ok(Response::new(BroadcastStream::new(rx)))
    }
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let addr = "[::1]:50051".parse()?;
    let (tx, _) = broadcast::channel(100);
    
    let service = MessageServiceImpl { tx };
    
    Server::builder()
        .add_service(MessageServiceServer::new(service))
        .serve(addr)
        .await?;

    Ok(())
}
```

3. Implementar el cliente (client.rs):
```rust
use messages::message_service_client::MessageServiceClient;
use messages::{Message, SubscribeRequest};
use tonic::Request;

pub mod messages {
    tonic::include_proto!("messages");
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut client = MessageServiceClient::connect("http://[::1]:50051").await?;

    // Enviar mensaje
    let message = Message {
        id: String::new(),
        content: "¡Hola mundo!".into(),
        sender: "usuario1".into(),
        timestamp: 0,
    };

    let response = client
        .send_message(Request::new(message))
        .await?;
    println!("Mensaje enviado: {:?}", response);

    // Suscribirse a mensajes
    let mut stream = client
        .subscribe_to_messages(Request::new(SubscribeRequest {
            user_id: "usuario1".into(),
        }))
        .await?
        .into_inner();

    while let Some(message) = stream.message().await? {
        println!("Mensaje recibido: {:?}", message);
    }

    Ok(())
}
```

## 2. Bases de datos NoSQL para Sistemas Distribuidos

### Ejercicio 2.1: Implementación de un Sistema de Caché Distribuido con Redis
**Objetivo**: Crear un sistema de caché distribuido para almacenar resultados de consultas frecuentes.

**Solución**:

```go
package main

import (
    "context"
    "encoding/json"
    "time"
    "github.com/go-redis/redis/v8"
)

type CacheService struct {
    client *redis.Client
}

type Product struct {
    ID    string  `json:"id"`
    Name  string  `json:"name"`
    Price float64 `json:"price"`
}

func NewCacheService() *CacheService {
    client := redis.NewClient(&redis.Options{
        Addr:     "localhost:6379",
        Password: "", 
        DB:       0,
    })
    
    return &CacheService{client: client}
}

func (c *CacheService) Set(ctx context.Context, key string, value interface{}, expiration time.Duration) error {
    json, err := json.Marshal(value)
    if err != nil {
        return err
    }
    
    return c.client.Set(ctx, key, json, expiration).Err()
}

func (c *CacheService) Get(ctx context.Context, key string, dest interface{}) error {
    val, err := c.client.Get(ctx, key).Result()
    if err != nil {
        return err
    }
    
    return json.Unmarshal([]byte(val), dest)
}

func main() {
    ctx := context.Background()
    cache := NewCacheService()
    
    // Ejemplo de uso
    product := Product{
        ID:    "1",
        Name:  "Laptop",
        Price: 999.99,
    }
    
    // Guardar en caché
    err := cache.Set(ctx, "product:1", product, 24*time.Hour)
    if err != nil {
        panic(err)
    }
    
    // Recuperar de caché
    var cachedProduct Product
    err = cache.Get(ctx, "product:1", &cachedProduct)
    if err != nil {
        panic(err)
    }
}
```

### Ejercicio 2.2: Sistema de Almacenamiento de Documentos con MongoDB
**Objetivo**: Implementar un sistema de almacenamiento de documentos con replicación.

**Solución**:

```go
package main

import (
    "context"
    "time"
    "go.mongodb.org/mongo-driver/bson"
    "go.mongodb.org/mongo-driver/mongo"
    "go.mongodb.org/mongo-driver/mongo/options"
)

type Document struct {
    ID        string    `bson:"_id,omitempty"`
    Title     string    `bson:"title"`
    Content   string    `bson:"content"`
    Tags      []string  `bson:"tags"`
    CreatedAt time.Time `bson:"created_at"`
    UpdatedAt time.Time `bson:"updated_at"`
}

type DocumentStore struct {
    client     *mongo.Client
    collection *mongo.Collection
}

func NewDocumentStore(uri string) (*DocumentStore, error) {
    ctx := context.Background()
    client, err := mongo.Connect(ctx, options.Client().ApplyURI(uri))
    if err != nil {
        return nil, err
    }
    
    collection := client.Database("documents").Collection("docs")
    
    return &DocumentStore{
        client:     client,
        collection: collection,
    }, nil
}

func (ds *DocumentStore) CreateDocument(ctx context.Context, doc *Document) error {
    doc.CreatedAt = time.Now()
    doc.UpdatedAt = doc.CreatedAt
    
    _, err := ds.collection.InsertOne(ctx, doc)
    return err
}

func (ds *DocumentStore) UpdateDocument(ctx context.Context, id string, updates bson.M) error {
    updates["updated_at"] = time.Now()
    
    _, err := ds.collection.UpdateOne(
        ctx,
        bson.M{"_id": id},
        bson.M{"$set": updates},
    )
    return err
}

func (ds *DocumentStore) FindDocuments(ctx context.Context, filter bson.M) ([]Document, error) {
    cursor, err := ds.collection.Find(ctx, filter)
    if err != nil {
        return nil, err
    }
    defer cursor.Close(ctx)
    
    var documents []Document
    if err = cursor.All(ctx, &documents); err != nil {
        return nil, err
    }
    
    return documents, nil
}

func main() {
    ctx := context.Background()
    store, err := NewDocumentStore("mongodb://localhost:27017")
    if err != nil {
        panic(err)
    }
    
    // Ejemplo de uso
    doc := &Document{
        Title:   "Mi Documento",
        Content: "Contenido del documento",
        Tags:    []string{"ejemplo", "documento"},
    }
    
    // Crear documento
    err = store.CreateDocument(ctx, doc)
    if err != nil {
        panic(err)
    }
    
    // Buscar documentos
    docs, err := store.FindDocuments(ctx, bson.M{"tags": "ejemplo"})
    if err != nil {
        panic(err)
    }
    
    // Actualizar documento
    err = store.UpdateDocument(ctx, doc.ID, bson.M{
        "content": "Contenido actualizado",
    })
    if err != nil {
        panic(err)
    }
}
```