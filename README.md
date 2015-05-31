# TopoKanji

> **15 seconds explanation**: for people learning Japanese as a second language, learning kanji in "visual decomposition" order (e.g. `一 → 二 → 三`, `言 → 五 → 口 → 語`, etc.) is better than in the order taught by [JLPT][] levels or in Japanese schools. This project provides such lists of kanji.

The goal of TopoKanji project is to provide people who want to learn Japanese [kanji][] with a properly ordered list of kanji which makes the learning process as fast, simple, and effective as possible.

Sample (first 100 kanji from [lists/aozora.txt](lists/aozora.txt)):

    人一乙丿丶亅丨二十入
    大了子力上八儿七九乂
    又丁刀冂囗口日乃几卜
    匕厶厂勹冫凵匚冖亠卩
    匸亻刂山出女三下小分
    中方目見扌手川士土之
    千万工久勺夕己丸心寸
    丈亡今凡才干乞也弓巾
    刃尸巳廴于广彳兀彡宀
    个幺廾弋巛夂彑屮忄阝

Final lists can be found in [`lists` directory](lists). Lists are only differ in order of kanji. Each file contains kanji, grouped by 10 per line, starting from simplest.

You can use them to build an [Anki][] deck or just as a guidance.

- Use `aozora.txt` if you're learning Japanese language primarily to be able to read Japanese novels.
- Use `wikipedia.txt` if your goal is to be able to read documents in Japanese.

I'm planning to build more lists later.

## What is a properly ordered list of kanji?

If you look at a kanji like 語, you can see it consists of at least three distinct parts: 言, 五, 口. Those are kanji by themselves too. The idea behind this project is to find the order of about 2000-2500 common kanji, in which no kanji appears before its' parts, so you only learn a new kanji when you already know its' components.

### Properties of properly ordered lists

1. **No kanji appear before it's parts (components).** In fact, in you treat kanji as nodes in a [graph][] structure, and connect them with directed edges, where each edge means "kanji A includes kanji B as a component", it all forms a [directed acyclic graph (DAG)][dag]. For any DAG, it is possible to build a [topological order][topsort], which is basically what "no kanji appear before it's parts" means.
2. **Simpler kanji come first.** From my personal experience, it is very important for people who study [logographic characters][logogram] for the first time.
3. **More frequently used kanji come first.** That way you learn useful characters as soon as possible.

### Algorithm

The last two properties above are contradicting: some simple kanji are pretty rare, while some 10+ strokes kanji are ubiquitous. To deal with this, I modified a [topological sorting algorithm][topsort] so that both factors add to the "weight" of each character: the more complex and rare a kanji is, the less likely it appears in the beginning of a list. 

The weighting formula is simple: `Weight = Complexity * Rarity`, where `Complexity` and `Rarity` are real numbers in range `[0, 1]`. Weight is used for sorting intermediate list of kanji during topological sorting.

Topological sorting is done by using a modified version of [Kahn (1962) algorithm][kahn] with intermediate sorting step. See source code for details.

## Used data

- Initial unsorted list contains only kanji which are present in [KanjiVG][] project, so for each character there is a data of its' shape and stroke order
- Characters are split into components using [CJK Decompositions Data][cjk] project, along with "fixes" to simplify final lists and avoid characters which are not present in initial list
- Statistical data of kanji usage frequencies was gathered by processing data from various sources:
  - [Wikimedia Downloads][wiki-dumps] - snapshot of all pages and articles of Japanese Wikipedia
  - [Aozora Bunko][aozora] - large collection of Japanese literature

## Which kanji are in the list

Initial list contain only common kanji, because it is build for people who just started learning kanji.

Kanji is considered "common" if:

