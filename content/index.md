---
title: Welcome to Quartz
---

## Orbital Supercruise System (MOC)
Добро пожаловать в архитектурную документацию 3D орбитальной системы для SS13 (TGstation build).

## Архитектура системы
graph TD    SSsupercruise <|-- Star_System    Star_System <|-- Orbital_Object    Orbital_Object <|-- Shuttle    Orbital_Object <|-- Planet    Orbital_Object <|-- Station    Orbital_Object <|-- Star        Shuttle <--> Flight_Console    Flight_Console <--> TGUI_UI    TGUI_UI <--> Canvas_Renderer
## Основные модули
1. Математика и Физика
- [[Orbital Vector]] — Базовый класс 3D вектора.
- [[Orbital Object]] — Базовый объект космоса (позиция, скорость, гравитация).
- [[Planet Physics]] — Орбитальная механика планет (аналитическая скорость).
2. Управление кораблем
- [[Orbital Shuttle]] — Главный датум шаттла.
- [[Flight Control Systems]] — SAS, RCS и ручное управление.
- [[Autopilot AI]] — Векторный автопилот (Travel, Orbit, Hold).
3. Интерфейсы (TGUI / React)
- [[Console Architecture]] — Разделение на Helm, Nav, Tactical.
- [[Canvas 3D Renderer]] — Проекция 3D в 2D (Painter's algorithm, Камера).
- [[TGUI Data Flow]] — Как данные полетят из DM в React.
4. Боевые и Сенсорные системы (План разработки)
- [[Radar & Transponders]] — Обнаружение объектов и скрытие кораблей.
- [[Orbital Combat]] — Снаряды, урон, оружие.
## Список задач (Roadmap)
- Базовая 3D проекция Canvas
- Векторная математика (/datum/orbital_vector)
- Автопилот (Travel, Orbit, Hold)
- Система SAS и RCS
- Фикс рывков планет (Аналитическая скорость)
- Фильтрация объектов в UI (Радары)
- Разделение консолей (Helm/Nav/Tac)
- Боевые системы (Снаряды)