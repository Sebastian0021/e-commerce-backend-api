# E-commerce Backend Api

Este proyecto es una aplicación backend robusta diseñada para gestionar las operaciones centrales de una tienda virtual. Incluye autenticación de usuarios con JWT y Passport.js, junto con endpoints de API estructurados para gestionar productos, carritos de compra y finalizar el proceso de compra con generación automática de tickets.

### Características

- **Gestión de Usuarios**: Registro, inicio de sesión y recuperación de información de usuario.
- **Autenticación**: Implementada con Passport.js usando estrategias Local y JWT. Los JWT se utilizan para la gestión de sesiones.
- **Gestión de Productos**: Operaciones CRUD para productos, incluyendo paginación, ordenación y consulta. Las imágenes de productos se pueden subir.
- **Gestión de Carritos**: Crear, ver, actualizar y eliminar productos dentro de un carrito de compra.
- **Proceso de Compra**: Endpoint para finalizar una compra, verificando stock, generando un ticket de compra y actualizando el carrito con productos no procesados.
- **Autorización Basada en Roles**: Los endpoints están protegidos según los roles de usuario (por ejemplo, `admin` para la gestión de productos y usuarios).
- **Validación de Entrada**: Middleware para validar datos de registro e inicio de sesión de usuarios.
- **Configuración de Entorno**: Utiliza `dotenv` para gestionar variables de entorno.

### Tecnologías Utilizadas

- **Node.js**
- **Express.js**: Framework web.
- **Mongoose**: Modelado de objetos MongoDB para Node.js.
- **Mongoose-Paginate-V2**: Para paginación en consultas de Mongoose.
- **Passport.js**: Para autenticación.
  - `passport-jwt`: Estrategia de autenticación JWT.
  - `passport-local`: Estrategia de autenticación por nombre de usuario y contraseña.
- **Bcrypt**: Para hashing de contraseñas.
- **JSON Web Token (JWT)**: Para el intercambio seguro de información.
- **Multer**: Para manejar `multipart/form-data`, principalmente para la subida de archivos.
- **Cookie-Parser**: Middleware para parsear cookies.
- **Express-Session**: Middleware para manejar sesiones.

### Instalación

Para configurar el proyecto localmente, sigue estos pasos:

1.  **Clonar el repositorio**:

    ```bash
    git clone [https://github.com/Sebastian0021/e-commerce-backend-api.git](https://github.com/Sebastian0021/e-commerce-backend-api.git)
    cd e-commerce-backend-api
    ```

2.  **Instalar dependencias**:

    ```bash
    npm install
    ```

3.  **Crear un archivo `.env`**:
    Crea un archivo `.env` en el directorio raíz del proyecto y añade las siguientes variables de entorno:

    ```
    PORT=3000
    URL_MONGO="your_mongodb_connection_string"
    JWT_SECRET="your_jwt_secret_key"
    JWT_EXPIRES_IN="1h"
    SESSION_SECRET="your_session_secret_key"
    COOKIE_AGE=3600000
    ```

    - `PORT`: El puerto en el que se ejecutará el servidor.
    - `URL_MONGO`: Tu cadena de conexión de MongoDB.
    - `JWT_SECRET`: Una clave secreta para firmar los JWT.
    - `JWT_EXPIRES_IN`: Tiempo de expiración para los JWT (por ejemplo, '1h', '7d').
    - `SESSION_SECRET`: Una clave secreta para firmar la cookie de ID de sesión.
    - `COOKIE_AGE`: La edad máxima (en milisegundos) de la cookie.

    El archivo `.env` y `node_modules` son ignorados por Git.

4.  **Ejecutar la aplicación**:

    ```bash
    npm run dev
    ```

    Esto iniciará el servidor usando `nodemon` y lo reiniciará automáticamente al detectar cambios en el código. Deberías ver `app listening on port 3000` (o el puerto especificado) y `MongoDB Connected: <host>` en la consola.

### Endpoints de la API

La API está estructurada con las siguientes rutas:

#### Sesiones

- `POST /api/sessions/register`: Registra un nuevo usuario.
- `POST /api/sessions/login`: Autentica un usuario y devuelve un JWT en una cookie.
- `GET /api/sessions/current`: Obtiene la información del usuario autenticado actual. Requiere JWT.
- `GET /api/sessions/logout`: Cierra la sesión del usuario eliminando la cookie JWT.

#### Usuarios

- `GET /api/users`: Obtiene todos los usuarios. (Solo administrador).
- `GET /api/users/current`: Obtiene la información del usuario autenticado actual.
- `GET /api/users/:uid`: Obtiene un usuario por ID.

#### Productos

