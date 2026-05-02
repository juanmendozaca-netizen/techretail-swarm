# TechRetail — Docker Swarm Deployment

Proyecto de orquestación de contenedores con Docker Swarm para la empresa TechRetail, desplegado sobre instancias AWS EC2.

## Arquitectura

- 1 nodo Manager — AWS EC2 Ubuntu 24.04
- 2 nodos Worker — AWS EC2 Ubuntu 24.04
- Red Overlay: `techretail_net`

## Servicios desplegados

| Servicio   | Imagen                    | Réplicas | Puerto |
|------------|---------------------------|----------|--------|
| frontend   | nginx:alpine              | 3 → 5    | 80     |
| backend    | node:18-alpine            | 2        | 3000   |
| database   | mysql:8                   | 1        | 3306   |
| cache      | redis:7-alpine            | 1        | 6379   |
| visualizer | dockersamples/visualizer  | 1        | 8080   |

## Requisitos previos

- 3 instancias AWS EC2 con Ubuntu 24.04
- Puertos abiertos en el Security Group: `22`, `80`, `2377`, `4789`, `7946`, `8080`
- Docker instalado en los 3 nodos

## Pasos de despliegue

### 1. Inicializar el clúster Swarm

En el Manager:

```bash
docker swarm init --advertise-addr 
```

En cada Worker:

```bash
docker swarm join --token  :2377
```

### 2. Crear el secret de base de datos

```bash
echo "tu_password_segura" | docker secret create db_password -
```

### 3. Desplegar el stack

```bash
docker stack deploy -c docker-compose.yml techretail
```

### 4. Verificar los servicios

```bash
docker stack services techretail
docker service ps techretail_frontend
```

### 5. Escalar el frontend dinámicamente

```bash
docker service scale techretail_frontend=5
```

## Comandos útiles

```bash
# Ver nodos del clúster
docker node ls

# Ver logs de un servicio
docker service logs techretail_backend --tail 20

# Ver réplicas de un servicio
docker service ps techretail_frontend

# Eliminar el stack
docker stack rm techretail

# Salir del swarm (workers primero, luego manager)
docker swarm leave
docker swarm leave --force
```

## Visualizer

Monitor visual del clúster disponible en:

```
http://<IP_PUBLICA_MANAGER>:8080
```

## Archivos del proyecto

| Archivo | Descripción |
|---------|-------------|
| `docker-compose.yml` | Definición completa del stack con todos los servicios |
| `README.md` | Documentación del proyecto |

## Autor

- **Nombre:** Juan Carlos Mendoza C.
- **Curso:** Desarrollo de Soluciones en la Nube
- **Fecha:** 01/05/2026
