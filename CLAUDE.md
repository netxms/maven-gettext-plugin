# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Maven Gettext Plugin - A Maven 2+ plugin for GNU Gettext integration with Java projects. Enables extraction of translatable messages from source code, merging translations, and generating compiled resource bundles.

**Coordinates:** `org.netxms:gettext-maven-plugin`

## Build Commands

```bash
# Build and install locally
mvn clean install

# Run tests
mvn test

# Generate plugin documentation
mvn clean verify

# View available plugin goals
mvn help:describe -Dplugin=org.netxms:gettext-maven-plugin

# Release (requires Java 11)
JAVA_HOME="$(/usr/libexec/java_home -v 11)" mvn release:clean release:prepare release:perform
```

## Architecture

All source code is in `src/main/java/org/xnap/commons/maven/gettext/`.

### Mojo Classes (Plugin Goals)

| Class | Goal | Phase | Purpose |
|-------|------|-------|---------|
| `GettextMojo` | `gettext` | generate-resources | Extract translatable strings via `xgettext` |
| `MergeMojo` | `merge` | generate-resources | Update .po files with new messages via `msgmerge` |
| `DistMojo` | `dist` | generate-resources | Compile .po to resource bundles via `msgfmt`/`msgcat` |
| `AttribMojo` | `attrib` | generate-resources | Update .po attributes via `msgattrib` |
| `ReportMojo` | `report` | process-sources | Generate translation statistics |

### Supporting Classes

- **AbstractGettextMojo**: Base class defining common parameters (outputDirectory, sourceDirectory, poDirectory, keysFile, encoding, includes/excludes)
- **GettextUtils**: Locale conversion (gettext→Java format), POT header manipulation, unicode escaping
- **LoggerStreamConsumer**: Bridges external command output to Maven logger

### External Tool Integration

All goals execute GNU Gettext tools via command-line (Plexus CLI utilities):
- `xgettext` - message extraction
- `msgmerge` - translation merging
- `msgfmt` - compilation to class format
- `msgcat` - compilation to properties format
- `msgattrib` - attribute manipulation

### DistMojo Output Formats

Uses strategy pattern with `CommandlineFactory` interface:
- **class**: `MsgFmtCommandlineFactory` - compiled .class resource bundles
- **java**: `MsgFmtSourceCommandlineFactory` - Java source files (uses temp directory)
- **properties**: `MsgCatCommandlineFactory` - property files

### Locale Handling

`GettextUtils.getJavaLocale()` converts gettext locales (e.g., `en_US`, `pt_BR_VARIANT@encoding`) to Java format with legacy mappings: `he→iw`, `yi→ji`, `id→in`.

## Configuration

Target Java: 11 (source and target)

Key parameters across mojos:
- `poDirectory`: Location of .po/.pot files (default: `src/main/po`)
- `keysFile`: Template filename (default: `keys.pot`)
- `targetBundle`: Package/class for output (e.g., `com.example.Messages`)
- `outputFormat`: `class`, `properties`, or `java`
- `printPOTCreationDate`: Include timestamp header (default: `false`)
