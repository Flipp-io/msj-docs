# PortAudio mit C++ und VS Code auf dem Mac

## Ziel
Diese Anleitung beschreibt, wie man auf einem macOS-System PortAudio mit C++ in Visual Studio Code einrichtet, um beispielsweise die Mikrofoneingabe in Echtzeit auszulesen. Die Schritte beinhalten Installation, Projektstruktur, Code, Build-Konfiguration und Ausführung.

---

## Voraussetzungen

- macOS (Intel oder Apple Silicon)
- Visual Studio Code installiert (siehe OpenCV-Anleitung)
- Homebrew installiert (siehe OpenCV-Anleitung)
- Terminal-Grundkenntnisse

---

## Schritt 1: PortAudio installieren

Im Terminal:
```bash
brew install portaudio
```

PortAudio wird installiert unter:
- Header: `/usr/local/include/portaudio.h`
- Library: `/usr/local/lib/libportaudio.dylib`

---

## Schritt 2: Projektstruktur anlegen

```bash
mkdir ~/portaudio-test
cd ~/portaudio-test
mkdir -p .vscode
touch main.cpp CMakeLists.txt .vscode/tasks.json
```

---

## Schritt 3: C++ Code (main.cpp)

```cpp
#include <portaudio.h>
#include <iostream>
#include <cmath>

float latestVolume = 0.0f;

static int audioCallback(const void* input, void*, unsigned long frameCount,
                         const PaStreamCallbackTimeInfo*, PaStreamCallbackFlags, void*) {
    const float* in = static_cast<const float*>(input);
    float sum = 0;
    for (unsigned long i = 0; i < frameCount; ++i) {
        sum += std::fabs(in[i]);
    }
    latestVolume = sum / frameCount;
    return paContinue;
}

int main() {
    Pa_Initialize();

    PaStream* stream;
    Pa_OpenDefaultStream(&stream, 1, 0, paFloat32, 44100, 256, audioCallback, nullptr);
    Pa_StartStream(stream);

    std::cout << "Mikrofon läuft... Drücke Enter zum Beenden." << std::endl;
    std::cin.get();

    Pa_StopStream(stream);
    Pa_CloseStream(stream);
    Pa_Terminate();

    return 0;
}
```

Dieser Code startet eine Audioaufnahme vom Standardmikrofon und berechnet eine einfache Lautstärkeanzeige über den Mittelwert der Absolutwerte der Samples.

---

## Schritt 4: CMakeLists.txt erstellen

**Zweck:** Diese Datei konfiguriert den CMake-Build für das Projekt. Sie definiert:
- Projektname
- C++-Standard
- Einbindung von PortAudio-Headern
- Verlinkung der PortAudio-Bibliothek

**Inhalt:**

```cmake
cmake_minimum_required(VERSION 3.10)
project(PortAudioTest)

set(CMAKE_CXX_STANDARD 17)

include_directories(/usr/local/include)
link_directories(/usr/local/lib)

add_executable(main main.cpp)

target_link_libraries(main portaudio)
```

---

## Schritt 5: Build-Task in VS Code konfigurieren

**Zweck:** Der Task in `.vscode/tasks.json` steuert, wie der Quellcode kompiliert wird. Hier wird angegeben:
- verwendeter Compiler (`clang++`)
- Include- und Library-Pfade
- zu linkende Bibliothek (PortAudio)

**Inhalt:**

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Build PortAudio Test",
      "type": "shell",
      "command": "/usr/bin/clang++",
      "args": [
        "-std=c++17",
        "-I/usr/local/include",
        "-L/usr/local/lib",
        "-lportaudio",
        "main.cpp",
        "-o",
        "main"
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "problemMatcher": ["$gcc"],
      "detail": "Kompiliert main.cpp mit PortAudio"
    }
  ]
}
```

---

## Schritt 6: Kompilieren und ausführen

1. Drücke `Cmd + Shift + B` in VS Code.
2. Wähle den Task "Build PortAudio Test".
3. Im Terminal (wenn `main` kompiliert wurde):

```bash
./main
```

Es wird die aktuelle Lautstärke deines Mikrofons gemessen und im Terminal ausgegeben. Beenden kannst du die Anwendung, indem du Enter drückst.

---

## Hinweise
- Achte darauf, dass VS Code und Terminal Zugriff auf dein Mikrofon haben (Systemeinstellungen > Sicherheit > Mikrofon).
- Bei Apple Silicon-Macs kann der Pfad zu Homebrew-Installationen abweichen (`/opt/homebrew/` statt `/usr/local/`).

---

