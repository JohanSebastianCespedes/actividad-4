# actividad-4
 1. Estructura de la Interfaz y Estilos (HTML & CSS)
Para la parte visual del chatbot, creé dos elementos principales fijados en la esquina inferior derecha mediante CSS (position: fixed):

El botón flotante (#chat-widget-button): Un botón circular con un gradiente dorado y verde que activa o desactiva la ventana del chat mediante la función toggleChat().

El contenedor de la ventana (#chat-box-container): Diseñado con un estilo oscuro acorde a la interfaz de la página. Tiene una cabecera con botón de cierre, un área central deslizable (#chat-messages) con overflow-y: auto para acumular el historial de mensajes, y la barra inferior de entrada con un <input> y su botón de envío.

2. Captura y Manejo de Mensajes (JavaScript)
Para hacer la interacción fluida, vinculé la función sendMessage() tanto al evento de clic del botón enviar como al presionar la tecla Enter (handleKeyPress(event)).

Cuando el usuario envía un mensaje:

Extraigo el texto escrito, lo limpio con .trim() y lo muestro inmediatamente en el historial como un mensaje del usuario llamando a appendMessage(text, 'user').

Convierto todo el texto a mayúsculas (.toUpperCase()) para evaluar fácilmente lo que escribió el usuario sin preocuparme por minúsculas o tildes.

3. Reconocimiento de Intención y Símbolos Cripto
Implementé una lógica de coincidencia que busca si el mensaje del usuario contiene el nombre o el símbolo de una criptomoneda dentro de una lista previa (['BTC', 'BITCOIN', 'ETH', 'ETHEREUM', 'SOL', 'SOLANA', 'XRP', 'DOGE', ...]):

Si el usuario escribe "Bitcoin" o "BTC", el código lo mapea automáticamente al símbolo estándar BTC.

Hago lo mismo para Ethereum (ETH), Solana (SOL) y otros activos populares.

4. Consulta a la API en Tiempo Real (fetch) e Integración Async/Await
Una vez identificado el activo:

Muestro un mensaje temporal indicando que estoy consultando el mercado.

Ejecuto una petición asíncrona mediante fetch() utilizando async/await conectándome a la API pública de CryptoCompare ([https://min-api.cryptocompare.com/data/price?fsym=SIMBOLO&tsyms=USD](https://min-api.cryptocompare.com/data/price?fsym=SIMBOLO&tsyms=USD)).

Al recibir la respuesta en JSON, formateo la cifra de dólares en pantalla con .toLocaleString() para mostrar el precio del activo actualizado en tiempo real.

5. Sistema de Respaldo y Manejo de Errores (Fallback / Try...Catch)
Para prevenir que el chatbot falle si no hay conexión a internet o si la API llega a estar fuera de servicio:

Envolví la petición dentro de un bloque try...catch.

Si la API no responde, el bloque catch entra en acción inmediatamente y consulta un objeto local llamado realPrices donde tengo guardadas las cotizaciones de referencia.

De esta manera, el chatbot siempre le entrega una respuesta válida al usuario y la aplicación se mantiene 100% estable sin importar las condiciones de la red.
