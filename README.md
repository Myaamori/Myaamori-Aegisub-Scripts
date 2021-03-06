# Myaamori's Script Collection

1. [Merge Scripts: Bidirectional script merging](#merge-scripts)
2. [Sub Digest: Processing ASS files from the CLI](#sub-digest)
3. [ASSParser: Parsing ASS files from automations](#assparser)
4. [Paste From Pad: Paste pad contents over existing lines](#paste-from-pad)
5. [Font Validator: Scan for common font-related issues in a muxed MKV](#font-validator)

## Merge Scripts

### Overview

![overview](https://raw.githubusercontent.com/TypesettingTools/Myaamori-Aegisub-Scripts/master/assets/ms_overview.png)

Merge Scripts is an advanced bidirectional script merger.
It allows you to define a set of imports in an ASS file, which can then be used to import and unimport other files, as well as export any changes to the original files.
This is particularly useful when you are using a VCS such as git to store your files in a modular layout, where e.g. dialogue, songs and typesetting are stored in separate files and where the filenames don't change.
In such a scenario, you only need to define the imports once, after which all scripts can be re-merged in a single click anytime something is changed.
No more manually having to merge and shift songs every time you change something in the OP!
The ability to export changes also means that the modular structure can be kept, while still allowing for e.g. easy QC from a single file.

### Importing files

To import files you must first create an import definition.
Merge Scripts can do this for you using the **Add file to include** option.
Simply select the file or files that you wish to include, and import definitions will be added for you at the end of the file.

![overview](https://raw.githubusercontent.com/TypesettingTools/Myaamori-Aegisub-Scripts/master/assets/ms_importdef.png)

This script contains import definitions for three files: `shoujo kageki 03.ass`, `insert 03.ass`, and `signs 03.ass`.
In order to actually import the files, we use the **Import all external files** option.
This will read the files specified in the import definitions and add their styles and dialogue lines to the current script.

You can also use **wildcards** to specify groups of files, e.g. `signs*.ass`.
Files imported in this manner will generally be treated as one single large file as far as Merge Scripts is concerned.

![after import](https://raw.githubusercontent.com/TypesettingTools/Myaamori-Aegisub-Scripts/master/assets/ms_postimport.png)

Every style will be prefixed with an identifier to keep track of what file the line came from.
Files can be imported selectively using **Import selected external file(s)**, which will only import files corresponding to the currently selected import definitions.

Once files have been imported, they can be unimported to get back the original file prior to importing using the **Remove all external files** option.
This will delete all lines that belong to imported files.
Much like with importing, you can remove files selectively using **Remove selected external file(s)**, which will only remove the file corresponding to the currently selected files (you may select either the import definition or any line from the file you wish to remove).

![import gui](https://raw.githubusercontent.com/TypesettingTools/Myaamori-Aegisub-Scripts/master/assets/ms_importgui.png)

In order to make selective importing and unimporting easier, there's also a GUI available through the **Select files to import or unimport** option.
It will detect all defined imports and ask which of the files you wish to import.
If you deselect an already imported file, it will be unimported.

### Exporting changes

The **Export changes** option allows you to export any changes you've made to the lines to their original files.
You will get a warning showing what files will be overwritten upon export.
Note that the script will currently *not* export script properties, but will leave the Script Info and Aegisub Project Garbage sections as they were in the source file.
Export changes will keep any extradata generated by automations intact.

### Shifted imports

![shifted import](https://raw.githubusercontent.com/TypesettingTools/Myaamori-Aegisub-Scripts/master/assets/ms_shiftedimport.png)

Shifted imports automatically shift the lines in the imported file so that the start time of a synchronization line in the external file matches the start time of the import definition.
Run **Add synchronization line for shifted imports** on the file to import to add a new synchronization line at the current video time, and **Add file to include (shifted)** to add a shifted import definition at the current video time.

When exporting changes to a shifted file, the timings will be shifted back to match the original position of the synchronization line.
This means that shifting the lines will not affect the timing in the source file, however relative adjustments to individual lines such as fixed scene bleeds will be exported.

### Conflicting script properties on import

![conflicting properties](https://raw.githubusercontent.com/TypesettingTools/Myaamori-Aegisub-Scripts/master/assets/ms_conflict.png)

When importing files, Merge Scripts will set script properties such as the resolution of the current script to match the imported files.
However, if there are conflicting values among the imported scripts, Merge Scripts will warn about this and let you choose which value to use.
If you hover over the dropdown box you can see the corresponding value from each individual file.
Note that **Merge Scripts will not attempt to resample the lines based on your selection**.
Thus, in the event of a conflict it is usually best to correct the offending file first.

### Generating a release version

Since you may not want to release the script publicly with all styles prefixed and the import definitions included (not that it would affect the viewing experience in any way), Merge Scripts also provides the **Generate release candidate** option.
This will allow you to save a new file with the prefixes from all styles, unused styles, and commented and empty lines removed.
It will also warn about conflicting style names.

### Other misc. features

* If you set the layer field of an import definition line, all lines imported from that file will have their layers incremented by that value.
This is useful for e.g. importing dialogue lines, if you want to ensure that they are automatically layered above typesetting.

## Sub Digest

### Overview

```
subdigest -i script.ass --selection-set style "Default|Alt" --keep-selected --remove-unused-styles --sort-field start ASC
```

Sub Digest is a utility for quick processing of ASS files.
At the basic level it can do e.g. sorting, simple replacement with regex, and modification of values using Python expressions.
However, most of its power comes from the use of *selections*, which make it possible to operate on just a subset of the file.

Sub Digest makes use of [python-ass](https://github.com/chireiden/python-ass), and inherits many of its conventions, including field names.
It is highly recommended to familiarize yourself with python-ass first before making use of Sub Digest.

```
$ subdigest --help
usage: subdigest [-h] [--fps FPS] [--get-field FIELD] [--keep-selected]
                 [--merge-file OTHER_FILE] [--modify-expr FIELD EXPR]
                 [--modify-field FIELD PATTERN REPLACE] [--move-selected {BOTTOM,TOP}]
                 [--ms-import] [--ms-import-filter FIELD PATTERN] [--ms-import-rc]
                 [--ms-import-rc-filter FIELD PATTERN] [--ms-remove-namespace]
                 [--remove-all-tags] [--remove-comments] [--remove-selected]
                 [--remove-unused-styles] [--selection-add FIELD PATTERN]
                 [--selection-add-expr EXPR] [--selection-clear]
                 [--selection-intersect FIELD PATTERN] [--selection-intersect-expr EXPR]
                 [--selection-set FIELD PATTERN] [--selection-set-expr EXPR]
                 [--selection-subtract FIELD PATTERN] [--selection-subtract-expr EXPR]
                 [--set-script-info FIELD VALUE] [--shift EXPR]
                 [--sort-expr EXPR {ASC,DESC}] [--sort-field FIELD {ASC,DESC}]
                 [--use-events] [--use-styles] [-i INPUT] [-o OUTPUT] [--in-place]

optional arguments:
  -h, --help            show this help message and exit
  -i INPUT, --input INPUT
                        Specify input file (default: stdin)
  -o OUTPUT, --output OUTPUT
                        Specify output file (default: stdout)
  --in-place            Perform operations in place

Subtitles:
  --fps FPS             Set the fps to use for the frames() function. Default is 24000/1001.
                        Output type: Subtitles.
  --get-field FIELD     Returns the given field from all lines in the current selection as
                        text, newline separated. Output type: Text.
  --keep-selected       Remove all lines not in the current selection. Clears the selection.
                        Output type: Subtitles.
  --merge-file OTHER_FILE
                        Append the styles and event lines from another file. Output type:
                        Subtitles.
  --modify-expr FIELD EXPR
                        Replace the value of the given field on all lines in the selection
                        with the result of the given expression. Output type: Subtitles.
  --modify-field FIELD PATTERN REPLACE
                        Replace occurrences of pattern with the given replacement string in
                        the given field on all lines in the current selection. Accepts
                        regular expressions. Output type: Subtitles.
  --move-selected {BOTTOM,TOP}
                        Move all selected lines to the top or bottom of the current section.
                        Output type: Subtitles.
  --ms-import           Import files according to import definitions in a file using Merge
                        Scripts syntax. Discards styles and lines already imported from
                        external files. Imported styles will be prefixed by a namespace
                        identifier, e.g. 1$Default. Output type: Subtitles.
  --ms-import-filter FIELD PATTERN
                        Import files according to import definitions in a file using Merge
                        Scripts syntax. Discards styles and lines already imported from
                        external files. Imported styles will be prefixed by a namespace
                        identifier, e.g. 1$Default. Only import files corresponding to
                        import definitions matching the given pattern. Output type:
                        Subtitles.
  --ms-import-rc        Equivalent to --ms-import --remove-comments --remove-unused-styles
                        --ms-remove-namespace. Output type: Subtitles.
  --ms-import-rc-filter FIELD PATTERN
                        Equivalent to --ms-import-filter FIELD PATTERN --remove-comments
                        --remove-unused-styles --ms-remove-namespace. Output type:
                        Subtitles.
  --ms-remove-namespace
                        Remove namespace identifiers from style names (e.g. change 1$Default
                        to Default). Output type: Subtitles.
  --remove-all-tags     Remove all tags (everything in the text field enclosed in {}) from
                        all dialogue lines. No-op if current section is not the events
                        section. Output type: Subtitles.
  --remove-comments     Removes all commented lines in current selection. Clears the
                        selection if the current selection is the events section. Output
                        type: Subtitles.
  --remove-selected     Remove all lines in the current selection. Clears the selection.
                        Output type: Subtitles.
  --remove-unused-styles
                        Remove all styles not used in any dialogue lines. Clears the
                        selection if the current section is the styles section. Output type:
                        Subtitles.
  --selection-add FIELD PATTERN
                        Set the selection to the union of the current selection and all
                        lines in the current section for which the given field matches the
                        given regex pattern. Output type: Subtitles.
  --selection-add-expr EXPR
                        Set the selection to the union of the current selection and all
                        lines in the current section for which expr returns true. Output
                        type: Subtitles.
  --selection-clear     Reset the selection (select all lines). Output type: Subtitles.
  --selection-intersect FIELD PATTERN
                        Set the selection to the intersection of the current selection and
                        all lines in the current section for which the given field matches
                        the given regex pattern. Output type: Subtitles.
  --selection-intersect-expr EXPR
                        Set the selection to the intersection of the current selection and
                        all lines in the current section for which expr returns true. Output
                        type: Subtitles.
  --selection-set FIELD PATTERN
                        Set the selection to all lines in the current section for which the
                        given field matches the given regex pattern. Output type: Subtitles.
  --selection-set-expr EXPR
                        Set the selection to the lines in the current section for which expr
                        returns true. Output type: Subtitles.
  --selection-subtract FIELD PATTERN
                        Set the selection to the current selection, minus all lines in the
                        current section for which the given field matches the given regex
                        pattern. Output type: Subtitles.
  --selection-subtract-expr EXPR
                        Set the selection to the current selection, minus all lines in the
                        current section for which expr returns true. Output type: Subtitles.
  --set-script-info FIELD VALUE
                        Output type: Subtitles.
  --shift EXPR          Shifts the start and end time by a specified amount. Example:
                        --shift "secs(-10)". Output type: Subtitles.
  --sort-expr EXPR {ASC,DESC}
                        Sort all lines in the current selection based on the return value of
                        expr, either ascending or descending. Output type: Subtitles.
  --sort-field FIELD {ASC,DESC}
                        Sort all lines in the current selection based on the given field,
                        either ascending or descending. Output type: Subtitles.
  --use-events          Set the current section to the events section. Output type:
                        Subtitles.
  --use-styles          Set the current section to the styles section. Output type:
                        Subtitles.
```

### Installation

Install Sub Digest with pip:

```
pip install --user git+https://github.com/TypesettingTools/Myaamori-Aegisub-Scripts/#subdirectory=scripts/sub-digest
```

This will add a `subdigest` executable along with the `subdigest.py` Python module.
If the executable is not in your path, you can run Sub Digest as `python -m subdigest --help`, or alternatively `py -3 -m subdigest --help` on Windows.

### Workflow

The arguments to Sub Digest are processed as consecutive commands, with the output of one command being fed as the input to the next.
If you run e.g. `--sort-field start ASC --sort-field style ASC`, it will first sort by start time and then by style name.

By default Sub Digests works on the events section of the file, which contains the dialogue lines.
You can use the `--use-styles` argument to switch to the styles section instead, so that any following arguments will modify the styles section.
Use `--use-events` to switch back.

### Selections

By default, all lines are treated as selected, meaning that most commands such as `--sort-field` will operate on all lines (in the current section).
If you wish to only operate on a subset of lines, use the `--selection-*` commands.
E.g. `--selection-set style "^Default"` will set the selection to all lines whose style begins with "Default", and `--selection-intersect TYPE "Comment"` will intersect the current selection with all commented lines.
To reset the selection, and thus select all lines again, use `--selection-clear`.
Operations that support selections include `--sort-*`, `--modify-*`, `--keep-selected` and `--remove-selected`.

### Expressions

Some commands have alternative versions that take arbitrary Python expressions.
In these expressions, the current line will be available as `_`.
There are also a number of functions available to make it easier to modify the start and end times of a line, which are represented as `datetime.timedelta` objects, namely: `mins(x)`, `secs(x)`, `millis(x)` and `frames(x)`, which take a float and return a corresponding `timedelta` object.
For `frames(x)`, you can set the framerate to use for calculations using `--fps`, e.g. `--fps 24000/1001`.
For instance, in order to add a lead-in of 100 ms to all lines, you can use `--modify-expr start "_.start - secs(0.1)"`.

### Use from Python (interactive session)

Sub Digest also provides a `subdigest.py` module which can be imported from Python.
This allows for the possibility of interactively editing an ASS file from the Python REPL.

```
$ python
>>> import ass
>>> import subdigest
>>> with open('script.ass') as f:
...   subs = subdigest.Subtitles(ass.parse(f))
...
>>> subs.
subs.get_field(                 subs.move_selection(            subs.selection                  subs.selection_intersect_expr(  subs.sort_expr(
subs.keep_selected(             subs.remove_all_tags(           subs.selection_add(             subs.selection_set(             subs.sort_field(
subs.merge_file(                subs.remove_selected(           subs.selection_add_expr(        subs.selection_set_expr(        subs.sub_file
subs.modify_expr(               subs.remove_unused_styles(      subs.selection_clear(           subs.selection_subtract(        subs.use_events(
subs.modify_field(              subs.section                    subs.selection_intersect(       subs.selection_subtract_expr(   subs.use_styles(
>>> subs.selection_set("style", "^Default")
<subdigest.Subtitles object at 0x7f9ba4d49e48>
>>> subs.modify_field("text", "-chan", "")
<subdigest.Subtitles object at 0x7f9ba4d49e48>
>>> with open('output.ass', 'w') as f:
...   subs.dump_file(f)
```

All `ass.document.Document` properties and methods are available directly from the `subdigest.Subtitles` object, including the `events` and `styles` objects.

Note that at the current point in time most operations are performed in-place, meaning that there is no way to undo a change.

### Examples

Get the text of dialogue lines as plain text, with override tags and comments removed:
```
subdigest -i script.ass --remove-all-tags --get-field text
```

Change all instances of `-chan` and `-san` to `{**-chan}` and `{**-san}`, respectively, for use with [Daiz's Autoswapper](https://github.com/Daiz/AegisubMacros/#autoswapperlua---autoswapper):
```
subdigest -i script.ass --in-place --modify-field text "(-(chan|san))" "{**\1}"
```

Extract only the dialogue lines from a muxed script:
```
subdigest -i script.ass -o dialogue.ass --selection-set style "Default|Alt" --keep-selected --remove-unused-styles --sort-field start ASC
```

Use together with [Prass](https://github.com/tp7/Prass/):
```
subdigest -i script.ass --selection-set style "Default" --keep-selected | prass tpp - --lead-in 100 --lead-out 200 | subdigest --sort-field start ASC
```

Merge files:
```
subdigest -i dialogue.ass --merge-file typesetting.ass --merge-file OP.ass
```

Remove all lines within the first 5 minutes:
```
subdigest -i dialogue.ass --set-selection-expr "_.start < mins(5)" --remove-selected
```

Replace styles with styles from a different ASS file:
```
subdigest -i previous_episode.ass --remove-selected > only_styles.ass # remove dialogue lines
subdigest -i current_episode.ass --use-styles --remove-selected --merge-file only_styles.ass
```

## ASSParser

### Overview

ASSParser is a module containing facilities for parsing ASS files and related utilities written in MoonScript for use with Aegisub automations.

### Usage

```moon
parser = require 'myaa.ASSParser'
file = open 'my script.ass'
parsed_file = parser.parse_file file
file\close!

for event in *parsed_file.events
    aegisub.log "#{event.text}\n"
```

### Documentation

#### parse_file(file)

Parses an ASS file.

Arguments:

* **file**: A file handle to the ASS file to parse

Returns: An `ASSFile` object representing the parsed file, containing the following fields:

* **styles**: A list of style lines.
* **events**: A list of event (dialogue) lines.
* **script_info**: A list of info lines representing the Script Info section.
* **script_info_mapping**: A table mapping Script Info keys to their corresponding values.
* **aegisub_garbage**: A list of info lines representing the Aegisub Project Garbage section.
* **aegisub_garbage_mapping**: A table mapping Aegisub Project Garbage keys to their corresponding values.
* **extradata**: A table mapping numerical extradata IDs to data lines (see below).
* **extradata_mapping**: A table mapping keys and values to their original ID.
`extradata_mapping[key][value]` is the numerical ID of the extradata line with key `key` and value `value`.

The style, dialogue and info lines follow the format described in [the Aegisub documentation](http://docs.aegisub.org/3.2/Automation/Lua/Subtitle_file_interface/#line-data-tables).
Data lines are similar to info lines, except they additionally have an `id` field containing the original numerical ID of the line in the source file.

Example:

```moon
assfile = parser.parse_file(file)
aegisub.log "The script resolution is #{assfile.script_info_mapping.PlayResX}x#{assfile.script_info_mapping.PlayResY}\n"
```

#### raw_to_line(raw, extradata=nil, format=nil)

Parses a raw line as it appears in the ASS file.

Arguments:

* **raw**: A text representation of the line.
* **extradata**: A mapping from extradata IDs to extradata lines. Corresponds to `ASSFile.extradata`.
If provided, the line will be automatically populated with extradata if extradata IDs are present on the line;
otherwise, the extradata IDs will be silently removed.
* **format**: An ordered list of field names corresponding to the `Format` line at the start of the styles and events sections.
You probably never need to use this.

Returns: A line object corresponding to the given text.

Example:

```moon
>>> json.encode parser.raw_to_line "Dialogue: 5,0:00:04.20,0:00:05.48,Default,,0,0,0,,{\\i1}Starlight."
{"layer":5,"extra":{},"margin_t":0,"margin_l":0,"style":"Default","actor":"","start_time":4200,"end_time":5480,"section":"[Events]","class":"dialogue","margin_r":0,"comment":false,"effect":"","text":"{\\i1}Starlight."}
```

#### line_to_raw(line)

Converts a line object into a text representation corresponding to how it would appear in an ASS file.

Arguments:

* **line**: The line object.

Returns: A raw text representation of the line.

Example:

```moon
>>> parser.line_to_raw {"layer":5,"extra":{},"margin_t":0,"margin_l":0,"style":"Default","actor":"","start_time":4200,"end_time":5480,"section":"[Events]","class":"dialogue","margin_r":0,"comment":false,"effect":"","text":"{\\i1}Starlight."}
Dialogue: 5,0:00:04.20,0:00:05.48,Default,,0,0,0,,{\i1}Starlight.
```

#### create_dialogue_line(fields)

Create a new dialogue line with the given values.
Non-specified values will be populated with default values.

Arguments:

* **fields**: A partial line object, i.e. a map with the fields you wish to specify and their values.

Returns: A new dialogue line object.

Example:

```moon
>>> parser.line_to_raw parser.create_dialogue_line {text: "Hello", style: "Alt"}
Dialogue: 0,0:00:00.00,0:00:00.00,Alt,,0,0,0,,Hello
```

#### create_style_line(fields)

Like `create_dialogue_line`, but for styles.

Arguments:

* **fields**: A map containing predefined style properties.

Returns: A new style line object.

#### inline_string_encode(input)

Encodes a string using a simple scheme reminiscent of URL encoding.
Used to encode extradata values.

Arguments:

* **input**: The string to encode.

Returns: An encoded string.

Example:

```moon
>>> parser.inline_string_encode '{"style":"Alt","text":"Hello"}'
{"style"#3A"Alt"#2C"text"#3A"Hello"}
```

#### inline_string_decode(input)

Decodes a string encoded using `inline_string_encode`.

Arguments:

* **input**: The string to decode.

Returns: A decoded string.

Example:

```moon
>>> parser.inline_string_decode '{"style"#3A"Alt"#2C"text"#3A"Hello"}'
{"style":"Alt","text":"Hello"}
```

#### uuencode(input)

Implements UUEncode as defined in the [ASS specification](http://moodub.free.fr/video/ass-specs.doc).
Used to encode extradata values.
Based on [the corresponding Aegisub implementation](https://github.com/TypesettingTools/Aegisub/blob/26ccf0b8e5374973b3a5edfe2bd958a1a2040da1/libaegisub/ass/uuencode.cpp#L29).

Arguments:

* **input**: The string to encode.

Returns: A UUEncoded string.

Example:

```moon
>>> parser.uuencode "hello"
;'6M<']
```

#### uudecode(input)

Decodes a UUEncoded string.

Arguments:

* **input**: The string to decode.

Returns: A decoded string.

Example:

```moon
>>> parser.uudecode ";'6M<']"
hello
```

#### decode_extradata_value(value)

Decodes an extradata value based on the prefix (`e` for `inline_string_decode`, `u` for `uudecode`).

Arguments:

* **value**: The extradata value to decode.

Returns: A decoded string.

Example:

```moon
>>> parser.decode_extradata_value "u;'6M<']"
hello
```

#### generate_styles_section(styles, callback)

Generates a V4+ Styles section from the given style lines.
Each outputted line is passed to the callback function.

Arguments:

* **styles**: A list of style objects.
* **callback**: A callback accepting one string argument, namely one line to output.

Example:

```moon
>>> parse.generate_styles_section style_list, (line) -> io.write line
[V4+ Styles]
Format: Name, Fontname, Fontsize, PrimaryColour, SecondaryColour, OutlineColour, BackColour, Bold, Italic, Underline, StrikeOut, ScaleX, ScaleY, Spacing, Angle, BorderStyle, Outline, Shadow, Alignment, MarginL, MarginR, MarginV, Encoding
Style: Default,Myriad Pro Light,79,&H00FFFFFF,&H000000FF,&H00613C38,&HC0171010,0,0,0,0,100,100,0,0,1,3.8,2.4,2,144,144,60,1
Style: Alt,Myriad Pro Light,79,&H00FFFFFF,&H000000FF,&H007B437F,&H96171010,0,0,0,0,100,100,0,0,1,3.8,2.4,2,144,144,50,1
```

#### generate_events_section(events, extradata_mapping=nil, callback)

Generates an Events section from the given dialogue lines.
If `extradata_mapping` is given, also generates an extradata section.
Each outputted line is passed to the callback function.

Arguments:

* **events**: A list of dialogue lines.
* **extradata_mapping**: A map from extradata key and value to numerical extradata ID, of the same format as `ASSFile.extradata_mapping`.
May be `{}`.
Note that `extradata_mapping` will be populated with new IDs for unseen extradata key/value pairs.
* **callback**: A callback accepting one string argument, namely one line to output.

Example:

```moon
>>> extradata_mapping = {}
>>> events, extradata = parser.generate_events_section dialogue_lines, extradata_mapping, (line) -> io.write line
[Events]
Format: Layer, Start, End, Style, Name, MarginL, MarginR, MarginV, Effect, Text
Dialogue: 0,0:23:13.78,0:23:16.16,Default,,0,0,0,,{=1}{\pos(562.423,712.71)\frz-7.456}T
Dialogue: 0,0:23:13.78,0:23:16.16,Default,,0,0,0,,{=2}{\pos(572.74,714.338)\frz-9.882}e
Dialogue: 0,0:23:13.78,0:23:16.16,Default,,0,0,0,,{=2}{\pos(582.064,715.864)\frz-9.882}x
Dialogue: 0,0:23:13.78,0:23:16.16,Default,,0,0,0,,{=2}{\pos(588.91,717.113)\frz-11.157}t

[Aegisub Extradata]
Data: 1,l0.MoveAlongPath,e{"orgLine"#3A"{\\clip(m 557 712 b 715 731 921 868 1200 354)}Text"#2C"settings"#3A{"relPos"#3Afalse#2C"aniPos"#3Atrue#2C"aniFrz"#3Atrue#2C"accel"#3A1#2C"cfrMode"#3Atrue#2C"reverseLine"#3Afalse#2C"flipFrz"#3Afalse}#2C"id"#3A"d5531114-a9ce-4bdd-b48e-a3f536cc954a"}
Data: 2,l0.MoveAlongPath,e{"settings"#3A{"relPos"#3Afalse#2C"aniPos"#3Atrue#2C"aniFrz"#3Atrue#2C"accel"#3A1#2C"cfrMode"#3Atrue#2C"reverseLine"#3Afalse#2C"flipFrz"#3Afalse}#2C"id"#3A"d5531114-a9ce-4bdd-b48e-a3f536cc954a"}
>>> json.encode extradata_mapping
{"l0.MoveAlongPath":{"{\"orgLine\":\"{\\\\clip(m 557 712 b 715 731 921 868 1200 354)}Text\",\"settings\":{\"relPos\":false,\"aniPos\":true,\"aniFrz\":true,\"accel\":1,\"cfrMode\":true,\"reverseLine\":false,\"flipFrz\":false},\"id\":\"d5531114-a9ce-4bdd-b48e-a3f536cc954a\"}":1,"{\"settings\":{\"relPos\":false,\"aniPos\":true,\"aniFrz\":true,\"accel\":1,\"cfrMode\":true,\"reverseLine\":false,\"flipFrz\":false},\"id\":\"d5531114-a9ce-4bdd-b48e-a3f536cc954a\"}":2}}
```

#### generate_script_info_section(lines, callback, bom=true)

Generates a Script Info section from the given info lines.
Each outputted line is passed to the callback function.

Arguments:

* **lines**: A list of info lines.
* **callback**: A callback accepting one string argument, namely one line to output.
* **bom**: If true, prepend the section with the UTF-8 byte order mark.
Aegisub expects all Unicode-encoded ASS files to start with a BOM;
don't set to false unless you know what you're doing.

#### generate_aegisub_garbage_section(lines, callback)

Generates an Aegisub Project Garbage section from the given info lines.
Each outputted line is passed to the callback function.

Arguments:

* **lines**: A list of info lines.
* **callback**: A callback accepting one string argument, namely one line to output.

#### generate_file(script_info=nil, aegisub_garbage=nil, styles=nil, events=nil, extradata_mapping=nil, callback)

Generates an ASS file from the given lines in one go.
If any argument is `nil`, the corresponding section will not be included.
Each outputted line is passed to the callback function.
The first output will always include the UTF-8 BOM.

Arguments:

* **script_info**: A list of info lines passed to `generate_script_info_section`.
* **aegisub_garbage**: A list of info lines passed to `generate_aegisub_garbage_section`.
* **styles**: A list of style lines passed to `generate_styles_section`.
* **events**: A list of dialogue lines passed to `generate_events_section`.
* **extradata_mapping**: A mapping from extradata key/value pairs to IDs, passed to `generate_events_section`.
* **callback**: A callback accepting one string argument, namely one line to output.

## Paste From Pad

Allows you to paste dialogue lines with annotated actors over existing, already-timed lines.
Think of it as a combination of Aegisub's text import and paste lines over (Shift+Ctrl+V) features.

The **Paste over from pad** option allows you to paste text in `Actor: Text` format, after which the contents of the existing lines will be replaced with the pasted text.
All line properties other than `Actor`, `Text` and `Comment` will be left untouched.
If the pasted content contains more lines than exist in the script already, new lines will be inserted, so you can also use this script as a substitute for Aegisub's built-in text import.

![paste](https://raw.githubusercontent.com/TypesettingTools/Myaamori-Aegisub-Scripts/master/assets/pfp_paste.png)

![paste result](https://raw.githubusercontent.com/TypesettingTools/Myaamori-Aegisub-Scripts/master/assets/pfp_paste_result.png)

Using **Copy to pad** you can do the inverse: The selected lines will be converted to `Actor: Text` format, and can then be copied back to the pad or used for diffing.

![copy](https://raw.githubusercontent.com/TypesettingTools/Myaamori-Aegisub-Scripts/master/assets/pfp_copy.png)

![copy result](https://raw.githubusercontent.com/TypesettingTools/Myaamori-Aegisub-Scripts/master/assets/pfp_copy_result.png)

## Font Validator

### Overview

Scans a muxed MKV file for issues such as missing fonts, missing glyphs and use of faux bold/italic.

```
$ fontvalidator --help
usage: fontvalidator [-h] [--ignore-drawings]
                     subtitles [additional_fonts [additional_fonts ...]]

Validate font usage in a muxed Matroska file or an ASS file.

positional arguments:
  subtitles          File containing the subtitles to verify. May be a Matroska file or an
                     ASS file. If a Matroska file is provided, any attached fonts will be
                     used.
  additional_fonts   List of additional fonts to use for verification. May be a Matroska
                     file with fonts attached, a directory containing font files, or a
                     single font file.

optional arguments:
  -h, --help         show this help message and exit
  --ignore-drawings  Don't warn about missing fonts only used for drawings.

$ fontvalidator video.mkv
Validating track English
- Could not find font Arial on line(s): 2036 2037 2038 2039 2040 2065 2074 2075 2084 2085 2086 2087 2138 2139 2140 2141 2142 2143 2144 2145 2146 2147 2148 2149 2150 2151 2152 2153 2154 2155 2156 2157 2158 2159 2160 2161 2162 2163 2164 2165 2166 2167 2168 2169 2170 2171 2172 2173 2174 2175 2176 2177 2178 2210 2211 2212 2213 2214 2215
- Could not find font SF Pro Display Light on line(s): 1925 1926 1927 1928 1929
- Requested weight 400 but got 700 for font Avenir LT Std 55 Roman on line(s): 1930 1931 1932 1933 1934 1935 1936
- Font DK nouveau crayon is missing glyphs ンゴちゃん on line(s): 1104
- Font Sazanami Mincho is missing glyphs ὦ on line(s): 911
5 issue(s) found
```

You can also run Font Validator directly on an ASS file and provide a muxed MKV to read fonts from as an additional argument.
This is useful for making the line numbers more accurate, as comments will be moved to the top of the script when muxing.

```
$ fontvalidator script.ass video.mkv
```

Alternatively, you can provide a list of directories containing fonts, or individual font files.

### Installation

Install Font Validator with pip:

```
pip install --user git+https://github.com/TypesettingTools/Myaamori-Aegisub-Scripts/#subdirectory=scripts/fontvalidator
```

This will add a `fontvalidator` executable along with the `fontvalidator.py` Python module.
If the executable is not in your path, you can run Font Validator as `python -m fontvalidator --help`, or alternatively `py -3 -m fontvalidator --help` on Windows.

### Common issues and how to fix them

#### Missing fonts

```
Could not find font [...] on line(s): [...]
```

In most cases this means you forgot to include a font when muxing.
To solve it, simply include the missing font.

This can also happen if you specified the name of a font incorrectly in a style definition or in an argument to an `\fn` tag.
libass supports using family names and full names for TrueType fonts, and family names and the PostScript name for OpenType fonts.
Use a tool such as `fc-scan` or `Font Info > PS/TTF Names` in FontForge to list the available names for a font, and ensure that you are using a valid name in the script.

#### Missing glyphs

```
Font [...] is missing glyphs [...] on line(s): [...]
```

This happens when you are using characters not supported by the current font.
This will cause the renderer to use a fallback font, or simply fail to render the characters completely.
To solve this, either ensure you are only using supported characters, or switch to another font that does support the characters in question.

#### Faux bold/italic

```
Faux bold used for font [...] (requested weight 700, got 400) on line(s): [...]
Faux italic used for font [...] on line(s): [...]
```

This happens when you have requested a bold or italic font but only a non-bold/non-italic font from the same family is available.
This will cause the renderer to use faux bold/italic, where it emulates bold/italic by simply making the base font thicker/slant.

This is not necessarily an issue, but it is generally preferable to use a true bold/italic font if one is available for the font family in question, as it will generally look better.
Additionally, if you do not mux the true bold/italic font, the result may end up looking slightly different if the user playing the file has it installed.

Another reason you may see this warning is if you are using a very thin font, e.g. one with weight 100.
Since non-bold (`\b0`) corresponds to weight 400 in ASS, the renderer will still apply faux bold even if you did not explicitly request a bold font.
To avoid this, explicitly specify the exact weight of the font you want to use, e.g. `\b100`.

#### Mismatched weight/slant

```
Requested weight 400 but got 700 for font [...] on line(s): [...]
Requested non-italic but got italic for font [...] on line(s): [...]
```

This happens when you requested e.g. a non-bold font, but only a bold version was available.
This generally does not affect the rendering of the font, but if the user has a better-matching font installed, they may end up with a different result.
To avoid this, simply specify the font style explicitly, by including e.g. `\b1` (or `\bX` where X is the weight of the font) for bold fonts or `\i1` for italic fonts.
