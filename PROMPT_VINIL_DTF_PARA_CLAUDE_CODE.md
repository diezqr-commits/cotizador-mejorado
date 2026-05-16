# PROMPT PARA CLAUDE CODE — MÓDULO VINIL / DTF

## INTRODUCCIÓN

Necesito que agregues un nuevo módulo de **VINIL / DTF** al cotizador HTML (`cotizador-mejorado.html`). Este es un módulo complejo de cálculo de costos con dos técnicas diferentes de impresión: DTF (Direct-to-Film) y Vinil (corte).

**Esto es FASE 1 del ERP** — el módulo es standalone (HTML + JS puro), sin backend ni base de datos. Los datos se guardan temporalmente en localStorage hasta que se migre a PostgreSQL. Mantén la estructura serializable en JSON para futuras integraciones.

---

## ESPECIFICACIÓN COMPLETA

### PARTE 1: DTF (Direct-to-Film)

#### Parámetros
- Ancho de tira: **58 cm** (fijo)
- Costo proveedor: **$300/metro lineal**
- Precio venta: **$600/metro lineal**
- Sin límite de largo (puede ser N metros)

#### Entrada por impresión DTF
- Nombre/identificador (texto)
- Tamaño (dropdown): 1/4 carta (13.97×10.8 cm) | 1/2 carta (21.59×13.97 cm) | Carta (21.59×27.94 cm)
- Cantidad de playeras (entero ≥ 1)
- Upload de imagen (reutilizar componente del módulo de promocionales)
- Checkbox: "¿Diseño adaptativo?" (+$150 por pedido, se cobra una sola vez)
- Placeholder: si no sube imagen, mostrar recuadro con la proporción exacta del tamaño elegido

#### Cálculos DTF

**Material:**
- Área por impresión = base × altura × cantidad de playeras
- Área total = suma de todas las impresiones DTF
- Largo consumido = área total ÷ 58 cm
- Si largo < 10 cm → redondear a 10 cm
- Costo material = (largo / 100) × 600

**Bajadas de plancha:**
- Cada impresión DTF consume 2 bajadas
- 1ª bajada de TODAS: $25
- Cada bajada subsecuente: $15
- NO se reinicia (cálculo continuo de todas las bajadas del pedido)
- Ejemplo: 3 impresiones = 6 bajadas = $25 + $15 + $15 + $15 + $15 + $15 = $110
- Ejemplo 2: Frente (2) + Espalda (2) = 4 bajadas = $25 + $15 + $15 + $15 = $70

**Diseño adaptativo:**
- Checkbox opcional
- +$150 por pedido (se cobra una sola vez, no por impresión)

**Acomodo de piezas:**
- Usar algoritmo bin packing 2D (estrategia skyline)
- Tira de 58 cm de ancho × N metros de largo
- Permitir rotación automática de piezas (a menos que exista un campo "sin rotación")
- Gap entre piezas: 0.3 cm (3 mm)
- Generar SVG visual del acomodo
- Botón "Guardar distribución" al lado del botón "Imprimir" → descarga SVG (no va en PDF cliente)

---

### PARTE 2: VINIL (Corte)

#### Parámetros
- Ancho de tira: **25 cm** (de un rollo de 50 cm)
- Costo proveedor: **$165/metro lineal** (de la tira de 25 cm)
- Precio venta: **$165/metro lineal** (mismo, es un producto de producción)
- Máximo continuo: **3 metros por corte** (si excede, dividir en múltiples cortes)
- Márgenes: 1.5 cm arriba + 1.5 cm abajo
- Separación entre cortes: 0.8 cm

#### Entrada por impresión vinil
- Nombre/identificador (texto)
- Tamaño (dropdown): 1/4 carta | 1/2 carta | Carta (mismo que DTF)
- Cantidad de playeras (entero ≥ 1)
- Upload de imagen
- Depilación (dropdown): Simple | Compleja (OBLIGATORIO, no hay opción "ninguna")
- Placeholder si no sube imagen

