# MakeOTF OpenType/CFF 编译器：用户指南

### **翻译须知**
font 根据上下文可能会翻译为「字体」和「字库」

## **概览**

MakeOTF 是创建 OpenType® 字体的工具，它需要用到源字体文件，以及包含 OpenType 布局特性的上层信息。我们将其设计为命令行工具——也就是 macOS® 或 Windows® 终端里面的一条指令。请注意，MakeOTF 是「字库数据的编译器」，而非「字体编辑器」。

MakeOTF 需要多个源文件，这些文件均可在 makeotf 指令的选项中指定：
  * **`font`（字库）**：通常命名为 `font.pfa` 或 `cidfont.ps`，可以是 Type1/CID 字体文件、TrueType 字体文件、或是 OpenType/CFF 字体文件。请注意，MakeOTF 只抽取源字库的字形轮廓。

  * **`features`（特性）**：文本文件，包含了用户定义的 OpenType 布局特性规则，还指定了 OpenType 表中某些字段的值。这些指定的值会覆盖掉 MakeOTF 计算的默认值。

  * **`FontMenuNameDB`（字体菜单名）**：文本文件，提供了该字体在 Windows 和 macOS 菜单中显示的文字。

  * **`GlyphOrderAndAliasDB`（字形顺序与代号）**：此文件的用途有三，其一，在字体文件内部，编排字形的编号顺序。其二，对于输出的 .otf 字库文件，允许其字形命名异于输入的源字体数据。这允许开发者在开发字体时使用更友好的名字，并使用不同于最终输出的 OpenType 文件中的名字。其三，提供各字形的 Unicode® 码位。默认情况下 MakeOTF 会提供某些字形的码位，但不会提供全部字形的。

  * 非必需的文件 **`fontinfo`**，若存在，MakeOTF 会检查其中的关键词，并设定某些特定的选项。

为避免每次构建 OpenType 字体时，都得输入选项来指定必要的文件，MakeOTF 可以保存一个字体工程文件，该文件存储了所有需要的选项。MakeOTF 总是会将最终使用的选项尽数写入一个名为 `current.fpr` 的文件，该文件的目录与输入的字体文件相同。（项目文件也可以用其他名字。）

使用 `makeotf` 命令时可以加入选项来设置参数，以改变 MakeOTF 构建 OpenType 字体的细节。若选项有冲突，命令行上的最后一个选项将覆盖任何先前与之冲突的选项。

## **使用 MakeOTF**

MakeOTF 分为两个部分，二者相互独立，调用时互不干扰：
  * **`makeotfexe`** 是个 C 语言程序，是事实上真正构建 OpenType 字体的程序，但得在命令行中输入选项，来明确地选取所有的源文件。

  * **`makeotf`** 是个命令行的 shell，它会调用 Python™ 脚本 **`makeotf.py`**。它将在一些标准位置查找源文件，并为选项填写默认值。它也可以从一个项目文件(`current.fpr`)中读取所需的选项，这样子在每次构建字体时，免去了反复输入所有选项的麻烦。当 **`makeotf.py`** 收集了所有的源文件后，它将调用 **`makeotfexe`** 程序。

调用 **`makeotf.py`** 时一般得用 makeotf 命令，这样子可以将前一次的选项记录在工程文件中。

使用 MakeOTF 的第一步是整理各个源文件。不妨将诸文件组织在目录树中：

```
MyFontFamily/
├── FontMenuNameDB
├── GlyphOrderAndAliasDB
├── Italic/
│   ├── MyFont-BoldItalic/
│   │   ├── features
│   │   ├── features.kern
│   │   └── font.pfa
│   ├── MyFont-Italic/
│   │   ├── features
│   │   ├── features.kern
│   │   └── font.pfa
│   └── features.family
└── Roman/
    ├── MyFont-Bold/
    │   ├── features
    │   ├── features.kern
    │   └── font.pfa
    ├── MyFont-Regular/
    │   ├── features
    │   ├── features.kern
    │   └── font.pfa
    └── features.family
```
这样编排的理由是，一些数据——如「字体菜单名」和「字形顺序与代号」文件——将在字族的所有成员之间共享，而其他数据将只适用于字族的其中一部分字族。将公用数据放置在目录树的较高层次，可以避免同一数据有多个副本，这样子，在数据需要更改时，需要编辑的文件会减少。这对于 `features.family` 文件尤为重要。

一般来说，对于不同的字库，其字形定位规则（如 kerning `features.kern`）一般不同，而字形替换规则却几乎相同。实践中有个不错的做法是，给字族中每个成员分别使用一个特性文件，跟源字库文件一起放在同一个目录下；对于放在上层目录的特性文件，则使用「include 陈述」来引用。上例中，在罗马体和意大利体的字族目录下，有两个独立的 `features.family` 字族特性文件，可为相应字族下属的各字体共用。要搞两个不同的字族特性文件，是因为罗马体和意大利体的字形替换（GSUB）规则不太一样——意大利体的应该多一点。

