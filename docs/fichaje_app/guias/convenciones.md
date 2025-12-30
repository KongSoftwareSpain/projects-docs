# Convenciones de Código

Este documento describe las convenciones y estándares de código que se siguen en el proyecto.

## 📂 Estructura de Archivos

### Backend (Node.js/Express)

**Nomenclatura:**

- Archivos: `camelCase.js` (ej: `asistenciaController.js`)
- Carpetas: `lowercase` (ej: `controllers`, `routes`)
- Modelos Sequelize: `UPPERCASE.js` (ej: `USUARIOS.js`, `PROYECTOS.js`)

**Organización:**

```
backend-AppServicios/
├── controllers/
│   └── [modulo]Controller.js    # Un controller por módulo
├── routes/
│   └── [modulo]Routes.js        # Un archivo de rutas por módulo
├── Model/
│   ├── init-models.js           # Inicialización
│   └── [TABLA].js               # Un archivo por tabla
├── middleware/
│   └── [nombre]Middleware.js
└── utils/
    └── [utilidad].js
```

### Frontend (Angular)

**Nomenclatura Angular:**

- Componentes: `nombre-componente.component.ts`
- Servicios: `nombre.service.ts`
- Guards: `nombre.guard.ts`
- Interfaces: `nombre.ts`
- Pipes: `nombre.pipe.ts`

**Organización:**

```
src/app/
├── pages/
│   └── nombre-pagina/
│       ├── nombre-pagina.component.ts
│       ├── nombre-pagina.component.html
│       ├── nombre-pagina.component.css
│       └── nombre-pagina.component.spec.ts
├── components/
│   └── nombre-componente/
│       └── [misma estructura]
├── services/
│   └── nombre/
│       ├── nombre.service.ts
│       └── nombre.service.spec.ts
└── interface/
    └── nombre.ts
```

## 🎨 Estilo de Código

### Backend (JavaScript)

**Formato general:**

```javascript
// ✅ BIEN
const funcionEjemplo = async (req, res) => {
  try {
    const { empresa } = req.user;
    const { parametro } = req.body;

    // Lógica de negocio
    const resultado = await db.TABLA.findAll({
      where: { id_empresa: empresa },
    });

    res.status(200).json(resultado);
  } catch (error) {
    console.error("Error en funcionEjemplo:", error.message);
    res.status(500).json({ message: "Error del servidor" });
  }
};
```

**Convenciones:**

- ✅ Usar `const` y `let`, nunca `var`
- ✅ Arrow functions para callbacks
- ✅ Async/await en lugar de promises con `.then()`
- ✅ Destructuring de objetos
- ✅ Template literals para strings

**Manejo de errores:**

```javascript
// ✅ BIEN - Try/catch en todos los controllers
const controller = async (req, res) => {
  try {
    // Lógica
  } catch (error) {
    console.error("Contexto del error:", error.message);
    res.status(500).json({ message: "Error del servidor" });
  }
};

// ❌ MAL - Sin manejo de errores
const controller = async (req, res) => {
  const data = await db.TABLA.findAll(); // Puede fallar
  res.json(data);
};
```

### Frontend (TypeScript)

**Formato general:**

```typescript
// ✅ BIEN
export class MiComponente implements OnInit {
  // Propiedades públicas primero
  titulo: string = "";
  datos: MiInterface[] = [];

  // Propiedades privadas después
  private subscription?: Subscription;

  constructor(private miService: MiService, private router: Router) {}

  ngOnInit(): void {
    this.cargarDatos();
  }

  cargarDatos(): void {
    this.miService.obtenerDatos().subscribe({
      next: (datos) => (this.datos = datos),
      error: (error) => console.error("Error:", error),
    });
  }

  ngOnDestroy(): void {
    this.subscription?.unsubscribe();
  }
}
```

**Convenciones TypeScript:**

- ✅ Tipado explícito en parámetros y retornos
- ✅ Interfaces para estructuras de datos
- ✅ Enums para valores constantes
- ✅ Readonly para propiedades inmutables
- ✅ Optional chaining (`?.`) y nullish coalescing (`??`)

