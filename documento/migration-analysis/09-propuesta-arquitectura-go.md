# 9. Propuesta de Arquitectura Go: Diseño y Estructura

## Arquitectura General del Sistema

### Estructura de Módulos
```
baileys-go/
├── cmd/
│   ├── baileys/              # CLI principal
│   ├── server/               # HTTP/gRPC server
│   └── tools/                # Herramientas de desarrollo
├── pkg/
│   ├── baileys/              # Core library
│   ├── events/               # Sistema de eventos
│   ├── signal/               # Signal Protocol integration
│   ├── websocket/            # WhatsApp WebSocket client
│   ├── storage/              # Storage backends
│   └── media/                # Media processing
├── internal/
│   ├── config/               # Configuration management
│   ├── metrics/              # Observability
│   └── proto/                # Generated protobuf code
├── examples/
│   ├── basic-bot/            # Bot básico
│   ├── web-integration/      # Integración web
│   └── enterprise/           # Uso empresarial
└── docs/
    ├── api/                  # API documentation
    ├── migration/            # Migration guides
    └── examples/             # Usage examples
```

## Core Package Design

### 1. Main Baileys Package
```go
// pkg/baileys/baileys.go
package baileys

import (
    "context"
    "github.com/whiskeysockets/baileys-go/pkg/events"
    "github.com/whiskeysockets/baileys-go/pkg/storage"
    "github.com/whiskeysockets/baileys-go/pkg/websocket"
)

type Client struct {
    config    *Config
    ws        *websocket.Client
    events    *events.Hub
    storage   storage.Provider
    signal    signal.Store
    logger    *zerolog.Logger
    ctx       context.Context
    cancel    context.CancelFunc
}

type Config struct {
    LogLevel        string        `yaml:"log_level"`
    Storage         StorageConfig `yaml:"storage"`
    WebSocket       WSConfig      `yaml:"websocket"`
    Signal          SignalConfig  `yaml:"signal"`
    EventBuffer     EventConfig   `yaml:"events"`
    Media           MediaConfig   `yaml:"media"`
}

// Constructor principal
func NewClient(config *Config) (*Client, error) {
    ctx, cancel := context.WithCancel(context.Background())
    
    // Initialize components
    logger := setupLogger(config.LogLevel)
    storage := storage.New(config.Storage)
    eventHub := events.NewHub(config.EventBuffer)
    signalStore := signal.NewStore(storage)
    wsClient := websocket.New(config.WebSocket, eventHub, logger)
    
    return &Client{
        config:  config,
        ws:      wsClient,
        events:  eventHub,
        storage: storage,
        signal:  signalStore,
        logger:  logger,
        ctx:     ctx,
        cancel:  cancel,
    }, nil
}

// API principal
func (c *Client) Connect() error {
    return c.ws.Connect(c.ctx)
}

func (c *Client) SendMessage(jid string, message *Message) (*MessageInfo, error) {
    return c.ws.SendMessage(c.ctx, jid, message)
}

func (c *Client) On(event EventType, handler EventHandler) {
    c.events.Subscribe(event, handler)
}

func (c *Client) Close() error {
    c.cancel()
    return c.ws.Close()
}
```

### 2. Event System Architecture
```go
// pkg/events/hub.go
package events

import (
    "context"
    "sync"
    "time"
)

type EventType string

const (
    ConnectionUpdate EventType = "connection.update"
    MessagesUpsert   EventType = "messages.upsert"
    ChatsUpdate      EventType = "chats.update"
    // ... más eventos
)

type Event struct {
    Type      EventType   `json:"type"`
    Data      interface{} `json:"data"`
    Timestamp time.Time   `json:"timestamp"`
    ID        string      `json:"id"`
}

type EventHandler func(Event) error

type Hub struct {
    subscribers map[EventType][]EventHandler
    buffer      *Buffer
    middleware  []Middleware
    mu          sync.RWMutex
    ctx         context.Context
}

func NewHub(config BufferConfig) *Hub {
    return &Hub{
        subscribers: make(map[EventType][]EventHandler),
        buffer:      NewBuffer(config),
        middleware:  []Middleware{},
    }
}

func (h *Hub) Subscribe(eventType EventType, handler EventHandler) {
    h.mu.Lock()
    defer h.mu.Unlock()
    h.subscribers[eventType] = append(h.subscribers[eventType], handler)
}

func (h *Hub) Emit(eventType EventType, data interface{}) error {
    event := Event{
        Type:      eventType,
        Data:      data,
        Timestamp: time.Now(),
        ID:        generateEventID(),
    }
    
    // Apply middleware
    for _, middleware := range h.middleware {
        if err := middleware.Process(&event); err != nil {
            return err
        }
    }
    
    // Check if buffering
    if h.buffer.IsActive() {
        return h.buffer.Add(event)
    }
    
    // Emit directly
    return h.emit(event)
}

func (h *Hub) emit(event Event) error {
    h.mu.RLock()
    handlers := h.subscribers[event.Type]
    h.mu.RUnlock()
    
    // Parallel execution
    var wg sync.WaitGroup
    errCh := make(chan error, len(handlers))
    
    for _, handler := range handlers {
        wg.Add(1)
        go func(h EventHandler) {
            defer wg.Done()
            if err := h(event); err != nil {
                errCh <- err
            }
        }(handler)
    }
    
    wg.Wait()
    close(errCh)
    
    // Collect errors
    var errors []error
    for err := range errCh {
        errors = append(errors, err)
    }
    
    if len(errors) > 0 {
        return fmt.Errorf("handlers failed: %v", errors)
    }
    
    return nil
}
```

