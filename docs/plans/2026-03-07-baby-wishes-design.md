# Baby Wishes V1 Design

## Objetivo

Definir el producto, dominio, reglas de negocio, módulos y arquitectura técnica de Baby Wishes antes de iniciar el desarrollo. La aplicación permite que una usuaria embarazada gestione una lista privada de regalos para su bebé y que sus amigas o familiares reserven y confirmen compras desde una experiencia autenticada y controlada por invitación.

## Principios de producto

- Una misma cuenta puede actuar como dueña de su lista y como miembro de listas ajenas.
- En v1 cada dueña tiene una sola lista activa.
- La lista es una única fuente de verdad; lo que cambia es la visibilidad contextual según la relación del usuario con esa lista.
- Toda la aplicación requiere autenticación salvo la landing.
- Las compras se realizan fuera de la aplicación y se confirman manualmente dentro del producto.

## Resumen funcional

### Flujo de dueña

La dueña crea su lista activa, añade artículos, configura la expiración de reservas, comparte un enlace de invitación y gestiona quién tiene acceso a la lista. Desde su panel privado puede ver todos los artículos, incluidos los libres, reservados y comprados, junto con la identidad de la persona que actuó sobre cada uno.

### Flujo de miembro invitado

Una usuaria invitada accede a una lista mediante un enlace compartido, pero siempre bajo autenticación. Una vez unida a la lista, puede ver los artículos disponibles y los reservados como estado anónimo, reservar artículos, cancelar sus reservas y marcar compras como confirmadas.

## Decisiones cerradas para V1

- Una dueña tiene una sola lista activa.
- Las invitaciones se gestionan con un enlace compartible único con caducidad de 30 días.
- Si una usuaria entra con un enlace válido y está autenticada, se une directamente a la lista sin paso intermedio de aprobación.
- La dueña puede expulsar miembros desde su lista privada.
- Las reservas caducan automáticamente; por defecto en 7 días, y la dueña puede configurar 7, 14 o 30 días.
- Los artículos reservados siguen visibles en la lista pública con el estado `Reservado`, pero sin mostrar quién lo reservó.
- Los artículos comprados desaparecen de la vista pública.
- La dueña puede ver en todo momento el detalle completo de artículos libres, reservados y comprados, incluido quién reservó o compró.
- Si se expulsa a un miembro, todos los artículos que esa usuaria hubiera reservado o marcado como comprados en esa lista vuelven a quedar `Libres`.
- Si una usuaria se registra o inicia sesión con Google y no tiene nombre visible, debe completarlo antes de continuar.

## Arquitectura del producto

El producto se organiza alrededor de un dominio central compuesto por lista, membresía, artículo, reserva y compra. No existen roles permanentes de cuenta; existen roles contextuales respecto a cada lista.

### Módulos principales

1. Identidad y perfil
   - Login con email y contraseña.
   - Login o registro con Google.
   - Onboarding para capturar nombre visible obligatorio.

2. Mi lista
   - Panel privado de la dueña.
   - CRUD de artículos.
   - Configuración de expiración de reservas.
   - Gestión y regeneración de enlace de invitación.
   - Vista de miembros invitados.
   - Expulsión de miembros.
   - Tabs de filtrado: `Todos`, `Libres`, `Reservados`, `Comprados`.

3. Listas a las que me uní
   - Listado de listas donde la usuaria es miembro.
   - Acceso al detalle de cada lista.

4. Invitaciones y membresías
   - Validación del enlace compartido.
   - Creación de membresía activa.
   - Control de acceso persistente a la lista.

5. Catálogo de artículos y ciclo de regalo
   - Estado operativo real de cada artículo.
   - Acciones de reservar, cancelar y confirmar compra.

6. Reglas automáticas y visibilidad contextual
   - Expiración de reservas.
   - Reversión de artículos al expulsar miembros.
   - Proyección distinta de una misma lista según owner o member.

## Modelo de dominio

