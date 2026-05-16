# ESPECIFICACIÓN MÓDULO VINIL / DTF

## 1. PARÁMETROS GENERALES

### DTF (Direct-to-Film)
- Ancho de tira: **58 cm** (fijo)
- Costo proveedor: **$300/metro lineal**
- Precio venta: **$600/metro lineal**
- Sin máximo de largo (puede ser N metros)

### VINIL (Corte)
- Ancho de tira: **25 cm** (de un rollo de 50 cm que se corta por la mitad)
- Costo proveedor: **$165/metro lineal** (de la tira de 25 cm)
- Precio venta: **$165/metro lineal** (misma banda para producción)
- Máximo continuo: **3 metros por corte**
- Márgenes: 1.5 cm de cada lado (arriba y abajo)
- Separación entre cortes: 0.8 cm

---

## 2. TAMAÑOS ESTÁNDAR

```
1/4 CARTA:  13.97 × 10.8 cm
1/2 CARTA:  21.59 × 13.97 cm
CARTA:      21.59 × 27.94 cm
```

---

## 3. ENTRADA POR MÓDULO

### Paso 0: Elegir técnica
- Radio buttons: **DTF** o **VINIL**

### Si elige DTF:
Cada impresión DTF tiene:
- Nombre/identificador (texto)
- Tamaño: dropdown (1/4, 1/2, carta) → fijas las medidas en cm
- Cantidad (entero ≥ 1)
- Upload de imagen (reutilizar componente promocionales)
- Placeholder si no sube imagen

**Cálculo por impresión DTF:**
- Área = base × altura × cantidad
- Largo consumido = (área total de TODAS las impresiones DTF) ÷ 58 cm
- Si largo < 10 cm → redondear a 10 cm

**Bajadas de plancha:**
- Por cada impresión DTF: 2 bajadas
- 1ª bajada del pedido: $25
- Cada bajada subsecuente: $15
- Ejemplo: 3 impresiones = 6 bajadas = $25 + $15 + $15 + $15 + $15 + $15 = $110
- **Se reinicia por playera** (si son 2 playeras diferentes, cada una empieza con $25)
  - Pregunta: ¿cómo sabe el sistema para cuántas playeras es? → *Agregar campo "cantidad de playeras"*

**Diseño adaptativo:**
- +$150 **por pedido** (se cobra una sola vez aunque haya 3 impresiones DTF)
- Checkbox opcional en el módulo

**Acomodo DTF:**
- Bin packing 2D en tira 58 cm × N metros
- Igual al módulo DTF que ya se especificó antes
- Mostrar SVG con "Guardar distribución"

---

### Si elige VINIL:
Cada impresión vinil tiene:
- Nombre/identificador (texto)
- Tamaño: dropdown (1/4, 1/2, carta)
- Cantidad de playeras (entero ≥ 1)
- Upload de imagen
- Depilación: dropdown (simple, compleja, ninguna) → **obligatorio en vinil**
- Placeholder si no sube imagen

**Cálculo por impresión vinil:**

1. **Material (precio lineal):**
   - Largo base del tamaño (tomar de tabla de tamaños)
   - Cantidad de playeras → se acomodan en tira de 25 cm × 3 metros máx

2. **Acomodo en tira de 25 cm:**
   - Base fija: 25 cm
   - Altura variable: suma de alturas de todos los cortes + márgenes + separaciones
   - Márgenes: 1.5 cm arriba, 1.5 cm abajo (total 3 cm)
   - Separación entre cortes: 0.8 cm cada uno
   - Fórmula: `largo_total = 1.5 + (h1 + 0.8 + h2 + 0.8 + h3...) + 1.5`
   - Si largo > 300 cm (3 m) → dividir en múltiples cortes

3. **Corte (die-cutting):**
   - Por cantidad total de playeras
   - Escala interpolada:
     ```
     1 playera:   $55 (1/4) | $66 (1/2) | $72 (carta)
     2 playeras:  $50 (1/4) | $60.33 (1/2) | $66.08 (carta)
     3 playeras:  $45 (1/4) | $54.67 (1/2) | $60.17 (carta)
     4 playeras:  $40 (1/4) | $49 (1/2) | $54.25 (carta)
     5 playeras:  $35 (1/4) | $43.33 (1/2) | $48.33 (carta)
     6 playeras:  $30 (1/4) | $37.67 (1/2) | $42.42 (carta)
     7 playeras:  $25 (1/4) | $32 (1/2) | $36.50 (carta)
     8 playeras:  $20 (1/4) | $26.33 (1/2) | $30.58 (carta)
     9 playeras:  $15 (1/4) | $20.67 (1/2) | $24.67 (carta)
     10+ playeras: $10 (1/4) | $15 (1/2) | $18.75 (carta)
     ```

