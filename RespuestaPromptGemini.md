Esta es una guía técnica detallada para estructurar tu ecosistema de agentes y desarrollar el MVP de la aplicación de joyería. Utilizaremos una estructura modular para que tu agente global pueda gestionar las habilidades de diseño, código y scraping de forma independiente.

---

## 1. Estructura de la Habilidad Agente Global (`.agents`)

Primero, configuramos el entorno de "cerebro" para tus agentes en la raíz de tu espacio de trabajo.

**Estructura de Directorios:**
```text
.agents/
├── SKILL.md                # Definición de capacidades y reglas
├── scripts/                # Automatizaciones (bash/python)
├── ejemplos/               # Templates de código y UI
└── resources/              # Activos, prompts y configuraciones
```

**Contenido de `.agents/SKILL.md`:**
> Este archivo define el comportamiento de tus agentes.
> * **Skill Diseño:** Generación de temas, paleta de colores (oro, diamante) y layouts.
> * **Skill Código:** Lógica de negocio en Dart y arquitectura Clean.
> * **Skill Scraping:** Extracción de metadatos de joyería competidora para poblar la base de datos.

---

## 2. Preparación del Entorno y Prerrequisitos

Antes de codificar, debemos asegurar que el "taller" esté listo.

### Instalación de Herramientas
Si no tienes las herramientas, ejecuta en tu terminal:
1.  **Flutter:** Descarga el SDK oficial y añade `/bin` a tu PATH.
2.  **Firebase CLI:** ```bash
    npm install -g firebase-tools
    ```
3.  **Flutterfire CLI:**
    ```bash
    dart pub global activate flutterfire_cli
    ```

### Verificación y Login
Ejecuta estos comandos para asegurar la conectividad:
```bash
flutter doctor          # Verifica Flutter
firebase login          # Abre navegador para autenticar
flutterfire configure   # Vincula tu proyecto "joyeria_b" con Firebase
```

---

## 3. Configuración del Proyecto `joyeria_b`

Crea el proyecto y configura las dependencias necesarias en `pubspec.yaml`.

```bash
flutter create joyeria_b
cd joyeria_b
```

**Archivo `pubspec.yaml` (Dependencias clave):**
```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0
  cloud_firestore: ^5.0.0
  cupertino_icons: ^1.0.6
```
*No olvides ejecutar `flutter pub get` después de guardar.*

---

## 4. Arquitectura del Proyecto y Código CRUD

Implementaremos una estructura organizada por capas dentro de `lib/`.

### Modelo de Producto (`lib/models/producto_model.dart`)
```dart
class Producto {
  String id;
  String nombre;
  double precio;
  String descripcion;

  Producto({required this.id, required this.nombre, required this.precio, required this.descripcion});

  Map<String, dynamic> toMap() => {
    'nombre': nombre,
    'precio': precio,
    'descripcion': descripcion,
  };

  factory Producto.fromFirestore(Map<String, dynamic> data, String id) {
    return Producto(
      id: id,
      nombre: data['nombre'] ?? '',
      precio: (data['precio'] ?? 0).toDouble(),
      descripcion: data['descripcion'] ?? '',
    );
  }
}
```

### Servicio de Firestore (`lib/services/firebase_service.dart`)
```dart
import 'cloud_firestore/cloud_firestore.dart';
import '../models/producto_model.dart';

class FirebaseService {
  final CollectionReference _db = FirebaseFirestore.instance.collection('productos');

  // CREATE
  Future<void> addProducto(Producto p) => _db.add(p.toMap());

  // READ
  Stream<List<Producto>> getProductos() {
    return _db.snapshots().map((snap) =>
        snap.docs.map((doc) => Producto.fromFirestore(doc.data() as Map<String, dynamic>, doc.id)).toList());
  }

  // UPDATE
  Future<void> updateProducto(Producto p) => _db.doc(p.id).update(p.toMap());

  // DELETE
  Future<void> deleteProducto(String id) => _db.doc(id).delete();
}
```

---

## 5. Interfaz de Usuario (UI)

### Pantalla Principal (`lib/screens/home_screen.dart`)
Diseño elegante con tonos dorados para la joyería.

```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';
import '../models/producto_model.dart';

class HomeScreen extends StatelessWidget {
  final FirebaseService _service = FirebaseService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Joyeria B - Catálogo"), backgroundColor: Colors.amber[800]),
      body: StreamBuilder<List<Producto>>(
        stream: _service.getProductos(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          final productos = snapshot.data!;
          return ListView.builder(
            itemCount: productos.length,
            item_builder: (context, index) {
              final p = productos[index];
              return ListTile(
                title: Text(p.nombre),
                subtitle: Text("\$${p.precio}"),
                trailing: IconButton(
                  icon: Icon(Icons.delete, color: Colors.red),
                  onPressed: () => _service.deleteProducto(p.id),
                ),
                onTap: () => _mostrarFormulario(context, p),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () => _mostrarFormulario(context, null),
      ),
    );
  }

  void _mostrarFormulario(BuildContext context, Producto? p) {
    // Aquí implementarías un Modal o Dialog para capturar datos
  }
}
```

---

## 6. Verificación de Funcionalidad

Para probar el proyecto con tus agentes:
1.  **Terminal de VS Code:** Abre la terminal integrada.
2.  **Ejecución:** Usa `flutter run` para desplegar en tu emulador o dispositivo Android.
3.  **Consola Firebase:** Verifica en la sección **Firestore Database** que los documentos se crean y eliminan en tiempo real.

> **Nota para Antigravity IDE:** Asegúrate de tener el plugin de Dart/Flutter activo para que la detección de la carpeta `.agents` sea fluida y permita la automatización de los scripts que definiste en el paso 1.
