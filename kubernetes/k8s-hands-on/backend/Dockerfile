
# Etapa 1: Build
FROM golang:1.25.5 AS builder

WORKDIR /app

# Copiar dependencias
COPY go.mod ./

# Descargar dependencias
RUN go mod download

# Copiar código fuente
COPY src/ ./src/

# Compilar binario estático para Linux
RUN CGO_ENABLED=0 GOOS=linux go build -o main ./src


# Etapa 2: Runtime
FROM alpine:latest

WORKDIR /app

# Copiar ejecutable
COPY --from=builder /app/main ./main

# Puerto
EXPOSE 8080

# Ejecutar aplicación
CMD ["./main"]