**Servicios:**

```typescript
// ✅ BIEN
@Injectable({ providedIn: "root" })
export class MiService {
  private apiUrl = `${environment.apiUrl}/mi-recurso`;

  constructor(private http: HttpClient) {}

  obtenerTodos(): Observable<MiInterface[]> {
    return this.http.get<MiInterface[]>(this.apiUrl);
  }

  obtenerPorId(id: number): Observable<MiInterface> {
    return this.http.get<MiInterface>(`${this.apiUrl}/${id}`);
  }

  crear(datos: Partial<MiInterface>): Observable<MiInterface> {
    return this.http.post<MiInterface>(this.apiUrl, datos);
  }
}
```

## 🗄️ Base de Datos

### Modelos Sequelize

**Convenciones:**

- Nombres de tabla: `UPPERCASE` (ej: `USUARIOS`, `PROYECTOS`)
- Nombres de columna: `snake_case` (ej: `id_empresa`, `fecha_creacion`)
- Primary key: siempre `id`
- Foreign keys: `[tabla]_id` (ej: `usuario_id`, `proyecto_id`)

**Ejemplo de modelo:**

```javascript
module.exports = function (sequelize, DataTypes) {
  return sequelize.define(
    "USUARIOS",
    {
      id: {
        autoIncrement: true,
        type: DataTypes.INTEGER,
        allowNull: false,
        primaryKey: true,
      },
      nombre: {
        type: DataTypes.STRING(100),
        allowNull: false,
      },
      id_empresa: {
        type: DataTypes.INTEGER,
        allowNull: false,
        references: {
          model: "EMPRESA",
          key: "id",
        },
      },
      fecha_creacion: {
        type: DataTypes.DATE,
        allowNull: false,
        defaultValue: DataTypes.NOW,
      },
    },
    {
      sequelize,
      tableName: "USUARIOS",
      timestamps: false, // Deshabilitado (usamos fecha_creacion manual)
      indexes: [
        {
          name: "PRIMARY",
          unique: true,
          using: "BTREE",
          fields: [{ name: "id" }],
        },
        {
          name: "idx_empresa",
          using: "BTREE",
          fields: [{ name: "id_empresa" }],
        },
      ],
    }
  );
};
```

### Queries

**Convenciones:**

```javascript
// ✅ BIEN - Usar Sequelize
const usuarios = await db.USUARIOS.findAll({
  where: {
    id_empresa: empresa,
    activo: true,
  },
  attributes: ["id", "nombre", "email"],
  include: [
    {
      model: db.EMPRESA,
      as: "empresa",
      attributes: ["nombre"],
    },
  ],
  order: [["nombre", "ASC"]],
  limit: 20,
  offset: 0,
});

// ❌ EVITAR - Queries SQL directas
const result = await sequelize.query(
  "SELECT * FROM USUARIOS WHERE id_empresa = ?",
  { replacements: [empresa] }
);
```

## 🔐 Seguridad

### Autenticación

**En routes:**

```javascript
const express = require("express");
const router = express.Router();
const authenticateToken = require("../middleware/authMiddleware");
const authorizeRol = require("../middleware/authorizeRol");
const controller = require("../controllers/miController");

// Ruta pública
router.post("/login", controller.login);

// Ruta protegida (solo autenticado)
router.get("/datos", authenticateToken, controller.obtenerDatos);

// Ruta protegida por rol
router.post(
  "/admin",
  authenticateToken,
  authorizeRol(["admin"]),
  controller.funcionAdmin
);

module.exports = router;
```

### Validación de Entrada

**En controllers:**

```javascript
// ✅ BIEN - Validar entrada
const crearUsuario = async (req, res) => {
  try {
    const { nombre, email, password } = req.body;

    // Validación
    if (!nombre || !email || !password) {
      return res.status(400).json({
        message: "Nombre, email y password son obligatorios",
      });
    }

    if (password.length < 8) {
      return res.status(400).json({
        message: "Password debe tener al menos 8 caracteres",
      });
    }

    // Lógica de creación
    const usuario = await db.USUARIOS.create({ nombre, email, password });
    res.status(201).json(usuario);
  } catch (error) {
    console.error("Error al crear usuario:", error.message);
    res.status(500).json({ message: "Error del servidor" });
  }
};
```