整理好所有必要的文件后，下一步是打开终端命令行，并使用 `cd` 指令，将当前的工作目录切换至编译字体的源文件所属的目录：
```bash
cd <path to font directory>
```
然后输入 `makeotf` 指令，以及必要的选项。举个例子：
```bash
makeotf –f myfont.pfa –ff myfeatures –b –r
```
或
```bash
makeotf –fp myproject.fpr
```

## **MakeOTF 选项**

| 选项 | 设置 | 行为描述 |
|--------|---------|-------------|
|`–fp`| `<文件路径>` | 指定工程文件的路径。若不输入该选项，则路径默认为 `current.fpr`。MakeOTF 将从该项目文件中读取选项。项目文件应该只包含异于默认值的值。`-fp` 选项可以和其他选项一起使用，但必须放在第一位；后面的选项将覆盖从项目文件中读取的选项。例如，`-fp release.fpr -o test.otf` 将使用 `release.fpr` 项目文件中的所有选项来构建 OpenType 字体，但成品会输出至 `test.otf` 而 非 `<PostScript 名称>.otf`。|
|`–f` | `<文件路径>` | 指定输入字体的路径。未给路径时，MakeOTF 会假定文件名为 `font.pfa`。|
|`–o` | `<文件路径>` | 指定输出字体的路径。未给路径时，MakeOTF 会假定文件名为 `<PostScript 名称>.otf`。|
|`–b` | | 将风格设置为粗体（Bold）。会影响字体风格关联（style-linking）。若不输入该选项，MakeOTF 会假定该字形不是粗体。|
|`–i` | | 将风格设置为意大利体（Italic）。会影响字体风格关联（style-linking）。若不输入该选项，MakeOTF 会假定该字形不是意大利体。|
|`–ff` | `<文件路径>` | Specify path to features file. If not provided, MakeOTF assumes the file name is features.|
|`-gs` | | Omit any glyphs that are not named in the GOADB file. Works only if either the -ga or -r options is specified.|
|`–mf` | `<文件路径>` | Specify path to FontMenuNameDB file. If not provided, MakeOTF will look in the current directory for a file named FontMenuNameDB, and then one directory up, and finally two directories up.|
|`–gf` | `<文件路径>` | Specify path to GlyphOrderAndAliasDB file. If not provided, MakeOTF will look in the current directory for a file named GlyphOrderAndAliasDB, and then one directory up, and finally two directories up. Also, if this option is specified, and the `–r` or `–ga` options are NOT specified, the effect is to use the Unicode assignments from the third column of the GOADB without renaming the glyphs.|
|`–r` | | 设定为「正式发布」模式。这一选项会开启「子过程化」（subroutinization）——将字库中抽取各字形的公共部分；应用「字形顺序与代号」文件；对于 name 表，将其 Version 字符串（Name ID 5）中的 Development 字眼去除。|
|`–S` | | Turn on subroutinization.|
|`–ga` | | Apply the GlyphOrderAndAliasDB file. Use when the `–r` option is NOT specified.|
|`-rev` | `[<数值>]` | 尝试在 makeotfexe 运行前先编辑特性文件,将 head 表的 fontRevision 值域自增。这一选项起作用的前提条件是，对 head 表的覆盖已在特性文件中先行定义。版本号是可选的，若选项中未填写版本号，则将它自增 5；若给了整数参数，则它自增的大小为该参数。如果给了比值参数，则会将版本号*设定*为该比值（这里不是自增！），保留三位小数（十进制）。|
|`–osbOn` | `<数值>` | Turn on the specified bit number(s) in the OS/2 table fsSelection field. In order to turn on more than one bit, must be used more than once. `–osbOn 7 –osbOn 8` will turn on bits 7 and 8. See section below on New OS/2 Bits.|
|`–⁠osbOff` | `<数值>` | Turn off the specified bit number(s) in the OS/2 table fsSelection field. Can be used more than once to turn OFF more than one bit at a time. `–osbOff 7 –osbOff 8` will turn off bits 7 and 8. See section below on New OS/2 Bits.|
|`-osv` | `<数值>` | Set version number of the OS/2 table. The default value is 3 if none of the bits specified only in version 4 and later are used; otherwise, the default version is 4. See section below on New OS/2 Bits.|
|`-addn` | | Replace the `.notdef` glyph in the source data (if any) with a standard `.notdef` glyph, that will match the font’s weight and width.|
|`-adds` | | Create any Apple Mac Symbol glyphs missing from the font. Added glyphs will match the font’s weight and width.|
|`-serif` | | Specify that any added glyph will use the serif Multiple Master built-in glyph data.|
|`-sans` | | Specify that any added glyphs will use the sans-serif Multiple Master builtin glyph data.|
|`-cs` | | Override the heuristics, and specify the script ID for the Mac cmap subtable.|
|`-cl`| | Override the heuristics, and specify the language ID for the Mac cmap subtable.|
|`-cm` |`<文件路径>`| Specify path to CID CMap encoding file for the font Mac encoding. (CID-keyed fonts only)|
|`-ch`|`<文件路径>`| Specify path to CID CMap encoding file for the font Unicode UTF-32 encoding for horizontal glyphs. (CID-keyed fonts only)|
|`-cv`|`<文件路径>`| Specify path to CID CMap encoding file for the font Unicode UTF-32 encoding for vertical glyphs. (CID-keyed fonts only)|
|`-ci`|`<文件路径>`| Specify path to Unicode Variation Sequence specification file.|
|`-dbl`| |Map glyph names to two Unicode values rather than one. This was the default behavior of makeotf in FDK 1.6 and earlier. The Adobe Type Department now discourages this practice. The option exists only to allow building fonts that match original versions. See `makeotf –h` for the hard-coded list of glyphs.|
|`-dcs`| |Set OS/2.DefaultChar to the Unicode value for `space`, rather than `.notdef`. The latter is correct by the OT spec, but QuarkXPress 6.5 requires the former in order to print OTF/CFF fonts.|
|`-fi` |`<文件路径>`| Path to the `fontinfo` file. If no path is given, the default is to look for first `fontinfo`, then `cidfontinfo`, in the current directory. Used to set some default values. This are overridden by any conflicting settings in the project file and then by command line options. This option is processed before any others, so if the path is relative, it is relative to the current working directory. All other relative paths are relative so the source font’s parent directory.|
|`-sp`|`<⁠文⁠件⁠路⁠径⁠>`|Save the current options to the file path provided, as well as to the current.fpr file.|
|`-nb`| |Turn off the Bold style. Can be used to override a project file setting, otherwise has no effect.|
|`-ni`| |Turn off the Italic style. Can be used to override a project file setting, otherwise has no effect.|
|`-nS`| |Turn off subroutinization.|
|`-nga`| |Turn off applying the GlyphOrderAndAliasDB file|
|`-naddn`| |Turn off adding a standard .notdef. Can be used to override a project file setting, otherwise has no effect.|
|`-nadds`| |Turn off adding Apple symbol glyphs. Can be used to override a project file setting, otherwise has no effect.| 