### 3. Event Buffer Implementation
```go
// pkg/events/buffer.go
package events

import (
    "sync"
    "time"
)

type Buffer struct {
    active      bool
    events      map[EventType][]Event
    consolidate map[EventType]Consolidator
    history     map[string]bool
    config      BufferConfig
    mu          sync.Mutex
    ticker      *time.Ticker
}

type BufferConfig struct {
    Size           int           `yaml:"size"`
    FlushInterval  time.Duration `yaml:"flush_interval"`
    ConsolidateAll bool          `yaml:"consolidate_all"`
}

type Consolidator interface {
    Consolidate(events []Event) (Event, error)
}

func NewBuffer(config BufferConfig) *Buffer {
    b := &Buffer{
        events:      make(map[EventType][]Event),
        consolidate: make(map[EventType]Consolidator),
        history:     make(map[string]bool),
        config:      config,
    }
    
    // Setup consolidators
    b.setupConsolidators()
    
    // Auto-flush ticker
    if config.FlushInterval > 0 {
        b.ticker = time.NewTicker(config.FlushInterval)
        go b.autoFlush()
    }
    
    return b
}

func (b *Buffer) Activate() {
    b.mu.Lock()
    defer b.mu.Unlock()
    b.active = true
}

func (b *Buffer) Add(event Event) error {
    b.mu.Lock()
    defer b.mu.Unlock()
    
    if !b.active {
        return ErrBufferNotActive
    }
    
    // Check for duplicates
    if b.isDuplicate(event) {
        return nil
    }
    
    // Add to buffer
    b.events[event.Type] = append(b.events[event.Type], event)
    b.history[event.ID] = true
    
    // Check buffer size
    if len(b.events[event.Type]) >= b.config.Size {
        return b.flushType(event.Type)
    }
    
    return nil
}

func (b *Buffer) Flush() (map[EventType]Event, error) {
    b.mu.Lock()
    defer b.mu.Unlock()
    
    if !b.active {
        return nil, ErrBufferNotActive
    }
    
    consolidated := make(map[EventType]Event)
    
    for eventType, events := range b.events {
        if len(events) == 0 {
            continue
        }
        
        // Consolidate events
        if consolidator, exists := b.consolidate[eventType]; exists {
            consolidated[eventType], _ = consolidator.Consolidate(events)
        } else {
            // Take last event if no consolidator
            consolidated[eventType] = events[len(events)-1]
        }
    }
    
    // Reset buffer
    b.reset()
    b.active = false
    
    return consolidated, nil
}

func (b *Buffer) setupConsolidators() {
    b.consolidate[MessagesUpsert] = &MessageConsolidator{}
    b.consolidate[ChatsUpdate] = &ChatConsolidator{}
    // ... más consolidators
}
```

