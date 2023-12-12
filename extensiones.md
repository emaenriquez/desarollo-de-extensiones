

# manifest.json
Estas claves contienen metadatos básicos de la extensión. Controlan cómo aparece la extensión en la página de extensiones y, cuando se publica, en Chrome Web Store.

```json
{
    "manifest_version": 3,
    "name": "Hello Extensions of the world!",
    "description": "Base Level Extension",
    "version": "1.0",
    "action": {
        "default_popup": "hello.html",
        "default_icon": "youtube.png"
    }
}
```

#  Proporciona los íconos

```json
{
  "icons": {
    "16": "images/icon-16.png",
    "32": "images/icon-32.png",
    "48": "images/icon-48.png",
    "128": "images/icon-128.png"
  }
}
```

Tamaño del ícono	Uso de íconos
16x16	            Ícono de página en las páginas y el menú contextual de la extensión
32 × 32	            Las computadoras con Windows suelen requerir este tamaño.
48 × 48	            Se muestra en la página Extensiones.
128 × 128	        Se muestra en la instalación y en Chrome Web Store.

# Declara la secuencia de comandos del contenido
Las extensiones pueden ejecutar secuencias de comandos que leen y modifican el contenido de una página. Estos se denominan secuencias de comandos de contenido.
```json
{
  "content_scripts": [
    {
      "js": ["scripts/content.js"],
      "matches": [
        "https://developer.chrome.com/docs/extensions/*",
        "https://developer.chrome.com/docs/webstore/*"
      ]
    }
  ]
}
```

# Incorporar secuencias de comandos en la pestaña activa
se compila una extensión que simplifica el estilo de la extensión de Chrome y de las páginas de documentación de Chrome Web Store para que sean más fáciles de leer.

En esta guía, explicaremos cómo hacer lo siguiente:

- Usa el service worker de extensión como coordinador de eventos.
- Conserva la privacidad del usuario mediante el permiso "activeTab".
- Ejecuta el código cuando el usuario haga clic en el ícono de la barra de herramientas de extensiones.
- Insertar y quitar una hoja de estilo con la API de Scripting
- Usa una combinación de teclas para ejecutar el código.

## Inicializa la extensión
Las extensiones pueden supervisar los eventos del navegador en segundo plano con el service worker de la extensión. Los service workers son entornos especiales de JavaScript que controlan eventos y finalizan cuando no son necesarios.

```json
{
  "background": {
    "service_worker": "background.js"
  },
}
```
## background.js
El primer evento que detectará nuestro service worker es runtime.onInstalled(). Este método permite que la extensión establezca un estado inicial o complete algunas tareas durante la instalación. Las extensiones pueden usar la API de Storage y IndexedDB para almacenar el estado de la aplicación. Sin embargo, en este caso, como solo se muestran dos estados, usaremos el texto de la insignia de acción para realizar un seguimiento de si la extensión está "ACTIVADA" o "DESACTIVADA".
```js
chrome.runtime.onInstalled.addListener(() => {
  chrome.action.setBadgeText({
    text: "OFF",
  });
});
```

## Habilita la acción de extensión
La acción de extensión controla el ícono de la barra de herramientas de la extensión. Por lo tanto, cuando el usuario haga clic en el ícono de la extensión, se ejecutará algún código (como en este ejemplo) o se mostrará una ventana emergente
```json
{
  "action": {
    "default_icon": {
      "16": "images/icon-16.png",
      "32": "images/icon-32.png",
      "48": "images/icon-48.png",
      "128": "images/icon-128.png"
    }
  },
}
```

Usa el permiso activeTab para proteger la privacidad del usuario
El permiso activeTab le otorga a la extensión la capacidad temporal de ejecutar código en la pestaña activa en ese momento. También permite el acceso a propiedades sensibles de la pestaña actual.

Este permiso se habilita cuando el usuario invoca la extensión. En este caso, el usuario invoca la extensión haciendo clic en la acción de la extensión.
```json
{
  "permissions": ["activeTab"],
}
```

# realiza un seguimiento del estado de la pestaña actual
Después de que el usuario haga clic en la acción de extensión, la extensión verificará si la URL coincide con una página de documentación
```js
const extensions = 'https://developer.chrome.com/docs/extensions'
const webstore = 'https://developer.chrome.com/docs/webstore'

chrome.action.onClicked.addListener(async (tab) => {
    if (tab.url.startsWith(extensions) || tab.url.startsWith(webstore)) {
    // Retrieve the action badge to check if the extension is 'ON' or 'OFF'
    const prevState = await chrome.action.getBadgeText({ tabId: tab.id });
    // Next state will always be the opposite
    const nextState = prevState === 'ON' ? 'OFF' : 'ON'

    // Set the action badge to the next state
    await chrome.action.setBadgeText({
      tabId: tab.id,
      text: nextState,
    })
}})
```