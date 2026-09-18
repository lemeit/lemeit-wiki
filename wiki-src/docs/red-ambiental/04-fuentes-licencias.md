# Fuentes, datos y licencias

Monitoreo Ambiental Escolar existe gracias al hardware y los datos de referencia de otros proyectos — algunos donaron equipos, otros publican los datos que usamos como contexto. Esta página deja explícito qué se usa de cada uno, bajo qué licencia, y cómo pensamos usar (y devolver) esos datos. No es un documento legal formal — es una nota de transparencia, escrita para que cualquiera (incluidos los propios proveedores) entienda de un vistazo qué hacemos con lo que nos dieron.

---

## PurpleAir

[purpleair.com](https://www.purpleair.com) fabrica los sensores **PurpleAir Flex** que forman el núcleo de esta red. Nacieron en 2015, cuando su fundador, Adrian Dybwad, quiso medir el polvo que levantaba una cantera cerca de su casa en Draper, Utah, y no encontró nada que no costara miles de dólares:

> "I was curious and wanted to know how much dust was blowing in. But there wasn't anything I could buy to see."

De ahí salió un sensor de bajo costo pensado para que cualquiera pudiera responderse esa misma pregunta. La filosofía se mantiene hoy:

> "PurpleAir is about the community. We're people worldwide with a common desire to know what we're breathing."

### Cómo se usan acá

Los **5 sensores PurpleAir Flex** de la red fueron una donación de [PurpleAir Collective](https://community.purpleair.com/t/purpleair-collective-june-july-2024/8771) (convocatoria de mitad de 2024), a partir de una propuesta de necesidad presentada por el autor — el proyecto quedó en 2° puesto de esa ronda. Los datos se consultan cada 15 minutos vía la API oficial de PurpleAir (`api.purpleair.com`) y se guardan en una base propia (Cloudflare D1) para poder ofrecer historial e informes — PurpleAir solo retiene ventanas cortas en su propia plataforma.

Desde que PurpleAir empezó a cobrar por el acceso a su API, ese acceso ya no es gratuito por defecto — hoy se costea con créditos propios del autor. Se está gestionando con PurpleAir un acceso sin cargo para este proyecto puntual, por tratarse de un uso educativo y por haber recibido los sensores como donación.

### Licencia

Los datos del mapa y de la API en tiempo real de PurpleAir se publican bajo su propia [licencia de datos](https://www.purpleair.com/license) (uso no comercial, con atribución). Este proyecto es enteramente educativo y sin fines de lucro, y cita a PurpleAir como fuente en cada tarjeta de sensor y en el pie de página del sitio. Para el texto completo y vigente, la referencia es siempre la página de licencia enlazada abajo.

**Links:** [Quiénes son](https://www.purpleair.com/about) · [Licencia de datos](https://www.purpleair.com/license) · [Términos de servicio](https://www.purpleair.com/policies/terms-of-service) · [Privacidad](https://www.purpleair.com/policies/privacy-policy)

---

## AirGradient

[airgradient.com](https://www.airgradient.com) fabrica los sensores **Open Air** que suman a la red la medición de CO2, VOC y NOx además de material particulado. Es hardware y firmware abierto: publican diseños y código bajo licencia **CC BY-SA 4.0**, e invitan explícitamente a modificarlos.

Tienen una postura fuerte sobre a quién le pertenecen los datos de calidad de aire:

> "Always read the fine print on data ownership. Choose monitors where YOU own the data and can share it freely." — sitio de AirGradient

Su CEO, Achim Haug, lo plantea incluso en términos éticos: restringir la propiedad de los datos es *"wrong and against humanity's best interests"*, aunque una empresa pueda seguir siendo rentable sosteniendo datos abiertos. Tienen hasta un [quiz interactivo](https://www.airgradient.com/aq-data-ownership-quiz/) para ayudar a detectar letra chica restrictiva en los términos de otros fabricantes.

### Cómo se usan acá

Los **2 sensores AirGradient Open Air** de la red están activos, hoy conectados en un domicilio particular a modo de prueba mientras se define en qué instituciones se instalan. Uno llegó como donación del programa OpenAQ Community Ambassadors y el otro se compró directo a AirGradient; ambos se gestionan igual, dentro del ecosistema propio de AirGradient. Los datos se consultan por la API propia de AirGradient y se guardan en la misma base D1 que PurpleAir, con el campo `proveedor` distinguiendo el origen.

Al estar dados de alta en el ecosistema de AirGradient, los dos sensores ya son visibles también en su [mapa público](https://map.airgradient.com) — sin necesidad de ningún paso extra de nuestra parte.

### Licencia

El hardware y firmware de AirGradient son **CC BY-SA 4.0** (uso y modificación libres con atribución). Los datos que sus monitores generan quedan bajo control de quien los opera — en este caso, del proyecto — que decide si publicarlos. Acá se publican en la API propia de Monitoreo Ambiental Escolar, con la misma filosofía de datos abiertos que promueve AirGradient.

**Links:** [Privacidad](https://www.airgradient.com/privacy-policy/) · [Términos](https://www.airgradient.com/terms-conditions/) · [Quiz de propiedad de datos](https://www.airgradient.com/aq-data-ownership-quiz/)

---

## OpenAQ

[openaq.org](https://openaq.org) es una organización sin fines de lucro cuya misión es abrir el acceso a datos de calidad de aire a nivel mundial — "fights air inequality by opening up air quality data". No venden datos ni información de usuarios, con una postura similar a la de AirGradient: el dato ambiental es un bien público.

### Cómo se usan acá

Hoy OpenAQ es sobre todo una referencia de diseño y de filosofía de proyecto — no hay integración activa de datos entre Monitoreo Ambiental Escolar y OpenAQ todavía. AirGradient ofrece la opción de publicar los datos de sus monitores en OpenAQ (queda a elección del operador); es algo a evaluar a futuro para los 2 sensores AirGradient de la red. Los sensores PurpleAir, en cambio, hoy no llegan a OpenAQ: desde que PurpleAir empezó a monetizar el acceso a su API, OpenAQ dejó de levantar sus monitores.

**Links:** [Privacidad](https://openaq.org/privacy/) · [Términos](https://openaq.org/terms/)

---

## Los datos que genera este proyecto

Siguiendo la misma filosofía de PurpleAir, AirGradient y OpenAQ, los datos que Monitoreo Ambiental Escolar genera y publica (metadata de sensores, lecturas históricas) están disponibles sin autenticación ni registro a través de la [API pública](https://aq.lemeit.ar/api.html) del proyecto — de lectura, CORS abierto, pensada para que cualquiera la consuma directo. Si reutilizás estos datos públicamente, agradecemos que menciones como fuente a [app.lemeit.ar/aq](https://app.lemeit.ar/aq).

