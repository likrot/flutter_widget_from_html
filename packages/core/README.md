# Flutter Widget from HTML (core)

![Flutter](https://github.com/daohoangson/flutter_widget_from_html/workflows/Flutter/badge.svg)
[![codecov](https://codecov.io/gh/daohoangson/flutter_widget_from_html/branch/master/graph/badge.svg)](https://codecov.io/gh/daohoangson/flutter_widget_from_html)
[![Pub](https://img.shields.io/pub/v/flutter_widget_from_html_core.svg)](https://pub.dev/packages/flutter_widget_from_html_core)

A Flutter package for building Flutter widget tree from HTML (supports most popular tags and stylings).

This `core` package implements html parsing and widget building logic so it's easy to extend and fit your app's use case. It tries to render an optimal tree: use `RichText` with specific `TextStyle`, merge text spans together, show images in sized box, etc.

If this is your first time here, consider using the [`flutter_widget_from_html`](https://pub.dev/packages/flutter_widget_from_html) package as a quick starting point.

## Usage

To use this package, add `flutter_widget_from_html_core` as a [dependency in your pubspec.yaml file](https://flutter.io/using-packages/).

See the [Demo app](https://github.com/daohoangson/flutter_widget_from_html/tree/master/demo_app) for inspiration.

### Example

```dart
const kHtml = '''
<h1>Heading 1</h1>
<h2>Heading 2</h2>
<h3>Heading 3</h3>
<h4>Heading 4</h4>
<h5>Heading 5</h5>
<h6>Heading 6</h6>
<p>A paragraph with <strong>strong</strong> <em>emphasized</em> text.</p>

<p>And of course, cat image:</p>
<figure>
  <img src="https://media.giphy.com/media/6VoDJzfRjJNbG/giphy-downsized.gif" width="250" height="171" />
  <figcaption>Source: <a href="https://gph.is/QFgPA0">https://gph.is/QFgPA0</a></figcaption>
</figure>
''';

class HelloWorldCoreScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: Text('HelloWorldCoreScreen')),
        body: SingleChildScrollView(
          child: Padding(
            padding: const EdgeInsets.all(8.0),
            child: HtmlWidget(
              kHtml,
              onTapUrl: (url) => showDialog(
                context: context,
                builder: (_) => AlertDialog(
                  title: Text('onTapUrl'),
                  content: Text(url),
                ),
              ),
            ),
          ),
        ),
      );
}

void main() => runApp(WidgetsApp(home: HelloWorldCoreScreen()));
```

<img src="../../demo_app/screenshots/HelloWorldCoreScreen.gif?raw=true" width="300" />

## Features

### HTML tags

Below tags are the ones that have special meaning / styling, all other tags will be parsed as text.

- A: underline, blue color, no default onTap action (use [`flutter_widget_from_html`](https://pub.dev/packages/flutter_widget_from_html) for that)
- H1/H2/H3/H4/H5/H6
- IMG with support for asset (`asset://`), data uri and network image without caching (use [`flutter_widget_from_html`](https://pub.dev/packages/flutter_widget_from_html) for that)
- LI/OL/UL with support for:
  - Attributes: `type`, `start`, `reversed`
  - Inline style `list-style-type` values: `lower-alpha`, `upper-alpha`, `lower-latin`, `upper-latin`, `circle`, `decimal`, `disc`, `lower-roman`, `upper-roman`, `square`
- TABLE/CAPTION/THEAD/TBODY/TFOOT/TR/TD/TH with support for:
  - TABLE attributes (`border`, `cellpadding`) and inline style (`border`)
  - TD/TH attributes `colspan`, `rowspan` are parsed but ignored during rendering, use [`flutter_widget_from_html`](https://pub.dev/packages/flutter_widget_from_html) if you need them
- ABBR, ACRONYM, ADDRESS, ARTICLE, ASIDE, B, BIG, BLOCKQUOTE, BR, CENTER, CITE, CODE,
  DD, DEL, DFN, DIV, DL, DT, EM, FIGCAPTION, FIGURE, FONT, FOOTER, HEADER, HR, I, IMG, INS,
  KBD, MAIN, NAV, P, PRE, Q, RP, RT, RUBY, S, SAMP, SECTION, STRIKE, STRONG, SUB, SUP, TT, U, VAR

However, these tags and their contents will be ignored:

- IFRAME (use [`flutter_widget_from_html`](https://pub.dev/packages/flutter_widget_from_html) for web view support)
- SCRIPT
- STYLE
- SVG (use [`flutter_widget_from_html`](https://pub.dev/packages/flutter_widget_from_html) for SVG support)

### Attributes

- dir: `auto`, `ltr` and `rtl`

### Inline stylings

- background (color only), background-color: hex values, `rgb()`, `hsl()` or named colors
- border-top, border-bottom: overline/underline with support for dashed/dotted/double/solid style
- color: hex values, `rgb()`, `hsl()` or named colors
- direction (similar to `dir` attribute)
- font-family
- font-size: absolute (e.g. `xx-large`), relative (`larger`, `smaller`) or values in `em`, `%`, `pt` and `px`
- font-style: italic/normal
- font-weight: bold/normal/100..900
- line-height: `normal` number or values in `em`, `%`, `pt` and `px`
- margin and margin-xxx: values in `em`, `pt` and `px`
- padding and padding-xxx: values in `em`, `pt` and `px`
- vertical-align: baseline/top/bottom/middle/sub/super
- text-align: center/justify/left/right
- text-decoration: line-through/none/overline/underline
- text-overflow: clip/ellipsis. Note: `text-overflow: ellipsis` should be used in conjuntion with `max-lines` or `-webkit-line-clamp` for better result.
- Sizing (width & height, max-xxx, min-xxx) with values in `em`, `pt` and `px`

## Extensibility

There are two ways to change the built widget tree.

1. Use callbacks like `customStylesBuilder` or `customWidgetBuilder` for small changes
2. Use a custom `WidgetFactory` for complete control of the rendering process

### Callbacks

For cosmetic changes like color, italic, etc., use `customStylesBuilder` to specify inline styles (see supported list above) for each DOM element. Some common conditionals:

- If HTML tag is H1 `e.localName == 'h1'`
- If the element has `foo` CSS class `e.classes.contains('foo')`
- If an attribute has a specific value `e.attributes['x'] == 'y'`

This example changes the color for a CSS class:

<table><tr><td>

```dart
const kHtml = 'Hello <span class="name">World</span>!';

class CustomStylesBuilderScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(
          title: Text('CustomStylesBuilderScreen'),
        ),
        body: Padding(
          padding: const EdgeInsets.all(8),
          child: HtmlWidget(
            kHtml,
            customStylesBuilder: (e) =>
                e.classes.contains('name')
                  ? {'color': 'red'}
                  : null,
          ),
        ),
      );
}
```

</td>
<td>
  <img src="../../demo_app/screenshots/CustomStylesBuilderScreen.png?raw=true" width="200" />
</td>
</tr>
</table>

For fairly simple widget, use `customWidgetBuilder`. You will need to handle the DOM element and its children manually. The next example renders a carousel:

```dart
const kHtml = '''
<p>...</p>
<div class="carousel">
  <div class="image">
    <img src="https://images.unsplash.com/photo-1514888286974-6c03e2ca1dba" />
  </div>
  ...
</div>
<p>...</p>
''';

class CustomWidgetBuilderScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(
          title: Text('CustomStylesBuilderScreen'),
        ),
        body: SingleChildScrollView(
          child: Padding(
            padding: const EdgeInsets.all(8.0),
            child: HtmlWidget(
              kHtml,
              customWidgetBuilder: (e) {
                if (!e.classes.contains('carousel')) return null;

                final srcs = <String>[];
                for (final child in e.children) {
                  for (final grandChild in child.children) {
                    srcs.add(grandChild.attributes['src']);
                  }
                }

                return CarouselSlider(
                  options: CarouselOptions(
                    autoPlay: true,
                    autoPlayAnimationDuration: const Duration(milliseconds: 250),
                    autoPlayInterval: const Duration(milliseconds: 1000),
                    enlargeCenterPage: true,
                    enlargeStrategy: CenterPageEnlargeStrategy.scale,
                  ),
                  items: srcs.map(_toItem).toList(growable: false),
                );
              },
            ),
          ),
        ),
      );

  static Widget _toItem(String src) => Container(
        child: Center(
          child: Image.network(src, fit: BoxFit.cover, width: 1000),
        ),
      );
}
```

<img src="../../demo_app/screenshots/CustomWidgetBuilderScreen.gif?raw=true" width="300" />

### Custom `WidgetFactory`

The HTML string is parsed into DOM elements and each element is visited once to collect `BuildMetadata` and prepare `BuildBit`s. See step by step how it works:

| Step | | Integration point |
| --- | --- | --- |
| 1 | Parse | `WidgetFactory.parse(BuildMetadata)` |
| 2 | Inform parents if any | `BuildOp.onChild(BuildMetadata)` |
| 3 | Populate default styling | `BuildOp.defaultStyles(BuildMetadata)` |
| 4 | Populate custom styling | `HtmlWidget.customStylesBuilder` |
| 5 | Parse styling key+value pairs, `parseStyle` may be called multiple times | `WidgetFactory.parseStyle(BuildMetadata, String, String)` |
| 6 | a. If a custom widget is provided, go to 7 | `HtmlWidget.customWidgetBuilder` |
|   | b. Loop through children elements to prepare `BuildBit`s | |
| 7 | Inform build ops | `BuildOp.onTree(BuildMetadata, BuildTree)` |
| 8 | a. If not a block element, go to 10 | |
|   | b. Build widgets from bits using a `Flattener` | Use existing `BuildBit` or extends from it, overriding `.swallowWhitespace` to control whitespace, etc. |
| 9 | Inform build ops | `BuildOp.onWidgets(BuildMetadata, Iterable<Widget>)` |
| 10 | The end | |

Notes:

- Text related styling can be changed with `TextStyleBuilder`, just register your callback and it will be called when the build context is ready.
  - The first parameter is a `TextStyleHtml` which is immutable and is calculated from the root down to each element, the callback must return a new `TextStyleHtml` by calling `copyWith`. It's recommended to return the same object if no change is needed.
  - Optionally, pass any object on registration and your callback will receive it as the second parameter.

```dart
// example 1: simple callback setting accent color from theme
meta.tsb((parent, _) =>
  parent.copyWith(
    style: parent.style.copyWith(
      color: parent.getDependency<ThemeData>().accentColor,
    ),
  ));

// example 2: callback using second param to set height
TextStyleHtml callback(TextStyleHtml parent, double value) =>
  parent.copyWith(height: value)

// example 2 (continue): register with some value
meta.tsb<double>(callback, 2.0);
```

- The root styling can be customized by overriding `WidgetFactory.onRoot(TextStyleBuilder)`
- Other complicated styling are supported via `BuildOp`

```dart
meta.register(BuildOp(
  onTree: (meta, tree) {
    tree.add(...);
  },
  onWidgets: (meta, widgets) => widgets.map((widget) => ...),
  ...,
  priority: 9999,
));
```

- Each metadata may have as many tsb callbacks and build ops as needed.

The example below replaces smilie inline image with an emoji:

```dart
const kHtml = """
<p>Hello <img class="smilie smilie-1" alt=":)" src="http://domain.com/sprites.png" />!</p>
<p>How are you <img class="smilie smilie-2" alt=":P" src="http://domain.com/sprites.png" />?
""";

const kSmilies = {':)': '🙂'};

class SmilieScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(
          title: Text('SmilieScreen'),
        ),
        body: Padding(
          padding: const EdgeInsets.all(8.0),
          child: HtmlWidget(
            kHtml,
            factoryBuilder: () => _SmiliesWidgetFactory(),
          ),
        ),
      );
}

class _SmiliesWidgetFactory extends WidgetFactory {
  final smilieOp = BuildOp(
    onTree: (meta, tree) {
      final alt = meta.element.attributes['alt'];
      tree.addText(kSmilies[alt] ?? alt);
    },
  );

  @override
  void parse(BuildMetadata meta) {
    final e = meta.element;
    if (e.localName == 'img' &&
        e.classes.contains('smilie') &&
        e.attributes.containsKey('alt')) {
      meta.register(smilieOp);
      return;
    }

    return super.parse(meta);
  }
}
```

<img src="../../demo_app/screenshots/SmilieScreen.png?raw=true" width="300" />
