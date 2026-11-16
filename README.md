# norn-html

Parse HTML into immutable Norn data, query it with CSS selectors, and build
and write HTML from data. The
package runs on stock Norn and needs no native helper or external executable.

## Install

Add a path dependency to your app’s `norn.edn`. Set the path relative to the
app’s manifest.

```clojure
{:name example
 :paths ["src"]
 :deps {norn-html {:path "../norn-libs/norn-html"}}}
```

## Parse and query

```clojure
(require '[norn.html :as html])

(def page (html/parse "<title>News</title><nav><a href=/latest>Latest &amp; greatest</a></nav>"))

(mapv (fn [link] {:href (html/attr link :href) :label (html/text link)})
      (html/select page "nav a[href]"))
;; => [{:href "/latest" :label "Latest & greatest"}]
```

| Function | Result |
| --- | --- |
| `(parse source)` | A document with `html`, `head`, and `body` elements, inferred when omitted. |
| `(parse-fragment source)` | A fragment containing the snippet’s top-level nodes. |
| `(read-file path)` | A document read from a UTF-8 file. |
| `(select node css)` | A vector of matching descendant elements in document order. |
| `(select-first node css)` | The first matching descendant, or `nil`. |
| `(attr node key)` | An attribute value by string or keyword name, or `nil`. |
| `(text node)` | Descendant text concatenated in order, including script and style text. |

An element is a map like this.

```clojure
{:type :element :tag "a" :attrs {"href" "/latest"} :children ["Latest & greatest"]}
```

Documents and fragments also have a `:children` vector. Text nodes are strings,
comments have `:type :comment` and `:text`, and doctypes have `:type :doctype`
and `:name`. Tag and attribute names are ASCII-lowercase strings.

Selectors support type, universal, ID, class, attribute presence and comparison,
selector lists, and descendant, child, adjacent-sibling, and general-sibling
combinators. Supported pseudo-classes are `:first-child`, `:last-child`,
`:only-child`, `:empty`, and `:root`. Attribute comparisons accept the `i` and
`s` case flags. Invalid or unsupported selectors signal `ParseError`.

## Build and write

```clojure
(html/render [:ul.links (for [{:keys [href label]} links] [:li [:a {:href href} label]])])
;; => "<ul class=\"links\"><li><a href=\"/latest\">Latest &amp; greatest</a></li></ul>"

(html/write (html/document {:title "Report"} [:h1 "Totals"] [:p "…"]))
```

| Function | Result |
| --- | --- |
| `(html form…)` | Nodes from hiccup-style forms: `[:tag attrs? children…]`, the tag maybe `:div.card#main`. |
| `(render form…)` | The HTML text of forms: `(write (html form…))`. |
| `(write node)` | A node as HTML text: text and attribute values escaped, script and style text as it is. |
| `(document opts body…)` | A whole page: doctype, `html` with `:lang`, a head with charset, viewport, `:title` and `:head`. |
| `(escape s)`, `(escape-attribute s)` | Text escaped for content, or for a double-quoted attribute. |
| `(raw html)` | A node written as it is, for markup you trust. |

In attributes, `true` is an attribute with no value, `nil` and `false` leave it
out, `:class` takes a vector, and `:style` a map (written sorted by property, so
shorthands come before their longhands). Children may be strings, numbers,
nodes, `nil`, or sequences of them, which are spliced in. In Norn `for` and
`map` give vectors, so a vector that doesn't start with a keyword counts as a
sequence. Attributes are written sorted by name.

What `parse` gives, `write` gives back as HTML that parses to it again, and
writing, parsing and writing again gives the same text (a property test checks
it).

## Interface

`api/` records the package's public interface. `norn api-diff` checks it before
a release; a breaking change needs a new major version.

## Parsing scope

The parser handles void elements, quoted and unquoted attributes, named and
numeric character references, raw script and style text, and commonly omitted
end tags such as `p`, `li`, `option`, and table cells. It also inserts table
rows and sections in common cases.

Browser-equivalent recovery for complex misnested formatting and misplaced
table content, context-specific fragment parsing, full SVG and MathML
namespaces, and byte encoding detection are outside this release’s scope.
Use the tree for inspection and extraction. It can contain scripts and other
active markup from the input.

The named character reference table uses the mappings from the
[WHATWG HTML Standard](https://html.spec.whatwg.org/entities.json), generated
from Python’s bundled copy.

## Test

From the package directory, run `norn test`.
