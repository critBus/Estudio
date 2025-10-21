# Flutter

## Estructura del Proyecto

- **Código principal:** Se encuentra en la carpeta `/lib`.
- **Plataformas:** Hay una carpeta por cada plataforma (por ejemplo, Android, iOS).
- **Tests:** Los tests están en la carpeta `/test`.
- **pubspec.lock:** Almacena información sobre las librerías usadas.
- **pubspec.yaml:** Contiene configuraciones generales del proyecto, como el nombre o la versión de Android objetivo.

## Main

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}
```

- `runApp` recibe un `Widget` como argumento.

## Widgets

Hay dos tipos principales de widgets en Flutter:

### StatelessWidget

- Es una pieza que se construye rápidamente y no mantiene estado por sí mismo.
- Flutter sabe cuándo debe volver a dibujarse.
- Se recomienda crear clases que extiendan de `StatelessWidget` en lugar de funciones que retornen widgets.
- No tiene estado interno.

**Ejemplo:**

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Center(child: Text("hola mundo")),
    );
  }
}
```

- **Constructor con `const`:** Es buena práctica usar `const` si el widget no cambiará después de ser construido.

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: CounterScreem(),
    );
  }
}
```

### StatefulWidget

- Similar a `StatelessWidget`, pero permite mantener un estado interno y tiene un ciclo de vida (inicialización, destrucción, etc.).
- Es esencial para animaciones y otros comportamientos dinámicos.

**Ejemplo:**

```dart
class CounterScreem extends StatefulWidget {
  const CounterScreem({super.key});

  @override
  State<CounterScreem> createState() => _CounterScreemState();
}

class _CounterScreemState extends State<CounterScreem> {
  int clickCounter = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('el titulo'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              "a",
              style: TextStyle(fontSize: 10, fontWeight: FontWeight.w100),
            ),
            Text("b")
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {},
        child: Icon(Icons.plus_one),
      ),
    );
  }
}
```

## Componentes Comunes

### Text

```dart
Text("hola mundo")
```

### Center

```dart
Center(child: Text("hola mundo"))
```

### MaterialApp

```dart
MaterialApp(
  home: Center(child: Text("hola mundo")),
);
```

- **Con temas y sin debug:**

```dart
MaterialApp(
  debugShowCheckedModeBanner: false,
  home: CounterScreem(),
  theme: ThemeData(useMaterial3: true, colorSchemeSeed: Colors.red),
);
```

### Column

```dart
Column(
  mainAxisAlignment: MainAxisAlignment.center,
  children: [Text("a"), Text("b")],
)
```

### Icon

```dart
Icon(Icons.plus_one)
```

### Botones

#### Propiedades Comunes

- **enableFeedback:** Ofrece una reacción al presionar el botón (vibración o sonido).
- **elevation:** Aumenta la profundidad de la sombra.

**Ejemplo:**

```dart
FloatingActionButton(
  enableFeedback: true,
  elevation: 5,
  onPressed: onPressed,
  child: Icon(icon),
);
```

#### FloatingActionButton

```dart
FloatingActionButton(
  shape: const StadiumBorder(),
  onPressed: () {
    clickCounter++;
    setState(() {});
  },
  child: Icon(Icons.plus_one),
)
```

#### IconButton

```dart
IconButton(
  onPressed: () {},
  icon: Icon(Icons.refresh_rounded),
)
```

#### FilledButton

```dart
FilledButton.tonal(
  onPressed: () {},
  child: const Text('Click me'),
)
```

### Scaffold

```dart
Scaffold(
  appBar: AppBar(
    title: const Text('el titulo'),
  ),
  body: Center(
    child: Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: [
        Text(
          "a",
          style: TextStyle(fontSize: 10, fontWeight: FontWeight.w100),
        ),
        Text("b")
      ],
    ),
  ),
  floatingActionButton: FloatingActionButton(
    onPressed: () {},
    child: Icon(Icons.plus_one),
  ),
)
```

### AppBar

```dart
AppBar(
  title: const Text('el titulo'),
)
```

- **Con icono leading:**

```dart
AppBar(
  title: const Text('el titulo'),
  leading: IconButton(
    onPressed: () {},
    icon: Icon(Icons.refresh_rounded),
  ),
)
```

- **Con acciones:**

```dart
AppBar(
  title: const Text('el titulo'),
  actions: [
    IconButton(
      onPressed: () {},
      icon: Icon(Icons.refresh_rounded),
    ),
    IconButton(
      onPressed: () {},
      icon: Icon(Icons.refresh_rounded),
    )
  ],
)
```

### SizedBox

- Un separador, generalmente usado entre elementos en una lista de widgets.

```dart
const SizedBox(
  height: 10,
)
```

### ClipRRect

```dart
ClipRRect(
  borderRadius: BorderRadius.circular(20),
  child: Image.network(
    "https://yesno.wtf/assets/yes/15-3d723ea13af91839a671d4791fc53dcc.gif"
  ),
)
```

### `return const`

- Usar `const` en el `return` de un widget optimiza el código, indicando que el widget no cambiará.

**Ejemplo:**

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: Scaffold(
        body: Center(
          child: Text('hola mundo'),
        ),
      ),
    );
  }
}
```

