# Request-IO Business SDK

[![Swift](https://img.shields.io/badge/Swift-6.2-orange.svg)](https://swift.org)
[![Platforms](https://img.shields.io/badge/Platforms-iOS%20|%20macOS%20|%20watchOS%20|%20tvOS-blue.svg)](https://developer.apple.com)
[![SPM](https://img.shields.io/badge/SPM-Compatible-brightgreen.svg)](https://swift.org/package-manager)

Swift SDK oficial para la **API de negocio completo** de Request-IO (Core + AlmaVip + Admin). Generado automáticamente desde la especificación OpenAPI 3.1.0.

> **Nota**: Este SDK es para funcionalidad de negocio completo. Para administración exclusiva, ver [`request-io-swift-admin-client`](https://github.com/CloudOnMobile/request-io-swift-admin-client).

## Características

- ✅ **Type-Safe**: Código Swift nativo con verificación de tipos en tiempo de compilación
- ✅ **Async/Await**: APIs modernas usando Swift Concurrency
- ✅ **Auto-Generated**: Sincronizado automáticamente con el backend mediante OpenAPI
- ✅ **Multi-Platform**: Soporte para iOS 16+, macOS 13+, watchOS 9+, tvOS 16+
- ✅ **Modular**: Acceso a módulos Core, AlmaVip, y Admin Dashboard

## Requisitos

- **iOS** 16.0+ / **macOS** 13.0+ / **watchOS** 9.0+ / **tvOS** 16.0+
- **Swift** 6.2+
- **Xcode** 15.0+

## Instalación

### Swift Package Manager

Agrega el paquete a tu `Package.swift`:

```swift
dependencies: [
    .package(url: "https://github.com/CloudOnMobile/request-io-swift-business-sdk.git", from: "1.0.0")
]
```

O en Xcode:

1. File → Add Package Dependencies...
2. Pega la URL: `https://github.com/CloudOnMobile/request-io-swift-business-sdk`
3. Selecciona "Up to Next Major Version" y especifica `1.0.0`

## Uso Rápido

### Importar el SDK

```swift
import RequestIOBusiness
import OpenAPIRuntime
import OpenAPIURLSession
```

### Crear Cliente

```swift
let client = Client(
    serverURL: try Servers.server1(), // https://api.request-io.com
    transport: URLSessionTransport()
)
```

### Autenticación

```swift
// Con JWT Token
let transport = URLSessionTransport()
let authenticatedClient = Client(
    serverURL: try Servers.server1(),
    transport: transport,
    middlewares: [AuthMiddleware(token: "your-jwt-token")]
)
```

### Ejemplos de Uso

#### Obtener Dashboard de Administrador

```swift
do {
    let response = try await client.get_sol_api_sol_v1_sol_admin_sol_dashboard(
        query: .init(
            period: .last_30_days,
            includeAlmaVipMetrics: true
        )
    )

    switch response {
    case .ok(let okResponse):
        let dashboard = try okResponse.body.json
        print("Total Users: \(dashboard.quickStats.totalUsers)")
        print("Active Users: \(dashboard.quickStats.activeUsers)")
    case .unauthorized:
        print("Error: No autorizado")
    default:
        print("Error inesperado")
    }
} catch {
    print("Error: \(error)")
}
```

#### Buscar Aeropuertos (AlmaVip)

```swift
do {
    let response = try await client.get_sol_api_sol_v1_sol_almavip_sol_airports(
        query: .init(
            searchTerm: "Madrid",
            page: 1,
            size: 20,
            sortBy: .name,
            sortOrder: .asc
        )
    )

    switch response {
    case .ok(let okResponse):
        let page = try okResponse.body.json
        for airport in page.items {
            print("\(airport.name) (\(airport.iataCode))")
        }
    default:
        print("Error buscando aeropuertos")
    }
} catch {
    print("Error: \(error)")
}
```

#### Obtener Lista de Caterers

```swift
do {
    let response = try await client.get_sol_api_sol_v1_sol_almavip_sol_caterers(
        query: .init(page: 1, size: 20)
    )

    switch response {
    case .ok(let okResponse):
        let page = try okResponse.body.json
        for caterer in page.items {
            print("\(caterer.name) - \(caterer.email)")
        }
    default:
        print("Error obteniendo caterers")
    }
} catch {
    print("Error: \(error)")
}
```

## Middleware de Autenticación

Crea un middleware personalizado para agregar el token JWT a todas las requests:

```swift
import OpenAPIRuntime
import Foundation

struct AuthMiddleware: ClientMiddleware {
    let token: String

    func intercept(
        _ request: HTTPRequest,
        baseURL: URL,
        operationID: String,
        next: (HTTPRequest, URL) async throws -> HTTPResponse
    ) async throws -> HTTPResponse {
        var request = request
        request.headerFields[.authorization] = "Bearer \(token)"
        return try await next(request, baseURL)
    }
}
```

Uso:

```swift
let client = Client(
    serverURL: try Servers.server1(),
    transport: URLSessionTransport(),
    middlewares: [AuthMiddleware(token: yourJWTToken)]
)
```

## Estructura del Proyecto

```
request-io-swift-business-sdk/
├── Package.swift                      # Configuración del paquete
├── Sources/
│   └── RequestIOBusiness/
│       ├── openapi.json               # Especificación OpenAPI (auto-actualizada)
│       ├── openapi-generator-config.yaml  # Configuración del generador
│       └── RequestIOBusiness.swift    # Punto de entrada del SDK
├── Tests/
│   └── RequestIOBusinessTests/
└── Examples/                          # (Próximamente) Apps de ejemplo
```

## Actualización del SDK

El código Swift se genera automáticamente desde el backend cada vez que ejecutas `swift build`. Para obtener las últimas APIs:

```bash
# 1. Asegúrate de que el backend está corriendo
cd ../request-io_back
swift run App serve --env develop.env

# 2. En otra terminal, actualiza el spec
cd ../request-io-swift-business-sdk
curl http://localhost:8080/openapi.json -o Sources/RequestIOBusiness/openapi.json

# 3. Regenera el cliente
swift build
```

## Endpoints Disponibles

El SDK incluye soporte completo para:

### Core Module
- 👤 **User Management**: Registro, login, perfil, roles
- 📋 **Order Requests**: CRUD de solicitudes, transiciones de estado
- 💬 **Comments**: Sistema de comentarios con archivos adjuntos
- 📊 **Audit Logs**: Historial de cambios

### AlmaVip Module
- ✈️ **Airports**: Búsqueda avanzada, gestión de aeropuertos
- 🍽️ **Caterers**: Gestión de caterers, reglas de negocio
- 👥 **Customers**: Gestión de clientes
- 🍕 **Menu**: Categorías, items, gestión de menús
- 📦 **Orders**: Sistema completo de pedidos
- 🖼️ **Image Upload**: Subida de imágenes con S3

### Admin Module
- 📈 **Dashboard**: Métricas y estadísticas consolidadas
- 🔔 **Notifications**: Sistema de notificaciones administrativas
- 🔍 **Audit**: Logs de auditoría con análisis avanzado
- ⚙️ **System Config**: Configuración del sistema

Ver documentación completa de endpoints en [Backend Docs](https://github.com/CloudOnMobile/request-io_back/tree/main/docs).

## Troubleshooting

### Error: "Reference not found in components"

Si ves este error al hacer build, significa que el spec OpenAPI está desactualizado. Solución:

```bash
# Actualiza el spec desde el backend
curl http://localhost:8080/openapi.json -o Sources/RequestIOBusiness/openapi.json
swift build
```

### Warnings sobre "nullable property"

Los warnings sobre `nullable` property son esperados y no afectan el funcionamiento. OpenAPI Generator los maneja automáticamente traduciendo a `type: ["null", ...]`.

## Contribuir

Este SDK se genera automáticamente. Para contribuir:

1. **Cambios en la API**: Modifica el backend en `request-io_back`
2. **Wrappers personalizados**: Agrega extensiones en `Sources/RequestIOBusiness/Extensions/`
3. **Ejemplos**: Contribuye apps de ejemplo en `Examples/`

## Roadmap

- [ ] Wrappers de alto nivel (AdminDashboardClient, AlmaVipClient)
- [ ] Apps de ejemplo para iOS y macOS
- [ ] Soporte de Keychain para almacenamiento de tokens
- [ ] Retry policies y manejo de errores avanzado
- [ ] Cache de requests
- [ ] Documentación DocC completa

## License

Este proyecto está licenciado bajo MIT License.

## Links

- **Backend Repository**: [request-io_back](https://github.com/CloudOnMobile/request-io_back)
- **Admin SDK**: [request-io-swift-admin-client](https://github.com/CloudOnMobile/request-io-swift-admin-client)
- **API Documentation**: [Backend Docs](https://github.com/CloudOnMobile/request-io_back/tree/main/docs)
- **Issues**: [GitHub Issues](https://github.com/CloudOnMobile/request-io-swift-business-sdk/issues)

---

**Versión**: 1.0.0-alpha
**Generado con**: [swift-openapi-generator](https://github.com/apple/swift-openapi-generator)
**OpenAPI**: 3.1.0