4. **Depilación (removal de excesos):**
   - Por cantidad de playeras × costo unitario
   - Simple: $6 (1/4) | $10 (1/2) | $15 (carta)
   - Compleja: $10 (1/4) | $18 (1/2) | $25 (carta)

---

## 4. VALIDACIONES

- Cantidad > 0
- Tamaño válido (1/4, 1/2, carta)
- DTF: ¿cantidad de playeras es obligatorio para calcular bajadas? → SÍ
- VINIL: Depilación es obligatoria → elegir simple o compleja

---

## 5. TABLA DE COTIZACIÓN FINAL

Para DTF:
```
| Impresión | Tamaño | Cantidad | Área | Precio material | Bajadas | Subtotal |
```

Para VINIL:
```
| Impresión | Tamaño | Cantidad | Largo consumido | Material | Corte | Depilación | Subtotal |
```

**Totales:**
```
COSTO DE MATERIAL (DTF + VINIL)
+ BAJADAS DE PLANCHA (solo DTF)
+ DISEÑO ADAPTATIVO (si aplica, una sola vez)
+ CORTE (solo VINIL, por cantidad total)
+ DEPILACIÓN (solo VINIL, por impresión)
= TOTAL
```

---

## 6. BOTÓN "GUARDAR DISTRIBUCIÓN"

- Solo para DTF (vinil no necesita acomodo visual complejo)
- SVG con la tira de 58 cm y acomodo de impresiones
- No va en PDF cliente
- Al lado del botón "Imprimir"

---

## 7. CASO DE PRUEBA

**Entrada:**
- Técnica: **VINIL**
- Impresión 1: 1/4 carta, 5 playeras, depilación simple
- Impresión 2: carta, 5 playeras, depilación compleja

**Cálculos esperados:**

*Impresión 1 (1/4 carta × 5 playeras):*
- Tamaño: 13.97 × 10.8 cm
- Acomodo vertical en tira 25 cm: 1.5 + 10.8 + 0.8 + 10.8 + 0.8 + 10.8 + 0.8 + 10.8 + 0.8 + 10.8 + 1.5 = 60.1 cm = 0.601 m
- Material: 0.601 m × $165 = $99.17
- Corte (5 playeras): $35 (interpolado para 1/4 carta)
- Depilación (5 playeras): 5 × $6 = $30
- Subtotal: $99.17 + $35 + $30 = $164.17

*Impresión 2 (carta × 5 playeras):*
- Tamaño: 21.59 × 27.94 cm
- Acomodo vertical: 1.5 + 27.94 + 0.8 + 27.94 + 0.8 + 27.94 + 0.8 + 27.94 + 0.8 + 27.94 + 1.5 = 176.14 cm = 1.7614 m
- Material: 1.7614 m × $165 = $290.63
- Corte (5 playeras): $48.33 (interpolado para carta)
- Depilación (5 playeras): 5 × $25 = $125
- Subtotal: $290.63 + $48.33 + $125 = $463.96

**Total esperado: $164.17 + $463.96 = $628.13**

---

## 8. ESTRUCTURAS DE DATOS

### DTF Item
```javascript
{
  id: "dtf_1",
  nombre: "Frente playera",
  tamaño: "carta", // "1/4" | "1/2" | "carta"
  cantidad: 10, // cantidad de playeras
  imagen: "data:image/..." o null,
  diseñoAdaptativo: true,
  bajadas: 2 // siempre 2 por impresión DTF
}
```

### VINIL Item
```javascript
{
  id: "vinil_1",
  nombre: "Logo",
  tamaño: "1/2",
  cantidad: 5, // cantidad de playeras
  imagen: "data:image/..." o null,
  depilación: "simple" // "simple" | "compleja" | "ninguna"
}
```

### Cotización (serializable para BD futura)
```javascript
{
  modulo: "vinil_dtf",
  tecnica: "vinil" o "dtf" o "ambas",
  items: [... array de items],
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