- `GET /api/products`: Obtiene todos los productos. Admite parámetros `limit`, `page`, `sort` (precio asc/desc) y `query` (categoría/disponibilidad). (Solo administrador).
- `GET /api/products/:pid`: Obtiene un único producto por ID. (Solo administrador).
- `POST /api/products`: Agrega un nuevo producto. Admite la subida de archivos `thumbnails`. (Solo administrador).
- `PUT /api/products/:pid`: Actualiza un producto por ID. Admite la subida de archivos `thumbnails`. (Solo administrador).
- `DELETE /api/products/:pid`: Elimina un producto por ID. (Solo administrador).

#### Carritos

- `POST /api/carts`: Crea un nuevo carrito vacío.
- `GET /api/carts`: Obtiene todos los carritos.
- `GET /api/carts/:cid`: Lista los productos en un carrito específico.
- `PUT /api/carts/:cid`: Actualiza un carrito con un arreglo de productos.
- `PUT /api/carts/:cid/products/:pid`: Actualiza la cantidad de un producto específico en un carrito.
- `DELETE /api/carts/:cid/products/:pid`: Elimina un producto específico de un carrito.
- `DELETE /api/carts/:cid`: Elimina todos los productos de un carrito.
- `POST /api/carts/:cid/purchase`: Finaliza el proceso de compra para un carrito específico.
  - **Descripción**: Verifica el stock de los productos en el carrito. Aquellos con stock suficiente se procesan, restando la cantidad del inventario y generando un ticket de compra. Los productos sin stock suficiente se devuelven en una lista y permanecen en el carrito.
  - **Autenticación**: Requiere JWT.
  - **Respuesta Exitosa (201)**:
    ```json
    {
      "status": "success",
      "message": "Purchase completed successfully",
      "ticket": { "...datos del ticket..." },
      "productsNotPurchased": [ "id_del_producto_1", ... ]
    }
    ```
  - **Respuesta de Error (400)**:
    ```json
    {
      "status": "error",
      "message": "No products could be purchased due to lack of stock",
      "productsNotPurchased": [ "id_del_producto_1", ... ]
    }
    ```

### Estructura del Proyecto

```
e-commerce-backend-api/
├── .gitignore
├── README.md
├── package.json
├── package-lock.json
└── src/
    ├── app.js
    ├── config/
    │   ├── db.config.js
    │   ├── dotenv.config.js
    │   ├── multer.config.js
    │   └── passport.config.js
    ├── controllers/
    │   ├── cart.controller.js
    │   ├── product.controller.js
    │   ├── session.controller.js
    │   └── user.controller.js
    ├── dao/
    │   ├── dto/
    │   │   ├── cart.dto.js
    │   │   ├── product.dto.js
    │   │   ├── user.dto.js
    │   │   └── user.session.dto.js
    │   ├── models/
    │   │   ├── cart.model.js
    │   │   ├── product.model.js
    │   │   ├── ticket.model.js
    │   │   └── user.model.js
    │   ├── cart.dao.js
    │   ├── product.dao.js
    │   ├── user.dao.js
    │   └── ticket.dao.js
    ├── middlewares/
    │   └── validation.middleware.js
    ├── repositories/
    │   ├── cart.repository.js
    │   ├── product.repository.js
    │   ├── session.repository.js
    │   ├── user.repository.js
    │   └── ticket.repository.js
    ├── routes/
    │   ├── cart.router.js
    │   ├── product.router.js
    │   ├── session.router.js
    │   └── user.router.js
    └── utils/
        ├── __dirname.js
        ├── bcrypt.js
        ├── cookieExtractor.js
        ├── jwt.js
        └── passportCall.js
```

### Esquema de la Base de Datos

#### Modelo de Usuario (`src/dao/models/user.model.js`)

- `first_name`: String, requerido
- `last_name`: String, requerido
- `email`: String, único
- `age`: Number, requerido
- `password`: String, requerido (hash)
- `cart`: ObjectId (referencia a 'Carts')
- `role`: String

#### Modelo de Producto (`src/dao/models/product.model.js`)

- `title`: String, requerido
- `description`: String, requerido
- `code`: String, requerido
- `price`: Number, requerido
- `status`: Boolean, requerido
- `stock`: Number, requerido
- `category`: String, requerido
- `thumbnails`: Array de Strings

#### Modelo de Carrito (`src/dao/models/cart.model.js`)

- `products`: Array de objetos
  - `product`: ObjectId (referencia a 'products'), requerido
  - `quantity`: Number (predeterminado: 1)

#### Modelo de Ticket (`src/dao/models/ticket.model.js`)

- `code`: String, único, requerido
- `purchase_datetime`: Date, requerido (predeterminado: fecha/hora actual)
- `amount`: Number, requerido
- `purchaser`: String (referencia a 'User'), requerido