当存在多个选项时，选项是否被应用，取决于选项的先后顺序：`–r –nS` 不会将字体子程序化，但 `–nS –r` 会。参数的读取顺序是：先读取 fontinfo 文件的键值对，然后读取指定的项目文件，最后，从左往右读取命令行里的选项。后读取的参数会覆盖先读取的。

「子程序化」是一个作用于字库中各字形的过程，它会把各字形中的共同元素分解为单独的「子程序」。这可以减少最终字体的大小，但对于大型字体（如 CID 字体）可能需要极多的内存和时间。MakeOTF 对于大型的罗马字体可能需要 64 MB 的内存，只需要 32 MB 就可以完成大部分的工作，但是它可能需要768 MB 的内存来对一个5 MB 的 CID 字体进行子程序化。如果全部在内存中完成，子程序化的速度相当快：罗马字体几分钟，CID 字体半小时到三小时，但如果系统不得不使用虚拟内存，所需时间可能会增加 20 倍以上。子程序化对罗马字体和韩文 CID 字体一般会有用；而至于日文和中文 CID 字体，由于重复的路径元素较少，使用子程序化一般只能减少百分之几的大小。

## 字体菜单名（FontMenuNameDB）：第二版

> Previous versions of MakeOTF used a different version of the `FontMenuNameDB` file, and wrote the Macintosh font menu names differently than the Windows font menu names, and not according to the OpenType spec. This is because of some history of the early efforts to get OpenType fonts working on the Mac OS. However, for some years Apple has been following the OpenType spec when making Apple OpenType fonts, and has fully supported the OpenType font menu names. As a result, this version of MakeOTF has implemented new syntax for the `FontMenuNameDB`, and will create the name table according to the OpenType spec when this new syntax is used.

> Fonts made with earlier versions of MakeOTF will not be disadvantaged, as all Adobe fonts to date and many third-party fonts were made this way, and all programs look first to the Windows font menu names when they exist, as this is where the style-linking names can most reliably be found.

> The earlier version of the `FontMenuNameDB` may still be used. The main reason to change is that the newer version is easier to explain and understand.

「字体菜单名」文件用来指定字体在菜单中的名称信息。它是文本文件，可能包含多个字体的信息。每个字体的条目从 `[<PostScript 名称>]` 开始，后面几行是各种各样的菜单名。这些名称会用来构建 name 表的字串，以描述该字体。这使得我们可以将所有字体的菜单名统统放进一个单独的文件，以便在整个字族中找出各个菜单名。

|**字体条目描述**|
|--------------------------|
|`[<PostScript 名称>]`|
|`f=<优先选择的字族名称>`|
|`s=<子字族名称>` （又称：风格名称）|
|`l=<兼容的字族菜单名称>`|
|`m=1,<Macintosh 兼容全名>`|

