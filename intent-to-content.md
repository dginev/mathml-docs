---
title: "Intent: Transformation to Content MathML"
layout: wgnote
---

*Author*: Deyan Ginev

<style>
pre.def {
  padding: .5em 1em;
  background: #fafafa;
  background: #8ccbf2;
  margin: 1.2em 0;
  border-left: 0.5em solid #8ccbf2;
  border-left: 0.5em solid var(--def-border);
  color: black;
  color: var(--def-text);
}
.markdown-body table tr {
  background-color: white !important;
}
.markdown-body pre {
  display: table-cell;
  background-color: white !important;
}
.hljs{padding:.5em;color:#383a42;background:#fafafa}
.hljs-comment,.hljs-quote{color:#717277;font-style:italic}
.hljs-doctag,.hljs-formula,.hljs-keyword{color:#a626a4}
.hljs-deletion,.hljs-name,.hljs-section,.hljs-selector-tag,.hljs-subst{color:#ca4706;font-weight:700}
.hljs-literal{color:#0b76c5}
.hljs-addition,.hljs-attribute,.hljs-meta-string,.hljs-regexp,.hljs-string{color:#42803c}
.hljs-built_in,.hljs-class .hljs-title{color:#9a6a01}
.hljs-attr,.hljs-number,.hljs-selector-attr,.hljs-selector-class,.hljs-selector-pseudo,.hljs-template-variable,.hljs-type,.hljs-variable{color:#986801}
.hljs-bullet,.hljs-link,.hljs-meta,.hljs-selector-id,.hljs-symbol,.hljs-title{color:#336ae3}
.hljs-emphasis{font-style:italic}
.hljs-strong{font-weight:700}
.hljs-link{text-decoration:underline}
</style>

## Abstract

This note suggests a two-step alogrithm for transforming intent expressions into Content MathML.
First, we offer rules to build an operator tree with symbols in the "intent" virtual content dictionary.
Second, we suggest a "phrase book" for mapping those operator trees into the classic subset of Content MathML.

## Preliminaries - The Grammar for <code class="attribue">intent</code>

This is the December 2022 state of MathML 4's [5.1.1 The Grammar for intent](https://w3c.github.io/mathml/#mixing_intent_grammar).

<p>The value of the <code class="attribute">intent</code> attribute, after ignoring white
space between tokens, should match the following grammar.</p>

<pre class="def bnf">
intent             := concept-or-literal | number | reference | application
concept-or-literal := NCName
number             := '-'? digit+ ( '.' digit+ )?
reference          := '$' NCName
application        := intent hint? '(' arguments? ')'
arguments          := intent ( ',' intent )*
hint               := '@' ( 'prefix' | 'infix' | 'postfix' | 'function' | 'silent' )
</pre>

<p>Here <a href="https://www.w3.org/TR/REC-xml-names/#NT-NCName"><code>NCName</code></a>
is as defined in [[xml-names]], and <code>digit</code> is a character in the range 0–9.</p>

## Creating an Operator Tree

Starting with an intent expression, we offer a `to_content` procedure to create an operator tree, via the following list of transformation rules:

The rules are *ordered*, and should be matched top-first to bottom-last.

<table style="width: 80rem; overflow:visible;">
<thead><tr>
  <th>intent input</th><th>to_content output</th><th>Extra Requirement</th>
</tr></thead>
<tbody>
<tr>
<td>hint</td><td>ignore</td><td></td>
</tr><tr>
<td>literal</td><td>ignore</td><td></td>
</tr><tr>
<td>concept</td>
<td>
<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>concept<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>
</td><td></td>
</tr><tr>
<td>number</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">cn</span>&gt;</span>number<span class="hljs-tag">&lt;/<span class="hljs-name">cn</span>&gt;</span>
</code></pre>
</td><td></td>
</tr><tr>
<td>reference</td><td>

<pre><code class="hljs">to_content(ref_target_node)
</code></pre>
</td><td></td>
</tr><tr>
<td>

<pre><code class="hljs">intent hint? '(' arguments? ')'
</code></pre>
</td>
<td>ignore</td>
<td>"intent" head is a literal,<br> arguments are all literals</td>
</tr><tr>
<td>

<pre><code class="hljs">intent hint? '(' arguments? ')'</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">apply</span>&gt;</span>
  to_content(intent)
  to_content(argument_1)
  ...
  to_content(argument_n)
<span class="hljs-tag">&lt;/<span class="hljs-name">apply</span>&gt;</span>
</code></pre>
</td><td>"intent" head is <br>concept, reference, <br>
number, compound expression,<br> or literal different from "_".
</td>
</tr><tr>

<td>

<pre><code class="hljs">_ hint? '('
  (pre_concept_arguments ',')?
  only_concept_argument
  (',' post_arguments)? ')'
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">apply</span>&gt;</span>
  to_content(only_concept_argument)
  to_content(pre_concept_arguments_1)
  ...
  to_content(pre_concept_arguments_n)
  to_content(post_concept_arguments_1)
  ...
  to_content(post_concept_arguments_n)
<span class="hljs-tag">&lt;/<span class="hljs-name">apply</span>&gt;</span>
</code></pre>

</td><td>"intent" head is the underscore literal,<br> arguments contain <br><strong>exactly one</strong> concept argument.</td>
</tr><tr>
<td>

<pre><code class="hljs">_ hint? '(' arguments? ')'</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">apply</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>narrative-pieces<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
  to_content(argument_1)
  ...
  to_content(argument_n)
<span class="hljs-tag">&lt;/<span class="hljs-name">apply</span>&gt;</span>
</code></pre>

</td><td>"intent" head is the underscore literal,<br> all other cases</td>
</tr>
</tbody>
</table>


## Example transformations to operator trees


<table style="width: 80rem; overflow:visible;">
<thead><tr>
  <th>intent input</th><th>to_content output</th><th>Extra Requirement</th>
</tr></thead>
<tbody>
<tr><td>
<pre><code class="hljs">_@silent</code></pre>
</td>
<td></td>
</tr><tr>
<td>
<pre><code class="hljs">_to</code></pre>
</td><td></td>
</tr><tr>
<td>
<pre><code class="hljs">euler-constant</code></pre>
</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>euler-constant<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td>
</tr><tr>
<td><pre><code class="hljs">0.01</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">cn</span>&gt;</span>0.01<span class="hljs-tag">&lt;/<span class="hljs-name">cn</span>&gt;</span>
</code></pre>

</td>
</tr><tr>
<td>

<pre><code class="hljs">$operator</code></pre>
</td><td>

<pre><code class="hljs xml">
to_content(<span class="hljs-tag">&lt;<span class="hljs-name">mo</span> arg="operator"&gt;</span>+<span class="hljs-tag">&lt;/<span class="hljs-name">mo</span>&gt;</span>)
</code></pre>
</td>
</tr><tr>
<td>

<pre><code class="hljs">absolute-value($inner)</code></pre>
</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">apply</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>absolute-value<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
  to_content(<span class="hljs-tag">&lt;<span class="hljs-name">mrow</span> arg="inner"&gt;</span>...<span class="hljs-tag">&lt;/<span class="hljs-name">mrow</span>&gt;</span>)
<span class="hljs-tag">&lt;/<span class="hljs-name">apply</span>&gt;</span>
</code></pre>
</td><td>"intent" head is <br>concept, reference, <br>
number, compound expression,<br> or literal different from "_".
</td>
</tr><tr>
<td>

<pre><code class="hljs">$op@infix(1,2,3)</code></pre>
</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">apply</span>&gt;</span>
  to_content(<span class="hljs-tag">&lt;<span class="hljs-name">mo</span> arg="op"&gt;</span>...<span class="hljs-tag">&lt;/<span class="hljs-name">mo</span>&gt;</span>)
  <span class="hljs-tag">&lt;<span class="hljs-name">cn</span>&gt;</span>1<span class="hljs-tag">&lt;/<span class="hljs-name">cn</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">cn</span>&gt;</span>2<span class="hljs-tag">&lt;/<span class="hljs-name">cn</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">cn</span>&gt;</span>3<span class="hljs-tag">&lt;/<span class="hljs-name">cn</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">apply</span>&gt;</span>
</code></pre>

</td><td>"intent" head is <br>concept, reference, <br>
number, compound expression,<br> or literal different from "_".
</td>
</tr><tr>
<td>

<pre><code class="hljs">inverse(f)(x)
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">apply</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">apply</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>inverse<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>f<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
 <span class="hljs-tag">&lt;/<span class="hljs-name">apply</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>x<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">apply</span>&gt;</span>
</code></pre>

</td><td>"intent" head is <br>concept, reference, <br>
number, compound expression,<br> or literal different from "_".
</td>

</tr><tr>
<td>

<pre><code class="hljs">_our_altered_minus(1,2,3)
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">apply</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>_our_altered_minus<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">cn</span>&gt;</span>1<span class="hljs-tag">&lt;/<span class="hljs-name">cn</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">cn</span>&gt;</span>2<span class="hljs-tag">&lt;/<span class="hljs-name">cn</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">cn</span>&gt;</span>3<span class="hljs-tag">&lt;/<span class="hljs-name">cn</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">apply</span>&gt;</span>
</code></pre>

</td><td>"intent" head is <br>concept, reference, <br>
number, compound expression,<br> or literal different from "_".
</td>
</tr><tr>
<td>

<pre><code class="hljs">_note(_used,_for,_brevity)</code></pre>
</td><td>
</td><td>"intent" head is a literal,<br> arguments are all literals
</td>
</tr><tr>
<td>

<pre><code class="hljs">_(_the, open-interval, _from, $arg1, _to, $arg2)</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">apply</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>open-interval<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
  to_content(<span class="hljs-tag">&lt;<span class="hljs-name">msub</span> arg="arg1"&gt;</span>...<span class="hljs-tag">&lt;/<span class="hljs-name">msub</span>&gt;</span>)
  to_content(<span class="hljs-tag">&lt;<span class="hljs-name">msub</span> arg="arg2"&gt;</span>...<span class="hljs-tag">&lt;/<span class="hljs-name">msub</span>&gt;</span>)
<span class="hljs-tag">&lt;/<span class="hljs-name">apply</span>&gt;</span>
</code></pre>

</td><td>"intent" head is the underscore literal,<br> arguments contain <br><strong>exactly one</strong> concept argument.
</td>
</tr><tr>
<td>

<pre><code class="hljs">_(_the, open-interval, _from, x, _to, $arg2)</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">apply</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>narrative-pieces<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>open-interval<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>x<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
  to_content(<span class="hljs-tag">&lt;<span class="hljs-name">mi</span> arg="arg2"&gt;</span>...<span class="hljs-tag">&lt;/<span class="hljs-name">mi</span>&gt;</span>)
<span class="hljs-tag">&lt;/<span class="hljs-name">apply</span>&gt;</span>
</code></pre>

</td><td>"intent" head is the underscore literal,<br> all other cases</td>
</tr></tbody></table>


## Phrase book to classic Content MathML

The following (partial) map can be used to rewrite the virtual content dictionary trees as classic Content MathML.

It covers the chapter [4.4 Content MathML for Specific Operators and Constants](https://www.w3.org/TR/MathML3/chapter4.html#contm.opel).

Note that there are quite often known aliases for mathematical  concepts (in narration, "addition" may be referred to as simply speaking the operator "plus" sign), and they would need to be included inside such phrase books in order to gain coverage (or, alternatively, an external "alias-resolution" mechanism would be needed, maybe provided by an Intent Open list).

<table style="width: 80rem; overflow:visible;">
<thead><tr>
  <th>intent CD input symbol</th><th>classic Content MathML output</th>
</tr></thead>
<tbody>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>\p{Letter}<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">ci</span>&gt;</span>\p{Letter}<span class="hljs-tag">&lt;/<span class="hljs-name">ci</span>&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>plus<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">plus</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>addition<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">plus</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>interval<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">interval</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>inverse<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">inverse</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>function-composition<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">compose</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>identity-function<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">ident</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>domain<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">domain</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>codomain<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">codomain</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>image<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">image</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>piecewise-function<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">piecewise</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>otherwise<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">otherwise</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>quotient<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">quotient</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>factorial<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">factorial</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>division<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">divide</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>maximum<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">max</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>minimum<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">min</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>subtraction<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">minus</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>exponentiation<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">power</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>remainder<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">rem</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>multiplication<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">times</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>root<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">root</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>greatest-common-divisor<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">gcd</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>and<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">and</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>or<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">or</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>exclusive-or<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">xor</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>not<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">not</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>implies<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">implies</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>for-all<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">forall</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>exists<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">exists</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>absolute-value<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">abs</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>complex-conjugate<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">conjugate</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>argument<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">arg</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>real-part<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">real</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>imaginary part<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">imaginary</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>lowest-common-multiple<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">lcm</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>floor<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">floor</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>ceiling<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">ceiling</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>equals<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">eq</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>not-equals<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">neq</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>greater-than<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">gt</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>less-than<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">lt</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>greater-than-or-equal<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">geq</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>less-than-or-equal<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">leq</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>equivalent<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">equivalent</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>approximately<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">approx</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>factor-of<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">factorof</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>integral<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">int</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>differentiation<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">diff</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>partial-differentiation<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">partialdiff</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>divergence<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">divergence</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>gradient<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">grad</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>curl<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">curl</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>laplacian<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">laplacian</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>set<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">set</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>list<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">list</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>union<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">union</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>intersect<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">intersect</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>set-inclusion<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">in</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>set-exclusion<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">notin</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>subset<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">subset</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>proper-subset<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">prsubset</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>not-subset<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">notsubset</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>not-proper-subset<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">notprsubset</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>set-difference<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">setdiff</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>cardinality<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">card</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>cartesian-product<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">cartesianproduct</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>sum<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">sum</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>product<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">product</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>limits<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">limit</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>tends-to<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">tendsto</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>sine<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">sin</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>cosine<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">cos</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>tangent<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">tan</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>secant<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">sec</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>cosecant<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csc</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>catangent<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">cot</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>arcsine<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">arcsin</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>arccosine<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">arccos</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>arctangent<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">arctan</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>arcsecant<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">arcsec</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>arccosecant<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">arccsc</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>arccotangent<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">arccot</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>hyperbolic-sine<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">sinh</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>hyerpbolic-cosine<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">cosh</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>hyperbolic-tangent<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">tanh</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>hyperbolic-secant<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">sech</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>hyperbolic-cosecant<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csch</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>hyperbolic-catangent<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">coth</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>hyperbolic-arcsine<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">arcsinh</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>hyperbolic-arccosine<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">arccosh</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>hyperbolic-arctangent<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">arctanh</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>hyperbolic-arcsecant<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">arcsech</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>hyperbolic-arccosecant<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">arccsch</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>hyperbolic-arccotangent<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">arccoth</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>exponential-function<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">exp</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>natural-logarithm<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">ln</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>logarithm<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">log</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>mean<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">mean</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>standard-deviation<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">sdev</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>variance<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">variance</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>median<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">median</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>mode<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">mode</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>moment<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">moment</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>vector<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">vector</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>matrix<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">matrix</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>determinant<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">determinant</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>transpose<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">transpose</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>selector<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">selector</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>vector-product<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">vectorproduct</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>scalar-product<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">scalarproduct</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>outer-product<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">outerproduct</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>integer-numbers<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">integers</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>real-numbers<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">reals</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>rational-numbers<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">rationals</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>natural-numbers<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">naturalnumbers</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>complex-numbers<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">complexes</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>prime-numbers<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">primes</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>euler-constant<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">exponentiale</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>imaginary-number<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">imaginaryi</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>not-a-number<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">notanumber</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>true<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">true</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>false<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">false</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>empty-set<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">emptyset</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>pi<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">pi</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>euler-gamma<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">eulergamma</span>/&gt;</span>
</code></pre>

</td></tr>
<tr><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">csymbol</span> cd="intent"&gt;</span>infinity<span class="hljs-tag">&lt;/<span class="hljs-name">csymbol</span>&gt;</span>
</code></pre>

</td><td>

<pre><code class="hljs xml"><span class="hljs-tag">&lt;<span class="hljs-name">infinity</span>/&gt;</span>
</code></pre>

</td></tr></tbody></table>