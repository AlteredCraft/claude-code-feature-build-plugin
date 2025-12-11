# Changelog

All notable changes to the artifact-workflow plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] - 2025-12-11

### Added
- **Phase 3.5: ADR Review Gate** - Systematic architectural decision review process before build completion
  - Mandatory review phase between execution and completion
  - Compares planned vs actual implementation to identify architectural decisions
  - References `adr-manager` skill for decision criteria and examples
  - Ensures ADR documentation is considered for every build

- **Enhanced adr-manager skill** with concrete example scenarios
  - Added ✅ examples of decisions that should be documented (e.g., technology choices, architectural patterns)
  - Added ❌ examples of routine work that doesn't require ADR entries
  - Centralized decision criteria in one authoritative location

### Changed
- **Simplified Phase 3 ADR guidance** - Removed duplicative criteria, now references `adr-manager` skill
- **Updated Key Principles** - Added principle emphasizing ADR review is mandatory for all builds
- **Streamlined build command** - Reduced ~40 lines by eliminating duplication and consolidating guidance in `adr-manager` skill
- **Made context links mandatory** in `adr-manager` skill - All ADR entries must link back to their originating build

### Improved
- Eliminated duplication between build command and adr-manager skill documentation
- Made ADR documentation workflow more systematic and less likely to be skipped
- Two-pronged approach: real-time capture (Phase 3) + systematic review (Phase 3.5)

## [1.0.0] - Initial Release

Initial release of artifact-workflow plugin with:
- `/build` command for artifact-driven development workflow
- Four-phase workflow: Interrogation, Planning, Execution, Completion
- Three per-build artifacts: todo.md, implementation-plan.md, walkthrough.md
- Shared ADR.md for architecture decision records
- Skills: task-list-creator, implementation-plan-creator, walkthrough-creator, adr-manager
- Mandatory approval gates between phases