### TextStyle

- Algunos componentes aceptan el parámetro `style`.

```dart
Text(
  "el texto",
  style: TextStyle(fontSize: 10, fontWeight: FontWeight.w100),
)
```

### ThemeData

```dart
ThemeData(useMaterial3: true, colorSchemeSeed: Colors.red)
```

- **Modo oscuro:**

```dart
ThemeData(
  useMaterial3: true,
  colorSchemeSeed: _ separatelyThemes[selectedColor],
  brightness: Brightness.dark,
)
```

## Estado

### Estado a Nivel de Widget

- Sus hijos pueden acceder a él.

### Estado Global de la Aplicación

- Es un estado al que todos los widgets pueden acceder, sin importar su posición en el árbol de widgets.

### Actualizar Estado

- Para actualizar el estado, se debe llamar a `setState`.

**Ejemplo:**

```dart
onPressed: () {
  setState(() {
    clickCounter++;
  });
}
```

- O simplemente:

```dart
onPressed: () {
  clickCounter++;
  setState(() {});
}
```

- Al llamar a `setState`, Flutter actualiza el widget y sus hijos si es necesario.

### Argumentos `const`

- Se puede usar `const` en argumentos que no cambiarán.

**Ejemplo:**

```dart
Text(
  'Click${clickCounter > 1 ? "s" : ""}',
  style: const TextStyle(fontSize: 25),
)
```

## Bordes

### StadiumBorder

- Bordes circulares.

```dart
const StadiumBorder()
```

**Ejemplo:**

```dart
FloatingActionButton(
  shape: const StadiumBorder(),
  onPressed: () {
    clickCounter++;
    setState(() {});
  },
  child: Icon(Icons.plus_one),
)
```

### BorderRadius

```dart
BorderRadius.circular(20)
```

**Ejemplo:**

```dart
Container(
  decoration: BoxDecoration(
    color: Colors.black,
    borderRadius: BorderRadius.circular(20),
  ),
  child: Padding(
    padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 10),
    child: Text(
      'el texto',
      style: TextStyle(color: Colors.blue),
    ),
  ),
)
```

## VoidCallback

```dart
final VoidCallback? onPressed;

FloatingActionButton(
  onPressed: onPressed,
  child: Icon(icon),
)
```

## Colores

```dart
const Color _customColor = Color.fromARGB(15, 211, 109, 107);

const List<Color> _colorThemes = [
  Color.fromARGB(255, 44, 36, 156),
  Colors.red
];
```

## Padding

```dart
Padding(
  padding: const EdgeInsets.all(8.0),
  child: CircleAvatar(),
)
```