### Entidades

#### User

Representa una cuenta autenticada. Contiene email, proveedor de autenticación, nombre visible y metadatos básicos.

#### Wishlist

Lista activa de una dueña. Contiene configuración de reservas, invitación activa y metadatos de presentación.

#### WishlistMember

Relaciona a una usuaria con una lista a la que pertenece. Es la fuente real de autorización continua, no el enlace.

#### WishlistItem

Artículo de una lista. Incluye:

- nombre
- imagen
- precio
- enlace opcional
- sitio opcional
- estado derivado de su actividad

#### ItemReservation

Registro de quién reservó un artículo, cuándo lo hizo y cuándo expira.

#### ItemPurchase

Registro de confirmación manual de compra de un artículo por parte de una usuaria.

#### WishlistInvitation

Invitación activa con token, fecha de expiración y estado.

### Estados de artículo

- `Libre`
- `Reservado`
- `Comprado`

### Transiciones válidas

- `Libre -> Reservado`
- `Reservado -> Libre` por cancelación, expiración o expulsión
- `Reservado -> Comprado` por confirmación manual
- `Comprado -> Libre` por expulsión del miembro que lo marcó como comprado

## Navegación y pantallas

### Landing pública

Pantalla de marketing y acceso. Es la única ruta no autenticada.

### Auth y onboarding

- Login y registro por email y contraseña.
- Acceso con Google.
- Solicitud de nombre visible si falta.
- Reanudación del destino de invitación tras autenticación.

### Home autenticada

Hub simple con dos entradas principales:

- `Mi lista`
- `Listas a las que me uní`

### Mi lista

Vista privada de la dueña con:

- resumen de lista
- enlace de invitación
- configuración de reservas
- CRUD de artículos
- tabs `Todos`, `Libres`, `Reservados`, `Comprados`
- listado de miembros con icono y nombre
- acción de expulsar miembros

### Listas a las que me uní

Listado de listas donde la usuaria tiene membresía activa.

### Detalle de lista invitada

Vista única de la lista, pero recortada por permisos:

- artículos `Libres`
- artículos `Reservados` como estado anónimo
- artículos `Comprados` ocultos
- acciones para reservar, cancelar y confirmar compra propia

### Mis reservas

Vista agregada para seguir reservas activas en las listas donde participa.

### Entrada por invitación

Ruta por token que valida invitación, fuerza autenticación si hace falta y crea membresía cuando el enlace es válido.

## Permisos y visibilidad

### Owner

Es la creadora de la lista.

Puede:

- ver todos los artículos y todos los miembros
- ver quién reservó o compró cada artículo
- crear, editar y eliminar artículos
- configurar la expiración de reservas
- generar o regenerar la invitación activa
- expulsar miembros

### Member

Es una usuaria con membresía activa en una lista ajena.

Puede:

- ver artículos `Libres`
- ver artículos `Reservados` como estado anónimo
- no ver artículos `Comprados`
- reservar artículos `Libres`
- cancelar su propia reserva
- confirmar la compra de su propia reserva

### Outsider

Es una usuaria no autenticada o autenticada sin membresía en la lista.

No puede acceder al contenido de la lista.

## Casos límite

- Acceso con enlace sin login: el destino se retiene hasta completar auth y onboarding.
- Enlace caducado: no crea membresía y muestra estado de invitación no válida.
- Enlace válido con miembro ya existente: no duplica membresía.
- Dueña entrando por su propio enlace: se resuelve como owner sin efectos laterales.
- Reserva sobre artículo ya no libre: el backend rechaza la mutation y la interfaz se refresca con el estado actual.
- Reserva expirada durante la navegación: cualquier acción posterior revalida el estado en servidor.
- Expulsión de miembro: se elimina acceso y se revierten reservas y compras de esa usuaria a `Libre`.

## Arquitectura técnica

### Stack

- TanStack Start
- TypeScript
- Tailwind CSS
- shadcn/ui
- Convex
- Netlify
- GitHub

