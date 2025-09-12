# 7. Mapeo de Dependencias: Ecosistema Node.js vs Go

## Análisis de package.json

### Dependencias Core (8 paquetes)
```json
{
  "@cacheable/node-cache": "^1.4.0",        // Caching en memoria
  "@hapi/boom": "^9.1.3",                   // HTTP error handling
  "async-mutex": "^0.5.0",                  // Locks para file operations
  "axios": "^1.6.0",                        // HTTP client
  "libsignal": "git+https://...",           // Signal Protocol (C++)
  "music-metadata": "^11.7.0",              // Audio metadata extraction
  "pino": "^9.6",                          // Logging estructurado
  "protobufjs": "^7.2.4",                  // Protocol Buffers
  "ws": "^8.13.0"                          // WebSocket client
}
```

### Dependencias de Desarrollo (24 paquetes)
```json
{
  // TypeScript ecosystem
  "typescript": "^5.8.2",
  "@types/node": "^16.0.0",
  "@types/ws": "^8.0.0",
  
  // Testing
  "jest": "^30.0.5", 
  "@types/jest": "^30.0.0",
  
  // Build tools
  "tsc-esm-fix": "^3.1.2",
  "esbuild-register": "^3.6.0",
  
  // Code quality
  "eslint": "^9.31.0",
  "prettier": "^3.5.3"
}
```

### Peer Dependencies (Opcionales)
```json
{
  "audio-decode": "^2.1.3",      // Audio processing
  "jimp": "^1.6.0",              // Image processing  
  "link-preview-js": "^3.0.0",   // Link metadata
  "sharp": "*"                    // Image optimization
}
```

## Mapeo a Go Ecosystem

### 1. Core Dependencies

#### HTTP Client & Error Handling
```go
// axios + @hapi/boom → Go equivalents
import (
    "net/http"
    "github.com/go-resty/resty/v2"      // HTTP client avanzado
    "github.com/pkg/errors"             // Error wrapping
)

// Reemplazo para axios
client := resty.New()
resp, err := client.R().
    SetHeader("Content-Type", "application/json").
    SetBody(payload).
    Post("https://api.example.com")

// Error handling similar a @hapi/boom
type HTTPError struct {
    StatusCode int    `json:"statusCode"`
    Error      string `json:"error"`
    Message    string `json:"message"`
}
```

#### Caching System
```go
// @cacheable/node-cache → Go alternatives
import (
    "github.com/patrickmn/go-cache"         // In-memory cache
    "github.com/go-redis/redis/v8"          // Redis client
    "github.com/bluele/gcache"              // Advanced caching
)

// Reemplazo directo
cache := cache.New(5*time.Minute, 10*time.Minute)
cache.Set("key", "value", cache.DefaultExpiration)
value, found := cache.Get("key")
```

#### Concurrency & Locks
```go
// async-mutex → sync.Mutex + context
import (
    "sync"
    "context"
    "time"
)

type FileManager struct {
    locks map[string]*sync.Mutex
    mu    sync.RWMutex
}

func (fm *FileManager) LockFile(path string) *sync.Mutex {
    fm.mu.Lock()
    defer fm.mu.Unlock()
    
    if _, exists := fm.locks[path]; !exists {
        fm.locks[path] = &sync.Mutex{}
    }
    return fm.locks[path]
}
```

#### Logging System
```go
// pino → zerolog/logrus
import (
    "github.com/rs/zerolog/log"
    "github.com/sirupsen/logrus"
)

// Structured logging similar a pino
log.Info().
    Str("chatId", chatID).
    Int("messageCount", count).
    Msg("Processing messages")
```

#### Protocol Buffers
```go
// protobufjs → protoc-gen-go
// Proto file compilation:
// protoc --go_out=. --go_opt=paths=source_relative WAProto.proto

import "google.golang.org/protobuf/proto"

// Usage
data, err := proto.Marshal(message)
err = proto.Unmarshal(data, &message)
```

#### WebSocket Client
```go
// ws → gorilla/websocket
import (
    "github.com/gorilla/websocket"
    "github.com/nhooyr/websocket"          // Alternative moderna
)

// Gorilla WebSocket (más popular)
conn, _, err := websocket.DefaultDialer.Dial(url, headers)
defer conn.Close()

// Message handling
for {
    messageType, message, err := conn.ReadMessage()
    if err != nil {
        break
    }
    processMessage(messageType, message)
}
```

### 2. Signal Protocol Dependency

#### libsignal Migration Strategy
```go
// Opción 1: CGO Bindings
/*
#cgo LDFLAGS: -lsignal-ffi
#include <signal_ffi.h>
*/
import "C"

type SignalStore struct {
    store *C.SignalProtocolStore
}

// Opción 2: Pure Go Implementation  
import "github.com/RadicalApp/complete-signal-protocol-go"

// Opción 3: gRPC Service
import (
    "google.golang.org/grpc"
    "your-org/signal-service/pb"
)
```

### 3. Media Processing

#### Image Processing
```go
// jimp/sharp → imaging/gg
import (
    "github.com/disintegration/imaging"
    "github.com/fogleman/gg"
    "image"
    "image/jpeg"
    "image/png"
)

// Resize image (sharp equivalent)
src, err := imaging.Open("input.jpg")
resized := imaging.Resize(src, 800, 0, imaging.Lanczos)
err = imaging.Save(resized, "output.jpg")

// Image manipulation (jimp equivalent)
dc := gg.NewContext(800, 600)
dc.DrawImage(src, 0, 0)
dc.SavePNG("output.png")
```