```dart
Padding(
  padding: const EdgeInsets.symmetric(horizontal: 10),
  child: Column(
    children: [
      Expanded(
        child: Container(
          color: Colors.red,
        ),
      ),
      Text("mundo")
    ],
  ),
)
```

## CircleAvatar

```dart
CircleAvatar(
  backgroundImage: NetworkImage("la direccion en internet"),
)
```

```dart
CircleAvatar(
  backgroundColor: Colors.red,
)
```

## ListView

```dart
Expanded(
  child: ListView.builder(
    itemBuilder: (context, index) {
      return Text("$index");
    },
  ),
)
```

- **Con cantidad límite:**

```dart
ListView.builder(
  itemCount: 100,
  itemBuilder: (context, index) {
    return Text("$index");
  },
)
```

### Controlador

- Se usa para controlar el desplazamiento, por ejemplo, para mover la lista al final después de agregar un nuevo elemento.

**Ejemplo:**

```dart
class ChatProvider extends ChangeNotifier {
  final chatScrollController = ScrollController();
  List<Message> messageList = [
    Message(text: "Hola", fromWho: FromWho.me),
    Message(text: "dime", fromWho: FromWho.her)
  ];

  Future<void> sendMessage(String text) async {
    final newMessage = Message(text: text, fromWho: FromWho.me);
    messageList.add(newMessage);
    notifyListeners();
    moveToScrollToBottom();
  }

  void moveToScrollToBottom() {
    chatScrollController.animateTo(
      chatScrollController.position.maxScrollExtent,
      duration: const Duration(seconds: 300),
      curve: Curves.easeOut,
    );
  }
}
```

```dart
@override
Widget build(BuildContext context) {
  final chatProvider = context.watch<ChatProvider>();
  return ListView.builder(
    controller: chatProvider.chatScrollController,
    itemCount: chatProvider.messageList.length,
    itemBuilder: (context, index) {
      final message = chatProvider.messageList[index];
      return message.fromWho == FromWho.her
          ? const HerMessageBubble()
          : MyMessageBubble(message: message);
    },
  );
}
```

## Container

- Similar a un `div` en HTML.
- Se puede decorar con `BoxDecoration`.

**Ejemplo:**

```dart
Container(
  decoration: BoxDecoration(
    color: Colors.black,
    borderRadius: BorderRadius.circular(20),
  ),
  child: Padding(
    padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 10),
    child: Text(
      'el texto',
      style: TextStyle(color: Colors.blue),
    ),
  ),
)
```

## Esquema de Colores

- Para obtener los colores del tema principal.

**Ejemplo:**

```dart
class MyMessageBubble extends StatelessWidget {
  const MyMessageBubble({super.key});

  @override
  Widget build(BuildContext context) {
    final colors = Theme.of(context).colorScheme;
    return Column(
      crossAxisAlignment: CrossAxisAlignment.end,
      children: [
        Container(
          decoration: BoxDecoration(
            color: colors.primary,
            borderRadius: BorderRadius.circular(20),
          ),
          child: Padding(
            padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 10),
            child: Text(
              'el texto',
              style: TextStyle(color: Colors.blue),
            ),
          ),
        ),
        SizedBox(
          height: 10,
        )
      ],
    );
  }
}
```

## Imágenes

```dart
Image.network(
  "https://yesno.wtf/assets/yes/15-3d723ea13af91839a671d4791fc53dcc.gif",
)
```

- **Con ajuste de tamaño:**

```dart
Image.network(
  "https://yesno.wtf/assets/yes/15-3d723ea13af91839a671d4791fc53dcc.gif",
  width: size.width * 0.7,
  height: 150,
  fit: BoxFit.cover,
)
```

## MediaQuery

- Para obtener propiedades del dispositivo, como el tamaño.

**Ejemplo:**

```dart
final size = MediaQuery.of(context).size;
```

## Entrada de Texto

### TextFormField

```dart
TextFormField();
```

- **Con decoración:**

