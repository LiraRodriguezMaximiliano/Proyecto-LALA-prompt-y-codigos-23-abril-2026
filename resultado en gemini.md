¡Hola! Es un gusto saludarte. Como desarrollador senior, entiendo perfectamente la estructura que buscas. Vamos a construir este sistema CRUD (Create, Read, Update, Delete) utilizando Flutter y Firebase, integrando la filosofía de **Antigravity** (que se enfoca en una arquitectura limpia, desacoplada y de alto rendimiento).

Aquí tienes el plan de trabajo detallado y la guía técnica para tus estudiantes.

---

## 1. Estructura de Directorios Inicial
Primero, preparamos el entorno local desde la terminal:

```bash
mkdir xflutterlira0627
cd xflutterlira0627
flutter create crudlala
cd crudlala
```

---

## 2. Configuración de Firebase y Librerías
Para integrar Firebase, necesitamos las dependencias principales. Modifica tu archivo `pubspec.yaml` de la siguiente manera:

### 5. Modificación de `pubspec.yaml`
```yaml
dependencies:
  flutter:
    sdk: flutter
  # 4. Librerías necesarias
  firebase_core: ^3.0.0      # Núcleo de Firebase
  cloud_firestore: ^5.0.0    # Base de datos NoSQL
  cupertino_icons: ^1.0.6
```
> **Nota:** Después de guardar, ejecuta `flutter pub get` en tu terminal.

