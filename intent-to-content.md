---
title: "Intent: Transformation to Content MathML"
layout: wgnote
---

*Author*: Deyan Ginev

## Abstract

This note suggests a two-step alogrithm for transforming intent expressions into Content MathML.
First, we offer rules to build an operator tree with symbols in the "intent" virtual content dictionary.
Second, we suggest a "phrase book" for mapping those operator trees into the Pragmatic subset of Content MathML.

# Creating an Operator Tree

Starting with an intent expression, we offer a `to_content` procedure to create an operator tree, via the following list of transformation rules:

The rules are *ordered*, and should be executed top to bottom.

<table width="100%">
<thead>
  <th>intent input</th><th>to_content output</th><th>Extra Requirement</th>
</thead>
<tbody>
<tr>
<td>hint</td><td> ignore</td></tr><tr>
<td>literal</td><td> ignore</td></tr><tr>
<td>concept</td><td>

<pre class="xml">
<csymbol cd="intent">concept</csymbol>
</pre>
</td></tr><tr>
<td>number</td><td>

<pre class="xml">
<cn>number</cn>
</pre>

</td></tr><tr>
<td>reference</td><td>

<pre class="xml">
to_content(ref_target_node)
</pre>
</td></tr><tr>
<td>

<pre>
intent hint? '(' arguments? ')'
</pre>

</td><td>

<pre class="xml">
<apply>
  to_content(intent)
  to_content(argument_1)
  ...
  to_content(argument_n)
</apply>
</pre>
</td><td>"intent" head is <br>concept, reference, <br>
number, or compound expression.
</td></tr><tr><td>

<pre>
intent hint? '(' arguments? ')'
</pre>
</td><td>ignore
</td><td>"intent" head is a literal,<br> arguments are all literals
</td></tr><tr><td>

<pre>
intent hint? '('
  (pre_concept_arguments ',')?
  only_concept_argument
  (',' post_arguments)? ')'
</pre>

</td><td>

<pre class="xml">
<apply>
  to_content(only_concept_argument)
  to_content(pre_concept_arguments_1)
  ...
  to_content(pre_concept_arguments_n)
  to_content(post_concept_arguments_1)
  ...
  to_content(post_concept_arguments_n)
</apply>
</pre>

</td><td>"intent" head is a literal,<br> arguments contain <br><strong>exactly one</strong> concept argument.
</td><tr><td>

<pre>
intent hint? '(' arguments? ')'
</pre>

</td><td>

<pre class="xml">
<apply>
  <csymbol cd="intent">narrative_pieces</csymbol>
  to_content(argument_1)
  ...
  to_content(argument_n)
</apply>
</pre>

</td><td>"intent" head is a literal,<br> all other cases
</td></tr></tbody></table>



## Example transformations to operator trees


<table width="100%">
<thead>
  <th>intent input</th><th>to_content output</th><th>Extra Requirement</th>
</thead>
<tbody>
<tr>
<td>

<pre>
_@silent
</pre>

</td><td></td></tr><tr>
<td>

<pre>
_to
</pre>

</td><td></td></tr><tr>
<td>

<pre>
euler-constant
</pre>

</td><td>

<pre class="xml">
<csymbol cd="intent">euler-constant</csymbol>
</pre>
</td></tr><tr>
<td>

<pre>
0.01
</pre>

</td><td>

<pre class="xml">
<cn>0.01</cn>
</pre>

</td></tr><tr>
<td>

<pre>
$operator
</pre>
</td><td>

<pre class="xml">
to_content(<mo arg="operator">+</mo>)
</pre>
</td></tr><tr>
<td>

<pre>
absolute-value($inner)
</pre>
</td><td>

<pre class="xml">
<apply>
  <csymbol cd="intent">absolute-value</csymbol>
  to_content(<mrow arg="inner">...</mrow>)
</apply>
</pre>
</td><td>"intent" head is <br>concept, reference, <br>
number, or compound expression.
</td></tr>
<tr>
<td>

<pre>
$op@infix(1,2,3)
</pre>
</td><td>

<pre class="xml">
<apply>
  to_content(<mo arg="op">...</mo>)
  <cn>1</cn>
  <cn>2</cn>
  <cn>3</cn>
</apply>
</pre>
</td><td>"intent" head is <br>concept, reference, <br>
number, or compound expression.
</td></tr>
<tr>
<td>

<pre>
inverse(f)(x)
</pre>

</td><td>

<pre class="xml">
<apply>
  <apply>
    <csymbol cd="intent">inverse</csymbol>
    <csymbol cd="intent">f</csymbol>
  </apply>
  <csymbol cd="intent">x</csymbol>
</apply>
</pre>

</td><td>"intent" head is <br>concept, reference, <br>
number, or compound expression.
</td></tr>
<tr><td>

