# Práctica — Módulo 1: Fundamentos de DevOps y CI/CD

**Tiempo estimado:** 60 a 75 minutos
**Modalidad:** individual · se entrega un repositorio de GitHub

---

## Qué vas a hacer

Cuatro ejercicios cortos, uno por herramienta. **No son cuatro cosas separadas: es una sola aplicación que va atravesando el toolchain.**

```
   Ejercicio 1          Ejercicio 2          Ejercicio 3          Ejercicio 4
      GIT                 DOCKER             KUBERNETES           TERRAFORM
       │                     │                    │                    │
  versionás la  ──▶   la empaquetás  ──▶   la desplegás   ──▶  describís su
     página            en una imagen         con réplicas        infra en código
```

Al terminar vas a tener, en un solo repositorio, una landing estática versionada, contenerizada, desplegada y con su infraestructura declarada. Es un pipeline entero en miniatura.

## Antes de empezar

| Herramienta | Verificá con | Obligatoria |
|---|---|---|
| Git | `git --version` | Sí |
| Docker | `docker --version` | Sí |
| Minikube | `minikube version` | No (ejercicio 3 es opcional) |
| Terraform | `terraform --version` | Sí |

> Los ejercicios 1, 2 y 4 corren en cualquier máquina sin cuenta de nube ni tarjeta de crédito. El 3 necesita Minikube; si no lográs instalarlo, hay una alternativa al final del ejercicio.

**Descargá los archivos de apoyo** de esta práctica: contienen la página web, el Dockerfile, los manifiestos de Kubernetes y la configuración de Terraform, todos comentados línea por línea.

---

# Ejercicio 1 — Git: versionar el trabajo

**Objetivo:** entender que Git guarda una línea de tiempo de tu proyecto, y que podés trabajar en paralelo con ramas sin romper nada.
**Tiempo:** 15 minutos

## Paso 1 — Crear tu repositorio

Entrá a GitHub y creá un repositorio nuevo llamado `practica-devops-modulo1`. Marcá **público** y tildá "Add a README file".

Después, cloná el repositorio a tu máquina:

```bash
git clone https://github.com/TU-USUARIO/practica-devops-modulo1.git
cd practica-devops-modulo1
```

Verificá dónde estás parado:

```bash
git status
git log --oneline
```

Deberías ver la rama `main` limpia y un commit inicial (el del README que creó GitHub).

## Paso 2 — Tu primer commit

Creá un archivo nuevo:

```bash
echo "# Práctica Módulo 1 — TU NOMBRE" > NOTAS.md
echo "Fecha: $(date +%Y-%m-%d)" >> NOTAS.md
```

Y ahora el ciclo de tres pasos que vas a repetir toda tu carrera:

```bash
git status                      # 1. ¿qué cambió?
git add NOTAS.md                # 2. preparo lo que quiero guardar
git commit -m "docs: notas iniciales de la práctica"   # 3. lo guardo
```

Fijate en `git status` **antes y después** de cada comando. Es la mejor forma de entender qué hace cada uno.

## Paso 3 — Trabajar en una rama

Acá está el concepto que más cuesta al principio, y se entiende mejor haciéndolo que leyéndolo.

```bash
git checkout -b feature/agregar-autor
```

Editá `NOTAS.md`, agregá cualquier línea, y hacé commit:

```bash
git commit -am "feat: agrego mi presentación"
```

Ahora volvé a `main` y mirá el archivo:

```bash
git checkout main
cat NOTAS.md
```

**Tu línea desapareció.** No se borró: está en la otra rama. Volvé a mirarla:

```bash
git checkout feature/agregar-autor
cat NOTAS.md
```

Ahí está de nuevo. Eso es una rama: una línea de tiempo paralela.

## Paso 4 — Fusionar y subir

```bash
git checkout main
git merge feature/agregar-autor
git log --oneline --graph --all
git push origin main
```

Entrá a tu repositorio en GitHub: los commits tienen que estar ahí.

## ✅ Cómo saber que salió bien

- `git log --oneline` muestra al menos 3 commits
- Tu repositorio en GitHub muestra el archivo `NOTAS.md`
- `git branch` lista `main` y `feature/agregar-autor`

## ⚠️ Errores comunes

| Error | Qué significa | Solución |
|---|---|---|
| `Please tell me who you are` | Git no sabe quién sos | `git config --global user.name "Tu Nombre"` y `git config --global user.email "tu@email.com"` |
| `Authentication failed` al hacer push | GitHub ya no acepta contraseña | Generá un Personal Access Token en GitHub → Settings → Developer settings, y usalo como contraseña |
| `nothing to commit` | No editaste ningún archivo, o te olvidaste del `git add` | Corré `git status` para ver qué está pasando |
| `fatal: not a git repository` | Estás fuera de la carpeta del proyecto | `cd practica-devops-modulo1` |

---

# Ejercicio 2 — Docker: empaquetar la aplicación

**Objetivo:** construir una imagen con tu app adentro y correrla como contenedor. Que funcione igual en tu máquina que en cualquier otra.
**Tiempo:** 20 minutos

## Paso 1 — Preparar los archivos

Copiá `index.html`, `style.css`, `Dockerfile` y `.dockerignore` a una carpeta `docker/` dentro de tu repositorio.

Abrí `index.html` y **reemplazá `TU NOMBRE ACÁ` por tu nombre**. Abrí el `Dockerfile` y leelo entero: cada línea está comentada.

## Paso 2 — Construir la imagen

```bash
cd docker
docker build -t hola-devops:v1 .
```

> El punto final del comando **no es un error de tipeo**: le dice a Docker "el contexto de construcción es esta carpeta". Es el error de tipeo más frecuente de la clase.

