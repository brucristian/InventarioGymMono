# InventarioGymMono
graph TD
    %% Definición de la Leyenda y Estilo
    classDef domain fill:#f9f,stroke:#333,stroke-width:2px,color:black;
    classDef application fill:#ccf,stroke:#333,stroke-width:2px,color:black;
    classDef infrastructure fill:#ff9,stroke:#333,stroke-width:2px,color:black;
    classDef port fill:#fff,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5,color:black;

    %% --- HEXÁGONO INTERNO (Capa de Dominio) ---
    subgraph DomainLayer ["📦 domain (El Corazón)"]
        direction TB
        
        subgraph DomainModel ["📁 model"]
            SocioEntity[("Socio (Entity)
            - id: UUID
            - fechaVencimiento: LocalDate
            + estaVencido()
            + renovarMembresia(meses)")]
            
            ProductoEntity[("Producto (Entity)
            - stockActual: int
            - stockMinimo: int
            + requerirReposicion()")]
            
            TransaccionEntity[("Transaccion (Entity)
            - monto: BigDecimal
            + validarMonto()")]
        end

        subgraph DomainRepositoryPorts ["📁 repository (Puertos de Salida)"]
            SocioRepoPort["<<interface>>
            SocioRepository
            + buscarPorId(UUID)
            + guardar(Socio)"]:::port
            
            ProductoRepoPort["<<interface>>
            ProductoRepository
            + buscarPorId(Long)
            + guardar(Producto)"]:::port
        end
        
        %% Relaciones internas del dominio
        SocioEntity -.-> TransaccionEntity
    end

    %% --- CAPA DE APLICACIÓN (Casos de Uso) ---
    subgraph ApplicationLayer ["📦 application (El Orquestador)"]
        direction TB
        
        subgraph UseCases ["📁 usecases (Servicios)"]
            RenovacionUC["RenovacionMembresiaUseCase
            (Servicio de Aplicación)
            + ejecutar(socioId, meses)"]:::application
            
            VentaUC["RegistrarVentaUseCase
            (Servicio de Aplicación)
            + ejecutar(productoId, cant)"]:::application
        end
        
        subgraph DTOs ["📁 dto"]
            SocioDTO["SocioDTO
            (Datos simples para la UI)"]:::application
        end
    end

    %% --- CAPA DE INFRAESTRUCTURA (Adaptadores Externos) ---
    subgraph InfrastructureLayer ["📦 infrastructure (Los Enchufes)"]
        direction TB
        
        subgraph PersistenceAdapters ["📁 persistence (Adaptadores de Salida)"]
            JpaSocioAdapter["JpaSocioRepository
            (Implementación SQL/JPA)"]:::infrastructure
            
            JpaProductoAdapter["JpaProductoRepository
            (Implementación SQL/JPA)"]:::infrastructure
        end
        
        subgraph UIAdapters ["📁 ui (Adaptadores de Entrada)"]
            DesktopUI["JavaFX / Swing UI
            (Interfaz de Escritorio)"]:::infrastructure
            
            RestApi["Controlador REST
            (Para una App Móvil futura)"]:::infrastructure
        end
        
        subgraph Config ["📁 config"]
            AppConfig["AppConfig
            (Inyección de Dependencias)"]:::infrastructure
        end
    end

    %% --- FLUJO DE DEPENDENCIAS Y LLAMADAS (Las Flechas) ---
    
    %% 1. Flujo de Entrada (UI -> Application)
    DesktopUI ==>|llama| RenovacionUC
    RestApi ==>|llama| VentaUC
    
    %% 2. Flujo de Aplicación (Application -> Domain)
    RenovacionUC -->|orquesta| SocioEntity
    VentaUC -->|orquesta| ProductoEntity
    
    %% 3. Flujo de Salida (Application -> Ports -> Infrastructure)
    %% El Servicio usa la interfaz del puerto
    RenovacionUC -.->|usa| SocioRepoPort
    VentaUC -.->|usa| ProductoRepoPort
    
    %% El adaptador implementa el puerto
    JpaSocioAdapter --|>|implementa| SocioRepoPort
    JpaProductoAdapter --|>|implementa| ProductoRepoPort

    %% El adaptador usa la base de datos real
    JpaSocioAdapter -.->|persiste en| SQLDB[(Base de Datos SQL)]
    JpaProductoAdapter -.->|persiste en| SQLDB
    
    %% Nota sobre DTOs
    DesktopUI -.->|usa| SocioDTO
    RenovacionUC -.->|retorna| SocioDTO
    
    %% Estilos de los nodos principales
    class SocioEntity,ProductoEntity,TransaccionEntity domain;
    class SQLDB fill:#ddd,stroke:#333;