#### Audio Processing
```go
// music-metadata → go-audio
import (
    "github.com/go-audio/audio"
    "github.com/go-audio/wav"
    "github.com/dhowden/tag"              // Metadata extraction
)

// Audio metadata (music-metadata equivalent)
file, err := os.Open("audio.mp3")
metadata, err := tag.ReadFrom(file)
fmt.Printf("Title: %s, Artist: %s", metadata.Title(), metadata.Artist())
```

#### Link Preview
```go
// link-preview-js → go-link-preview
import (
    "github.com/PuerkitoBio/goquery"
    "net/http"
)

// Link metadata extraction
resp, err := http.Get(url)
doc, err := goquery.NewDocumentFromReader(resp.Body)

title := doc.Find("title").Text()
description := doc.Find("meta[name='description']").AttrOr("content", "")
image := doc.Find("meta[property='og:image']").AttrOr("content", "")
```

### 4. Build & Development Tools

#### Build Pipeline
```bash
# package.json scripts → Makefile/Go tools
.PHONY: build test lint format

build:
	go build -o bin/baileys-go ./cmd/baileys

test:
	go test -v ./...

lint:
	golangci-lint run

format:
	gofmt -s -w .
	goimports -w .
```

#### Testing Framework
```go
// jest → testify/ginkgo
import (
    "testing"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
)

func TestEventBuffer(t *testing.T) {
    buffer := NewEventBuffer()
    
    buffer.Buffer()
    buffer.Add("test.event", testData)
    
    events := buffer.Flush()
    assert.Len(t, events, 1)
    assert.Equal(t, testData, events["test.event"])
}
```

#### Code Generation
```go
// TypeScript types → Go structs
//go:generate go run tools/generate-types.go

//go:generate protoc --go_out=. WAProto.proto
//go:generate mockgen -source=interfaces.go -destination=mocks/mock.go
```

## Dependency Migration Matrix

| Node.js Package | Go Equivalent | Migration Effort | Notes |
|-----------------|---------------|------------------|-------|
| ws | gorilla/websocket | ⭐⭐ Easy | Direct replacement |
| axios | resty/net/http | ⭐⭐ Easy | Similar API |
| pino | zerolog/logrus | ⭐⭐ Easy | Structured logging |
| async-mutex | sync.Mutex | ⭐⭐ Easy | Built-in Go |
| protobufjs | protoc-gen-go | ⭐⭐⭐ Medium | Different workflow |
| @hapi/boom | custom/errors | ⭐⭐⭐ Medium | Custom implementation |
| libsignal | CGO/Pure Go | ⭐⭐⭐⭐⭐ Hard | Critical dependency |
| @cacheable/node-cache | go-cache | ⭐⭐ Easy | Multiple options |
| music-metadata | dhowden/tag | ⭐⭐⭐ Medium | Limited features |

## Package Management Strategy

### go.mod Structure
```go
module github.com/whiskeysockets/baileys-go

go 1.21

require (
    github.com/gorilla/websocket v1.5.0
    github.com/go-resty/resty/v2 v2.7.0
    github.com/rs/zerolog v1.29.1
    google.golang.org/protobuf v1.31.0
    github.com/patrickmn/go-cache v2.1.0+incompatible
    github.com/stretchr/testify v1.8.4
)

require (
    // CGO dependencies for Signal Protocol
    // Custom signal bindings
)
```

### Vendoring Strategy
```bash
# Vendoring critical dependencies
go mod vendor

# Custom forks for modified packages
replace github.com/gorilla/websocket => ./vendor/websocket-custom

# Signal protocol bindings
replace github.com/whiskeysockets/libsignal-go => ./signal
```

## Performance & Size Comparison

### Binary Size
```bash
# Node.js bundle
node_modules/: ~200MB
Built bundle: ~50MB

# Go binary
Static binary: ~15-25MB
With CGO: ~20-30MB
```

### Runtime Dependencies
```bash
# Node.js runtime requirements
- Node.js v20+
- npm/yarn
- Native addons compilation

# Go binary requirements  
- None (static binary)
- libc (if CGO enabled)
```

## Risk Assessment

### 🟢 **LOW RISK** (Fácil migración)
- HTTP client (axios → resty)
- Logging (pino → zerolog)  
- Caching (node-cache → go-cache)
- WebSocket (ws → gorilla/websocket)

### 🟡 **MEDIUM RISK** (Esfuerzo moderado)
- Protocol Buffers (workflow diferente)
- Error handling (custom implementation)
- Media processing (features limitadas)

### 🔴 **HIGH RISK** (Crítico para el proyecto)
- **libsignal**: Dependency más crítica
- **music-metadata**: Features específicas pueden faltar
- **Build pipeline**: Workflow completamente diferente

## Recomendaciones

### Estrategia de Migración por Fases
1. **Fase 1**: Core dependencies (HTTP, logging, caching)
2. **Fase 2**: WebSocket y Protocol Buffers  
3. **Fase 3**: Signal Protocol (CGO implementation)
4. **Fase 4**: Media processing y optimizations

### Dependencies Específicas
```bash
# Core packages recomendados
go get github.com/gorilla/websocket
go get github.com/rs/zerolog
go get github.com/go-resty/resty/v2
go get google.golang.org/protobuf

# Testing & quality
go get github.com/stretchr/testify
go get github.com/golangci/golangci-lint

# Signal Protocol (evaluar opciones)
# Opción A: CGO bindings
# Opción B: Pure Go implementation  
# Opción C: gRPC microservice
```

**CONCLUSIÓN**: La mayoría de dependencias tienen equivalentes directos en Go. El mayor riesgo está en **libsignal** que requiere estrategia específica (CGO vs Pure Go vs microservice). El ecosistema Go es maduro y tiene alternativas robustas para todas las funcionalidades core.