Mirá la salida: Docker ejecuta cada instrucción del Dockerfile en orden y va creando capas. Cuando termina, verificá:

```bash
docker images | head -3
```

Tu imagen debería pesar **alrededor de 50 MB**. Para comparar: una máquina virtual con el mismo servidor web pesa entre 1 y 2 GB.

## Paso 3 — Correr el contenedor

```bash
docker run -d -p 8080:80 --name mi-landing hola-devops:v1
```

Qué significa cada parte:

| Flag | Qué hace |
|---|---|
| `-d` | *detached*: corre en segundo plano y te devuelve la terminal |
| `-p 8080:80` | conecta el puerto 8080 de **tu máquina** con el 80 **del contenedor** |
| `--name mi-landing` | le pone un nombre para no tener que usar el ID |

Verificá que está vivo y abrilo:

```bash
docker ps
```

Ahora andá a **http://localhost:8080** en el navegador. Ahí está tu página, servida desde adentro de un contenedor.

## Paso 4 — Explorar (esto es lo divertido)

```bash
docker logs mi-landing              # los logs del servidor
docker exec -it mi-landing sh       # entrar ADENTRO del contenedor
```

Una vez adentro, probá:

```sh
ls /usr/share/nginx/html   # ahí están tus archivos
cat /etc/os-release        # es un Alpine Linux completo
exit
```

Y comprobá la velocidad, que es el punto de todo esto:

```bash
docker stop mi-landing     # se apaga al instante
docker start mi-landing    # vuelve en menos de un segundo
```

Cuando termines, limpiá:

```bash
docker stop mi-landing && docker rm mi-landing
```

## ✅ Cómo saber que salió bien

- `docker images` lista `hola-devops:v1`
- `localhost:8080` muestra tu página con tu nombre
- `docker ps` muestra el contenedor con estado `Up`

## ⚠️ Errores comunes

| Error | Qué significa | Solución |
|---|---|---|
| `Cannot connect to the Docker daemon` | Docker Desktop no está abierto | Abrilo y esperá a que el ícono deje de moverse |
| `port is already allocated` | Ya hay algo usando el 8080 | Usá otro puerto: `-p 8081:80` |
| `COPY failed: file not found` | Estás construyendo desde otra carpeta | Asegurate de estar en la carpeta donde está el Dockerfile |
| La página carga sin estilos | El CSS no se copió | Revisá que el Dockerfile tenga la línea `COPY style.css` |
| `name is already in use` | Ya existe un contenedor con ese nombre | `docker rm -f mi-landing` y volvé a intentar |

---

# Ejercicio 3 — Kubernetes: orquestar (opcional)

**Objetivo:** ver qué agrega Kubernetes sobre Docker: réplicas, autoreparación y una dirección estable.
**Tiempo:** 25 minutos
**Requiere:** Minikube instalado

## Paso 1 — Levantar el clúster

```bash
minikube start
kubectl get nodes
```

Deberías ver un nodo en estado `Ready`. Ese es tu clúster de Kubernetes, corriendo en tu propia máquina.

## Paso 2 — Desplegar

Copiá `deployment.yaml` y `service.yaml` a una carpeta `k8s/` de tu repositorio. Leelos: están comentados línea por línea.

```bash
cd k8s
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Verificá:

```bash
kubectl get deployments
kubectl get pods
kubectl get services
```

Tenés que ver **dos Pods** en estado `Running`. Kubernetes creó dos copias porque el manifiesto dice `replicas: 2`.

## Paso 3 — La autoreparación (el momento clave)

Abrí una terminal y dejá esto corriendo:

```bash
kubectl get pods --watch
```

En **otra terminal**, matá un Pod a propósito:

```bash
kubectl delete pod <nombre-de-un-pod>
```

Mirá la primera terminal: Kubernetes detecta que falta una réplica y **levanta una nueva automáticamente, en segundos**. Nadie le avisó. Eso es lo que hace un orquestador.

## Paso 4 — Ver los logs y acceder

```bash
kubectl logs <nombre-de-un-pod>
kubectl describe pod <nombre-de-un-pod>    # útil cuando algo falla
minikube service hola-devops-svc           # abre el navegador solo
```

## Paso 5 — Limpiar

```bash
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
minikube stop
```

## 🔁 Si no podés instalar Minikube

No pasa nada, es lo esperable en el primer módulo. Hacé esto en su lugar:

1. Leé `deployment.yaml` y `service.yaml` completos.
2. Escribí en `NOTAS.md`, con tus palabras, respondiendo:
   - ¿Qué hace el campo `replicas` y qué pasaría si lo ponés en 5?
   - ¿Por qué el `selector` del Service tiene que coincidir con las `labels` del Deployment?
   - ¿Qué diferencia hay entre `requests` y `limits` en la sección `resources`?
3. Probá **Killercoda** (killercoda.com), que te da un clúster real en el navegador sin instalar nada, y corré los mismos comandos ahí.

## ✅ Cómo saber que salió bien

- `kubectl get pods` muestra 2 Pods `Running`
- Al borrar un Pod, aparece uno nuevo automáticamente
- `minikube service hola-devops-svc` abre la página en el navegador

## ⚠️ Errores comunes

| Error | Qué significa | Solución |
|---|---|---|
| Pod en `ImagePullBackOff` | Kubernetes no encuentra la imagen | Usá `nginx:alpine` en el manifiesto, o cargá la tuya con `minikube image load hola-devops:v1` y descomentá `imagePullPolicy: Never` |
| Pod en `Pending` | No hay recursos en el nodo | `minikube stop && minikube start --memory=4096` |
| `connection refused` de kubectl | El clúster no está levantado | `minikube start` |
| `error validating data` | Un problema de indentación en el YAML | En YAML la indentación es con **espacios**, nunca con tabs |

---