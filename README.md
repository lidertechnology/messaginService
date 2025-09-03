## Informe Detallado del Servicio de Mensajería Push

# 1. Propósito y Filosofía del Servicio
El MessagingService es la pieza fundamental para integrar notificaciones push en todas tus aplicaciones. Su propósito es actuar como una capa de abstracción sobre el SDK de Firebase Cloud Messaging (FCM), centralizando toda la lógica en tu biblioteca lidertechLibCentralModule.

Esto garantiza que todos tus proyectos Lidertech, sin importar su tipo (web, móvil con Capacitor), sigan la misma convención para:

* Solicitar permisos de notificación al usuario.

* Obtener el token único del dispositivo.

* Escuchar los mensajes entrantes en tiempo real.

* Gestionar el estado de cada operación de manera uniforme a través de StatesEnum.

# 2. Funcionalidades y Métodos
El servicio expone tres elementos principales:

states (Signal): Una señal que indica el estado actual del servicio. Los componentes pueden reaccionar a estos estados para mostrar la UI adecuada (ej. un spinner durante la carga, un mensaje de éxito, o una alerta de error).

* StatesEnum.INACTIVE: Estado inicial.

* StatesEnum.LOADING: Indica que se está solicitando el token.

* StatesEnum.SUCCESS: Se establece cuando el token ha sido obtenido con éxito.

* StatesEnum.UNAUTHORIZED: Si el usuario deniega los permisos.

StatesEnum.ERROR: Si ocurre un error al obtener el token.

messagePayload (Signal): Una señal que se actualiza cada vez que un mensaje push llega a la aplicación en primer plano. Los componentes se pueden suscribir a esta señal para reaccionar a los datos del mensaje.

obtenerToken() (Método): Es el método principal. Se encarga de solicitar los permisos de notificación al usuario y, si se conceden, obtiene y retorna el token del dispositivo. Este token es lo que tu backend necesita para enviar notificaciones push personalizadas a este dispositivo en específico.

escucharMensajesEnPrimerPlano() (Método): Activa la escucha de mensajes push cuando la aplicación está abierta y en uso.

# 3. Configuración en la Consola de Firebase
Para que el servicio funcione, es indispensable que configures correctamente tu proyecto en la consola de Firebase.

* Habilitar Firebase Cloud Messaging:

* Ve a tu proyecto en la consola de Firebase.

* Navega a Engage > Messaging.

* Sigue los pasos para habilitar el servicio de mensajería.

* Generar la Clave VAPID:

* En la consola de Firebase, haz clic en el icono de Engranaje > Configuración del proyecto.

* Selecciona la pestaña Cloud Messaging.

* En la sección Configuración web, busca la Clave de pares de claves de VAPID. Si no está generada, haz clic en Generar par de claves.

* Copia la clave pública de VAPID y guárdala en tu archivo de entorno (environment.ts) para que el servicio pueda acceder a ella.



# 4. Ejemplo de Uso en un Componente
Una vez configurado, puedes usar el servicio en cualquier componente de tu aplicación.
Componente de Ejemplo

TypeScript:

    import { Component, inject, OnInit } from '@angular/core';
    import { MessagingService } from '../core/services/messaging.service';
    import { StatesEnum } from '../shared/enums/states.enum';
    
    @Component({
      selector: 'app-notificacion',
      standalone: true,
      template: `
        @if (messagingService.states() === tant.LOADING) {
          <p>Cargando...</p>
        } @else if (messagingService.states() === tant.SUCCESS) {
          <p>Notificaciones habilitadas. Tu token es: {{ token() }}</p>
        } @else if (messagingService.states() === tant.UNAUTHORIZED) {
          <p>Permisos de notificación denegados. Por favor, habilítalos en tu navegador.</p>
        } @else {
          <button (click)="habilitarNotificaciones()">Habilitar Notificaciones</button>
        }
    
        @if (messagingService.messagePayload()) {
          <div class="notificacion-recibida">
            <h3>Mensaje Recibido:</h3>
            <pre>{{ messagingService.messagePayload() | json }}</pre>
          </div>
        }
      `
    })
    export class NotificacionComponent implements OnInit {
      private messagingService = inject(MessagingService);
      public readonly tant = StatesEnum;
      public readonly token = signal<string | null>(null);
    
      ngOnInit() {
        this.messagingService.escucharMensajesEnPrimerPlano();
      }
    
      async habilitarNotificaciones() {
        this.token.set(await this.messagingService.obtenerToken());
      }
    }


Con esta guía, tienes todo lo necesario para integrar el servicio de mensajería en cualquier proyecto de Lidertech, asegurando que cada implementación sea consistente y óptima.