```dart
TextFormField(
  decoration: InputDecoration(
    enabledBorder: OutlineInputBorder(
      borderSide: BorderSide(color: colors.primary),
      borderRadius: BorderRadius.circular(20),
    ),
    filled: true,
    suffixIcon: IconButton(
      icon: const Icon(Icons.send_outlined),
      onPressed: () {},
    ),
  ),
)
```

### TextEditingController

- Controla el texto del campo de entrada.

**Ejemplo:**

```dart
final textController = TextEditingController();

TextFormField(
  controller: textController,
  decoration: inputDecoration,
  onFieldSubmitted: (value) {
    textController.clear();
  },
)
```

- **Limpiar el texto:**

```dart
textController.clear();
```

- **Obtener el texto:**

```dart
final textValue = textController.value.text;
```

### FocusNode

- Gestiona el foco del campo de texto.

**Ejemplo:**

```dart
final focusNode = FocusNode();

TextFormField(
  onTapOutside: (event) {
    focusNode.unfocus();
  },
  focusNode: focusNode,
  controller: textController,
  decoration: inputDecoration,
  onFieldSubmitted: (value) {
    textController.clear();
    focusNode.requestFocus();
  },
)
```

- **Solicitar foco:**

```dart
focusNode.requestFocus();
```

- **Perder foco:**

```dart
focusNode.unfocus();
```

## Gestores de Estado

Manejar el estado de la aplicación es crucial. A continuación, se describen algunos de los gestores de estado más populares en Flutter.

### Provider

- Recomendado por Flutter, robusto y fácil de usar.
- Se debe crear una clase que extienda de `ChangeNotifier`.

**Instalación:**

```bash
flutter pub add provider
```

**Uso:**

- En el widget padre:

```dart
MultiProvider(
  providers: [ChangeNotifierProvider(create: (_) => ChatProvider())],
  child: MaterialApp(
    debugShowCheckedModeBanner: false,
    title: 'Material App',
    home: const ChatScreen(),
  ),
)
```

- En un widget hijo:

```dart
import 'package:provider/provider.dart';

@override
Widget build(BuildContext context) {
  final chatProvider = context.watch<ChatProvider>();
  // ...
}
```

- **Notificar cambios:**

```dart
class ChatProvider extends ChangeNotifier {
  List<Message> messageList = [];

  void sendMessage(String text) {
    final newMessage = Message(text: text, fromWho: FromWho.me);
    messageList.add(newMessage);
    notifyListeners();
  }
}
```

### Otros Gestores de Estado

- **Riverpod:** Fácilmente testeable.
- **InheritedWidget & InheritedModel:** Solución nativa de Flutter.
- **BLoC / RX y Flutter BLoC + Cubits:** Basado en Streams y Observables.
- **Get_it:** Útil para acceder a servicios sin `BuildContext`.
- **MobX:** Popular y basado en observables.
- **GetX:** Muy popular, pero oculta conceptos importantes para principiantes.

**Consejo:** Prueba varios gestores de estado y elige el que mejor se adapte a tus necesidades.

## Dio

- Para comunicación HTTP.

**Instalación:**

```bash
flutter pub add dio
```

# Reutiliar codigo y cambiar nombre

Para que tu nueva aplicación Flutter se reconozca como una app **completamente independiente** (y no como una actualización de la anterior), **debes modificar los identificadores únicos de cada plataforma**. Estos identificadores son los que el sistema operativo usa para diferenciar apps. Aquí te explico paso a paso qué debes cambiar:

---

### 🔑 **1. Cambia el identificador de Android (Package Name)**

El **Package Name** es el identificador único de tu app en Android. Si es igual al de la app anterior, el sistema la tratará como una actualización.

#### Pasos:

1. **Abre el archivo `android/app/build.gradle`**  
   Busca la sección `defaultConfig` y modifica el `applicationId` (debe ser único, ej: `com.tuempresa.nuevaapp`):
   
   ```gradle
   defaultConfig {
       applicationId "com.tuempresa.nuevaapp" // 👈 ¡Cámbialo aquí!
       minSdkVersion 21
       targetSdkVersion 34
       // ...
   }
   ```

2. **Actualiza el `AndroidManifest.xml`**  
   En `android/app/src/main/AndroidManifest.xml`, verifica que el atributo `package` coincida con el nuevo `applicationId`:
   
   ```xml
   <manifest xmlns:android="http://schemas.android.com/apk/res/android"
       package="com.tuempresa.nuevaapp"> <!-- ¡Mismo valor que applicationId! -->
   ```

3. **Renombra las carpetas del código Java/Kotlin** (¡Importante!)  
   Si tu estructura original era:
   
   ```
   android/app/src/main/java/com/tuempresa/app/
   ```
   
   Debes renombrar las carpetas para que coincidan con el nuevo `package`:
   
   ```
   android/app/src/main/java/com/tuempresa/nuevaapp/
   ```
   
   - **En Android Studio:** Haz clic derecho en la carpeta `java > com.tuempresa.app` → **Refactor > Rename** → Confirma los cambios.
   
   - **Manualmente:** Renombra las carpetas y actualiza el paquete en `MainActivity.kt`/`MainActivity.java`:
     
     ```kotlin
     package com.tuempresa.nuevaapp // 👈 Asegúrate de que coincida
     ```

---

### 🍎 **2. Cambia el identificador de iOS (Bundle ID)**

En iOS, el **Bundle ID** es el identificador único. Si es igual al de la app anterior, el sistema lo reconocerá como la misma app.

#### Pasos:

1. **Abre el proyecto en Xcode**  
   
   - Ejecuta `open ios/Runner.xcworkspace` desde la terminal.
   - En el panel izquierdo, selecciona **Runner** (en la sección *PROJECT*) → Pestaña **General**.

2. **Cambia el Bundle Identifier**  
   
   - En **Identity > Bundle Identifier**, usa un valor único (ej: `com.tuempresa.nuevaapp`):
     ![Xcode Bundle Identifier](https://i.imgur.com/5lZJ8yP.png)

3. **Verifica el `Info.plist`**  
   Asegúrate de que `CFBundleIdentifier` en `ios/Runner/Info.plist` coincida:
   
   ```xml
   <key>CFBundleIdentifier</key>
   <string>com.tuempresa.nuevaapp</string> <!-- ¡Mismo valor que en Xcode! -->
   ```

---

### 🧹 **3. Limpia el proyecto y reconstruye**

Después de los cambios, **elimina archivos de caché** para evitar conflictos:

```bash
flutter clean
flutter pub get
```

Luego genera una nueva versión de la app:

```bash
flutter build apk --release    # Para Android
flutter build ipa --release    # Para iOS
```

---

### ⚠️ **Notas clave**

- **Nunca uses el mismo Package Name ni Bundle ID** que la app anterior. Deben ser **únicos** (ej: `com.tuempresa.appv2`).
- Si usas **Firebase**, crea un nuevo proyecto en la [consola de Firebase](https://console.firebase.google.com/) y reemplaza los archivos de configuración (`google-services.json` y `GoogleService-Info.plist`).
- El nombre visible de la app (el que aparece debajo del ícono) se cambia en `pubspec.yaml` con el campo `name`, pero **esto no afecta el identificador único**.

---

### ✅ ¿Cómo verificar que funcionó?

1. Instala la nueva app en tu dispositivo.
2. Si ves **dos apps distintas** (una antigua y otra nueva), ¡listo!  
   Si aún se sobrescribe, revisa:
   - Que el `applicationId` en `build.gradle` y el `Bundle Identifier` en Xcode sean **únicos**.
   - Que hayas renombrado las carpetas de Java/Kotlin en Android.

¡Con esto tu nueva app será completamente independiente! 🚀

# Crear App

```bash
flutter build apk --release
```