#### Cálculos VINIL

**Material (acomodo vertical en tira 25 cm):**
- Base fija: 25 cm
- Altura por tamaño:
  - 1/4 carta: 10.8 cm
  - 1/2 carta: 13.97 cm
  - Carta: 27.94 cm
- Fórmula de largo total: `1.5 + (h1 + 0.8 + h2 + 0.8 + h3... + hn) + 1.5`
  - Ejemplo: 5 piezas de 1/4 carta = 1.5 + (10.8 + 0.8 + 10.8 + 0.8 + 10.8 + 0.8 + 10.8 + 0.8 + 10.8) + 1.5 = 60.1 cm
- Si largo total > 300 cm (3 m) → dividir en múltiples cortes
- Costo material = (largo / 100) × 165

**Corte (die-cutting):**
- **Por cantidad de playeras DE ESE TAMAÑO**, no cantidad total
- Si tienes 1/4 carta en 10 playeras → se cobra al precio de 10 playeras en 1/4 = $10
- Si tienes carta en 3 playeras → se cobra al precio de 3 playeras en carta = $60.17
- Tabla de precios (interpolación lineal):
  ```
  1/4 CARTA:
  Cant: 1($55) | 2($50) | 3($45) | 4($40) | 5($35) | 6($30) | 7($25) | 8($20) | 9($15) | 10+($10)
  
  1/2 CARTA:
  Cant: 1($66) | 2($60.33) | 3($54.67) | 4($49) | 5($43.33) | 6($37.67) | 7($32) | 8($26.33) | 9($20.67) | 10+($15)
  
  CARTA:
  Cant: 1($72) | 2($66.08) | 3($60.17) | 4($54.25) | 5($48.33) | 6($42.42) | 7($36.50) | 8($30.58) | 9($24.67) | 10+($18.75)
  ```

**Depilación (removal de excesos):**
- Por cantidad de playeras × costo unitario
- Simple: $6 (1/4) | $10 (1/2) | $15 (carta)
- Compleja: $10 (1/4) | $18 (1/2) | $25 (carta)

---

## UI/UX

### Sección de entrada
1. **Radio buttons:** Elige técnica → DTF o VINIL
2. Si DTF:
   - Sección "Impresiones DTF"
   - Botón "Agregar impresión DTF"
   - Para cada impresión: campos como se describió arriba
   - Checkbox "Diseño adaptativo" (global para todo el módulo)
   - Botón eliminar por impresión
3. Si VINIL:
   - Sección "Impresiones Vinil"
   - Botón "Agregar impresión vinil"
   - Para cada impresión: campos como se describió arriba
   - Botón eliminar por impresión

### Sección de cotización
- Tabla con desglose detallado (ver sección de tabla más abajo)
- Botones "Imprimir" (genera PDF con cotización)
- Botón "Guardar distribución" (solo si hay DTF) → descarga SVG

### Tabla de cotización

**Para DTF:**
```
| Impresión | Tamaño | Cantidad | Área (cm²) | Precio unitario | Bajadas | Subtotal |
```

**Para VINIL:**
```
| Impresión | Tamaño | Cantidad | Largo (m) | Material | Corte | Depilación | Subtotal |
```

**Totales:**
```
MATERIAL (DTF + VINIL):          $X.XX
BAJADAS DE PLANCHA (solo DTF):   $X.XX
DISEÑO ADAPTATIVO:               $X.XX (0 si no aplica)
CORTE (VINIL individual):        $X.XX (0 si no hay vinil)
DEPILACIÓN (VINIL):              $X.XX (0 si no hay vinil)
                                 -------
TOTAL:                           $X.XX
```

---

## ALGORITMO BIN PACKING 2D (para DTF)

Implementar en JS puro, sin librerías externas:

1. **Expandir piezas:** cada impresión con cantidad N genera N rectángulos individuales
2. **Ordenar:** por max(base, altura) descendente
3. **Skyline algorithm:**
   - Mantener lista de segmentos (x_start, x_end, y_height) que representa el perfil superior
   - Para cada pieza, probar ambas orientaciones (si rotación está permitida)
   - Buscar la posición más baja donde quepa (bottom-left fill)
   - Aplicar gap de 0.3 cm al ancho/alto efectivo
   - Actualizar skyline y fusionar segmentos consecutivos con misma altura
4. **Devolver:** largo total en cm, placements (id, x, y, w, h, label), eficiencia %

---

## VALIDACIONES

- Cantidad > 0
- Tamaño válido (1/4, 1/2, carta)
- DTF: cantidad de playeras es obligatoria (campo requerido)
- VINIL: depilación es obligatoria (no puede estar vacío)
- Mostrar errores en rojo, claro

---

## ESTRUCTURAS DE DATOS (para serialización)

### DTF Item
```javascript
{
  id: "dtf_1",
  nombre: "Frente",
  tamaño: "carta",
  cantidad: 10,
  imagen: "data:image/..." o null,
  diseñoAdaptativo: true
}
```

### VINIL Item
```javascript
{
  id: "vinil_1",
  nombre: "Logo",
  tamaño: "1/2",
  cantidad: 5,
  imagen: "data:image/..." o null,
  depilación: "simple"
}
```

### Cotización completa (almacenable en localStorage + futura BD)
```javascript
{
  modulo: "vinil_dtf",
  tecnica: "dtf", // "dtf" | "vinil" | "ambas"
  items: [...],
  diseñoAdaptativo: true,
  totales: {
    material: 123.45,
    bajadas: 0,
    diseño: 0,
    corte: 0,
    depilacion: 0,
    total: 123.45
  }
}
```

---

## CASOS DE PRUEBA

### Caso 1: DTF (validar algoritmo de packing)
**Entrada:**
- 15 impresiones de 10×10 cm (1 playera)
- 30 impresiones de 3×9 cm (1 playera)
- 5 impresiones de 11×2 cm (1 playera)
- Sin diseño adaptativo

**Esperado:**
- Largo total: ~44 cm (con gap 0.3 cm será ~46-48 cm)
- Piezas: 50 colocadas
- Eficiencia: ~94%
- Material: (0.46 / 100) × 600 = ~$276
- Bajadas: 50 × 2 = 100 bajadas = $25 + (99 × $15) = $1,510
- Total aprox: ~$1,786

### Caso 2: VINIL (validar acomodo y costos)
**Entrada:**
- Impresión 1: 1/4 carta, 5 playeras, depilación simple
- Impresión 2: carta, 5 playeras, depilación compleja

**Esperado:**
- Impresión 1:
  - Largo: 0.601 m, Material: $99.17
  - Corte (5 pzas 1/4): $35
  - Depilación (5 × $6): $30
  - Subtotal: $164.17
- Impresión 2:
  - Largo: 1.7614 m, Material: $290.63
  - Corte (5 pzas carta): $48.33
  - Depilación (5 × $25): $125
  - Subtotal: $463.96
- **Total: $628.13**

---

## NOTAS IMPORTANTES

1. **Reutilizar componentes:** el upload de imagen debe ser idéntico al módulo de promocionales
2. **Placeholder inteligente:** si no hay imagen, mostrar un recuadro punteado gris con aspect-ratio correcto (base/altura)
3. **Sin backend todavía:** localStorage temporal, datos serializables para futuro
4. **SVG de distribución:** solo para DTF, no va en PDF cliente, descarga como archivo aparte
5. **Mantener UI consistente:** mismo estilo visual que los módulos existentes
6. **No romper módulos existentes:** asegúrate de que el código nuevo no afecte otros módulos

---

## ENTREGA

- Integración limpia en `cotizador-mejorado.html`
- Todas las funciones de cálculo en JS puro
- SVG generado dinámicamente (sin librerías gráficas)
- Datos serializables en JSON
- Validaciones claras
- Comentarios en el código para futuras migraciones a backend

¡Adelante! 🚀
