<p align="center">
  <a href="https://t.me/lunaraim2">
    <img src="assets/brand/lunar-aim-banner.png" alt="LunarAim 2.2 — новый интерфейс" width="760">
  </a>
</p>

<p align="center">
  <a href="https://t.me/in_the_skyyy"><img src="https://img.shields.io/badge/%40in__the__skyyy-2AABEE?style=for-the-badge&logo=telegram&logoColor=white" alt="Связаться с @in_the_skyyy"></a>
  <a href="https://t.me/lunaraim"><img src="https://img.shields.io/badge/%40lunaraim-2548BB?style=for-the-badge&logo=telegram&logoColor=white" alt="Канал LunarAim @lunaraim"></a>
  <a href="https://t.me/lunaraim2"><img src="https://img.shields.io/badge/%40lunaraim2-17C3E1?style=for-the-badge&logo=telegram&logoColor=white" alt="Канал LunarAim @lunaraim2"></a>
</p>

<p align="center"><a href="README.md">English</a> · <a href="README.ru-RU.md">Русский</a> · <a href="README.zh-CN.md">简体中文</a></p>

<h3 align="center">Создано с помощью</h3>

<p align="center">
  <a href="https://developer.microsoft.com/en-us/microsoft-edge/webview2/"><img src="https://img.shields.io/badge/WebView2-107C10?style=for-the-badge&logo=microsoftedge&logoColor=white" alt="Microsoft WebView2"></a>
  <a href="https://learn.microsoft.com/en-us/windows/win32/"><img src="https://img.shields.io/badge/Win32%20API-0078D4?style=for-the-badge&logo=windows&logoColor=white" alt="Win32 API"></a>
  <a href="https://cmake.org/"><img src="https://img.shields.io/badge/CMake-064F8C?style=for-the-badge&logo=cmake&logoColor=white" alt="CMake"></a>
  <a href="https://visualstudio.microsoft.com/vs/features/cplusplus/"><img src="https://img.shields.io/badge/C%2B%2B%20%2F%20MSVC-5C2D91?style=for-the-badge&logo=visualstudio&logoColor=white" alt="C++ и MSVC"></a>
  <a href="https://developer.nvidia.com/cuda-toolkit"><img src="https://img.shields.io/badge/CUDA-76B900?style=for-the-badge&logo=nvidia&logoColor=white" alt="NVIDIA CUDA"></a>
  <a href="https://developer.nvidia.com/tensorrt"><img src="https://img.shields.io/badge/TensorRT-76B900?style=for-the-badge&logo=nvidia&logoColor=white" alt="NVIDIA TensorRT"></a>
  <a href="https://pytorch.org/"><img src="https://img.shields.io/badge/PyTorch-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white" alt="PyTorch"></a>
  <a href="https://github.com/ultralytics/ultralytics"><img src="https://img.shields.io/badge/Ultralytics%20YOLO-111F68?style=for-the-badge&logoColor=white" alt="Ultralytics YOLO"></a>
</p>

<details>
<summary>Содержание</summary>
<br>