「字体菜单名」中各条目都至少需要包含 `<优先选择的字族名称>`，用“关键字 `f=` 再加字族名”来指定。

这些字体在很多程序里面，会以分级菜单的形式列出——「字族名称」为第一层；然后把字族名称相同的字体，统统纳入弹出的「风格名称」列表中。另外某些程序则是扁平化的菜单，把所有字体按照「全名」列出来—— `makeotf` 会将「字族名称+空格+风格名称」连接成为「全名」。

However, additional names may need to be supplied. This happens because style-linking (selecting another font of the same family by applying Bold or Italic options to the current font) only allows for families to have up to four members. This means that only four fonts can share the same Family name. If the family has more than four members, or has members which are not style-linked, it is then necessary to divide the family into sub-families which are style-linked, and to provide each sub-family with a unique compatible family menu name.

Only fonts that share the same compatible family menu name can be style-linked, and each 4-element group can only contain one font of each of the styles *Regular, Bold, Italic*, and *BoldItalic*.

The compatible family menu name is specified with the key `l=`, followed by the name. Whenever this name needs to be defined, a compatible Full name for the Mac must also be specified. The Macintosh compatible menu name is specified with the key `m=1,`, followed by the name. This name is normally built by concatenating the compatible family menu name, a space character, and the appropriate choice of one of the four supported styles.

Although these naming conventions are necessary for correct style-linking, they are not sufficient. The font style flags for *Bold* or *Italic* must also be set correctly. This can be done either with the makeotf options `–b` and `–i`, or by providing a fontinfo file located in the same directory as the font and which contains the key/value pairs `IsBoldStyle true` and `IsItalicStyle true`, with `true` or `false` being set appropriately.

Compatible family menu names are also used in font menus by applications that are not OpenType savvy.

If the family only has four faces that are meant to be style-linked, then the compatible and preferred family menu names are the same, and only the `f=` entry is needed. The names specified with the key `f=` will then be written to the name table Name ID 1. If there is a compatible family name entry which differs from the entry supplied with the key `f=` , then the `f=` family name is written to Name ID 16. The compatible name from l= is then written to name table Name ID 1, and the name table Name ID 2 is chosen by makeotf to be one of Regular, Bold, Italic, or BoldItalic, based on the font’s style flag. The value set by `m=1,` is written to the Mac platform name table Name ID 18.

The key `s=` only needs to be used when you want a style name which is different than the choice from the styles *Regular, Bold, Italic*, and *BoldItalic* that would be dictated by the font’s style. When used, and different than the default style name, the style name is written to name table NameID 17. Otherwise, it is written to name table NameID 2.

