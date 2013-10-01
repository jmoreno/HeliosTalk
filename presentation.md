## [helios](http://helios.io/),  an extensible [open-source](https://github.com/helios-framework/helios) mobile backend framework

### Ya, pero... ¿qué es?

* Aplicación Ruby construida sobre [Rack](http://rack.github.io/).
* Compuesto por varias aplicaciones Ruby hechas con Sinatra.
* Con una pequeña interfaz de administración web.

Note:
Rack es una especificación y una librería que permite a aplicaciones desarrolladas en Ruby, independietemente del framework elegido para desarrollarlas interactuar entre si. Para ello, obviamente tienen que soportar Rack. Gracias a esto, Helios se puede usar por si mismo, en aplicaciones hechas en Sinatra y en aplicaciones hechas en Rails.
En realidad, y como veremos a continuación, Helios se compone de varias aplicaciones hechas en Sinatra.
Sobre toda esta capa modular, Helios también incluye un pequeño front para realizar consultas.
Supongo que la idea de todo esto es ir creciendo, tanto en el número de módulos que integra el framework como en las habilidades de la parte front. En cualquier caso, si por si solo no crece, ya es un buen punto de partida para que nosotros vayamos incorporando otros módulos que sean de interés para nuestras aplicaciones.



## ¿Para qué sirve?

* Sincronización de datos
* Notificaciones PUSH
* Gestión de In-App Purchases
* Gestión de Passbook
* Gestión de Newstand
* Analíticas y logging
* Configuración remota

Note:
Pese a que estos son los servicios que, según la publicidad del sitio, incluye Helios, la parte de Analíticas y Configuración Remota no están integradas. Si echamos un vistazo a los repositorios que tiene Mattt en Github y filtramos los que contienen la palabra Rack, aparecen uno que se llama RemoteConfiguration y otro que se llama HTTP-Logger. Quizá en alguna versión posterior de Helios los veamos integrados.
Como puede verse, la oferta de Helios es el pack habitual de necesidades de una aplicación iOS no cubiertas por los frameworks de iOS y los servicios de iTunes Connect más alguna otra necesidad habitual como puede ser la sincronización de datos entre nuestro aplicación y un servidor. Gracias a servicios como Parse, muchas de estas necesidades han quedado cubiertas pero siempre queda la incertidumbre para el desarrollador que ha tenido una idea brillante pero no quiere arruinarse de... ¿cuanto tendré que pagar si me paso? Con Helios, si queremos ir en plan barato, siempre podemos tirar de nuestro Mac Mini de casa... o de una Raspberry Pi, ahí lo dejo...



## Sincronización de datos

### En el backend

* [Rack::Scaffold](https://github.com/mattt/rack-scaffold): Automatically generate RESTful CRUD services          

<br>

### En iOS

* [AFIncrementalStore](https://github.com/AFNetworking/AFIncrementalStore): Core Data Persistence with AFNetworking, Done Right... my ass!
* [AFNetworking](https://github.com/AFNetworking/AFNetworking): A delightful iOS and OS X networking framework.

Note:
En todos los servicios se va a repetir este patrón. Una o varias aplicaciones web nos proporcionan una seríe de métodos para realizar las funciones que nuestras apps demandan y una o varias librerías nos facilitan el acceso a dichos métodos en estas aplicaciones. Las aplicaciones de tipo Rack,  se encargan de crear las tablas necesarias para almacenar la información y además nos ofrecen los métodos necesarios para acceder a los datos, modificarlos, borrarlos, etc.


## [Rack::Scaffold](https://github.com/mattt/rack-scaffold)

* `GET /:resources`
* `POST /:resources`
* `GET /:resources/:id`
* `PUT /:resources/:id`
* `DELETE /:resources/:id`
* `GET /:resources/:id/:association` si hay relaciones

Note:
En el caso de Rack::Scaffold, las tablas que crea son aquellas que le entren a través de un xcdatamodel. También soporta ActiveRecord y Sequel, lo cual será útil cuando la estructura de datos que necesitemos en nuestro backend no coincida exactamente con el modelo de datos de nuestra aplicación. Los puntos de acceso que facilitan son la recuperación de todos los elementos, la creación de un nuevo elemento, la consulta de un elemento, la actualización de un elemento y el borrado de un elemento. CRUD. Los métodos GET soportan los parametros: page, per_page, offset y limit Los métodos POST y PUT devuelven el registro si todo ha ido correctamente


## [AFIncrementalStore](https://github.com/AFNetworking/AFIncrementalStore)

* Promete mucho pero AFNetworking es más seguro.

Note:
AFIncrementalStore es una librería que partiendo de NSIncrementalStore y usando AFNetworking se supone que recupera toda la información del backend, la actualiza en el almacen de persistencia que hayamos definido en nuestra app (plist, fichero o sqlite) y viceversa. No es una librería trivial, ya que el asunto es complejo pero todavía está un poco verde. No todos los ejemplos funcionan correctamente y el conocimiento que hay que tener para entender bien todo lo que sucede es avanzado. Utilizar AFNetworking es mucho más recomendable.



## Notificaciones PUSH

### En el backend

* [Rack::PushNotification](https://github.com/mattt/rack-push-notification): A Rack-mountable webservice for managing push notifications
* [Houston](https://github.com/nomad/houston): Apple Push Notifications; No Dirigible Required

<br>

### En iOS

* [Orbiter](https://github.com/mattt/Orbiter): Push Notification Registration for iOS

Note:
El servicio de notificaciones push se compone de dos aplicaciones web, Rack::PushNotification se encarga de crear y mantener las tablas necesarias para el control de las notificaciones push: registro de dispositivos, basicamente y Houston se encarga del envío. Para ayudarnos con la tarea del registro en la aplicación podemos usar Orbiter.


## [Rack::PushNotification](https://github.com/mattt/rack-push-notification)

* `PUT /push_notification/devices/:token`
* `DELETE /push_notification/devices/:token`
* `GET /push_notification/devices/?`
* `GET /push_notification/devices/:token/?`

Note:
Además de crear la tabla necesaria para el control de dispositivos, este servicio provee de un método para registrar dispositivos y otro para borrar dichos registros. Helios, por encima, nos provee de dos métodos más. Uno para conseguir el listado completo de dispositivos y otro para ver el detalle del dispositivo. Igual que en Scaffold, el get permite el uso de los parametros page, per_page, offset y limit pero además, permite hacer queries sobre toda la tabla mediante tsquery con el parametro "q".


## [Houston](https://github.com/nomad/houston)

* `POST /push_notification/message`
* Command-Line Interface 

Note:
El servicio que se encarga de hacer la notificación push contra los servidores de Apple es Houston. Helios lo combina con Rack::PushNotification para que la experiencia de uso sea más completa. La particularidad de Houston es que puede utilizarse desde el terminal mediante comandos. Para que funcione Houston hay que indicarle donde esta el certificado que proporciona Apple en el portal de desarrolladores que autoriza a nuestra aplicación a enviar notificaciones PUSH.


## [Orbiter](https://github.com/mattt/Orbiter)

* Integración con Helios

    - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
    {
        NSURL *serverURL = [NSURL URLWithString:@"http://demo.helios.javimoreno.me/devices/"];
        Orbiter *orbiter = [[Orbiter alloc] initWithBaseURL:serverURL credential:nil];
        [orbiter registerDeviceToken:deviceToken withAlias:nil success:^(id responseObject) {
            NSLog(@"Registration Success: %@", responseObject);
        } failure:^(NSError *error) {
            NSLog(@"Registration Error: %@", error);
        }];
    }
    
* También con Parse y Urban Airship

Note:
Para recibir notificaciones PUSH es necesario que registremos nuestro dispositvo. En el proceso de registro interviene nuestra aplicación, nuestro servidor, los servidores de Apple y nuestro dispositivo. Una vez que Apple ha hecho todas las verificaciones oportunas, nos facilita un código de identificación del dispositivo que tenemos que almacenar porque es el que tendremos que usar para enviar, desde nuestro servidor las notificaciones a dicho dispositivo. Orbiter es una librería que simplifica el proceso de registro, no solo para Helios si no también para los populares servicios Parse y Urban Airship.



## Gestión de In-App Purchases

### En el backend

* [Rack::InAppPurchase](https://github.com/mattt/rack-in-app-purchase)
* [Venice](https://github.com/nomad/venice): Ruby Gem for In-App Purchase Receipt Verification

<br>

### En iOS

* [Cargo Bay](https://github.com/mattt/CargoBay): The Essential StoreKit Companion

Note:
Entre las recomendaciones de Apple sobre las In-App Purchases destacan dos: Que la lista de productos disponibles para vender a través de tu aplicación proceda de un servicio web en lugar de estar precargado en la aplicación y que la verificación del recibo de compra que se obtiene cuando la transacción ha sido correcta se haga desde un servidor tuyo en lugar de hacerlo desde la propia aplicación. Todo en aras de la seguridad. Estas dos funciones pueden ser realizadas mediante Helios.
Al igual que en las notificaciones PUSH, la gestión de las In-App Purchases también depende de dos módulos, el Rack::InAppPurchase creará las tablas necesarias (Productos y Recibos) así como los puntos de acceso a los datos. Venice, por otro lado se encarga de la verificación del recibo contra los servidores de Apple.
En el lado iOS, Cargo Bay es una librería muy potente que nos facilitará mucho la taréa de gestionar las IAP desde una aplicación iOS.


## [Rack::InAppPurchase](https://github.com/mattt/rack-in-app-purchase)

* `POST /in_app_purchase/receipts/verify`
* `GET /in_app_purchase/products/identifiers`
* `GET /in_app_purchase/receipts/`

Note:
A diferencia del caso anterior, Rack::InAppPurchase integra Venice por lo que tanto la obtención de los productos disponibles como la verificación de los recibos los realiza este servicio. Helios vuelve a incluir un método por encima de Rack::InAppPurchase que permite recuperar todos los recibos verificados y hacer queries sobre la tabla de recibos mediante tsquery por medio del parametro "q". Igual que antes, se pueden usar los parametros page, per_page, offset y limit.


## [Venice](https://github.com/nomad/venice)

* Command-Line Interface 

Note:
Como ya hemos dicho antes, el servicio que realiza la verificación del recibo contra los servidores de Apple es Venice. Este servicio, igual que Houston puede ser usado desde el terminal.


## [Cargo Bay](https://github.com/mattt/CargoBay)

* Lista de productos disponibles:

    NSURL *url = [NSURL URLWithString:@"http://demo.helios.javimoreno.me/products/identifiers/"];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    [[CargoBay sharedManager] productsWithRequest:request success:^(NSArray *products, NSArray *invalidIdentifiers) {
        NSLog(@"Products: %@", products);
        NSLog(@"Invalid Identifiers: %@", invalidIdentifiers);
        _productsArray = [NSMutableArray arrayWithArray:products];
        [self.tableView reloadData];
    } failure:^(NSError *error) {
        NSLog(@"Error: %@", [error description]);
    }];

Note:
La librería Cargo Bay es, el AFNetworking de las In-App Purchases: sencilla, ligera y útil. Por un lado proporciona un método para descargar la lista de productos disponibles de nuestro servidor, cruzarlos con los que tenemos registrados en iTunes Connect y devolver dos arrays, uno con los productos que se pueden consumir y otro con los que no se pueden consumir. Aunque esto pueda parecer un sinsentido, si quisieramos quitar un In-App Purchase que estuviera dando algún problema, la forma más eficaz de hacerlo sería quitandolo de nuestro servidor. Por lo que podemos hacernos una idea de la importancia de este método.


## [Cargo Bay](https://github.com/mattt/CargoBay)

* Seguimiento de los pagos:

    - (void)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)options {
      [[CargoBay sharedManager] setPaymentQueueUpdatedTransactionsBlock:^(SKPaymentQueue *queue, NSArray *transactions) {
        NSLog(@"Updated Transactions: %@", transactions);
      }];

      [[SKPaymentQueue defaultQueue] addTransactionObserver:[CargoBay sharedManager]];

      // ...
    }

Note:
Cuando realizamos compras dentro de nuestra aplicación, hay que poner un "observer" que nos informe de cuando se ha realizado la compra para hacer lo que corresponda: descargar contenido, cambiar los ajustes, etc. Cargo Bay se encarga de estas taréas.


## [Cargo Bay](https://github.com/mattt/CargoBay)

* Verificación de la compra:

    [[CargoBay sharedManager] verifyTransaction:transaction password:nil success:^(NSDictionary *receipt) {
      NSLog(@"Receipt: %@", receipt);
    } failure:^(NSError *error) {
        NSLog(@"Error %d (%@)", [error code], [error localizedDescription]);
    }];

Note:
Por último, aunque en iOS 7 ya hay un método nuevo dentro del framework de StoreKit para realizar la verificación de los recibos desde la propia aplicación contra los servidores de Apple, Cargo Bay lo hace tanto para iOS 7 como para versiones anteriores de iOS.



## Gestión de Passbook

### En el backend

* [Rack::Passbook](https://github.com/mattt/rack-passbook)
* [Dubai](https://github.com/nomad/dubai): Generate and Preview Passbook Passes

<br>

### En iOS

* [AFNetworking](https://github.com/AFNetworking/AFNetworking): A delightful iOS and OS X networking framework.

Note:
En un esquema que ya nos tiene que empezar a ser familiar, Passbook se basa en otros dos servicios: Rack::Passbook se encarga de crear las tablas de "Passes" y "Registros" y Dubai se encarga de la generación del pase. A diferencia de los servicios anteriores, Dubai no está integrado en Helios. 
Para usar los diferentes métodos que proporciona Rack::Passbook no hay ninguna librería específica para iOS pero para eso está AFNetworking, no?


## [Rack::Passbook](https://github.com/mattt/rack-passbook)

* `GET /passbook/passes/:passTypeIdentifier/:serialNumber`
* `GET /passbook/devices/:deviceLibraryIdentifier/registrations/:passTypeIdentifier[?passesUpdatedSince=tag]`
* `POST /passbook/devices/:deviceLibraryIdentifier/registrations/:passTypeIdentifier/:serialNumber`
* `DELETE /passbook/devices/:deviceLibraryIdentifier/registrations/:passTypeIdentifier/:serialNumber`
* `GET /passbook/passes/`

Note:
Si hasta ahora habíamos visto que nuestros servidor se utilizaba para realizar comunicaciones con los servidores de Apple, con Passbook tenemos que tener en cuenta que los servidores de Apple también se van a comunicar con nuestro servidor y por tanto, si queremos que funcionen nuestros pases, tenemos que seguir sus instrucciones al pie de la letra.    
El primer método permite acceder a la información actualizada de un pase concreto, asociado a uno de los tipos de pase que hemos registrado en el provisioning portal.   
El segundo método nos proporciona los códigos de identificación de cada uno de los pases vinculados a un dispositivo para un tipo de pase concreto.   
El tercer método permite registrar un dispositivo a un pase asociado a un tipo de pase. A este método se le acompaña de un token para poder realizar notificaciones push.
El cuarto método permita borrar dispositivos del registro.
Por encima de estos métodos, Helios vuelve a crear un método que permite consultar todos los pases que se han registrado, permitiendo como en veces anteriores la búsqueda de texto completo.


## [Dubai](https://github.com/nomad/dubai)

* Command-Line Interface 

Note:
Como ya hemos comentado, a diferencia de los servicios anteriores, Dubai no está integrado en Helios. Por justificarlo de alguna forma, al margen de que haya cinco tipos diferentes de pases (Tarjetas de embarque, cupones, entradas, tarjetas-regalo y genéricos), la información que acompaña a cada pase es muy variadas y requiere de una personalización en una clase específica de nuestro servicio web. Dubai será la libreria con la que generaremos los pases una vez tengamos claro como van a ser y que información vamos a enviar.



## Gestión de Newsstand

### En el backend

* [Rack::Newsstand](https://github.com/mattt/rack-newsstand): Automatically generate webservice endpoints for Newsstand

<br>

### En iOS

* [AFNetworking](https://github.com/AFNetworking/AFNetworking): A delightful iOS and OS X networking framework.

Note:
En la versión inicial de Helios solo estaban los cuatro servicios anteriores. Tras unas semanas muy movidas, en las que cambiaron algunas cosas que, desgraciadamente, no actualizaron en el readme.md introdujeron un servicio para newsstand. El servicio Rack::Newsstand crea las tablas necesarias para almacenar el contenido a descargar en Newsstand. Dentro de la aplicación, para acceder al contenido


## [Rack::Newsstand](https://github.com/mattt/rack-newsstand)

* `GET /newsstand/issues`
* `GET /newsstand/issues/:name`
* `POST /newsstand/issues`

Note:
Tal y como hemos comentado antes, Rack::Newsstand proporciona los métodos para que las aplicaciones de tipo newsstand descarguen el nuevo contenido que haya que descargar. Estos métodos proporcionan la información en plist o en atom-xml en función del tipo de llamada que se efectue.
Adicionalmente, Helios aporta un método para grabar nuevos contenidos que puedan ser descargados. En la configuración del servicio también se puede indicar si los contenidos a descargar están en Amazon Web Services, Google Cloud Storage o Rackspace



## Analíticas y logging

### En el backend

* [Rack::HTTPLogger](https://github.com/mattt/rack-http-logger): Log metrics from HTTP request parameters according to l2met conventions

<br>

### En iOS

* [Antenna](https://github.com/mattt/Antenna): Extensible Remote Logging

Note:
Aunque Rack::HTTPLogger no está integrado dentro de Helios, es fácil de añadir a un proyecto existente. Lo que aporta este servicio es que da la oportunidad a servicios externos de grabar en el log de la aplicación y de esta forma poder monitorizar algunos eventos. Esto puede ser interesante, por ejemplo, en passbook ya que cuando los servidores de Apple acceden a nuestros servidores para recuperar información, van a intentar escribir en nuestro log los problemas que se hayan producido si es que se ha producido alguno. 
Adicionalmente, hay una librería que permite a una aplicación iOS grabar eventos en el log de nuestro servidor de la misma forma que lo haría cualquier otro servicio.



## Configuración remota

### En el backend

* [Rack::RemoteConfiguration](https://github.com/mattt/rack-remote-configuration): Serve property list or JSON configuration files

<br>

### En iOS

* [Ground Control](https://github.com/mattt/GroundControl): Remote Configuration for iOS
* [SkyLab](https://github.com/mattt/SkyLab): Multivariate & A/B Testing for iOS and Mac

Note:
En la página de presentación de Helios, se nos comenta la existencia de dos librerías para integrar configuración remota y A/B testing en nuestras aplicaciones. Con Rack::RemoteConfiguration podríamos servir datos para que Ground Control los procesará. En el caso de SkyLab sería cuestión de ver el tipo de test que queremos realizar y ver si lo que mejor nos encaja es crear un recurso mediante Rack::Scaffold y cargarlo para que alimente nuestros tests. Estas funcionalidades se alejan ya un poco de lo que la mayoría de los desarrolladores puede necesitar pero dan una idea de hasta que punto podemos personalizar Helios para ajustarlo a nuestras necesidades.



## Anexos

* [Nomad](http://nomad-cli.com): world-class command line utilities for iOS development
* [Rocket](http://rocket.github.io/): a hybrid approach to real-time cloud applications
* [PostgresApp](https://github.com/PostgresApp/PostgresApp): The easiest way to get started with PostgreSQL on the Mac
* [Induction](https://github.com/Induction/Induction): A Polyglot Database Client for Mac OS X
* [NSHipster](http://nshipster.com/): a journal of the overlooked bits in Objective-C and Cocoa
* [ASCIIwwdc](http://asciiwwdc.com): Searchable full-text transcripts of WWDC sessions

Note:
Para terminar, y por si a alguno se le han escapado, se incluyen otros desarrollos open-source de Mattt.     
Nomad es un conjunto de herramientas que pueden usarse desde el terminal. Houston, Dubai y Venice se encuentran entre ellas pero también otras para subir contenido al portal de desarrolladores y para compilar y generar archivos de distribución.
Rocket es una apuesta de futuro. Basandose en Rack::Scaffold y AFNetworking se podrían llegar a desarrollar aplicaciones con streaming de información al cliente cuando se realicen cambios en los datos del servidor. Algo que ahora está muy de moda.
Finalmente un par de aplicaciones muy útiles si se van a hacer pruebas de Helios en el Mac: Postgreapp para gestionar una base de datos PostgreSQL e Induction, un programa para poder consultar los datos.
Y finalmente NSHipster, un pozo de sabiduría semanal en el que Mattt nos explica algún framework o librería de Cocoa y ASCIIwwdc, un servicio para poder buscar sobre los subtitulos de la wwdc de 2013 y de esta forma encontrar el video que tiene lo que andamos buscando.