## 📅 Manejo de Fechas

**Usar js-joda:**

```javascript
// Backend
const { LocalDateTime, ZoneId } = require("@js-joda/core");

// Obtener fecha actual
const ahora = LocalDateTime.now(ZoneId.of("Europe/Madrid"));

// Parsear fecha
const fecha = LocalDateTime.parse("2025-12-30T10:30:00");

// Formatear fecha
const fechaFormateada = ahora.format(
  DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm")
);
```

```typescript
// Frontend
import { LocalDateTime, ZoneId, DateTimeFormatter } from "@js-joda/core";

// Mismo uso que en backend
const ahora = LocalDateTime.now(ZoneId.of("Europe/Madrid"));
```

## 🎯 Patrones Comunes

### Controller Pattern (Backend)

```javascript
// Estructura estándar de un controller
const db = require("../Model");
const { Op } = require("sequelize");

const obtenerTodos = async (req, res) => {
  try {
    const { empresa } = req.user;
    const { page = 1, limit = 20 } = req.query;

    const offset = (page - 1) * limit;

    const { count, rows } = await db.TABLA.findAndCountAll({
      where: { id_empresa: empresa },
      limit: parseInt(limit),
      offset: parseInt(offset),
      order: [["fecha_creacion", "DESC"]],
    });

    res.status(200).json({
      data: rows,
      pagination: {
        total: count,
        page: parseInt(page),
        limit: parseInt(limit),
        totalPages: Math.ceil(count / limit),
      },
    });
  } catch (error) {
    console.error("Error en obtenerTodos:", error.message);
    res.status(500).json({ message: "Error del servidor" });
  }
};

const obtenerPorId = async (req, res) => {
  try {
    const { empresa } = req.user;
    const { id } = req.params;

    const registro = await db.TABLA.findOne({
      where: {
        id: id,
        id_empresa: empresa,
      },
    });

    if (!registro) {
      return res.status(404).json({ message: "No encontrado" });
    }

    res.status(200).json(registro);
  } catch (error) {
    console.error("Error en obtenerPorId:", error.message);
    res.status(500).json({ message: "Error del servidor" });
  }
};

const crear = async (req, res) => {
  try {
    const { empresa } = req.user;
    const datos = req.body;

    const nuevo = await db.TABLA.create({
      ...datos,
      id_empresa: empresa,
    });

    res.status(201).json(nuevo);
  } catch (error) {
    console.error("Error en crear:", error.message);
    res.status(500).json({ message: "Error del servidor" });
  }
};

const actualizar = async (req, res) => {
  try {
    const { empresa } = req.user;
    const { id } = req.params;
    const datos = req.body;

    const [updated] = await db.TABLA.update(datos, {
      where: {
        id: id,
        id_empresa: empresa,
      },
    });

    if (updated === 0) {
      return res.status(404).json({ message: "No encontrado" });
    }

    res.status(200).json({ message: "Actualizado correctamente" });
  } catch (error) {
    console.error("Error en actualizar:", error.message);
    res.status(500).json({ message: "Error del servidor" });
  }
};

const eliminar = async (req, res) => {
  try {
    const { empresa } = req.user;
    const { id } = req.params;

    const deleted = await db.TABLA.destroy({
      where: {
        id: id,
        id_empresa: empresa,
      },
    });

    if (deleted === 0) {
      return res.status(404).json({ message: "No encontrado" });
    }

    res.status(200).json({ message: "Eliminado correctamente" });
  } catch (error) {
    console.error("Error en eliminar:", error.message);
    res.status(500).json({ message: "Error del servidor" });
  }
};

module.exports = {
  obtenerTodos,
  obtenerPorId,
  crear,
  actualizar,
  eliminar,
};
```

### Service Pattern (Frontend)