For complete description of these issues, please read the [OpenType specification section on the name table](https://docs.microsoft.com/en-us/typography/opentype/spec/name).

### **Examples**

|**Regular font of Adobe® Garamond® Pro**|
|----------------------------------------|
|`[AGaramondPro-Regular]`|
|`f=Adobe Garamond Pro`|
|`s=Regular`|

The `l=` key is not used, so the compatible family menu name is the same as the Preferred Family name. The `m=1,` key is not used, so the Macintosh compatible menu name built by MakeOTF will be Adobe Garamond Pro Regular. The key `s=` is not actually necessary.

|**Bold font of Adobe Garamond Pro**|
|-----------------------------------|
|`[AGaramondPro-Bold]`|
|`f=Adobe Garamond Pro`|
|`s=Bold`|

The `l=` key is not used, so the compatible family menu name is the same as the Preferred Family name. This font will be style-linked with the Regular, because they share the same compatible family
menu name.

|**Italic font of Adobe Garamond Pro**|
|-------------------------------------|
|`[AGaramondPro-Italic]`|
|`f=Adobe Garamond Pro`|
|`s=Italic`|

Same as Bold, above.

|**Semibold font of Adobe Garamond Pro**|
|---------------------------------------|
|`[AGaramondPro-Semibold]`|
|`f=Adobe Garamond Pro`|
|`s=Semibold`|
|`l=Adobe Garamond Pro Sb`|
|`m=1,Adobe Garamond Pro Sb`|

This font needs to be part of a new style-linking subgroup. This means the key `l=` is used to set a compatible family menu name different from Preferred Family name. In consequence, a Macintosh compatible Full name is also assigned with the key `m=1,`. The default style will be Regular. The compatible family menu names are abbreviated to Adobe Garamond Pro Sb rather than Adobe Garamond Pro Semibold, in order to keep the menu names, of the remaining fonts belonging to this style-linked subgroup, within the 31-character limit. The key `s=` had to be used, as the preferred style name is not “Regular”, the style-linking style.

|**Semibold Italic font of Adobe Garamond Pro**|
|----------------------------------------------|
|`[AGaramondPro-SemiboldItalic]`|
|`f=Adobe Garamond Pro`|
|`s=Semibold Italic`|
|`l=Adobe Garamond Pro Sb`|
|`m=1,Adobe Garamond Pro Sb Italic`|

This font is part of the same style-linking subgroup as the font in the previous example. In order to be style-linked, they share the same `l=` key, but the default style will be set to Italic in this case. The Macintosh compatible Full menu name was be built by appending the default style to the compatible family menu name.

### **Note on length restrictions**

Because of limitations in operating systems and common applications, it is recommended that none of these keys contain names longer than 31 characters. This results in menu names for legacy environments being 31 characters or less.

The OpenType menu name (used in Adobe InDesign® and future Adobe applications) may be thought of as the combination of the `f=` and `s=` keys. Since each of these can be up to 31 characters, the OpenType menu name can be up to 62 characters.

### **Note on accented/extended characters**

To use accented characters in Latin menu names, one needs two `f=` entries: one for Windows, and one for Mac. The Windows entry specifies the character by Unicode. The Mac entry specifies it by the hex value of the Mac char code from the font’s mac cmap table, when different from the ASCII value. The Unicode/character code value is preceded by a backslash (`\`):

|**Arnold Böcklin**     ||
|-----------------------|---|
|`f=Arnold B`**`\00f6`**`cklin` | Windows |
|`f=1,Arnold B`**`\9a`**`cklin` | Mac     |

`00f6` is the hex representation of the Unicode for odieresis (`ö`). Unicode hex values must have 4 digits. `9a` is MacRoman encoding for `odieresis`. Mac char code hex values must have 2 digits for Western script encodings, and may have 2 or 4 digits for CJK encodings.

In general, the `f=`, `s=`, and `l=` can be extended to set non-Latin strings by adding the triplet (platform code, script code, language code) after the equal sign (=). The values are the same as described for the name table name ID strings. For example:

|**#### 小塚明朝 Pro (Kozuka Mincho® Pro)**||
|-----------------------------------------|---|
|`[KozMinPro-Bold]`                       ||
|`f=3,1,0x411,\5c0f\585a\660e\671d Pro`   | Windows, Unicode, Japanese |
|`f=1,1,11,\8f\ac\92\cb\96\be\92\a9 Pro`  | Mac, Japanese, Japanese |

## **字形顺序与代号（GOADB）**

GOADB文件用于重命名和建立字体中字形的顺序。它是个简单的文本文件，每个字形名称占一行。每一行至少包含两个字段，可以再放第三个字段；一行中的字段用制表符隔开（技术上讲，任何数量的空格都可以，但首选是一个ASCII TAB）。空行将被忽略。以 `#` 开头的行是注释，也会被忽略。第一个字段是要在输出字体中使用的最终字形名称。第二个字段是源字体数据中使用的「友好」名称。第三个字段是 Unicode 值，以 `uniXXXX` 或 `uXXXX[XX]` 的形式指定（见[注释](#unicode_note)）。可以通过给出一个逗号分隔的值列表为一个字形指定多个Unicode值，例如：`uni0020,uni00A0`。`XXXX` 十六进制值**必须**是数字（0-9）或大写字母。包含小写字母的值将被忽略。不要求源字体在此文件中有任何已命名的字形。

<a name="unicode_note"></a>
Note: Unicode values can be used in the form `uniXXXX` or `uXXXX[XX]` where `XXXX[XX]` is a hexadecimal Unicode value. The number of `X` is determined by the codepoint. For example, `U+0903` can be written as either `uni0903` or `u0903`. If the codepoint requires 5 or 6 digits, for example `U+F0002` or `U+F00041`, then the format must contain the same number of digits: `uF0002` or `uF00041`. This only applies when assigning Unicode values using column 3.


It should be noted that the ordering, renaming, and Unicode override operations are applied only if the `–r` option or the `-ga` option is specified. These operations work as follows:

  1) Establish a specific glyph order in the output OpenType font. Any glyph names in *Appendix A – Standard Strings* of [Adobe’s Technical Note #5176](http://wwwimages.adobe.com/www.adobe.com/content/dam/acom/en/devnet/font/pdfs/5176.CFF.pdf), The Compact Font Format Specification, will be ordered first in the font, in the order given in the CFF specification. All other glyphs will be ordered as in the GOADB file. Glyphs not named in either the CFF specification nor in the GOADB file, will be ordered as they were in the original source font.

  2) Rename glyphs in the source font to different names in the output OpenType font. Note that both the source font file and the features definition file must use the same glyph names – one cannot use a source font file with development names, together with a features file that contains final names, unless the options `–r` or `–ga` are used.

  3) Override the default Unicode encoding by MakeOTF. MakeOTF will assign Unicode values to glyphs in a non-CID font when possible. (For a CID font, the Unicode values are provided by the Adobe CMap files.) The rules used are as follows:

      a) If the third field of the GOADB record for a glyph contains a Unicode value in the form uniXXXX or uXXXX\[XX\] (see [note](#unicode_note)), assign that Unicode value to the glyph. Else b);
      
      b) If a glyph name is in the [Adobe Glyph List For New Fonts](https://github.com/adobe-type-tools/agl-aglfn/blob/master/aglfn.txt), use the assigned Unicode value. Else c);
      
      c) If the glyph name is in the form uniXXXX or uXXXX\[XX\] (see [note](#unicode_note)), assign the Unicode value. Else d);

      d) Do not assign any Unicode value.

Note that MakeOTF cannot re-order glyphs when the source font is a TrueType or OpenType/TTF font: the glyph order in the source font and the glyph order in the `GlyphOrderAndAliasDB` file must be the same. It can, however, still rename glyphs and assign Unicode values.

Note that MakeOTF no longer assigns glyphs Unicode values from the Private Use Area (PUA) block. If such Unicode values are needed, they must be specified in a `GOADB` file.

## **字体信息（fontinfo）**

字体信息文件是个简单的文本文件，包含键值对。每行有两个用空格隔开的字段，前者是关键词，后者是值。makeotf 会在源字体文件的目录下寻找字体信息文件，若存在，则以之为默认值。它们可以被项目文件或者 `makeotf` 命令行选项覆盖。makeotf 支持的键值如下：These values will be overridden if they are also set by a project file, and then by any `makeotf` command line options.

| Keyword | Values | Effect |
|---------|--------|--------|
|`IsBoldStyle`|`true/false` | Set the font style to Bold. Same as the command line option `–b`|
|`IsItalicStyle`|`true/false` | Set the font style to Italic. Same as the command line option `–i`|
|`PreferOS/2TypoMetrics`|`true/false` | Set the OS/2 table fsSelection bit 7, `USE_TYPO_METRICS`. Same as the command line option `–osbOn 7`.|
|`IsOS/2WidthWeigthSlopeOnly`|`true/false` | Set the OS/2 table fsSelection bit 8, `WEIGHT_WIDTH_SLOPE_ONLY`. Same as the command line option `–osbOn 8`.|
|`IsOS/2OBLIQUE`|`true/false` | Set the OS/2 table fsSelection bit 9, `OBLIQUE`. Same as the command line option `–osbOn 9`.|

## **Adobe 字符码位映射表（CMap）**
A CMap file maps character codes to glyph selectors. It is only necessary for CID fonts. This file may be located anywhere within the file system, as long as the directory path is stored in the `FontEnvironment.txt` file, or the file is chosen explicitly via the UI. The default CMap files are automatically selected by MakeOTF when the `cidfontinfo` file is selected.

The Mac CMap file provides the encoding for a Mac platform cmap subtable in the OpenType font cmap table. This is required. The script and language of this Mac platform cmap subtable is determined by heuristics in MakeOTF, but can be specified by MakeOTF options. These apply only to CID source fonts.

The Vertical and Horizontal CMap files supply the Unicode encoding for all the glyphs in the font. Note that for CID fonts, only glyphs named in these three files are encoded in any of the OpenType font cmap table subtables. The many alternates can often be accessed only through OpenType layout features.

When looking for default CID CMap files, MakeOTF uses the following rules:
  1) The default CMap files will be in a subdirectory of the default path which is specified in the `FontEnvironment.txt` file. If this is a relative rather than absolute path, it will be assumed to be under FDK/Tools/SharedData/.

  2) The CMap files will be under a directory which is named after the Registry-Order-Supplement from the cidfontinfo file, e.g. Adobe-Japan1-4, Adobe-GB1-4.

  3) The files will be named:

  | for R-O       | Mac CMap    | Unicode H CMap | Unicode V CMap |
  |---------------|-------------|----------------|----------------|
  | Adobe-Japan1: | 83pv-RKSJ-H | UniJIS-UTF32-H | UniJIS-UTF32-V |
  | Adobe-Japan2: | 83pv-RKSJ-H | UniJIS-UTF32-H | UniJIS-UTF32-V |
  | Adobe-GB1:    | GBpc-EUC-H  | UniGB-UTF32-H  | UniGB-UTF32-V  |
  | Adobe-CNS1:   | B5pc-H      | UniCNS-UTF32-H | UniCNS-UTF32-V |
  | Adobe-Korea1: | KSCpc-EUC-H | UniKS-UTF32-H  | UniKS-UTF32-V  |

  If one wants to use a CMap, which would not be found by these assumptions, one must specify them explicitly on the command line while using `makeotf`.