### 4. WebSocket Client Design
```go
// pkg/websocket/client.go
package websocket

import (
    "context"
    "crypto/tls"
    "github.com/gorilla/websocket"
    "github.com/whiskeysockets/baileys-go/pkg/events"
    "github.com/whiskeysockets/baileys-go/pkg/noise"
)

type Client struct {
    conn        *websocket.Conn
    events      *events.Hub
    noise       *noise.Handler
    frameQueue  chan Frame
    config      Config
    logger      *zerolog.Logger
    ctx         context.Context
    cancel      context.CancelFunc
}

type Config struct {
    URL             string            `yaml:"url"`
    Headers         map[string]string `yaml:"headers"`
    Timeout         time.Duration     `yaml:"timeout"`
    ReconnectDelay  time.Duration     `yaml:"reconnect_delay"`
    MaxReconnects   int               `yaml:"max_reconnects"`
    TLSConfig       *tls.Config       `yaml:"-"`
}

func New(config Config, eventHub *events.Hub, logger *zerolog.Logger) *Client {
    ctx, cancel := context.WithCancel(context.Background())
    
    return &Client{
        events:     eventHub,
        config:     config,
        logger:     logger,
        frameQueue: make(chan Frame, 1000),
        ctx:        ctx,
        cancel:     cancel,
    }
}

func (c *Client) Connect(ctx context.Context) error {
    dialer := &websocket.Dialer{
        TLSClientConfig:  c.config.TLSConfig,
        HandshakeTimeout: c.config.Timeout,
    }
    
    conn, _, err := dialer.DialContext(ctx, c.config.URL, c.makeHeaders())
    if err != nil {
        return fmt.Errorf("websocket connect failed: %w", err)
    }
    
    c.conn = conn
    c.noise = noise.NewHandler(c.events, c.logger)
    
    // Start goroutines
    go c.readPump()
    go c.writePump()
    go c.frameProcessor()
    
    return nil
}

func (c *Client) readPump() {
    defer c.conn.Close()
    
    c.conn.SetReadDeadline(time.Now().Add(60 * time.Second))
    c.conn.SetPongHandler(func(string) error {
        c.conn.SetReadDeadline(time.Now().Add(60 * time.Second))
        return nil
    })
    
    for {
        select {
        case <-c.ctx.Done():
            return
        default:
            _, message, err := c.conn.ReadMessage()
            if err != nil {
                c.handleError(err)
                return
            }
            
            frame, err := c.noise.DecodeFrame(message)
            if err != nil {
                c.logger.Error().Err(err).Msg("frame decode failed")
                continue
            }
            
            select {
            case c.frameQueue <- frame:
            case <-c.ctx.Done():
                return
            }
        }
    }
}

func (c *Client) writePump() {
    ticker := time.NewTicker(54 * time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-c.ctx.Done():
            return
        case <-ticker.C:
            c.conn.SetWriteDeadline(time.Now().Add(10 * time.Second))
            if err := c.conn.WriteMessage(websocket.PingMessage, nil); err != nil {
                return
            }
        }
    }
}

func (c *Client) frameProcessor() {
    for {
        select {
        case <-c.ctx.Done():
            return
        case frame := <-c.frameQueue:
            if err := c.processFrame(frame); err != nil {
                c.logger.Error().Err(err).Msg("frame processing failed")
            }
        }
    }
}
```

### 5. Signal Protocol Integration
```go
// pkg/signal/store.go
package signal

import (
    "context"
    "database/sql"
    "github.com/whiskeysockets/baileys-go/pkg/storage"
)

/*
#cgo LDFLAGS: -lsignal-ffi
#include <signal_ffi.h>
*/
import "C"

type Store struct {
    storage     storage.Provider
    sessions    map[string]*Session
    identities  map[string]*Identity
    preKeys     map[uint32]*PreKey
    signedKeys  map[uint32]*SignedPreKey
    mu          sync.RWMutex
}

type Session struct {
    ID      string `json:"id"`
    JID     string `json:"jid"`
    Data    []byte `json:"data"`
    Created int64  `json:"created"`
}

func NewStore(storage storage.Provider) *Store {
    return &Store{
        storage:    storage,
        sessions:   make(map[string]*Session),
        identities: make(map[string]*Identity),
        preKeys:    make(map[uint32]*PreKey),
        signedKeys: make(map[uint32]*SignedPreKey),
    }
}

// Signal Protocol Store Interface Implementation
func (s *Store) LoadSession(ctx context.Context, address *Address) (*Session, error) {
    s.mu.RLock()
    defer s.mu.RUnlock()
    
    sessionID := address.String()
    if session, exists := s.sessions[sessionID]; exists {
        return session, nil
    }
    
    // Load from persistent storage
    session, err := s.storage.LoadSession(ctx, sessionID)
    if err != nil {
        return nil, err
    }
    
    s.sessions[sessionID] = session
    return session, nil
}

func (s *Store) StoreSession(ctx context.Context, address *Address, session *Session) error {
    s.mu.Lock()
    defer s.mu.Unlock()
    
    sessionID := address.String()
    s.sessions[sessionID] = session
    
    return s.storage.StoreSession(ctx, sessionID, session)
}

// CGO Signal Protocol Operations
func (s *Store) ProcessPreKeyBundle(bundle *PreKeyBundle) error {
    cBundle := C.signal_prekey_bundle_new(
        C.uint32_t(bundle.RegistrationID),
        C.uint32_t(bundle.DeviceID),
        C.uint32_t(bundle.PreKeyID),
        (*C.uint8_t)(&bundle.PreKeyPublic[0]),
        C.size_t(len(bundle.PreKeyPublic)),
        // ... más parameters
    )
    defer C.signal_prekey_bundle_free(cBundle)
    
    result := C.signal_process_prekey_bundle(cBundle)
    if result != 0 {
        return fmt.Errorf("signal processing failed: %d", result)
    }
    
    return nil
}
```

