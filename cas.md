# Content-Addressable Storage (CAS) e Idempotencia

*Notas de estudio a partir del análisis de `gentle-pi`*

---

## 1. Qué es Content-Addressable Storage, desde cero

La forma "normal" de guardar y referenciar datos es por **ubicación** (*location-addressable*): un archivo vive en `/home/user/docs/informe.pdf`, una fila en una tabla se referencia por su `id` autoincremental. La dirección te dice **dónde está** el dato, no dice nada sobre **qué contiene**.

Content-Addressable Storage invierte esto: la dirección del dato **es una función matemática de su contenido** — típicamente un hash criptográfico (SHA-256, SHA-1, etc.).

```
ubicación tradicional:  id=4821          → contenido puede cambiar sin que el id cambie
CAS:                     hash(contenido)  → si el contenido cambia, el hash cambia. Siempre.
```

Ejemplo concreto: si tenés el string `"hola mundo"`, su SHA-256 es siempre:

```
sha256("hola mundo") = 227ba3f8de62dc5f0bfd6688faceb08b1e693d919cbcc4e60fbc1e5638b09ea6
```

No importa quién lo calcule, en qué máquina, en qué momento — el resultado es siempre el mismo. Y si cambiás **un solo carácter**, el hash resultante es completamente distinto (efecto avalancha). Esta propiedad es la base de todo lo que sigue.

---

## 2. Git ya es, en el fondo, un sistema CAS

Esto es algo que probablemente ya intuías usando Git a diario, pero vale la pena hacerlo explícito porque es exactamente el mecanismo que `gentle-pi` reutiliza:

- Cada **blob** (contenido de un archivo) se guarda bajo el hash SHA de su contenido.
- Cada **tree** (una carpeta) se guarda bajo el hash de la lista de blobs/trees que contiene.
- Cada **commit** se guarda bajo el hash de su tree + metadata (autor, mensaje, padre).

```bash
$ echo "hola mundo" | git hash-object --stdin
3b18e512dba79e4c8300dd08aeb37f8e728b8dad
```

Este comando no *guarda* nada todavía — solo te muestra qué hash *tendría* ese contenido si lo guardaras. Esa es la esencia de CAS: el identificador se puede calcular **antes** de decidir si guardás algo, porque depende únicamente del contenido.

Consecuencia directa: **dos commits en dos repos distintos, en dos máquinas distintas, que representan exactamente el mismo árbol de archivos, tienen el mismo hash de tree.** Git no necesita preguntarle a nadie "¿este árbol ya existe?" — lo puede calcular localmente y comparar.

`gentle-pi` no inventa un sistema CAS nuevo — dice explícitamente que usa **CAS bajo el directorio común de Git** (`.git`), aprovechando la infraestructura de direccionamiento por contenido que Git ya provee, para atar sus propios "receipts" de aprobación a un árbol de contenido específico.

---

## 3. El problema que CAS resuelve acá: el receipt tiene que significar algo exacto

Repasemos el escenario: el sistema revisa un cambio, lo aprueba, y emite un **receipt** ("esto fue aprobado"). La pregunta crítica de diseño es: **¿aprobado con respecto a qué, exactamente?**

Si el receipt dijera simplemente `"PR #42: approved"`, sería ambiguo — ¿aprobado en qué estado del PR? Un PR puede recibir commits nuevos después de la aprobación. Con direccionamiento por ubicación (el número de PR), el receipt queda desconectado del contenido real que fue revisado.

Con CAS, el receipt se ata al **hash del árbol exacto**:

```
receipt = {
  approved: true,
  tree_hash: "a3f9e21...",   ← el árbol EXACTO que fue revisado
  lens_results: [...],
  timestamp: ...
}
```

Si alguien modifica un solo carácter de un archivo después de la aprobación, el hash del nuevo árbol es distinto. El receipt viejo, atado al hash anterior, **deja de aplicar automáticamente** — no porque alguien lo revoque manualmente, sino porque matemáticamente ya no corresponde al estado actual del código. No hace falta "invalidar" nada; la validez se cae sola por construcción.

Esto es lo que en el repo se describe como: *"Compact authority uses content-derived CAS under the Git common directory. Exact retries are idempotent; stale/semantic retries [...] fail closed."*

---

## 4. Idempotencia: la otra mitad de la historia

**Idempotencia** significa que ejecutar una operación una vez o múltiples veces produce el mismo resultado — no hay efectos secundarios acumulativos por repetir la llamada.

Ejemplo clásico:

```
PUT /usuarios/42 { nombre: "Ignacio" }   → idempotente (repetirlo no cambia nada más)
POST /usuarios/42/incrementar-saldo      → NO idempotente (repetirlo suma de nuevo)
```

¿Por qué importa esto en un sistema con agentes de IA? Porque los agentes (y las redes, y los procesos) **fallan y reintentan constantemente**. Un timeout de red, un proceso que se cae a mitad de camino, un usuario que aprieta el botón dos veces — todos estos son eventos normales, no excepcionales. Si tu sistema no es idempotente ante reintentos, cada uno de esos eventos comunes se convierte en un bug potencial (doble cobro, doble commit, estado corrupto).

### Cómo CAS te da idempotencia gratis

Acá está la conexión elegante entre los dos conceptos: si la clave de tu operación **es** el hash del contenido, entonces reintentar la misma operación con el mismo contenido **genera la misma clave**, y por lo tanto el sistema puede reconocer "esto ya lo procesé" sin necesidad de lógica adicional de deduplicación.

```
Intento 1: START(tree_hash=a3f9e21) → crea registro bajo esa clave
Intento 2 (reintento por timeout): START(tree_hash=a3f9e21) → misma clave, mismo registro
                                     → el sistema detecta que ya existe, no duplica nada
```