For further information on CMaps, please read the many [Adobe Technical Notes](https://github.com/adobe-type-tools/cmap-resources)

## **〔Unicode 变体选择序列〕文件（UVS File）**
A UVS file is a list of records, each of which specifies the glyph which should be displayed for a given combination of a Unicode encoding value and a Unicode Variation Selector. You can read about Unicode Variation Sequence in the publication “Unicode Technical Standard #37 , Ideographic Variation Database , at http://unicode.org/reports/tr37/. The format depends on the source font format: CID-keyed fonts require three fields in each record, non-CID fonts require only two.

#### For CID-keyed fonts:
The `makeotf` command will look for a UVS file under FDK/Tools/SharedData/Adobe CMAPS/<R-OS>, where <R-O-S> stands for the font’s CID Registry-Order-Supplement. The file will be assumed to be named “`<R-O>_sequences.txt`. For a CID font with the R-O-S “Adobe-Japan1-6”. the default UVS file path would be “FDK/Tools/SharedData/Adobe CMAPS/Adobe-Japan1-6/Adobe-Japan1_sequences.txt“ . The option `-ci <path>` may be used to specify a UVS file path other than the default.

The file format is an ASCII text file. Each line represents one Unicode Variation Sequence(UVS). A line may be blank, or contain a comment that starts with the character “#”.

Each line contains three fields. Fields are separated by a semicolon followed by whitespace. The fields are:
  1) Variation Sequence. Two hexadecimal value, with no leading “x” or “0x, separated by a space. The first is the Base Unicode Value, and the second is the Variation Selector value..
  2) Adobe CID Registry and Order. The CID Registry and Order names, joined by a hyphen.
  3) CID Value. A decimal value, prefixed by “CID+”.