- it is among the 2000-3000 most frequently used (according to some statistical data)
- at least one of the following conditions are met:
  - it is among the [Jouyou kanji][jouyou]
  - it is a [Kangxi radical][kangxi], but not a complex (by the number of strokes) one
  - it is used in common words (a tricky condition, don't rely just on this)

## Files and formats

Files in `lists` directory are final lists. They contain kanji grouped by 10 per line.

Files in `data` directory:

- `kanji.txt` - list of kanji characters included in final ordered lists
- `kanjivg.txt` - list of kanji from [KanjiVG][]
- `cjk-decomp-{VERSION}.txt` - data from [CJK Decompositions Data][cjk], without any modifications
- `cjk-decomp-override.txt` - data to override some CJK's decompositions
- `kanji-frequency/*.json` - kanji frequency tables

All files are encoded in UTF-8, without [byte order mark (BOM)][bom]. All files, except for `cjk-decomp-{VERSION}.txt`, have unix-style [line endings][eol], `LF`.

### kanji.txt

Contains initial list of kanji.

- Each character must be listed in `kanjivg.txt`
- Each character must be listed in one of `cjk-decomp-*.txt` files
- Each character must appear on a line which number is equal to a number of strokes in the character
- No extra whitespace or any other symbols, comments, etc.

### kanjivg.txt

Simple list of characters which are present in KanjiVG project. Those are from the list of *.svg files in [KanjiVG's Github repository][kanjivg-github].

### cjk-decomp-{VERSION}.txt

Data file from [CJK Decompositions Data][cjk] project, see [description of its' format][cjk-format].

### cjk-decomp-override.txt

Same format as `cjk-decomp-{VERSION}.txt`, except the only purpose of each pictorial configuration record here is to override the one from `cjk-decomp-{VERSION}.txt`. The type of decomposition is always `fix`, which just means "fix a record for the same character from original file".

Special character `0` is used to distinguish invalid decompositions (which lead to characters with no graphical representation) from those which just can't be decomposed further into something meaningful. For example, `一:fix(0)` means that this kanji can't be further decomposed, since it's just a single stroke.

### kanji-frequency/*.json

Kanji usage frequency data in [JSON][] format. Each file contain an array of arrays (rows). Each row contains three fields:

1. (string) Kanji itself. `"all"` is a special case in the first row.
2. (integer) How many times it was found in the analyzed data set. For `"all"` it is a total number of kanji, including repetitions.
3. (float) Fraction of total amount of data this character represents. For `"all"` it is `1` (e.i. 100%).

Currently present data:

- `aozora.json` - data from [Aozora Bunko][aozora], about 12900 files containing various works of Japanese literature (May 2015)
- `wikipedia.json` - data from [Wikimedia Downloads][wiki-dumps], full snapshot of Japanese Wikipedia (May 2015)

## Usage

You must have Node.js and Git installed

1. `git clone https://github.com/THIS/REPO.git`
2. `npm install`
3. `node build.js`

### Testing data modifications

    $ node build.js

Does not change any files, only validates the data and writes to terminal.

Options:

- (optional) `--chars-per-line=`, positive integer, default is `50`.
  Number of characters per line to display when printing a sorted list.
- (optional) `--use-freq-table=`, possible values: `aozora` (default), `wikipedia`.
  Name of kanji frequency table to use.

### Building final lists

    $ node build.js --override-final-lists

This will override final lists in `lists` directory.

## Contributing

Consider *not* adding or removing kanji from the initial list. It is made for beginners, so don't include uncommon kanji or kanji used primarily in names. Remove kanji from the initial list *only* if you sure it's not common by any criteria.

All other contributions, suggestions, fixes, and requests are welcome!

## License

This is a multi-license project. Choose any license from this list:

- [CC-BY-4.0](http://creativecommons.org/licenses/by/4.0/)
- [MIT](http://opensource.org/licenses/MIT)
- [Apache 2.0](http://www.apache.org/licenses/LICENSE-2.0)
- [LGPL 3.0](http://www.gnu.org/licenses/lgpl-3.0.html)
- [ODC-By 1.0](http://opendatacommons.org/licenses/by/1.0/)
- [Eclipse 1.0](https://www.eclipse.org/legal/epl-v10.html)

[jlpt]: http://www.jlpt.jp/e/
[anki]: http://ankisrs.net/
[kanji]: https://en.wikipedia.org/wiki/Kanji
[graph]: https://en.wikipedia.org/wiki/Graph_(mathematics)
[dag]: https://en.wikipedia.org/wiki/Directed_acyclic_graph
[topsort]: https://en.wikipedia.org/wiki/Topological_sorting
[logogram]: https://en.wikipedia.org/wiki/Logogram
[kahn]: http://dl.acm.org/citation.cfm?doid=368996.369025
[wiki-dumps]: https://dumps.wikimedia.org/
[aozora]: http://www.aozora.gr.jp/
[jouyou]: https://en.wikipedia.org/wiki/J%C5%8Dy%C5%8D_kanji
[kangxi]: https://en.wikipedia.org/wiki/Kangxi_radical
[kanjivg]: http://kanjivg.tagaini.net/
[kanjivg-github]: https://github.com/KanjiVG/kanjivg
[cjk]: https://cjkdecomp.codeplex.com/
[cjk-format]: https://cjkdecomp.codeplex.com/wikipage?title=cjk-decomp
[json]: http://json.org/
[bom]: https://en.wikipedia.org/wiki/Byte_order_mark
[eol]: https://en.wikipedia.org/wiki/Newline