Esto es exactamente lo que dice el repo: *"Exact retries are idempotent."* Un reintento exacto (mismo contenido, mismo hash) es seguro por diseño — no por una capa extra de código que chequea "¿ya hice esto antes?", sino porque la estructura misma de direccionamiento hace que sea imposible crear un duplicado accidental.

---

## 5. Por qué "reintento exacto" vs "reintento con cambios" necesitan tratamiento distinto

Esta es la parte más sutil, y el repo la marca explícitamente: *"stale/semantic retries [...] fail closed."*

Hay una diferencia importante entre dos escenarios que a simple vista parecen "lo mismo":

| Escenario | Qué pasó | Tratamiento correcto |
|---|---|---|
| **Reintento exacto** | Se reenvía la misma operación, sobre el mismo contenido (mismo hash) — típicamente porque una respuesta se perdió en la red | Idempotente: se detecta y se devuelve el resultado ya calculado, sin reprocesar |
| **Reintento "stale" o "semántico"** | Se reenvía una operación pero el contenido subyacente **cambió** entre medio (otro hash) | NO es el mismo request — es una operación nueva sobre datos viejos, y debe rechazarse (fail closed), no procesarse como si fuera válida |

El peligro real está en el segundo caso: si el sistema tratara cualquier reintento como automáticamente válido sin comparar hashes, se abriría la puerta a aprobar cambios sobre un estado que ya no es el que originalmente se revisó — literalmente el mismo problema del trust boundary que charlamos antes, pero manifestado como un bug de concurrencia en vez de como una decisión de diseño explícita.

Esto es un ejemplo de un principio general en sistemas distribuidos: **la idempotencia solo es segura si podés distinguir con certeza "es la misma operación" de "parece la misma operación pero no lo es."** CAS te da esa certeza gratis, porque el hash *es* la definición de "mismo contenido" — no hay ambigüedad posible.

---

## 6. Un pseudocódigo simplificado para fijar la idea

```python
def start_review(git_tree):
    tree_hash = sha256(serialize(git_tree))  # CAS: la clave ES el contenido

    existing = storage.get(tree_hash)
    if existing is not None:
        # Reintento exacto detectado: mismo árbol, misma clave.
        # No reprocesamos, devolvemos lo que ya existía. Idempotente.
        return existing

    # Primera vez que vemos este árbol exacto: procesamos de cero.
    result = run_review_process(git_tree)
    storage.set(tree_hash, result)
    return result


def finalize_approval(tree_hash, review_result):
    current_tree_hash = sha256(serialize(get_current_git_state()))

    if current_tree_hash != tree_hash:
        # El código cambió desde que arrancó la revisión.
        # Esto NO es un reintento válido — fail closed.
        raise StaleCandidateError()

    return issue_receipt(tree_hash, review_result)
```

Notá que en ningún lado hay una tabla separada de "operaciones ya procesadas" con lógica manual de deduplicación — la clave de almacenamiento **es** la prueba de que el contenido es idéntico.

---

## 7. Dónde más aparece esta idea (para que la reconozcas)

Content-addressing e idempotencia vía hash no son exclusivos de Git ni de este proyecto — es un patrón que aparece una y otra vez en sistemas que necesitan integridad fuerte:

- **Docker/OCI images**: cada capa de una imagen se referencia por su hash de contenido (`sha256:abc123...`). Dos Dockerfiles distintos que producen la misma capa exacta comparten el mismo blob en disco.
- **IPFS**: sistema de archivos distribuido donde *todo* se direcciona por hash de contenido — es CAS llevado al extremo como diseño central de todo el sistema.
- **Claves de idempotencia en APIs de pago** (Stripe, MercadoPago): cuando mandás un `Idempotency-Key` en un request de cobro, es el mismo patrón — la clave garantiza que reintentar un request de red no duplique el cobro. En este caso la clave no siempre es un hash de contenido, pero cumple el mismo rol conceptual.
- **Package managers** (npm, pnpm, cargo): el `pnpm-lock.yaml` que usás en tus proyectos fija versiones exactas por integridad de hash (`integrity: sha512-...`) — exactamente para que "instalar de nuevo" sea reproducible byte a byte, no "probablemente lo mismo".

---

## 8. Preguntas para seguir pensando

1. En tu backend de **Reparto** (SQLite + Hono), ¿hay alguna operación que hoy no sea idempotente y debería serlo? Pensá en el flujo de creación de pedidos si el cliente hace doble tap.
2. ¿Cómo implementarías una clave de idempotencia para un webhook de MercadoPago (algo que ya debatiste en el pasado con `external_reference` nulo)? ¿Sería CAS puro (hash del payload) o una clave de idempotencia explícita que te manda el proveedor?
3. CAS te da idempotencia "gratis" para escritura, pero ¿qué pasa con operaciones que tienen efectos externos (mandar un email, cobrar una tarjeta)? ¿Cómo extenderías la idea para que esas tampoco se dupliquen ante un reintento?
4. ¿Qué overhead de performance tiene calcular hashes constantemente comparado con IDs autoincrementales simples? ¿Cuándo vale la pena pagar ese costo?

---

## 9. Lectura de referencia

- *Pro Git* (Scott Chacon), capítulo "Git Internals — Git Objects" — la mejor explicación práctica de cómo Git implementa CAS internamente.
- Documentación de **Idempotency Keys** de Stripe — ejemplo real de este patrón aplicado a pagos.
- IPFS whitepaper — CAS llevado a su forma más pura como sistema de archivos distribuido.
- El repo mismo: [`Gentleman-Programming/gentle-pi`](https://github.com/Gentleman-Programming/gentle-pi), sección "Bounded review transactions".
