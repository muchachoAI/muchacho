Muchacho
![Muchacho](https://forgejo.canariannode.uk/muchacho/muchacho/media/branch/main/docs/muchacho.png)
Muchacho es un motor local de modelos y un cliente de línea de comandos escrito principalmente en Go, con ejecutores nativos para GGUF y otras arquitecturas incluidas en este repositorio. Permite descargar, crear, ejecutar, inspeccionar y publicar modelos sin renunciar al control de tus datos ni de tu hardware.
Características
Ejecución local interactiva y API HTTP compatible con las estructuras incluidas en `api/`.
Transferencias content-addressed con SHA-256, reanudación y subidas multiparte.
Manifiestos simples y listas de variantes para distintos ejecutores.
Gestión de modelos, procesos, copias, etiquetas y borrado.
Detección de CPU, memoria y aceleradores mediante las bibliotecas nativas incluidas.
Registro remoto independiente del daemon local.
Cliente y servidor reunidos en un único binario `muchacho`.

Descarga
Instalación Automatica (ACTUALMENTE SOLO LINUX AMD64)
```sh
curl -fsSL https://muchacho.uk/install.sh | sh
```

Compilación desde el código fuente:(CUALQUIER ARQ)
```sh
git clone https://forgejo.canariannode.uk/muchacho/muchacho.git
cd muchacho
```

Compilación
Se necesita Go 1.26.10 o posterior. Para compilar el binario principal:
```sh
go build -o muchacho .
./muchacho --version
./muchacho --help
```
Para compilar también los ejecutores nativos, instala CMake y un compilador C/C++ compatible:
```sh
cmake -B build . && cmake --build build --parallel 4 && go build -o muchacho .
```
El build de Go puede realizarse sin regenerar los componentes nativos ya disponibles. Para ejecutar inferencia se necesita el ejecutor nativo correspondiente a la plataforma y al formato del modelo.
Inicio rápido
Inicia el daemon local:
```sh
muchacho serve
```
El listener predeterminado es `http://127.0.0.1:11435`. Desde otra terminal:
```sh
muchacho pull usuario/modelo:latest
muchacho run usuario/modelo:latest
muchacho list
muchacho ps
muchacho show usuario/modelo:latest
muchacho stop usuario/modelo:latest
```
Comandos de gestión disponibles:
```text
run      Descarga si es necesario y ejecuta un modelo
pull     Descarga un manifiesto y todos sus blobs
push     Publica blobs y manifiesto usando tu clave
create   Construye un modelo desde un Modelfile
serve    Inicia la API local en el puerto 11435
list     Lista los modelos locales
ps       Lista los modelos cargados
rm       Elimina una referencia local
cp       Copia una referencia local
show     Muestra configuración, Modelfile y metadatos
stop     Descarga un modelo de memoria
```
Registry Muchacho
El origen predeterminado es `https://muchacho.uk`. Para utilizar una instancia local de Muchacho_web:
```sh
export MUCHACHO_REGISTRY=http://localhost:2050
muchacho pull library/modelo:latest
```
El registry puede usar HTTP en desarrollo o HTTPS en producción. `MUCHACHO_REGISTRY` acepta un origen sin ruta, por ejemplo `https://registry.example.com` o `http://127.0.0.1:2050`.
La autenticación de publicación utiliza la clave privada OpenSSH Ed25519 de:
```text
~/.muchacho/id_ed25519
```
Genera una clave si todavía no tienes una:
```sh
mkdir -p ~/.muchacho
ssh-keygen -t ed25519 -f ~/.muchacho/id_ed25519
```
Registra únicamente el contenido de `~/.muchacho/id_ed25519.pub` en la consola web. La clave privada no debe salir del equipo.
Crear y publicar
Ejemplo de `Modelfile`:
```text
FROM ./modelo.gguf
PARAMETER temperature 0.7
PARAMETER num_ctx 4096
SYSTEM """
Eres un asistente claro y preciso.
"""
```
Después:
```sh
muchacho create usuario/modelo:latest -f ./Modelfile
muchacho push usuario/modelo:latest
```
Cada blob se almacena por su digest. La publicación comprueba las capas existentes antes de transferirlas y finaliza con el manifiesto exacto de la etiqueta.
Configuración
Variable	Función	Valor predeterminado
`MUCHACHO_HOST`	Dirección del daemon local	`127.0.0.1:11435`
`MUCHACHO_REGISTRY`	Registry remoto	`https://muchacho.uk`
`MUCHACHO_MODELS`	Directorio de manifiestos y blobs	`~/.muchacho/models`
`MUCHACHO_ORIGINS`	Orígenes HTTP permitidos	Orígenes locales
`MUCHACHO_KEEP_ALIVE`	Tiempo de permanencia en memoria	`5m`

`MUCHACHO_NUM_PARALLEL`	Peticiones paralelas por modelo	Autodetectado
`MUCHACHO_MAX_LOADED_MODELS`	Modelos cargados por dispositivo	Autodetectado
`MUCHACHO_MAX_TRANSFER_STREAMS`	Transferencias simultáneas	`4`
`MUCHACHO_DEBUG`	Logs adicionales	Desactivado
Las variables heredadas de plataformas nativas, como `CUDA_VISIBLE_DEVICES`, se documentan en la ayuda del comando `serve`.
Almacenamiento local
```text
~/.muchacho/
├── id_ed25519
├── id_ed25519.pub
├── logs/
└── models/
    ├── blobs/
    ├── manifests/
    └── manifests-v2/
```
Los blobs son inmutables y pueden compartirse entre modelos. Las operaciones de borrado respetan las referencias restantes antes de liberar contenido.
Desarrollo y pruebas
```sh
go test ./types/model ./envconfig ./api ./manifest
go test ./...
```
La prueba completa puede necesitar dependencias nativas y plataformas específicas. Los tests de protocolo no sustituyen la validación contra un registry real.
Licencia
Consulta LICENSE. Las dependencias y componentes nativos conservan sus avisos de licencia correspondientes.
