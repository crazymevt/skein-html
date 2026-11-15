# norn-html

Parse HTML into immutable Norn data and query it with CSS selectors. The
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