```typescript
import { Injectable } from "@angular/core";
import { HttpClient, HttpParams } from "@angular/common/http";
import { Observable } from "rxjs";
import { environment } from "../../environments/environment";
import { MiInterface } from "../interface/mi-interface";

@Injectable({
  providedIn: "root",
})
export class MiService {
  private apiUrl = `${environment.apiUrl}/mi-recurso`;

  constructor(private http: HttpClient) {}

  obtenerTodos(page: number = 1, limit: number = 20): Observable<any> {
    const params = new HttpParams()
      .set("page", page.toString())
      .set("limit", limit.toString());

    return this.http.get<any>(this.apiUrl, { params });
  }

  obtenerPorId(id: number): Observable<MiInterface> {
    return this.http.get<MiInterface>(`${this.apiUrl}/${id}`);
  }

  crear(datos: Partial<MiInterface>): Observable<MiInterface> {
    return this.http.post<MiInterface>(this.apiUrl, datos);
  }

  actualizar(id: number, datos: Partial<MiInterface>): Observable<any> {
    return this.http.put<any>(`${this.apiUrl}/${id}`, datos);
  }

  eliminar(id: number): Observable<any> {
    return this.http.delete<any>(`${this.apiUrl}/${id}`);
  }
}
```

## 📝 Comentarios y Documentación

**Comentarios en código:**

```javascript
// ✅ BIEN - Comentarios útiles
// Verificar si el usuario tiene parte_auto habilitado
const parteAuto = await checkParteAuto(empresa);

// ❌ MAL - Comentarios obvios
// Crear usuario
const usuario = await db.USUARIOS.create(datos);
```

**JSDoc para funciones complejas:**

```javascript
/**
 * Crea un parte de trabajo automáticamente al fichar entrada
 * @param {number} usuarioId - ID del usuario
 * @param {number} proyectoId - ID del proyecto
 * @param {string} fecha - Fecha en formato ISO
 * @returns {Promise<Object>} Parte de trabajo creado
 */
const crearParteAutomatico = async (usuarioId, proyectoId, fecha) => {
  // Implementación
};
```

## 🚫 Anti-patrones a Evitar

### Backend

❌ **No usar callbacks anidados:**

```javascript
// ❌ MAL
db.USUARIOS.findOne({ where: { id: 1 } }, (err, user) => {
  if (err) return res.status(500).json({ error: err });
  db.PROYECTOS.findAll({ where: { usuario_id: user.id } }, (err, projects) => {
    // Callback hell
  });
});

// ✅ BIEN
const user = await db.USUARIOS.findOne({ where: { id: 1 } });
const projects = await db.PROYECTOS.findAll({ where: { usuario_id: user.id } });
```

❌ **No modificar req.user directamente:**

```javascript
// ❌ MAL
req.user.nuevaPropiedad = "valor";

// ✅ BIEN
const datosExtendidos = { ...req.user, nuevaPropiedad: "valor" };
```

### Frontend

❌ **No suscribirse sin desuscribirse:**

```typescript
// ❌ MAL
ngOnInit() {
  this.miService.obtenerDatos().subscribe(datos => {
    this.datos = datos;
  });
  // Memory leak!
}

// ✅ BIEN
private subscription?: Subscription;

ngOnInit() {
  this.subscription = this.miService.obtenerDatos().subscribe(datos => {
    this.datos = datos;
  });
}

ngOnDestroy() {
  this.subscription?.unsubscribe();
}
```

❌ **No manipular DOM directamente:**

```typescript
// ❌ MAL
document.getElementById("miElemento").innerHTML = "Nuevo texto";

// ✅ BIEN - Usar data binding
<div>{{ miTexto }}</div>;
```

## 🔧 Herramientas de Desarrollo

**Recomendadas:**

- **Editor**: VS Code
- **Extensiones VS Code**:
  - ESLint
  - Prettier
  - Angular Language Service
  - SQL Server (mssql)

**Configuración Prettier** (crear `.prettierrc`):

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": false,
  "printWidth": 100,
  "tabWidth": 2
}
```

---

**Siguiente**: [Introducción para Nuevos Desarrolladores](../onboarding/intro.md)
