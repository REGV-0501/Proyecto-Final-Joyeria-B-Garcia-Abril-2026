Para modelar una joyería en una base de datos moderna (usualmente NoSQL como MongoDB por su flexibilidad), debemos estructurar la información de manera que los productos, las ventas y los clientes estén bien relacionados.

Aquí tienes un ejemplo de diseño de base de datos llamado `Joyeria_Elite`.

---

## 1. Estructura de la Base de Datos

En un modelo documental, organizamos la información en **Colecciones** (equivalente a tablas) y **Documentos** (equivalente a filas/registros).

### Colección: `productos`
Esta colección almacena el inventario. Es vital manejar atributos específicos como el tipo de metal y la pureza.

| Atributo | Tipo de Dato | Descripción |
| :--- | :--- | :--- |
| `_id` | ObjectId | Identificador único del sistema. |
| `nombre` | String | Nombre comercial (ej. "Anillo de Compromiso"). |
| `tipo` | String | Categoría (Anillo, Collar, Reloj, Pulsera). |
| `material` | Object | Sub-documento con `metal` (Oro) y `pureza` (18k). |
| `piedras` | Array | Lista de gemas incrustadas (Diamante, Esmeralda). |
| `precio` | Decimal128 | Precio de venta actual. |
| `stock` | Int32 | Cantidad disponible en tienda. |

**Ejemplo de Documento:**
```json
{
  "nombre": "Solitario de Diamante",
  "tipo": "Anillo",
  "material": { "metal": "Oro Blanco", "pureza": "14k" },
  "piedras": ["Diamante 0.5ct"],
  "precio": 1250.00,
  "stock": 5
}
```

---

### Colección: `clientes`
Almacena la información de contacto y preferencias de compra.

| Atributo | Tipo de Dato | Descripción |
| :--- | :--- | :--- |
| `nombre` | String | Nombre completo del cliente. |
| `email` | String | Correo electrónico (único). |
| `telefono` | String | Número de contacto. |
| `fecha_registro` | Date | Fecha en que se creó la cuenta. |
| `lealtad` | Boolean | Indica si pertenece al programa VIP. |

---

### Colección: `ventas`
Registra las transacciones. Aquí se suelen "embeber" o referenciar los datos del producto y cliente.

| Atributo | Tipo de Dato | Descripción |
| :--- | :--- | :--- |
| `fecha_venta` | Date | Marca de tiempo de la transacción. |
| `cliente_id` | Reference | Vínculo al ID del cliente. |
| `articulos` | Array | Lista de IDs de productos comprados. |
| `total` | Decimal128 | Monto total de la venta (incluyendo impuestos). |
| `metodo_pago` | String | Tarjeta, Efectivo o Transferencia. |

---

## 2. Ejemplo de Implementación (Comandos)

Si estuvieras usando una consola de base de datos (como mongosh), los comandos para inicializar esto serían:

1. **Crear/Usar la base de datos:**
   `use Joyeria_Elite`

2. **Insertar un documento en la colección de productos:**
```javascript
db.productos.insertOne({
    nombre: "Reloj de Lujo Chrono",
    tipo: "Reloj",
    material: {
        metal: "Acero Inoxidable",
        acabado: "Pulido"
    },
    precio: 850.50,
    stock: 12,
    especificaciones: ["Resistente al agua 50m", "Cristal de zafiro"]
});
```

> **Nota de experto:** En joyería es fundamental usar tipos de datos de precisión decimal (como `Decimal128`) para el precio. Nunca uses "Float" o "Double" para dinero, ya que los errores de redondeo binario podrían hacerte perder centavos (o pesos) en cálculos grandes.

¿Te gustaría que profundizara en cómo crear una consulta para buscar productos por un rango de precio específico o por tipo de metal?