### 3. Configuración en la Consola de Firebase
1. Ve a [Firebase Console](https://console.firebase.google.com/).
2. Crea un proyecto llamado `crud-empleados`.
3. En el menú lateral, ve a **Firestore Database** y haz clic en **Crear base de datos**.
4. Selecciona "Comenzar en modo de prueba" (para desarrollo) y elige una ubicación de servidor.
5. Crea una colección llamada `empleados`.

---

## 6 y 7. Implementación del CRUD (Práctica Guiada)
Bajo la filosofía **Antigravity**, separaremos la lógica de la interfaz. Esto facilita que el "agente" de datos sea independiente de la vista.

### Estructura de archivos sugerida:
* `lib/services/firebase_service.dart` (El Agente de Datos)
* `lib/pages/home_page.dart` (La Interfaz de Usuario)
* `lib/main.dart` (Punto de entrada)

#### A. El Agente de Datos (`firebase_service.dart`)
Este archivo maneja la lógica de negocio y la comunicación con Firestore.

```dart
import 'package:cloud_firestore/cloud_firestore.dart';

FirebaseFirestore db = FirebaseFirestore.instance;

// LEER (Read)
Future<List> getEmpleados() async {
  List empleados = [];
  QuerySnapshot querySnapshot = await db.collection('empleados').get();
  for (var doc in querySnapshot.docs) {
    Map<String, dynamic> data = doc.data() as Map<String, dynamic>;
    data['uid'] = doc.id;
    empleados.add(data);
  }
  return empleados;
}

// CREAR (Create)
Future<void> addEmpleado(String nombre, int edad, double salario) async {
  await db.collection('empleados').add({
    'nombre': nombre,
    'edad': edad,
    'salario': salario,
  });
}

// ACTUALIZAR (Update)
Future<void> updateEmpleado(String uid, String newNombre, int newEdad, double newSalario) async {
  await db.collection('empleados').doc(uid).set({
    'nombre': newNombre,
    'edad': newEdad,
    'salario': newSalario,
  });
}

// BORRAR (Delete)
Future<void> deleteEmpleado(String uid) async {
  await db.collection('empleados').doc(uid).delete();
}
```

---

## 8. Metodología: Agentes, Roles y Flujo de Trabajo
Para que los estudiantes entiendan cómo escalar, definimos estos roles:

| Componente | Rol | Skill (Habilidad) |
| :--- | :--- | :--- |
| **Data Agent** | Repositorio | Consultas CRUD a Firebase. |
| **UI Agent** | Presentador | Renderizar widgets con colores atractivos. |
| **Flow Manager** | Navegación | Controlar el paso entre la lista y el formulario. |

### Código de la Interfaz (`home_page.dart`)
Utilizaremos colores vibrantes (Deep Purple y Amber) para que sea visualmente atractivo.

```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Panel de Empleados'),
        backgroundColor: Colors.deepPurpleAccent,
      ),
      body: FutureBuilder(
        future: getEmpleados(),
        builder: (context, snapshot) {
          if (snapshot.hasData) {
            return ListView.builder(
              itemCount: snapshot.data?.length,
              itemBuilder: (context, index) {
                return Card(
                  elevation: 5,
                  margin: const EdgeInsets.symmetric(horizontal: 10, vertical: 6),
                  child: ListTile(
                    tileColor: Colors.white,
                    leading: const CircleAvatar(backgroundColor: Colors.amber, child: Icon(Icons.person)),
                    title: Text(snapshot.data?[index]['nombre'], style: const TextStyle(fontWeight: FontWeight.bold)),
                    subtitle: Text('Edad: ${snapshot.data?[index]['edad']} | Salario: \$${snapshot.data?[index]['salario']}'),
                    trailing: IconButton(
                      icon: const Icon(Icons.delete, color: Colors.redAccent),
                      onPressed: () async {
                        await deleteEmpleado(snapshot.data?[index]['uid']);
                        setState(() {}); // Refrescar flujo
                      },
                    ),
                  ),
                );
              },
            );
          } else {
            return const Center(child: CircularProgressIndicator());
          }
        },
      ),
      floatingActionButton: FloatingActionButton(
        backgroundColor: Colors.deepPurple,
        onPressed: () => _showForm(context),
        child: const Icon(Icons.add),
      ),
    );
  }

  // Flujo para agregar empleado (Dialogo)
  void _showForm(BuildContext context) {
    final nameCtrl = TextEditingController();
    final edadCtrl = TextEditingController();
    final salCtrl = TextEditingController();

    showModalBottomSheet(
      context: context,
      isScrollControlled: true,
      builder: (_) => Padding(
        padding: EdgeInsets.only(bottom: MediaQuery.of(context).viewInsets.bottom, left: 20, right: 20, top: 20),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(controller: nameCtrl, decoration: const InputDecoration(labelText: 'Nombre')),
            TextField(controller: edadCtrl, decoration: const InputDecoration(labelText: 'Edad'), keyboardType: TextInputType.number),
            TextField(controller: salCtrl, decoration: const InputDecoration(labelText: 'Salario'), keyboardType: TextInputType.number),
            const SizedBox(height: 20),
            ElevatedButton(
              style: ElevatedButton.styleFrom(backgroundColor: Colors.deepPurple),
              onPressed: () async {
                await addEmpleado(nameCtrl.text, int.parse(edadCtrl.text), double.parse(salCtrl.text));
                Navigator.pop(context);
                setState(() {});
              },
              child: const Text('Guardar Empleado', style: TextStyle(color: Colors.white)),
            )
          ],
        ),
      ),
    );
  }
}
```

### Inicialización en `main.dart`
```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'pages/home_page.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Inicializa el agente core de Firebase
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'CRUD Empleados Antigravity',
      theme: ThemeData(primarySwatch: Colors.deepPurple),
      home: HomePage(),
    );
  }
}
```

---

### Resumen del Flujo de Trabajo
1.  **Agente de Datos:** Escucha y envía información a Firestore (Independiente).
2.  **Roles:** El `FutureBuilder` actúa como el observador que espera los datos.
3.  **Skills:** Cada función (`add`, `delete`) es una habilidad específica del servicio.
4.  **UI:** Estilo moderno con `Card`, `ListTile` y `FloatingActionButton`.

Este diseño permite que el estudiante no solo copie código, sino que entienda que los datos fluyen desde la nube hacia una interfaz reactiva.