<pre>
_(_used,_for,_brevity)
</pre>
</td><td>
</td><td>"intent" head is a literal,<br> arguments are all literals
</td></tr><tr><td>

<pre>
_(_the, open-interval, _from, $arg1, _to $arg2)
</pre>

</td><td>

<pre class="xml">
<apply>
  <csymbol cd="intent">open-interval</csymbol>
  to_content(<msub arg="arg1">...</msub>)
  to_content(<msub arg="arg2">...</msub>)
</apply>
</pre>

</td><td>"intent" head is a literal,<br> arguments contain <br><strong>exactly one</strong> concept argument.
</td><tr><td>

<pre>
_(_the, open-interval, _from, x, _to, $arg2)
</pre>

</td><td>

<pre class="xml">
<apply>
  <csymbol cd="intent">narrative_pieces</csymbol>
  <csymbol cd="intent">open-interval</csymbol>
  <csymbol cd="intent">x</csymbol>
  to_content(<mi arg="arg2">...</mi>)
</apply>
</pre>

</td><td>"intent" head is a literal,<br> all other cases
</td></tr></tbody></table>


# Phrase book to Pragmatic Content MathML

The following (partial) map can be used to rewrite the virtual content dictionary trees as Pragmatic Content MathML.

It covers the chapter [4.4 Content MathML for Specific Operators and Constants](https://www.w3.org/TR/MathML3/chapter4.html#contm.opel).

Note that there are quite often known aliases for mathematical  concepts (in narration, "addition" may be referred to as simply speaking the operator "plus" sign), and they would need to be included inside such phrase books in order to gain coverage (or, alternatively, an external "alias-resolution" mechanism would be needed, maybe provided by an Intent Open list).

<table width="100%">
<thead>
  <th>intent CD symbol input</th><th>Pragmatic CMML output</th><th>Extra Requirement</th>
</thead>
<tbody>
<tr><td>

<pre class="xml">
<csymbol cd="intent">\p{Letter}</csymbol>
</pre>

</td><td>

<pre class="xml">
<ci>\p{Letter}</ci>
</pre>

</td><td></td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">plus</csymbol>
</pre>

</td><td>

<pre class="xml">
<plus/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">addition</csymbol>
</pre>

</td><td>

<pre class="xml">
<plus/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">interval</csymbol>
</pre>

</td><td>

<pre class="xml">
<interval>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">inverse</csymbol>
</pre>

</td><td>

<pre class="xml">
<inverse>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">function-composition</csymbol>
</pre>

</td><td>

<pre class="xml">
<compose/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">identity-function</csymbol>
</pre>

</td><td>

<pre class="xml">
<ident/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">domain</csymbol>
</pre>

</td><td>

<pre class="xml">
<domain/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">codomain</csymbol>
</pre>

</td><td>

<pre class="xml">
<codomain/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">image</csymbol>
</pre>

</td><td>

<pre class="xml">
<image/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">piecewise-function</csymbol>
</pre>

</td><td>

<pre class="xml">
<piecewise>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">otherwise</csymbol>
</pre>

</td><td>

<pre class="xml">
<otherwise>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">quotient</csymbol>
</pre>

</td><td>

<pre class="xml">
<quotient/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">factorial</csymbol>
</pre>

</td><td>

<pre class="xml">
<factorial/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">division</csymbol>
</pre>

</td><td>

<pre class="xml">
<divide/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">maximum</csymbol>
</pre>

</td><td>

<pre class="xml">
<max/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">minimum</csymbol>
</pre>

</td><td>

<pre class="xml">
<min/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">subtraction</csymbol>
</pre>

</td><td>

<pre class="xml">
<minus/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">exponentiation</csymbol>
</pre>

</td><td>

<pre class="xml">
<power/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">remainder</csymbol>
</pre>

</td><td>

<pre class="xml">
<rem/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">multiplication</csymbol>
</pre>

</td><td>

<pre class="xml">
<times/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">root</csymbol>
</pre>

</td><td>

<pre class="xml">
<root/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">greatest-common-divisor</csymbol>
</pre>

</td><td>

<pre class="xml">
<gcd/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">and</csymbol>
</pre>

</td><td>

<pre class="xml">
<and/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">or</csymbol>
</pre>

</td><td>

<pre class="xml">
<or/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">exclusive-or</csymbol>
</pre>

</td><td>

<pre class="xml">
<xor/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">not</csymbol>
</pre>

</td><td>

<pre class="xml">
<not/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">implies</csymbol>
</pre>

</td><td>

<pre class="xml">
<implies/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">for-all</csymbol>
</pre>

</td><td>

<pre class="xml">
<forall/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">exists</csymbol>
</pre>

</td><td>

<pre class="xml">
<exists/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">absolute-value</csymbol>
</pre>

</td><td>

<pre class="xml">
<abs/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">complex-conjugate</csymbol>
</pre>

</td><td>

<pre class="xml">
<conjugate/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">argument</csymbol>
</pre>

</td><td>

<pre class="xml">
<arg/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">real-part</csymbol>
</pre>

</td><td>

<pre class="xml">
<real/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">imaginary part</csymbol>
</pre>

</td><td>

<pre class="xml">
<imaginary/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">lowest-common-multiple</csymbol>
</pre>

</td><td>

<pre class="xml">
<lcm/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">floor</csymbol>
</pre>

</td><td>

<pre class="xml">
<floor/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">ceiling</csymbol>
</pre>

</td><td>

<pre class="xml">
<ceiling/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">equals</csymbol>
</pre>

</td><td>

<pre class="xml">
<eq/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">not-equals</csymbol>
</pre>

</td><td>

<pre class="xml">
<neq/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">greater-than</csymbol>
</pre>

</td><td>

<pre class="xml">
<gt/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">less-than</csymbol>
</pre>

</td><td>

<pre class="xml">
<lt/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">greater-than-or-equal</csymbol>
</pre>

</td><td>

<pre class="xml">
<geq/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">less-than-or-equal</csymbol>
</pre>

</td><td>

<pre class="xml">
<leq/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">equivalent</csymbol>
</pre>

</td><td>

<pre class="xml">
<equivalent/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">approximately</csymbol>
</pre>

</td><td>

<pre class="xml">
<approx/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">factor-of</csymbol>
</pre>

</td><td>

<pre class="xml">
<factorof/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">integral</csymbol>
</pre>

</td><td>

<pre class="xml">
<int/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">differentiation</csymbol>
</pre>

</td><td>

<pre class="xml">
<diff/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">partial-differentiation</csymbol>
</pre>

</td><td>

<pre class="xml">
<partialdiff/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">divergence</csymbol>
</pre>

</td><td>

<pre class="xml">
<divergence/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">gradient</csymbol>
</pre>

</td><td>

<pre class="xml">
<grad/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">curl</csymbol>
</pre>

</td><td>

<pre class="xml">
<curl/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">laplacian</csymbol>
</pre>

</td><td>

<pre class="xml">
<laplacian/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">set</csymbol>
</pre>

</td><td>

<pre class="xml">
<set>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">list</csymbol>
</pre>

</td><td>

<pre class="xml">
<list>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">union</csymbol>
</pre>

</td><td>

<pre class="xml">
<union/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">intersect</csymbol>
</pre>

</td><td>

<pre class="xml">
<intersect/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">set-inclusion</csymbol>
</pre>

</td><td>

<pre class="xml">
<in/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">set-exclusion</csymbol>
</pre>

</td><td>

<pre class="xml">
<notin/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">subset</csymbol>
</pre>

</td><td>

<pre class="xml">
<subset/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">proper-subset</csymbol>
</pre>

</td><td>

<pre class="xml">
<prsubset/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">not-subset</csymbol>
</pre>

</td><td>

<pre class="xml">
<notsubset/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">not-proper-subset</csymbol>
</pre>

</td><td>

<pre class="xml">
<notprsubset/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">set-difference</csymbol>
</pre>

</td><td>

<pre class="xml">
<setdiff/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">cardinality</csymbol>
</pre>

</td><td>

<pre class="xml">
<card/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">cartesian-product</csymbol>
</pre>

</td><td>

<pre class="xml">
<cartesianproduct/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">sum</csymbol>
</pre>

</td><td>

<pre class="xml">
<sum/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">product</csymbol>
</pre>

</td><td>

<pre class="xml">
<product/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">limits</csymbol>
</pre>

</td><td>

<pre class="xml">
<limit/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">tends-to</csymbol>
</pre>

</td><td>

<pre class="xml">
<tendsto/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">sine</csymbol>
</pre>

</td><td>

<pre class="xml">
<sin/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">cosine</csymbol>
</pre>

</td><td>

<pre class="xml">
<cos/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">tangent</csymbol>
</pre>

</td><td>

<pre class="xml">
<tan/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">secant</csymbol>
</pre>

</td><td>

<pre class="xml">
<sec/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">cosecant</csymbol>
</pre>

</td><td>

<pre class="xml">
<csc/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">catangent</csymbol>
</pre>

</td><td>

<pre class="xml">
<cot/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">arcsine</csymbol>
</pre>

</td><td>

<pre class="xml">
<arcsin/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">arccosine</csymbol>
</pre>

</td><td>

<pre class="xml">
<arccos/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">arctangent</csymbol>
</pre>

</td><td>

<pre class="xml">
<arctan/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">arcsecant</csymbol>
</pre>

</td><td>

<pre class="xml">
<arcsec/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">arccosecant</csymbol>
</pre>

</td><td>

<pre class="xml">
<arccsc/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">arccotangent</csymbol>
</pre>

</td><td>

<pre class="xml">
<arccot/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">hyperbolic-sine</csymbol>
</pre>

</td><td>

<pre class="xml">
<sinh/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">hyerpbolic-cosine</csymbol>
</pre>

</td><td>

<pre class="xml">
<cosh/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">hyperbolic-tangent</csymbol>
</pre>

</td><td>

<pre class="xml">
<tanh/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">hyperbolic-secant</csymbol>
</pre>

</td><td>

<pre class="xml">
<sech/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">hyperbolic-cosecant</csymbol>
</pre>

</td><td>

<pre class="xml">
<csch/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">hyperbolic-catangent</csymbol>
</pre>

</td><td>

<pre class="xml">
<coth/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">hyperbolic-arcsine</csymbol>
</pre>

</td><td>

<pre class="xml">
<arcsinh/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">hyperbolic-arccosine</csymbol>
</pre>

</td><td>

<pre class="xml">
<arccosh/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">hyperbolic-arctangent</csymbol>
</pre>

</td><td>

<pre class="xml">
<arctanh/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">hyperbolic-arcsecant</csymbol>
</pre>

</td><td>

<pre class="xml">
<arcsech/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">hyperbolic-arccosecant</csymbol>
</pre>

</td><td>

<pre class="xml">
<arccsch/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">hyperbolic-arccotangent</csymbol>
</pre>

</td><td>

<pre class="xml">
<arccoth/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">exponential-function</csymbol>
</pre>

</td><td>

<pre class="xml">
<exp/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">natural-logarithm</csymbol>
</pre>

</td><td>

<pre class="xml">
<ln/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">logarithm</csymbol>
</pre>

</td><td>

<pre class="xml">
<log/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">mean</csymbol>
</pre>

</td><td>

<pre class="xml">
<mean/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">standard-deviation</csymbol>
</pre>

</td><td>

<pre class="xml">
<sdev/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">variance</csymbol>
</pre>

</td><td>

<pre class="xml">
<variance/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">median</csymbol>
</pre>

</td><td>

<pre class="xml">
<median/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">mode</csymbol>
</pre>

</td><td>

<pre class="xml">
<mode/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">moment</csymbol>
</pre>

</td><td>

<pre class="xml">
<moment/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">vector</csymbol>
</pre>

</td><td>

<pre class="xml">
<vector>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">matrix</csymbol>
</pre>

</td><td>

<pre class="xml">
<matrix>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">determinant</csymbol>
</pre>

</td><td>

<pre class="xml">
<determinant/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">transpose</csymbol>
</pre>

</td><td>

<pre class="xml">
<transpose/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">selector</csymbol>
</pre>

</td><td>

<pre class="xml">
<selector/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">vector-product</csymbol>
</pre>

</td><td>

<pre class="xml">
<vectorproduct/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">scalar-product</csymbol>
</pre>

</td><td>

<pre class="xml">
<scalarproduct/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">outer-product</csymbol>
</pre>

</td><td>

<pre class="xml">
<outerproduct/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">integer-numbers</csymbol>
</pre>

</td><td>

<pre class="xml">
<integers/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">real-numbers</csymbol>
</pre>

</td><td>

<pre class="xml">
<reals/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">rational-numbers</csymbol>
</pre>

</td><td>

<pre class="xml">
<rationals/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">natural-numbers</csymbol>
</pre>

</td><td>

<pre class="xml">
<naturalnumbers/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">complex-numbers</csymbol>
</pre>

</td><td>

<pre class="xml">
<complexes/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">prime-numbers</csymbol>
</pre>

</td><td>

<pre class="xml">
<primes/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">euler-constant</csymbol>
</pre>

</td><td>

<pre class="xml">
<exponentiale/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">imaginary-number</csymbol>
</pre>

</td><td>

<pre class="xml">
<imaginaryi/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">not-a-number</csymbol>
</pre>

</td><td>

<pre class="xml">
<notanumber/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">true</csymbol>
</pre>

</td><td>

<pre class="xml">
<true/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">false</csymbol>
</pre>

</td><td>

<pre class="xml">
<false/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">empty-set</csymbol>
</pre>

</td><td>

<pre class="xml">
<emptyset/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">pi</csymbol>
</pre>

</td><td>

<pre class="xml">
<pi/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">euler-gamma</csymbol>
</pre>

</td><td>

<pre class="xml">
<eulergamma/>
</pre>

</td></tr>
<tr><td>

<pre class="xml">
<csymbol cd="intent">infinity</csymbol>
</pre>

</td><td>

<pre class="xml">
<infinity/>
</pre>

</td></tr></tbody></table>