### 6. Storage Layer Design
```go
// pkg/storage/provider.go
package storage

import (
    "context"
    "database/sql"
    _ "github.com/mattn/go-sqlite3"
)

type Provider interface {
    // Authentication
    LoadCreds(ctx context.Context) (*AuthCreds, error)
    StoreCreds(ctx context.Context, creds *AuthCreds) error
    
    // Signal Protocol
    LoadSession(ctx context.Context, id string) (*Session, error)
    StoreSession(ctx context.Context, id string, session *Session) error
    
    // Messages & Chats
    LoadMessages(ctx context.Context, jid string, limit int) ([]*Message, error)
    StoreMessage(ctx context.Context, message *Message) error
    
    // Close & Cleanup
    Close() error
}

type SQLiteProvider struct {
    db     *sql.DB
    config SQLiteConfig
}

type SQLiteConfig struct {
    Path         string `yaml:"path"`
    MaxOpenConns int    `yaml:"max_open_conns"`
    MaxIdleConns int    `yaml:"max_idle_conns"`
    WAL          bool   `yaml:"wal_mode"`
}

func NewSQLiteProvider(config SQLiteConfig) (*SQLiteProvider, error) {
    db, err := sql.Open("sqlite3", config.Path+"?_journal_mode=WAL&_sync=NORMAL")
    if err != nil {
        return nil, err
    }
    
    db.SetMaxOpenConns(config.MaxOpenConns)
    db.SetMaxIdleConns(config.MaxIdleConns)
    
    provider := &SQLiteProvider{
        db:     db,
        config: config,
    }
    
    if err := provider.migrate(); err != nil {
        return nil, err
    }
    
    return provider, nil
}

func (p *SQLiteProvider) LoadCreds(ctx context.Context) (*AuthCreds, error) {
    var creds AuthCreds
    query := `
        SELECT registration_id, noise_key, identity_key, signed_identity_key, 
               signed_pre_key, created_at 
        FROM auth_creds LIMIT 1`
    
    row := p.db.QueryRowContext(ctx, query)
    err := row.Scan(
        &creds.RegistrationID,
        &creds.NoiseKey,
        &creds.IdentityKey,
        &creds.SignedIdentityKey,
        &creds.SignedPreKey,
        &creds.CreatedAt,
    )
    
    if err == sql.ErrNoRows {
        return nil, ErrCredsNotFound
    }
    
    return &creds, err
}

func (p *SQLiteProvider) migrate() error {
    migrations := []string{
        `CREATE TABLE IF NOT EXISTS auth_creds (
            id INTEGER PRIMARY KEY,
            registration_id INTEGER NOT NULL,
            noise_key BLOB NOT NULL,
            identity_key BLOB NOT NULL,
            signed_identity_key BLOB NOT NULL,
            signed_pre_key BLOB NOT NULL,
            created_at INTEGER NOT NULL
        )`,
        `CREATE TABLE IF NOT EXISTS sessions (
            id TEXT PRIMARY KEY,
            jid TEXT NOT NULL,
            data BLOB NOT NULL,
            created_at INTEGER NOT NULL
        )`,
        `CREATE TABLE IF NOT EXISTS messages (
            id TEXT PRIMARY KEY,
            jid TEXT NOT NULL,
            from_me BOOLEAN NOT NULL,
            message_data BLOB NOT NULL,
            timestamp INTEGER NOT NULL,
            INDEX idx_messages_jid (jid),
            INDEX idx_messages_timestamp (timestamp)
        )`,
    }
    
    for _, migration := range migrations {
        if _, err := p.db.Exec(migration); err != nil {
            return err
        }
    }
    
    return nil
}
```

## Configuration Management

