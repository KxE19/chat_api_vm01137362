<template>
  <div
    class="flex flex-col flex-grow w-full max-w-xl bg-white shadow-xl rounded-lg overflow-hidden"
  >
    <div class="flex flex-col flex-grow h-0 p-4 overflow-auto">
      <!-- Mostrar mensajes del chat -->
      <div
        class="flex w-full mt-2 space-x-3 max-w-xs"
        v-for="(message, index) in messages"
        :key="index"
        :class="{ 'ml-auto justify-end': message.sentByUser }"
      >
        <!-- Imagen de perfil -->

        <!--El ! es una negación, por lo que cuando message.sentByUser es false (el bot envió el mensaje), se ejecuta este bloque.-->
        <div v-if="!message.sentByUser" class="flex-shrink-0 h-10 w-10 rounded-full bg-gray-300">
          <img :src="botProfilePic" alt="bot profile" class="h-10 w-10 rounded-full" />
        </div>
        <div v-if="message.sentByUser" class="flex-shrink-0 h-10 w-10 rounded-full bg-gray-300">
          <img :src="userProfilePic" alt="user profile" class="h-10 w-10 rounded-full" />
        </div>

        <div>
          <!-- Texto del mensaje -->
          <div
            :class="
              message.sentByUser
                ? 'bg-blue-600 text-white p-3 rounded-l-lg rounded-br-lg'
                : 'bg-gray-300 p-3 rounded-r-lg rounded-bl-lg'
            "
          >
            <p class="text-sm">{{ message.text }}</p>
            <!-- Mostrar imagen si la respuesta de la API tiene una -->
            <img
              v-if="message.image"
              :src="message.image"
              alt="response image"
              class="mt-2 rounded-md"
            />
          </div>
          <!-- Hora del mensaje -->
          <span class="text-xs text-gray-500 leading-none">{{
            getRelativeTime(message.time)
          }}</span>
        </div>
      </div>
    </div>

    <div class="bg-gray-300 p-4">
      <!-- Input para enviar mensaje -->
      <input
        v-model="newMessage"
        @keydown.enter="sendMessage"
        class="flex items-center h-10 w-full rounded px-3 text-sm"
        type="text"
        placeholder="Escribe tu mensaje…"
      />
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      newMessage: '', // Variable para el mensaje nuevo
      userProfilePic: 'https://i.pinimg.com/736x/99/61/a6/9961a677cbed9f1b3e2c7def5fd40120.jpg', // Propiedad de la Imagen del perfil del usuario
      botProfilePic: 'https://i.pinimg.com/236x/83/90/ea/8390eaaad5d2d427c951f62bee64eb4e.jpg', // Propiedad de la Imagen del perfil del bot
      messages: [
        {
          text: 'Hola un gusto, dime ¿Cuál es tu pregunta el día de hoy?',
          time: new Date(),
          sentByUser: false
        }
      ] // Mensaje inicial del chat
    }
  },
  methods: {
    //getCurrentTime retorna la fecha y hora actuales. Se utiliza para marcar la hora en la que se envían los mensajes.
    getCurrentTime() {
      return new Date() // Guardamos la hora actual como un objeto Date
    },
    //getRelativeTime convierte la marca de tiempo en un formato legible (por ejemplo, "Hace 2 minutos"). Se usa para mostrar cuánto tiempo ha pasado desde que se envió un mensaje.
    getRelativeTime(time) {
      const now = new Date()
      const diffMs = now - new Date(time)
      const diffMins = Math.floor(diffMs / (1000 * 60))

      //Esta variable diffMins almacena la cantidad de minutos que han pasado desde que se envió el mensaje hasta el momento actual.

      if (diffMins < 1) {
        return 'Hace un momento'
        //Si ha pasado exactamente 1 minuto, devuelve 'Hace 1 minuto'
      } else if (diffMins === 1) {
        return 'Hace 1 minuto'
      } else if (diffMins < 60) {
        return `Hace ${diffMins} minutos`
      } else {
        const hours = Math.floor(diffMins / 60)
        if (hours === 1) {
          return 'Hace 1 hora'
        } else {
          return `Hace ${hours} horas`
        }
      }
    },
    //Esta función se ejecuta cuando el usuario presiona "Enter". Envía el mensaje del usuario, realiza la solicitud a la API yesno.wtf y luego añade la respuesta del bot al chat.
    async sendMessage() {
      // Valida que el mensaje no esté vacío
      if (this.newMessage.trim() !== '') {
        // Agrega el mensaje del usuario al chat con la hora actual
        this.messages.push({
          text: this.newMessage,
          time: this.getCurrentTime(),
          ////Un valor booleano que indica si el mensaje fue enviado por el usuario (true) o por el bot (false).
          sentByUser: true
        })

        // Guardar el mensaje en una variable temporal y limpiar el campo
        const userMessage = this.newMessage
        this.newMessage = ''

        // Hacer la solicitud a la API
        try {
          const response = await fetch('https://yesno.wtf/api')
          const data = await response.json()

          // Agregar la respuesta del servidor al chat con la hora actual
          this.messages.push({
            text: data.answer, // Respuesta de la API
            image: data.image, // Imagen de la API
            time: this.getCurrentTime(),
            //Un valor booleano que indica si el mensaje fue enviado por el usuario (true) o por el bot (false).
            sentByUser: false
          })
        } catch (error) {
          // Manejo de error si falla la petición a la API
          console.error('Error fetching from API:', error)
          this.messages.push({
            text: 'Lo siento, hubo un error al procesar tu solicitud.',
            time: this.getCurrentTime(),
            sentByUser: false
          })
        }
      }
    }
  }
}
</script>