- [Что показывает репозиторий](#что-показывает-репозиторий)
- [Архитектура](#архитектура)
  - [Границы runtime](#границы-runtime)
  - [Что нового](#что-нового)
- [Галерея интерфейса](#галерея-интерфейса)
  - [Дашборд runtime](#дашборд-runtime)
  - [Настройка детектора](#настройка-детектора)
  - [Инструменты runtime](#инструменты-runtime)
- [Эксперимент с детекцией](#эксперимент-с-детекцией)
- [Публичная и закрытая часть](#публичная-и-закрытая-часть)
- [Материалы](#материалы)

</details>

> **Публичный обзор** Lunar V2: границы runtime, интеграция frontend, визуальный язык и эксперимент с компьютерным зрением. Production-приложение и его реализация остаются закрытыми.

## Что показывает репозиторий

LunarAim 2.2 — нативное Windows-приложение с компактным наблюдаемым runtime-ядром. Интерфейс работает как клиент backend через структурированный bridge, а платформозависимый ввод и GPU-функциональность скрыты за явными границами.

- **CoreRuntime** управляет запуском, выполнением, остановкой, конфигурацией, snapshot-состояниями и telemetry.
- **Frontend Bridge** предоставляет встроенному UI структурированный native API.
- **Компонуемые контракты** позволяют независимо заменять aim, detection, trigger и mouse output.
- **Изолированные платформенные модули** разделяют Win32 Raw Input и CUDA / TensorRT runtime.
- **Инструменты оператора** показывают состояние runtime, производительность, конфигурацию, фильтры, binds и логи.

## Архитектура

```mermaid
flowchart TB
    frontend["Embedded frontend"]
    bridge["FrontendHost / FrontendBridge"]
    dispatcher["Native API dispatcher"]
    runtime["CoreRuntime<br/>starting → running → stopping<br/>error / stopped"]
    aim["AimController"]
    trigger["Trigger"]
    detector["Detector"]
    capture["Capture"]
    mouse["Mouse"]
    detectorContract["IDetector contract"]
    colorbot["Colorbot<br/>(OpenCV)"]
    cuda["CUDA / TensorRT module<br/>loaded separately"]
    rawInput["Win32 Raw Input<br/>separate thread"]
    triggerContract["ITrigger"]
    triggerV1["TriggerV1"]
    triggerV2["TriggerV2"]
    mouseContract["IMouse"]
    winapi["WinAPI"]
    lunarBox["LunarBox V1 / V2"]
    vdriver["Virtual Driver<br/>(VDriver)"]
    desktopDuplicator["DesktopDuplicator<br/>(DirectX)"]

    frontend --> bridge --> dispatcher --> runtime
    runtime --> aim
    runtime --> trigger
    runtime --> detector
    runtime --> capture
    runtime --> mouse
    detector --> detectorContract
    detectorContract --> colorbot
    detectorContract --> cuda
    trigger --> triggerContract
    triggerContract --> triggerV1
    triggerContract --> triggerV2
    mouse --> mouseContract
    mouseContract --> winapi
    mouseContract --> lunarBox
    mouseContract --> vdriver
    capture --> rawInput
    capture --> desktopDuplicator

    classDef core fill:#2548BB,color:#ffffff,stroke:#17C3E1,stroke-width:2px
    classDef boundary fill:#0f2f35,color:#ffffff,stroke:#17C3E1
    classDef module fill:#17243d,color:#ffffff,stroke:#2548BB
    class runtime core
    class bridge,dispatcher,detectorContract,triggerContract,mouseContract boundary
    class colorbot,cuda,rawInput,triggerV1,triggerV2,winapi,lunarBox,vdriver,desktopDuplicator module
```

### Границы runtime

Runtime разделён на явные границы, чтобы каждая часть могла развиваться независимо от деталей реализации верхних уровней.

- **`CoreRuntime`.** Владеет жизненным циклом, worker thread, путём к конфигурации, snapshot-состояниями и telemetry производительности.
- **`FrontendBridge`.** Преобразует UI-запросы в стабильный native API через dispatcher и handlers.
- **`IDetector`.** Отделяет aim-логику от реализации детектора, включая Color и CUDA-пути.
- **`ITrigger`.** Отделяет orchestration runtime от механизма срабатывания.
- **`IMouse`.** Отделяет прикладную логику от способа отправки mouse-команд.
- **Raw Input module.** Выполняет сбор Win32-ввода в отдельном потоке и через собственную границу состояния.
- **CUDA runtime module.** Выносит зависимости CUDA / TensorRT из основного executable в отдельно загружаемый GPU-модуль.

### Что нового

LunarAim 2.2 собирает основные runtime-задачи в единый наблюдаемый поток. Встроенный frontend общается через структурированный bridge, а не обращается к внутренним деталям напрямую; `CoreRuntime` задаёт единый жизненный цикл запуска, работы, остановки и обработки ошибок.

Контракты runtime упрощают расширение системы: aim control, detection, triggering и mouse output можно заменять независимо. Платформенная логика также изолирована: Win32 Raw Input работает через собственную границу состояния, а CUDA / TensorRT загружается как отдельный GPU-модуль.

Конфигурация, snapshot runtime, telemetry производительности и логи стали самостоятельными частями operator experience. UI показывает текущее состояние системы, оставляя конкретные реализации за ясными интерфейсами.

## Галерея интерфейса

UI задуман как операторская панель runtime с отдельными представлениями состояния, детекции, поведения ввода и диагностики.

### Дашборд runtime

Дашборд сразу показывает состояние runtime, активную конфигурацию, frame timing, FPS и telemetry.

<p align="center">
  <img src="assets/screenshots/dashboard-running-dark.png" alt="Тёмный дашборд LunarAim" width="49%">
  <img src="assets/screenshots/dashboard-running-light.png" alt="Светлый дашборд LunarAim" width="49%">
</p>

### Настройка детектора

<p align="center">
  <img src="assets/screenshots/colorbot-dark.png" alt="Настройка Colorbot — тёмная тема" width="49%">
  <img src="assets/screenshots/colorbot-light.png" alt="Настройка Colorbot — светлая тема" width="49%">
</p>

### Инструменты runtime

<p align="center">
  <img src="assets/screenshots/binds-light.png" alt="Настройка binds — светлая тема" width="49%">
  <img src="assets/screenshots/logs-dark.png" alt="Логи ядра — тёмная тема" width="49%">
</p>

<details>
<summary>Дополнительные экраны интерфейса</summary>
<br>

<p align="center">
  <img src="assets/screenshots/colorbot-dark-detail.png" alt="Детали настройки Colorbot" width="49%">
  <img src="assets/screenshots/filters-dark.png" alt="Редактор фильтров" width="49%">
  <img src="assets/screenshots/dashboard-stopped-dark.png" alt="Дашборд остановленного runtime" width="49%">
  <img src="assets/screenshots/runtime-terminal.png" alt="Терминальный вывод runtime" width="49%">
</p>
</details>

## Эксперимент с детекцией

В репозитории сохранён небольшой публичный snapshot эксперимента с детектором для CS2:

- размечено 127 игровых демок;
- модель обучалась 150 эпох;
- сохранены графики обучения, статистика разметки и пространственные распределения;
- checkpoint `yolo11n_cs2.pt` добавлен как артефакт эксперимента.

<p align="center">
  <img src="experiments/yolo-cs2/results.png" alt="Графики обучения и валидации YOLO" width="80%">
</p>

<p align="center">
  <img src="experiments/yolo-cs2/labels.jpg" alt="Распределение меток датасета" width="42%">
  <img src="experiments/yolo-cs2/labels_correlogram.jpg" alt="Коррелограмма меток датасета" width="42%">
</p>

## Публичная и закрытая часть

Это **showcase-репозиторий**, а не дистрибутив исходников Lunar V2. Здесь опубликованы архитектурные материалы, design documentation, снимки интерфейса и отдельные ML-артефакты. Production-код, приватные интеграции, credentials и proprietary infrastructure намеренно исключены.

<details>
<summary>Структура репозитория</summary>
<br>

```text
.
├── assets/
│   ├── brand/          # логотипы и обложка презентации
│   └── screenshots/    # именованные снимки UI
├── docs/               # презентация и публичные заметки
├── experiments/
│   └── yolo-cs2/       # метрики, графики и checkpoint детектора
└── README.md
```

</details>

## Материалы

- [Презентация об архитектуре](docs/Lunar.pdf)
- [Описание проекта](docs/project-description.md)
- [Скриншоты интерфейса](assets/screenshots/)
- [Эксперимент с CS2-детектором](experiments/yolo-cs2/)

<br>

<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/brand/LunAimW.svg">
    <source media="(prefers-color-scheme: light)" srcset="assets/brand/LunAimB.svg">
    <img src="assets/brand/LunAimB.svg" alt="Логотип LunarAim" width="96">
  </picture>
  <h2>LunarAim 2.2</h2>
  <sub>Архитектура native runtime · структурированный UI bridge · GPU-детекция · 2026</sub>
</div>
