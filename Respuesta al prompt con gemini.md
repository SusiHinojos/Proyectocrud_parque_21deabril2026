Hola. Como desarrollador de software, entiendo perfectamente el reto. Integrar Flutter con Firebase es el estándar de oro para apps móviles modernas, pero darle ese giro educativo usando la metodología de **Antigravity** (un framework conceptual para la orquestación de agentes y flujos de trabajo) eleva el proyecto de un simple CRUD a un sistema inteligente.

Aquí tienes el plan de trabajo detallado y la guía técnica para el proyecto **bdcrudparque**.

---

## 1. Configuración Inicial y Firebase

### Paso 1: Creación del Proyecto
En tu terminal, ejecuta los siguientes comandos para iniciar el proyecto y entrar al directorio:
```bash
flutter create crudatracciones
cd crudatracciones
```

### Paso 2: Configuración en la Consola de Firebase
1. Ve a [Firebase Console](https://console.firebase.google.com/).
2. Crea un nuevo proyecto llamado `bdcrudparque`.
3. En el menú lateral, ve a **Firestore Database** y haz clic en "Crear base de datos".
4. Selecciona "Iniciar en modo de prueba" (para desarrollo rápido) y elige una ubicación de servidor.
5. Crea una **Colección** llamada `atracciones`. Agrega un documento de prueba con los campos: `nombre` (string), `estaturaMinima` (number), `boleto` (number).

### Paso 3: Integración de Librerías
Abre tu archivo `pubspec.yaml` y añade las dependencias necesarias. La indentación es crítica en este archivo:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0  # Librería base
  cloud_firestore: ^5.0.0 # Librería para la BD
```
Luego ejecuta `flutter pub get` en la terminal.

---

## 2. Metodología Antigravity: Agentes y Flujos

Para que los estudiantes entiendan este paradigma, dividiremos la lógica no solo en funciones, sino en **entidades de responsabilidad**.

### Estructura de Agentes
| Agente | Rol | Skill (Habilidad) |
| :--- | :--- | :--- |
| **Agente de Datos** | Interface con Firestore | CRUD: Crear, Leer, Actualizar, Borrar. |
| **Agente de Validación** | Calidad de entrada | Validar tipos de datos (Nombre, Estatura, Precio). |
| **Agente de UI** | Orquestador de Interfaz | Renderizar widgets y capturar eventos de usuario. |

### Flujo de Trabajo (Workflow)
1. **Trigger:** Usuario presiona "Guardar".
2. **Acción Agente Validación:** Revisa que los campos no estén vacíos.
3. **Acción Agente de Datos:** Ejecuta la instrucción hacia Firebase.
4. **Respuesta:** Actualización en tiempo real de la lista mediante `StreamBuilder`.

---

## 3. Estructura de Carpetas Sugerida
```text
crudatracciones/
├── lib/
│   ├── agents/          # Lógica de Antigravity (Negocio)
│   │   ├── data_agent.dart
│   │   └── validator_agent.dart
│   ├── models/          # Estructura del objeto
│   │   └── atraccion_model.dart
│   ├── ui/              # Pantallas y Widgets
│   │   └── home_screen.dart
│   └── main.dart        # Punto de entrada e inicialización
```

---

## 4. Implementación del Código (Totalmente Funcional)

### Inicialización (`main.dart`)
```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'ui/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Conecta con la consola de Firebase
  runApp(MaterialApp(home: HomeScreen()));
}
```

### El Agente de Datos (`agents/data_agent.dart`)
Este agente posee el "Skill" de comunicarse con la nube.

```dart
import 'cloud_firestore/cloud_firestore.dart';

class DataAgent {
  final FirebaseFirestore _db = FirebaseFirestore.instance;

  // Skill: Crear
  Future<void> crearAtraccion(String nombre, double estatura, double precio) async {
    await _db.collection('atracciones').add({
      'nombre': nombre,
      'estaturaMinima': estatura,
      'boleto': precio,
    });
  }

  // Skill: Leer (Stream en tiempo real)
  Stream<QuerySnapshot> getAtracciones() {
    return _db.collection('atracciones').snapshots();
  }

  // Skill: Actualizar
  Future<void> actualizarAtraccion(String id, String nombre, double estatura, double precio) async {
    await _db.collection('atracciones').doc(id).update({
      'nombre': nombre,
      'estaturaMinima': estatura,
      'boleto': precio,
    });
  }

  // Skill: Borrar
  Future<void> borrarAtraccion(String id) async {
    await _db.collection('atracciones').doc(id).delete();
  }
}
```

### Interfaz de Usuario (`ui/home_screen.dart`)
Aquí el **Agente de UI** presenta la información.

```dart
import 'package:flutter/material.dart';
import '../agents/data_agent.dart';

class HomeScreen extends StatelessWidget {
  final DataAgent _agenteDatos = DataAgent();
  final TextEditingController _nombreCtrl = TextEditingController();
  final TextEditingController _estaturaCtrl = TextEditingController();
  final TextEditingController _boletoCtrl = TextEditingController();

  void _mostrarFormulario(BuildContext context, {String? id}) {
    showModalBottomSheet(
      isScrollControlled: true,
      context: context,
      builder: (_) => Padding(
        padding: EdgeInsets.only(bottom: MediaQuery.of(context).viewInsets.bottom, left: 15, right: 15, top: 15),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(controller: _nombreCtrl, decoration: InputDecoration(labelText: 'Nombre')),
            TextField(controller: _estaturaCtrl, decoration: InputDecoration(labelText: 'Estatura Mínima'), keyboardType: TextInputType.number),
            TextField(controller: _boletoCtrl, decoration: InputDecoration(labelText: 'Precio Boleto'), keyboardType: TextInputType.number),
            ElevatedButton(
              onPressed: () {
                if (id == null) {
                  _agenteDatos.crearAtraccion(_nombreCtrl.text, double.parse(_estaturaCtrl.text), double.parse(_boletoCtrl.text));
                } else {
                  _agenteDatos.actualizarAtraccion(id, _nombreCtrl.text, double.parse(_estaturaCtrl.text), double.parse(_boletoCtrl.text));
                }
                Navigator.pop(context);
              },
              child: Text(id == null ? 'Crear' : 'Actualizar'),
            )
          ],
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("CRUD Parque - Antigravity Agents")),
      body: StreamBuilder(
        stream: _agenteDatos.getAtracciones(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          return ListView.builder(
            itemCount: snapshot.data!.docs.length,
            itemBuilder: (context, index) {
              var doc = snapshot.data!.docs[index];
              return ListTile(
                title: Text(doc['nombre']),
                subtitle: Text("Estatura: ${doc['estaturaMinima']}m - \$${doc['boleto']}"),
                trailing: Row(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    IconButton(icon: Icon(Icons.edit), onPressed: () => _mostrarFormulario(context, id: doc.id)),
                    IconButton(icon: Icon(Icons.delete), onPressed: () => _agenteDatos.borrarAtraccion(doc.id)),
                  ],
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () => _mostrarFormulario(context),
      ),
    );
  }
}
```

---

### Resumen para Estudiantes
1. **Firebase** es nuestra bodega de datos.
2. **Flutter** es el cuerpo de nuestra aplicación.
3. **Antigravity** es la mente: en lugar de escribir código desordenado, creamos un **Agente de Datos** que sabe cómo hablar con Firebase, y un **Agente de UI** que sabe cómo hablar con el usuario. Cada uno tiene sus "Skills" (métodos) bien definidos.