### YAML Configuration
```yaml
# config.yaml
log_level: "info"

storage:
  type: "sqlite"
  sqlite:
    path: "./baileys.db"
    max_open_conns: 10
    max_idle_conns: 5
    wal_mode: true

websocket:
  url: "wss://web.whatsapp.com/ws/chat"
  timeout: "30s"
  reconnect_delay: "5s"
  max_reconnects: 10

signal:
  use_cgo: true
  store_type: "sqlite"

events:
  buffer:
    size: 100
    flush_interval: "1s"
    consolidate_all: true

media:
  max_size: "50MB"
  allowed_types: ["image/jpeg", "image/png", "video/mp4"]
  storage_path: "./media"

observability:
  metrics_enabled: true
  tracing_enabled: true
  health_check_port: 8080
```

## Build & Deployment

### Dockerfile
```dockerfile
# Multi-stage build
FROM golang:1.21-alpine AS builder

# Install dependencies for CGO
RUN apk add --no-cache gcc musl-dev pkgconfig

# Install libsignal-c
RUN apk add --no-cache libsignal-c-dev

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=1 go build -o baileys-go ./cmd/baileys

# Runtime stage
FROM alpine:latest

RUN apk add --no-cache ca-certificates libsignal-c

WORKDIR /app
COPY --from=builder /app/baileys-go .
COPY --from=builder /app/config.yaml .

EXPOSE 8080

CMD ["./baileys-go", "--config", "config.yaml"]
```

### Makefile
```makefile
.PHONY: build test lint fmt proto docker

# Variables
BINARY_NAME=baileys-go
VERSION=$(shell git describe --tags --always --dirty)
LDFLAGS=-ldflags "-X main.version=$(VERSION)"

# Build targets
build:
	CGO_ENABLED=1 go build $(LDFLAGS) -o bin/$(BINARY_NAME) ./cmd/baileys

build-static:
	CGO_ENABLED=1 go build -a -installsuffix cgo $(LDFLAGS) -o bin/$(BINARY_NAME) ./cmd/baileys

# Development
test:
	go test -v -race ./...

test-coverage:
	go test -v -race -coverprofile=coverage.out ./...
	go tool cover -html=coverage.out

lint:
	golangci-lint run

fmt:
	gofmt -s -w .
	goimports -w .

# Protobuf generation
proto:
	protoc --go_out=. --go_opt=paths=source_relative proto/*.proto

# Docker
docker:
	docker build -t baileys-go:$(VERSION) .

docker-push:
	docker push baileys-go:$(VERSION)

# Release
release:
	goreleaser release --rm-dist

# Development setup
dev-setup:
	go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
	go install golang.org/x/tools/cmd/goimports@latest
	go install google.golang.org/protobuf/cmd/protoc-gen-go@latest

# Clean
clean:
	rm -rf bin/ dist/ coverage.out
```

## Performance Optimizations

### Memory Pool Management
```go
// pkg/pools/pools.go
package pools

import (
    "sync"
)

var (
    // Event pools
    EventPool = sync.Pool{
        New: func() interface{} {
            return &Event{}
        },
    }
    
    // Buffer pools for WebSocket frames
    BufferPool = sync.Pool{
        New: func() interface{} {
            return make([]byte, 4096)
        },
    }
    
    // Message pools
    MessagePool = sync.Pool{
        New: func() interface{} {
            return &Message{}
        },
    }
)

func GetEvent() *Event {
    return EventPool.Get().(*Event)
}

func PutEvent(e *Event) {
    e.Reset()
    EventPool.Put(e)
}
```

### Metrics & Observability
```go
// pkg/metrics/metrics.go
package metrics

import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
)

var (
    MessagesProcessed = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: "baileys_messages_processed_total",
            Help: "Total number of messages processed",
        },
        []string{"type", "direction"},
    )
    
    WebSocketConnections = promauto.NewGaugeVec(
        prometheus.GaugeOpts{
            Name: "baileys_websocket_connections",
            Help: "Current WebSocket connections",
        },
        []string{"status"},
    )
    
    EventProcessingDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: "baileys_event_processing_duration_seconds",
            Help: "Event processing duration",
        },
        []string{"event_type"},
    )
)
```

Esta arquitectura Go proporciona:

- **Modularidad**: Packages bien definidos y separados
- **Performance**: Goroutines, channels, y memory pools
- **Observability**: Metrics, logging, y health checks
- **Flexibility**: Interfaces para easy testing y extensión
- **Production-ready**: Docker, CI/CD, y deployment automation

La arquitectura mantiene la funcionalidad core de Baileys mientras aprovecha las fortalezas de Go para mejor performance, concurrency, y deployment simplicity.