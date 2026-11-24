# Resumen diario de stock con n8n

Flujo de n8n que revisa el inventario de una tienda, detecta qué productos están por debajo de su stock mínimo y envía por correo la propuesta de pedido agrupada por proveedor, con lo que cuesta cada pedido.

## El problema

En muchas tiendas pequeñas, la reposición se decide revisando el inventario a mano: mirar qué falta, calcular cuánto pedir y separar los pedidos por proveedor. Es una tarea que se repite cada día y en la que es fácil que se escape algún producto. Este flujo lo hace solo y deja el resumen en el correo, listo para hacer los pedidos.

## Qué hace

1. **Lee el inventario** desde un CSV con el stock actual, el stock mínimo, el proveedor y el precio de coste de cada producto.
2. **Filtra** los productos cuyo stock está igual o por debajo del mínimo.
3. **Calcula** cuántas unidades hay que pedir de cada uno (mínimo − stock) y el coste de ese pedido.
4. **Agrupa** los pedidos por proveedor y suma el coste de cada uno y el total.
5. **Separa** los productos que están justo en el mínimo, que no hace falta pedir todavía pero conviene vigilar.
6. **Envía un correo** con el resumen por Gmail. Si no falta nada, envía un aviso de que no hay pedidos.

Además incluye un **flujo de errores**: si algo falla, llega un correo con el nombre del flujo, el último nodo que se ejecutó, el mensaje de error y el enlace a la ejecución.

## Ejemplo de resultado

Con el inventario de ejemplo de [`datos/productos_inventados.csv`](datos/productos_inventados.csv), el correo queda así:

```
PEDIDOS POR PROVEEDOR

Deportes Rius SL (coste: 12.40 €):
  - Red de canasta: pedir 2 (hay 3, mínimo 5)

Tèxtils Garbí SL (coste: 33.60 €):
  - Calcetines de running (pack 3): pedir 7 (hay 8, mínimo 15)

FitSupply Ibérica SL (coste: 59.40 €):
  - Esterilla de yoga: pedir 6 (hay 0, mínimo 6)

Pala Norte SL (coste: 34.00 €):
  - Bote de pelotas de pádel: pedir 10 (hay 2, mínimo 12)

TOTAL: 139.40 €

AL LÍMITE (dentro del stock mínimo, vigilar):
  - Raqueta de pádel de iniciación: hay 4, mínimo 4
```

## Tecnologías

- **n8n** (Community Edition) sobre **Docker**.
- Nodos: lectura de fichero, extracción de CSV, Filter, If, Edit Fields, Aggregate y Gmail.
- Expresiones en **JavaScript** dentro de n8n para agrupar por proveedor y construir el mensaje.
- **Error Trigger** para avisar de fallos.

## Estructura

```
.
├── workflows
│   ├── resumen-diario-stock.json   # flujo principal
│   └── error-workflow.json         # aviso por correo cuando algo falla
├── datos
│   └── productos_inventados.csv    # inventario de ejemplo (datos ficticios)
└── docker-compose.yml              # n8n local con la carpeta datos montada
```

## Cómo ponerlo en marcha

Necesitas Docker Desktop instalado.

1. Levanta n8n desde esta carpeta:

   ```powershell
   docker compose up -d
   ```

   Abre http://localhost:5678 y crea tu usuario. La carpeta `datos` queda montada en `/home/node/datos`, que es donde el flujo busca el CSV.

2. Importa los dos flujos desde **Workflows → Import from File**, con los JSON de la carpeta `workflows`.

3. Crea una credencial de **Gmail (OAuth2)** en n8n y asígnala a los nodos de envío. En los JSON aparece como `REEMPLAZAR_ID_CREDENCIAL`.

4. Cambia el destinatario `tu-correo@ejemplo.com` en los nodos de Gmail por tu dirección.

5. En los ajustes del flujo principal, elige **Error Workflow** como flujo de errores.

6. Ejecuta el flujo con **Execute workflow**. Para que se ejecute solo cada día, cambia el disparador manual por un **Schedule Trigger** a la hora que te venga bien (la zona horaria ya está en Europe/Madrid).

Para adaptarlo a una tienda real basta con sustituir el CSV por el inventario exportado del sistema que se use, con las mismas columnas: `id, nombre, categoria, stock, stock_minimo, proveedor, precio_coste`.

## Nota

Los productos, proveedores y cifras del CSV son inventados. Los JSON no llevan credenciales ni direcciones de correo reales.
