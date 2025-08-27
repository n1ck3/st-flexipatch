# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is st-flexipatch, a fork of st (simple terminal) that uses preprocessor directives to conditionally include patches at compile time. Based on st 0.9.2, the project contains both patched and original code, allowing selective inclusion of patches via configuration.

## Build Commands

- **Quick build and install**: `./install.sh` - Cleans, removes config files, rebuilds, installs system-wide, and sends USR1 signal to reload
- **Clean build**: `make clean && make`
- **Incremental build**: `make`
- **Install**: `sudo make install`
- **Note**: No unit tests - this is a terminal emulator

## Repository Structure

### Key Files
- `patches.def.h` - Default patch configuration (copy to `patches.h` before building)
- `config.def.h` - Default st configuration (copy to `config.h` before building)
- `config.mk` - Build configuration (compiler flags, optional libraries)
- `st.c` - Main terminal source with VT100 emulation and conditional patch code
- `x.c` - X11 window management and rendering
- `patch/` - Directory containing all patch implementations

### Branch Strategy
- `master` - Pristine copy of upstream (NEVER modify)
- `prod` - Production customizations on top of master
- `test` - Testing branch based on prod

## Architecture

The codebase uses preprocessor directives to conditionally include patches:
- Patches are controlled via `#define PATCH_NAME_PATCH 1` in patches.h
- Code blocks are wrapped in `#if PATCH_NAME_PATCH` directives
- Each patch typically has corresponding .c and .h files in the patch/ directory

### Core Components
- **Terminal emulation** - VT100/ANSI escape sequence processing (st.c)
- **Window management** - X11 rendering and event handling (x.c)
- **Patches** - Modular enhancements for appearance, behavior, and functionality

### Enabled Patches (from patches.h)
Key patches currently enabled include:
- `ALPHA_PATCH` - Terminal transparency support
- `DRAG_AND_DROP_PATCH` - Drag and drop file paths into terminal
- `W3M_PATCH` - Support for w3m image display
- `WIDE_GLYPHS_PATCH` - Proper wide character rendering
- `XRESOURCES_PATCH` - Configure st via Xresources
- `XRESOURCES_RELOAD_PATCH` - Reload Xresources with SIGUSR1

## Development Workflow

### Updating from Upstream
1. Ensure upstream remote: `git remote add upstream https://github.com/bakkeby/st-flexipatch.git`
2. Update master: `git checkout master && git fetch upstream master && git rebase --rebase-merges upstream/master && git push origin master`
3. Rebase prod: `git checkout prod && git rebase --rebase-merges master` (resolve conflicts if any)
4. Rebase test: `git checkout test && git rebase --rebase-merges prod`
5. Build and test: `./install.sh`
6. Push: `git push --force-with-lease origin prod && git push --force-with-lease origin test`

Note: `--rebase-merges` preserves merge commits and avoids re-resolving previous conflicts.

### Adding/Modifying Patches
1. Edit `patches.h` to enable/disable patches
2. Some patches require uncommenting lines in `config.mk`:
   - `ALPHA_PATCH`: Uncomment `XRENDER` line
   - `LIGATURES_PATCH`: Uncomment ligatures-related lines
   - `SIXEL_PATCH`: Uncomment sixel-related lines
   - `NETWMICON_PATCH`: Uncomment netwmicon line
3. Modify patch code in `patch/` directory if needed
4. Follow existing preprocessor directive patterns
5. Test with `./install.sh`

### Quick Reference Commands
- **Check status**: `git status`
- **View recent commits**: `git log --oneline --graph --decorate -10`
- **Compare branches**: `git diff master..prod` or `git diff prod..test`
- **Emergency rollback**: `git checkout prod && git reset --hard origin/prod`

## Code Style
- **Language**: C99 standard
- **Indentation**: Tabs (not spaces)
- **Naming**: snake_case for functions/variables, SCREAMING_SNAKE_CASE for macros
- **Braces**: K&R style
- **Error handling**: Check system calls, use `die()` for fatal errors
- **Memory**: Free all allocated resources, especially X resources

## Signal Handling
- **SIGUSR1**: Reload background image (if BACKGROUND_IMAGE_RELOAD_PATCH enabled) or Xresources (if XRESOURCES_RELOAD_PATCH enabled)