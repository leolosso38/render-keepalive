# render-keepalive

Mantiene despierto el backend de **Admin360** alojado en Render (plan free).

## Problema

Los servicios web del plan free de Render se suspenden a los 15 minutos sin
tráfico entrante y tardan alrededor de un minuto en levantarse de nuevo. El
primer usuario que entra después de la suspensión se come esa espera.

## Solución

Dos workflows programados que corren en este repositorio:

| Workflow | Frecuencia | Función |
| --- | --- | --- |
| `keep-render-awake.yml` | cada 5 minutos | `GET` al health endpoint del backend para generar tráfico entrante y evitar la suspensión |
| `keepalive.yml` | día 1 de cada mes | Commit de `.heartbeat` para que el repositorio no quede inactivo |

## Por qué un repositorio aparte

GitHub Actions es gratuito e ilimitado en repositorios **públicos**. En un
repositorio privado, el plan Free incluye 2.000 minutos por mes y cada
ejecución se redondea a un minuto completo: un cron cada 5 minutos consumiría
8.640 minutos mensuales, agotaría la cuota en pocos días y, al no haber método
de pago configurado, GitHub bloquearía Actions hasta el ciclo siguiente —
incluyendo el CI del repositorio principal.

Este repositorio es público pero no contiene código de la aplicación:
únicamente la URL del endpoint de salud, que ya es accesible desde Internet.

El segundo workflow existe porque un repositorio público que solo tiene cron y
no recibe commits se considera inactivo, y GitHub desactiva los workflows
programados después de 60 días en ese estado.

## Configuración

`HEALTH_URL` es una variable opcional. Si no está definida, se usa:

```
https://sistema-backend-6lh1.onrender.com/health
```

Para cambiarla sin editar los workflows: **Settings > Secrets and variables >
Actions > Variables > New repository variable**.

## Notas

- Se consulta `/health` (liveness) y no `/api/health/arca` (readiness): el
  primero no toca base de datos ni ARCA, y se ejecuta 288 veces por día.
- El cold start medido de este servicio es de ~33 segundos. El `--max-time 90`
  deja margen suficiente para ese arranque incluso bajo carga.
- Render otorga 750 instance hours por mes y por workspace. Mantener un
  servicio activo las 24 horas consume alrededor de 730 horas, así que el
  margen es chico: un segundo servicio free excedería el cupo.
- `timeout-minutes` acota el consumo de minutos si una ejecución queda colgada.
