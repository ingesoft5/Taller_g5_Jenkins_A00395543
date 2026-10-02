Usuario admin: admin
Contraseña: password


1. Empaquetamiento Inmutable: ¿Por por qué está prohibido el uso del tag :latest al publicar artefactos en Nexus? ¿Que
esquema de versionado garantiza trazabilidad exacta entre un artefacto publicado y el commit de código que lo originó?

R// Porque es mutable, no habria forma de llevar correctamente el versionado porque no se podria diferenciar entre las versiones ya que se puede sobrescribir.Usar el hash del commit es una buena alternativa para garantizar la trazabilidad exacta entre un artefacto publicado y el commit de código que lo origino.


2. Webhooks vía Smee.io: ¿Por por qué es necesario un relay como Smee.io cuando el servidor Jenkins no tiene IP pública ni puertos expuestos a Internet? ¿Qué sucede con la automatización si el canal de Smee se detiene mientras un desarrollador hace push? 

R// Es necesario porque Jenkins esta corriendo en local y no tiene IP publica, por lo que GitHub no puede acceder a él. si el canal de Smee se detiene, la automatización se detendra. Cuando se hace push, GitHub envía el evento a Smee.io, pero como tu cliente Smee está caído, el evento nunca llega a Jenkins. El pipeline no se disparará automáticamente. 