### Capas

#### Routing y app shell

TanStack Start gestionará rutas públicas, autenticadas y especiales de invitación.

Rutas base propuestas:

- `/`
- `/auth/login`
- `/auth/register`
- `/onboarding`
- `/app`
- `/app/my-list`
- `/app/joined-lists`
- `/app/joined-lists/$listId`
- `/app/my-reservations`
- `/invite/$token`

#### Dominio y backend

Convex será la fuente única de verdad para:

- perfiles
- listas
- invitaciones
- membresías
- artículos
- reservas
- compras
- expiraciones automáticas

Las queries devolverán proyecciones ya autorizadas y las mutations aplicarán las reglas de negocio y permisos.

#### Frontend

La interfaz se organizará por features, no por tipo de archivo. Bloques sugeridos:

- auth
- profile
- lists
- invitations
- items
- reservations
- purchases
- members

#### Plataforma

Netlify servirá la aplicación y GitHub alojará el código, CI y previews. Se separarán entornos de desarrollo y producción con variables de entorno para auth y Convex.

## Roadmap por fases

### Fase 0: bootstrap técnico

- Scaffold base del proyecto
- Convex integrado
- Tailwind y shadcn configurados
- Routing base y layouts
- Entornos y despliegue inicial

### Fase 1: núcleo de la lista propia

- Auth y onboarding
- Creación de la lista activa
- CRUD de artículos
- Configuración de expiración de reservas
- Tabs de owner
- Invitación activa

### Fase 2: invitaciones y membresías

- Ruta por token
- Redirección post login
- Creación de membresías
- Gestión de miembros
- Expulsión

### Fase 3: flujo de regaladora

- Listado de listas unidas
- Detalle de lista invitada
- Reserva
- Cancelación
- Confirmación manual de compra
- Vista agregada de reservas

### Fase 4: endurecimiento

- Expiración automática de reservas
- Reversión por expulsión
- Estados vacíos y errores
- Validación fina de permisos
- Observabilidad mínima

## MVP real

Incluye:

- autenticación y onboarding
- una lista activa por dueña
- CRUD de artículos
- invitación con acceso autenticado
- membresías
- reserva
- cancelación
- confirmación manual de compra
- filtros de owner
- expiración automática de reservas
- reversión por expulsión

Queda fuera del MVP:

- pagos internos
- notificaciones
- multi-lista por dueña
- actividad social avanzada
- importación masiva de artículos
- analítica compleja

## Estrategia de testing

### Prioridad 1: dominio y backend

Validar:

- permisos owner, member y outsider
- invitaciones válidas y caducadas
- creación de membresías
- reserva válida e inválida
- cancelación
- confirmación de compra
- expiración automática
- reversión por expulsión
- proyecciones correctas según contexto

### Prioridad 2: integración de flujos

Cubrir:

- entrada por invitación estando logged out
- login con Google y captura de nombre
- unión a lista
- reserva y actualización de la vista
- visibilidad privada de owner
- expulsión y restitución de artículos

### Prioridad 3: UI focalizada

Cubrir componentes críticos:

- tabs de filtro de owner
- formularios de artículo
- estados vacíos
- guards de navegación

## Preparación para evolución futura

La v1 debe dejar el dominio preparado para futuras extensiones sin rehacer el núcleo:

- pagos integrados
- múltiples listas por usuaria
- notificaciones automáticas
- analítica de conversión
- soporte para otros tipos de eventos o listas

## Criterios de éxito de V1

- Una usuaria puede crear y mantener su lista sin fricción.
- Una invitada puede entrar desde un enlace válido, autenticarse y unirse sin pasos manuales adicionales.
- La dueña mantiene control total sobre visibilidad y miembros.
- La vista pública nunca expone quién reservó un artículo ni artículos ya comprados.
- El estado de cada artículo se mantiene consistente ante reservas, compras, expiraciones y expulsiones.