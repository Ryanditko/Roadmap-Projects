# Changelog

All notable changes to this project will be documented in this file.

## [3.0.0] - 2026-04-05

### Major Changes

- **Repository Restructure**: All domain folders now organized under a central `projects/` directory
- **New Main README**: Generalized landing page without domain-specific project lists
- **Domain-Specific READMEs**: Created 6 comprehensive README files for each domain (Backend, Frontend, Data Science, Data Engineering, DevOps, System Design)
- **Professional Branding**: Added 7 high-quality banner images for visual consistency
- **Comprehensive Documentation**: Each domain README includes:
  - Domain overview and learning path
  - Complete project index (30 projects per domain: 10 beginner, 10 intermediate, 10 advanced)
  - Tech stack recommendations
  - Key concepts to master
  - Success tips and common mistakes
  - Curated resources and references

### Added

- **Main Banner Image**: `projects/images/banner-main.png`
- **Domain Banners**: 
  - `projects/images/banner-backend.png`
  - `projects/images/banner-frontend.png`
  - `projects/images/banner-data-science.png`
  - `projects/images/banner-data-engineering.png`
  - `projects/images/banner-devops.png`
  - `projects/images/banner-system-design.png`

- **Domain README Files**:
  - `projects/backend/README.md` - 30 backend projects with tech stack guidance
  - `projects/frontend/README.md` - 30 frontend projects with UI/UX focus
  - `projects/data-science/README.md` - 30 data science projects with ML focus
  - `projects/data-engineering/README.md` - 30 data engineering projects with ETL focus
  - `projects/devops/README.md` - 30 DevOps projects with automation focus
  - `projects/system-design/README.md` - 30 system design projects with architecture focus

### Changed

- **Main README**: Transformed to generalized presentation emphasizing learning paths across all 6 domains
- **Repository Structure**: All domains moved under `projects/` directory for better organization
- **Image References**: Updated all image paths to point to `projects/images/`
- **Navigation Links**: Updated all internal links to reflect new directory structure

### New Repository Structure

```
projects-library/
├── projects/
│   ├── backend/
│   ├── frontend/
│   ├── data-science/
│   ├── data-engineering/
│   ├── devops/
│   ├── system-design/
│   └── images/
├── README.md (generalized)
├── CHANGELOG.md
├── CONTRIBUTING.md
└── LICENSE
```

### Total Projects

- **Total Domains**: 6 (Backend, Frontend, Data Science, Data Engineering, DevOps, System Design)
- **Total Projects**: 180 idea-based project descriptions
- **Project Distribution**: 10 projects × 3 difficulty levels × 6 domains
- **Documentation**: Comprehensive README for each domain with learning paths and resources

---

## [2.0.0] - 2026-02-14

### Changed

- **Complete repository restructure**: Transformed from multi-language implementation repository to project ideas library
- **Language**: All content translated from Portuguese to English for broader accessibility
- **Content**: Removed all emojis for professional presentation
- **Technology sections**: Removed language-specific implementation details (JavaScript, Python, Clojure)
- **Focus**: Repository now provides project ideas and concepts rather than specific implementations

### Renamed

**Main Folders:**
- `iniciante/` → `beginner/`
- `intermediario/` → `intermediate/`
- `avancado/` → `advanced/`

**Beginner Projects:**
- `gerador-de-senha/` → `password-generator/`
- `lista-de-tarefas/` → `todo-list/`
- `calculadora-simples/` → `simple-calculator/`
- `conversor-de-moedas/` → `currency-converter/`
- `jogo-adivinhacao/` → `number-guessing-game/`
- `relogio-digital/` → `digital-clock/`
- `contador-de-palavras/` → `word-counter/`
- `simulador-dados-rpg/` → `rpg-dice-simulator/`
- `conversor-temperatura/` → `temperature-converter/`
- `gerador-qr-code/` → `qr-code-generator/`
- `temporizador-pomodoro/` → `pomodoro-timer/`
- `conversor-unidades/` → `unit-converter/`
- `bloco-de-notas/` → `online-notepad/`
- `simulador-moeda/` → `coin-flip-simulator/`
- `checklist-compras/` → `shopping-checklist/`

**Intermediate Projects:**
- `notas-markdown/` → `markdown-notes/`
- `encurtador-url/` → `url-shortener/`
- `dashboard-gastos/` → `expenses-dashboard/`
- `login-jwt/` → `jwt-login/`
- `api-clima/` → `weather-api/`
- `chat-tempo-real/` → `realtime-chat/`
- `gerenciador-senhas/` → `password-manager/`
- `kanban-tarefas/` → `kanban-board/`
- `quiz-ranking/` → `quiz-platform/`
- `gerador-relatorios-pdf/` → `pdf-report-generator/`

**Advanced Projects:**
- `ecommerce-completo/` → `ecommerce-platform/`
- `rede-social/` → `social-network/`
- `gestao-financeira/` → `financial-management/`
- `clone-trello/` → `trello-clone/`
- `streaming-videos/` → `video-streaming/`
- `saas-multiusuario/` → `saas-platform/`
- `chatbot-ia/` → `ai-chatbot/`
- `plataforma-cursos/` → `course-platform/`
- `recrutamento-ia/` → `ai-recruitment/`
- `investimentos-simulados/` → `investment-simulator/`

### Project Structure

Each project now contains:
- **Description**: Clear, concise project overview
- **Features**: Core functionality requirements
- **Learning Objectives**: Key concepts and skills to develop
- **Implementation Tips**: Practical guidance for building the project
- **Useful Resources**: Relevant documentation and learning materials
- **Additional Challenges**: Optional features to extend the project

### Purpose

This repository serves as a curated collection of project ideas for developers at all levels. Users can choose any project and implement it using their preferred programming language, framework, and tools.

---

## [1.0.0] - Initial Release

- Initial repository with 35 projects in Portuguese
- Multi-language implementations planned (JavaScript, Python, Clojure)
- Emoji-rich documentation
- Technology-specific suggestion