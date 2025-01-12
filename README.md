# rust_to_android_guide
 
 
 # Uso de una librería de Rust como librería C en Android

## Español

### Índice
1. Requisitos previos
2. Configurar tu proyecto Rust
3. Escribir funciones compatibles con C
4. Generar la cabecera C (.h)
5. Compilar la librería para Android
6. Integrar la librería en Android
7. Invocar las funciones desde Java/Kotlin

### Requisitos previos
1. **Instalar Rust**: Asegúrate de tener instalado Rust. Puedes instalarlo desde [rustup.rs](https://rustup.rs/).
2. **Instalar el NDK de Android**: Necesitarás el Android NDK para compilar código para Android. Descárgalo desde el [SDK Manager](https://developer.android.com/studio).
3. **Instalar herramientas adicionales**:
   - Cargo NDK: `cargo install cargo-ndk`
   - Bindgen o Cbindgen: `cargo install cbindgen`

### Pasos

1. **Configurar tu proyecto Rust**
   - Crea un nuevo proyecto Rust:
     ```bash
     cargo new --lib mi_libreria
     cd mi_libreria
     ```
   - Modifica el archivo `Cargo.toml` para establecer las opciones necesarias:
     ```toml
     [lib]
     crate-type = ["cdylib"]
     ```

2. **Escribir funciones compatibles con C**
   - Usa `#[no_mangle]` y la convención de llamadas `extern "C"` para las funciones que serán exportadas:
     ```rust
     #[no_mangle]
     pub extern "C" fn suma(a: i32, b: i32) -> i32 {
         a + b
     }
     ```

3. **Generar la cabecera C (.h)**
   - Configura un archivo `cbindgen.toml` en la raíz del proyecto:
     ```toml
     language = "C"
     output = "include/mi_libreria.h"
     ```
   - Ejecuta `cbindgen` para generar la cabecera:
     ```bash
     cbindgen . -o include/mi_libreria.h
     ```

4. **Compilar la librería para Android**
   - Usa `cargo-ndk` para compilar la librería:
     ```bash
     cargo ndk -t armeabi-v7a -t arm64-v8a -o ./target/android build --release
     ```
   - Los binarios generados estarán en `target/android/release`.

5. **Integrar la librería en Android**
   - Copia los archivos `.so` y la cabecera `.h` en tu proyecto Android.
   - Configura tu archivo `CMakeLists.txt` para incluir la librería:
     ```cmake
     cmake_minimum_required(VERSION 3.10)

     add_library(mi_libreria SHARED IMPORTED)
     set_target_properties(mi_libreria PROPERTIES IMPORTED_LOCATION
         ${CMAKE_SOURCE_DIR}/libs/${ANDROID_ABI}/libmi_libreria.so)

     target_link_libraries(
         # Otros enlaces
         mi_libreria
     )
     ```
   - Modifica tu archivo `build.gradle` para asegurar la compatibilidad:
     ```gradle
     android {
         externalNativeBuild {
             cmake {
                 path "src/main/cpp/CMakeLists.txt"
             }
         }
     }
     ```

6. **Invocar las funciones desde Java/Kotlin**
   - Usa `System.loadLibrary("mi_libreria")` para cargar la librería:
     ```kotlin
     class MiLibreria {
         init {
             System.loadLibrary("mi_libreria")
         }

         external fun suma(a: Int, b: Int): Int
     }
     ```

---

# Using a Rust Library as a C Library on Android

## English

### Index
1. Prerequisites
2. Set up your Rust project
3. Write C-compatible functions
4. Generate the C header (.h)
5. Compile the library for Android
6. Integrate the library into Android
7. Invoke functions from Java/Kotlin

### Prerequisites
1. **Install Rust**: Make sure Rust is installed. You can get it from [rustup.rs](https://rustup.rs/).
2. **Install Android NDK**: You'll need the Android NDK to compile code for Android. Download it from the [SDK Manager](https://developer.android.com/studio).
3. **Install additional tools**:
   - Cargo NDK: `cargo install cargo-ndk`
   - Bindgen or Cbindgen: `cargo install cbindgen`

### Steps

1. **Set up your Rust project**
   - Create a new Rust project:
     ```bash
     cargo new --lib my_library
     cd my_library
     ```
   - Update the `Cargo.toml` file to set the necessary options:
     ```toml
     [lib]
     crate-type = ["cdylib"]
     ```

2. **Write C-compatible functions**
   - Use `#[no_mangle]` and the `extern "C"` calling convention for exported functions:
     ```rust
     #[no_mangle]
     pub extern "C" fn add(a: i32, b: i32) -> i32 {
         a + b
     }
     ```

3. **Generate the C header (.h)**
   - Configure a `cbindgen.toml` file in the project's root:
     ```toml
     language = "C"
     output = "include/my_library.h"
     ```
   - Run `cbindgen` to generate the header:
     ```bash
     cbindgen . -o include/my_library.h
     ```

4. **Compile the library for Android**
   - Use `cargo-ndk` to compile the library:
     ```bash
     cargo ndk -t armeabi-v7a -t arm64-v8a -o ./target/android build --release
     ```
   - The generated binaries will be in `target/android/release`.

5. **Integrate the library into Android**
   - Copy the `.so` files and the `.h` header into your Android project.
   - Configure your `CMakeLists.txt` file to include the library:
     ```cmake
     cmake_minimum_required(VERSION 3.10)

     add_library(my_library SHARED IMPORTED)
     set_target_properties(my_library PROPERTIES IMPORTED_LOCATION
         ${CMAKE_SOURCE_DIR}/libs/${ANDROID_ABI}/libmy_library.so)

     target_link_libraries(
         # Other links
         my_library
     )
     ```
   - Update your `build.gradle` file to ensure compatibility:
     ```gradle
     android {
         externalNativeBuild {
             cmake {
                 path "src/main/cpp/CMakeLists.txt"
             }
         }
     }
     ```

6. **Invoke functions from Java/Kotlin**
   - Use `System.loadLibrary("my_library")` to load the library:
     ```kotlin
     class MyLibrary {
         init {
             System.loadLibrary("my_library")
         }

         external fun add(a: Int, b: Int): Int
     }
     ```