#### Example UVS file:

```
# Registered Adobe-Japan1 sequences
# Version 07/30/2007
# Prepared by Ken Lunde, Adobe Systems Incorporated
3402 E0100; Adobe-Japan1; CID+13698
3402 E0101; Adobe-Japan1; CID+13697
3402 E0102; Adobe-Japan1; CID+13699
3405 E0100; Adobe-Japan1; CID+15387
....
```

#### For non CID-keyed fonts:
The path to the variation sequence file must be specified with the option `-ci <file path>`.
The format for the UV and UVS sequence is the same. However, the Registry-Order field is omitted, and the glyph is identified by a glyph name from the GOADB file.

#### Example UVS file for a nonCID-keyed font:

```
3402 E0100; checkbox
3402 E0101; checkedbox
```

## **New `OS/2.fsSelection` Bits**

In 2006, Microsoft and Adobe began discussions to define three new bit values in the OS/2 table `fsSelection` field: bit 7 `USE_TYPO_METRICS`, bit 8 `WEIGHT_WIDTH_SLOPE_ONLY`, and bit 9, `OBLIQUE`.

These bits have meaning only in OS/2 table version 4 and later; in earlier versions, they are reserved, and should be set off. The Adobe Type Department will set the bits in our new fonts.

#### `USE_TYPO_METRICS`

When bit 7 is on, programs are supposed to use the OS/2 table sTypoAscent/Descent/LineGap for vertical line spacing. This bit was defined because the Windows layout libraries have been using the OS/2 table winAscent/Descent values instead. The next release of Windows Presentation Foundation, on which new versions of Microsoft programs such as Word will be based, and which is due in late 2006, will use the sTypo values instead – but only if this bit is on. This bit certifies that the OS/2 sTypo values are good, and have the side effect of being present only in new fonts, so that reflow of documents will happen less often than if Microsoft just changed the behavior for all fonts. For example, it is rumored that some TrueType fonts have bad values for the sTypo values, making it undesirable to apply the sTypo values for all existing fonts.

The next two bits have to do with the fact that future generations of Microsoft programs, and programs from other vendors that use the Windows Presentation Foundation (WPF) libraries, will use the Cascading Style Sheet (CSS) v.2 specification for specifying fonts. The provisions of this specification can be seen at https://www.w3.org/TR/css-fonts-3/. In brief, a CSS font declaration within a document will describe a font by specifying the family name, and one or more properties; there is no way to refer directly to a specific font. The possible styles are Regular, Italic and Oblique. Weight and width are specified with separate keywords, with a limited and defined set of possible values. An example (in a simplified syntax) is `font-family: TimesStd, style: italic, weight: 700, width: normal`. One requirement of a CSS family is that each font has to have a different set of properties. There cannot be two fonts in a CSS font family which both have the properties `style: italic, weight: 700, width: normal`. For OpenType fonts, the name table Preferred Family Name is used as the CSS family name. Nonetheless, OpenType font families may well contain more than one font with a particular set of properties, but this fact will create a conflict in a CSS-based environment. When WPF encounters one of such families, it will divide the family into groups, so that only one font in each group has a given set of properties. However, that will allow WPF to provide new family names for each group, and these might not correspond to the type designer’s initial intentions.

Similarly, WPF will infer the font’s properties by analyzing its Preferred Family and Style names, and may mistakenly conclude that any two fonts in a family cannot be uniquely assigned different sets of font properties. In this case, WPF will also divide the family into groups, and generate new family names for them. The `WEIGHT_WIDTH_SLOPE_ONLY` bit was added to deal with this latter case. When this bit is set in a font’s OS/2 table fsSelection field, the application is requested to trust that all fonts sharing the same Preferred Family name, will differ from all the remaining with respect to their CSS properties. That given, the application can use the OpenType name table font menu names.

