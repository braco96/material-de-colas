# Material de Colas — README base

Este directorio contiene **material de apoyo** para una app de colas / tickets (frontend). Incluye:
- Un **servicio Angular** (`websocket.service.ts`) para autenticación ligera y comunicación en **Socket.IO**.
- Una carpeta **estática** `public - estilo/` con páginas HTML/CSS de ejemplo para *Pantalla pública*, *Nuevo ticket* y *Escritorio*.
- Un audio `new-ticket.mp3` para reproducir cuando se genera/llama un ticket.

> ⚠️ **Importante:** Esto **no es** una app completa lista para producción. Son piezas reutilizables que se integran con un **backend Socket.IO** (no incluido aquí) y, opcionalmente, con un proyecto Angular donde se importe `ngx-socket-io`.

---

## Contenido

```
Material-de-Colas/
├─ services/
│  └─ websocket.service.ts         # Servicio Angular para Sockets (login, logout, eventos)
├─ classes/
│  └─ usuario.ts                   # Clase simple de Usuario (frontend)
└─ public - estilo/
   ├─ css/styles.css               # Estilos base para las vistas estáticas
   ├─ audio/new-ticket.mp3         # Sonido al crear/llamar ticket
   ├─ index.html                   # Menú: Pantalla pública / Crear tickets / Entrar a Escritorio
   ├─ nuevo-ticket.html            # Generación de nuevo ticket
   ├─ publico.html                 # Pantalla con últimos tickets llamados
   └─ escritorio.html              # Vista para atender el siguiente ticket
```

---

## Backend esperado

El material asume un **servidor de colas con Socket.IO** que expone eventos como (nombres frecuentes en cursos/demos):
- `configurar-usuario` → establece (o limpia) el nombre del usuario/escritorio en la sesión de socket.
- Otros eventos típicos del flujo de colas: *nuevo-ticket*, *atender-ticket*, *estado-actual*, *ultimos-4*, etc.

> Ajusta los nombres de evento a los que realmente emite tu backend. El servicio Angular ya incluye métodos básicos de **emit** y **listen**.

---

## Uso rápido de las páginas estáticas

Si quieres **visualizar rápidamente** las vistas de `public - estilo/`, puedes servirlas con un servidor estático:

```bash
# Opción 1: http-server
npm i -g http-server
http-server "public - estilo" -p 8080

# Opción 2: live-server (recarga en caliente)
npm i -g live-server
live-server "public - estilo" --port=8080
```

Luego abre en el navegador:
- `http://localhost:8080/index.html`
- `http://localhost:8080/publico.html`
- `http://localhost:8080/nuevo-ticket.html`
- `http://localhost:8080/escritorio.html`

> Estas páginas no incluyen la inicialización de Socket.IO por sí mismas; están pensadas como **maquetas UI**. Para conectarlas al backend, añade tu cliente de Socket.IO en cada HTML o reutiliza un *bundle* de tu proyecto SPA.

---

## Integración en un proyecto Angular

1) **Instala dependencias** (en tu proyecto Angular):
```bash
npm i ngx-socket-io
```

2) **Configura el módulo de sockets** (ejemplo en `app.module.ts`):
```ts
import { SocketIoModule, SocketIoConfig } from 'ngx-socket-io';

const config: SocketIoConfig = {
  url: 'http://localhost:5000', // Cambia por la URL de tu backend
  options: {}
};

@NgModule({
  imports: [
    SocketIoModule.forRoot(config),
    // ...
  ]
})
export class AppModule {}
```

3) **Copia el servicio** `services/websocket.service.ts` y la clase `classes/usuario.ts` a tu proyecto (ajusta rutas de importación si fuera necesario).

4) **Usa el servicio** en tus componentes:
```ts
constructor(private wsService: WebsocketService) {}

async ngOnInit() {
  // Establece el usuario/escritorio (persistido en localStorage por el servicio)
  await this.wsService.loginWS('Escritorio 1');

  // Escucha un evento cualquiera
  this.wsService.listen('ultimos-4').subscribe((payload) => {
    console.log('Últimos 4 tickets:', payload);
  });

  // Emite eventos
  this.wsService.emit('nuevo-ticket', null, (resp) => console.log('Ticket creado', resp));
}
```

5) **Cerrar sesión / limpiar usuario**:
```ts
this.wsService.logoutWS();
```

---

## Notas del `websocket.service.ts`

- **Persistencia:** guarda el `Usuario` en `localStorage` y lo restaura en reconexión.
- **Eventos base:**
  - `emit(evento, payload?, callback?)`
  - `listen(evento)` → `Observable`
  - `loginWS(nombre)` / `logoutWS()` → también emiten `configurar-usuario` al servidor.
- **Enrutamiento:** tras `logoutWS()` navega a la ruta raíz (`''`). Ajusta según tus rutas.

> Si tu backend exige *rooms* por escritorio o autenticación JWT, amplia el `payload` de `configurar-usuario` y agrega la lógica correspondiente.

---

## Conectar las vistas estáticas al backend (opcional)

Si no usas Angular y prefieres **conectar las páginas HTML** directamente al servidor:
1. Incluye cliente de Socket.IO en cada HTML, por ejemplo:
   ```html
   <script src="https://cdn.socket.io/4.7.5/socket.io.min.js"></script>
   <script>
     const socket = io('http://localhost:5000'); // backend
     socket.on('connect', () => console.log('Conectado'));
     // Escuchar/emitir según tu protocolo
   </script>
   ```
2. Añade la lógica de **generar/atender tickets** en cada página con `socket.emit(...)` y `socket.on(...)`.

---

## Personalización de estilo/UX

- Edita `public - estilo/css/styles.css` para adaptar colores, tipografías y tamaños.
- El audio `audio/new-ticket.mp3` puede reproducirse cuando el backend emita un evento tipo *ticket-llamado*.

---

## Requisitos

- **Node.js 14+** (recomendado)
- **Backend Socket.IO** accesible (puede ser `http://localhost:5000` u otra URL)
- Para Angular: **Angular 8+** y `ngx-socket-io`

---

## Licencia

Uso educativo / de ejemplo. Ajusta a la licencia de tu proyecto.
