# OpenCV mit C++ und VS Code auf dem Mac

## Ziel
Diese Anleitung beschreibt, wie man auf einem macOS-System OpenCV mit C++ in Visual Studio Code einrichtet, um beispielsweise ein Webcam-Bild live anzuzeigen. Die Schritte beinhalten Installation, Projektstruktur, Code, Build-Konfiguration und Ausführung.

---

## Voraussetzungen

- macOS (Intel oder Apple Silicon)
- Internetzugang
- Terminal-Grundkenntnisse

---

## Schritt 1: Visual Studio Code installieren

1. Gehe zu https://code.visualstudio.com/ und lade die macOS-Version herunter.
2. Entpacke das Archiv und verschiebe "Visual Studio Code.app" in den Programmordner.
3. Öffne VS Code und drücke `Cmd + Shift + P`, tippe `shell command`, und wähle **"Install 'code' command in PATH"**.
   - Damit kannst du später VS Code direkt über das Terminal mit `code .` öffnen.

---

## Schritt 2: Xcode Command Line Tools installieren

Im Terminal:
```bash
xcode-select --install
```
Damit wird der Compiler `clang++` und weitere Build-Werkzeuge installiert.

---

## Schritt 3: Homebrew installieren

Im Terminal:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
Dann prüfen:
```bash
brew --version
```

---

## Schritt 4: OpenCV und CMake installieren

Im Terminal:
```bash
brew install opencv cmake
```

OpenCV wird installiert unter:
- Header: `/usr/local/opt/opencv/include/opencv4`
- Libraries: `/usr/local/opt/opencv/lib`

---

## Schritt 5: Projektstruktur anlegen

```bash
mkdir ~/opencv-webcam
cd ~/opencv-webcam
mkdir -p .vscode
touch main.cpp CMakeLists.txt .vscode/tasks.json
```

---

## Schritt 6: C++ Code (main.cpp)

```cpp
#include <opencv2/opencv.hpp>
#include <iostream>

int main() {
    cv::VideoCapture cap(0);
    if (!cap.isOpened()) {
        std::cerr << "Webcam konnte nicht geöffnet werden." << std::endl;
        return -1;
    }

    cv::Mat frame;
    while (true) {
        cap >> frame;
        if (frame.empty()) break;

        cv::imshow("Webcam", frame);
        if (cv::waitKey(10) == 27) break;
    }

    cap.release();
    cv::destroyAllWindows();
    return 0;
}
```

---

## Schritt 7: CMakeLists.txt erstellen

```cmake
cmake_minimum_required(VERSION 3.10)
project(OpenCVWebcam)

set(CMAKE_CXX_STANDARD 17)

include_directories(/usr/local/opt/opencv/include/opencv4)
link_directories(/usr/local/opt/opencv/lib)

add_executable(main main.cpp)

target_link_libraries(main
    opencv_core
    opencv_highgui
    opencv_videoio
    opencv_imgproc
)
```

---

## Schritt 8: Build-Task in VS Code konfigurieren

In `.vscode/tasks.json`:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Build OpenCV Webcam",
      "type": "shell",
      "command": "/usr/bin/clang++",
      "args": [
        "-std=c++17",
        "-I/usr/local/opt/opencv/include/opencv4",
        "-L/usr/local/opt/opencv/lib",
        "-lopencv_core",
        "-lopencv_highgui",
        "-lopencv_videoio",
        "-lopencv_imgproc",
        "main.cpp",
        "-o",
        "main"
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "problemMatcher": ["$gcc"],
      "detail": "Kompiliert main.cpp mit OpenCV"
    }
  ]
}
```

---

## Schritt 9: Kompilieren und ausführen

1. Drücke `Cmd + Shift + B` in VS Code
2. Wähle den Task "Build OpenCV Webcam"
3. Im Terminal (wenn `main` kompiliert wurde):

```bash
./main
```

Ein Fenster mit dem Webcam-Bild sollte erscheinen. Mit `ESC` kannst du es schließen.

---

## Hinweise
- Stelle sicher, dass deine Webcam vom System freigegeben ist (Systemeinstellungen > Sicherheit).
- Wenn du Apple Silicon (M1/M2/M3) nutzt und Probleme auftreten, prüfe ggf. die Brew-Architektur (`/opt/homebrew` statt `/usr/local`).