#### `WEIGHT_WIDTH_SLOPE_ONLY`

When bit 8 is on, a program can be confident that all the fonts which share the same Preferred Font Family Name will all differ only in weight, width, and slope. The Preferred Font Family Name is defined as name table Windows platform Name ID 16 Preferred Family Name, or, if name ID 16 is not present, then Windows platform Name ID 1 Family Name. If the OS/2 table version is 4 and this bit is not on, the program will use heuristics to divide the faces with the same Preferred Family Name into CSS family groups, and assign family and property names.
 
Yet another issue is that WPF uses heuristics applied to the font menu names to determine if the font has the Oblique style. The Oblique style can be used for either a font that was designed as a slanted form of a typeface – as opposed to a true italic design – or for a font which has been created by a program by algorithmically slanting a base font. There is no information in current OpenType fonts to indicate whether a font is Oblique. The only way to know if a current OpenType font is Oblique is through the use of heuristics based on analyzing the font name. However, no set of heuristics will be perfect. The Oblique bit was proposed in order to solve this problem, thus providing a way to clearly indicate if the font should be assigned the CSS Oblique style. As an example from the CSS specification, a Regular font within a family could be classified as Oblique just because the word “Inclined” is used in its name.

#### `OBLIQUE`

When bit 9 is on, programs that support the CSS2 font specification standards will consider the font to have the Oblique style, otherwise not. If the document specifies an Oblique style font, and no Oblique font is present within the CSS font family, then the application may synthetically create an Oblique style by slanting the base font. This will occur even if there is an Italic font within the same CSS font family. However, if a document font specification requires an Italic style, but only an Oblique font is available, then the latter will be used. An additional proposal suggests that if this bit is not set and the OS/2 version is 4 or greater, then the application would look for Windows platform names ID 21 and 22. If present, name ID 21 would supply the CSS compatible family name, and name ID 22 would supply the CSS-compatible property name. As of the writing of this document, the status of this proposal is still uncertain – please check the latest OpenType specification. If this proposal is accepted, the additional names can be specified in the feature file.

Note that if any of these three new bits is turned on, with the option `–osbOn <n>`, then `makeotf` will require that all three values be explicitly set to be on or off. This is because, if any of new the bits is set on, `makeotf` will by default set the OS/2 version number to 4. When the version number is 4, then both the on and the off setting has a specific meaning for each bit.

## **Synthetic Glyphs**

MakeOTF includes two Multiple Master fonts built-in, one serif and one sans-serif. With these it can synthesize glyphs that match (more or less) the width and weight of the source font. It requires the glyphs zero and O to be present in the font, in order to determine the required weight and width. If the option `–adds` is used, the list of glyphs to generate will be derived from the concatenation of the following three groups of glyphs:
  1) Euro
  2) Apple Symbol glyphs. These glyphs were formerly supplied by the Macintosh ATM™ and the Laserwriter® drivers. This is no longer true for OpenType fonts.
  3) A miscellany of glyphs missing from some of the Adobe Type Library fonts, which were just a few glyphs short of containing the full Adobe PostScript Standard glyph set.

| Synthetic Glyphs |                |               |
|------------------|----------------|---------------|
| € Euro           | ∆ Delta        | Ω Omega       |
| ≈ approxequal    | ^ asciicircum  | ~ asciitilde  |
| @ at             | \ backslash    | \| bar        |
| ¦ brokenbar      | ¤ currency     | † dagger      |
| ‡ daggerdbl      | ° degree       | ÷ divide      |
| = equal          | ℮ estimated    | ⁄ fraction    |
| > greater        | ≥ greaterequal | ∞ infinity    |
| ∫ integral       | < less         | ≤ lessequal   |
| ℓ litre          | ¬ logicalnot   | ◊ lozenge     |
| − minus          | × multiply     | ≠ notequal    |
| № numbersign     | ½ onehalf      | ¼ onequarter  |
| ¶ paragraph      | ∂ partialdiff  | ‰ perthousand |
| π pi             | + plus         | ± plusminus   |
| ∏ product        | " quotedbl     | ' quotesingle |
| √ radical        | § section      | ∑ summation   |
| ¾ threequarters  | 0 zero         |               |

The glyphs are synthesized from the MM fonts which MakeOTF has built-in. It will try to match the glyph width of the zero and the dominant stem width of the target font. However, MakeOTF cannot stretch the MM font data to match very thick strokes, very wide glyphs, and it cannot match the design’s stem contrast.

---

#### Document Version History

Version 1.0 - Initial version  
Version 1.5 - Revisions for MakeOTF v2.5 - 2016  
Version 2.0 - Convert to Markdown, many minor updates and fixes - Josh Hadley, October 2019  
