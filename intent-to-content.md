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

```xml
<csymbol cd="intent">concept</csymbol>
```
</td></tr><tr>
<td>number</td><td>

```xml
<cn>number</cn>
```

</td></tr><tr>
<td>reference</td><td>

```xml
to_content(ref_target_node)
```
</td></tr><tr>
<td>

```perl
intent hint? '(' arguments? ')'
```

</td><td>

```xml
<apply>
  to_content(intent)
  to_content(argument_1)
  ...
  to_content(argument_n)
</apply>
```
</td><td>"intent" head is <br>concept, reference, <br>
number, or compound expression.
</td></tr><tr><td>

```perl
intent hint? '(' arguments? ')'
```
</td><td>ignore
</td><td>"intent" head is a literal,<br> arguments are all literals
</td></tr><tr><td>

```perl
intent hint? '('
  (pre_concept_arguments ',')?
  only_concept_argument
  (',' post_arguments)? ')'
```

</td><td>

```xml
<apply>
  to_content(only_concept_argument)
  to_content(pre_concept_arguments_1)
  ...
  to_content(pre_concept_arguments_n)
  to_content(post_concept_arguments_1)
  ...
  to_content(post_concept_arguments_n)
</apply>
```

</td><td>"intent" head is a literal,<br> arguments contain <br><strong>exactly one</strong> concept argument.
</td><tr><td>

```perl
intent hint? '(' arguments? ')'
```

</td><td>

```xml
<apply>
  <csymbol cd="intent">narrative_pieces</csymbol>
  to_content(argument_1)
  ...
  to_content(argument_n)
</apply>
```

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

```
_@silent
```

</td><td></td></tr><tr>
<td>

```
_to
```

</td><td></td></tr><tr>
<td>

```
euler-constant
```

</td><td>

```xml
<csymbol cd="intent">euler-constant</csymbol>
```
</td></tr><tr>
<td>

```
0.01
```

</td><td>

```xml
<cn>0.01</cn>
```

</td></tr><tr>
<td>

```
$operator
```
</td><td>

```xml
to_content(<mo arg="operator">+</mo>)
```
</td></tr><tr>
<td>

```perl
absolute-value($inner)
```
</td><td>

```xml
<apply>
  <csymbol cd="intent">absolute-value</csymbol>
  to_content(<mrow arg="inner">...</mrow>)
</apply>
```
</td><td>"intent" head is <br>concept, reference, <br>
number, or compound expression.
</td></tr>
<tr>
<td>

```perl
$op@infix(1,2,3)
```
</td><td>

```xml
<apply>
  to_content(<mo arg="op">...</mo>)
  <cn>1</cn>
  <cn>2</cn>
  <cn>3</cn>
</apply>
```
</td><td>"intent" head is <br>concept, reference, <br>
number, or compound expression.
</td></tr>
<tr>
<td>

```perl
inverse(f)(x)
```

</td><td>

```xml
<apply>
  <apply>
    <csymbol cd="intent">inverse</csymbol>
    <csymbol cd="intent">f</csymbol>
  </apply>
  <csymbol cd="intent">x</csymbol>
</apply>
```

</td><td>"intent" head is <br>concept, reference, <br>
number, or compound expression.
</td></tr>
<tr><td>

```perl
_(_used,_for,_brevity)
```
</td><td>
</td><td>"intent" head is a literal,<br> arguments are all literals
</td></tr><tr><td>

```

_(_the, open-interval, _from, $arg1, _to $arg2)
```

</td><td>

```xml
<apply>
  <csymbol cd="intent">open-interval</csymbol>
  to_content(<msub arg="arg1">...</msub>)
  to_content(<msub arg="arg2">...</msub>)
</apply>
```

</td><td>"intent" head is a literal,<br> arguments contain <br><strong>exactly one</strong> concept argument.
</td><tr><td>

```perl
_(_the, open-interval, _from, x, _to, $arg2)
```

</td><td>

```xml
<apply>
  <csymbol cd="intent">narrative_pieces</csymbol>
  <csymbol cd="intent">open-interval</csymbol>
  <csymbol cd="intent">x</csymbol>
  to_content(<mi arg="arg2">...</mi>)
</apply>
```

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

```xml
<csymbol cd="intent">\p{Letter}</csymbol>
```

</td><td>

```xml
<ci>\p{Letter}</ci>
```

</td><td></td></tr>
<tr><td>

```xml
<csymbol cd="intent">plus</csymbol>
```

</td><td>

```xml
<plus/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">addition</csymbol>
```

</td><td>

```xml
<plus/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">interval</csymbol>
```

</td><td>

```xml
<interval>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">inverse</csymbol>
```

</td><td>

```xml
<inverse>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">function-composition</csymbol>
```

</td><td>

```xml
<compose/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">identity-function</csymbol>
```

</td><td>

```xml
<ident/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">domain</csymbol>
```

</td><td>

```xml
<domain/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">codomain</csymbol>
```

</td><td>

```xml
<codomain/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">image</csymbol>
```

</td><td>

```xml
<image/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">piecewise-function</csymbol>
```

</td><td>

```xml
<piecewise>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">otherwise</csymbol>
```

</td><td>

```xml
<otherwise>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">quotient</csymbol>
```

</td><td>

```xml
<quotient/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">factorial</csymbol>
```

</td><td>

```xml
<factorial/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">division</csymbol>
```

</td><td>

```xml
<divide/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">maximum</csymbol>
```

</td><td>

```xml
<max/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">minimum</csymbol>
```

</td><td>

```xml
<min/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">subtraction</csymbol>
```

</td><td>

```xml
<minus/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">exponentiation</csymbol>
```

</td><td>

```xml
<power/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">remainder</csymbol>
```

</td><td>

```xml
<rem/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">multiplication</csymbol>
```

</td><td>

```xml
<times/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">root</csymbol>
```

</td><td>

```xml
<root/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">greatest-common-divisor</csymbol>
```

</td><td>

```xml
<gcd/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">and</csymbol>
```

</td><td>

```xml
<and/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">or</csymbol>
```

</td><td>

```xml
<or/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">exclusive-or</csymbol>
```

</td><td>

```xml
<xor/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">not</csymbol>
```

</td><td>

```xml
<not/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">implies</csymbol>
```

</td><td>

```xml
<implies/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">for-all</csymbol>
```

</td><td>

```xml
<forall/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">exists</csymbol>
```

</td><td>

```xml
<exists/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">absolute-value</csymbol>
```

</td><td>

```xml
<abs/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">complex-conjugate</csymbol>
```

</td><td>

```xml
<conjugate/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">argument</csymbol>
```

</td><td>

```xml
<arg/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">real-part</csymbol>
```

</td><td>

```xml
<real/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">imaginary part</csymbol>
```

</td><td>

```xml
<imaginary/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">lowest-common-multiple</csymbol>
```

</td><td>

```xml
<lcm/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">floor</csymbol>
```

</td><td>

```xml
<floor/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">ceiling</csymbol>
```

</td><td>

```xml
<ceiling/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">equals</csymbol>
```

</td><td>

```xml
<eq/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">not-equals</csymbol>
```

</td><td>

```xml
<neq/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">greater-than</csymbol>
```

</td><td>

```xml
<gt/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">less-than</csymbol>
```

</td><td>

```xml
<lt/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">greater-than-or-equal</csymbol>
```

</td><td>

```xml
<geq/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">less-than-or-equal</csymbol>
```

</td><td>

```xml
<leq/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">equivalent</csymbol>
```

</td><td>

```xml
<equivalent/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">approximately</csymbol>
```

</td><td>

```xml
<approx/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">factor-of</csymbol>
```

</td><td>

```xml
<factorof/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">integral</csymbol>
```

</td><td>

```xml
<int/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">differentiation</csymbol>
```

</td><td>

```xml
<diff/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">partial-differentiation</csymbol>
```

</td><td>

```xml
<partialdiff/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">divergence</csymbol>
```

</td><td>

```xml
<divergence/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">gradient</csymbol>
```

</td><td>

```xml
<grad/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">curl</csymbol>
```

</td><td>

```xml
<curl/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">laplacian</csymbol>
```

</td><td>

```xml
<laplacian/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">set</csymbol>
```

</td><td>

```xml
<set>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">list</csymbol>
```

</td><td>

```xml
<list>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">union</csymbol>
```

</td><td>

```xml
<union/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">intersect</csymbol>
```

</td><td>

```xml
<intersect/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">set-inclusion</csymbol>
```

</td><td>

```xml
<in/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">set-exclusion</csymbol>
```

</td><td>

```xml
<notin/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">subset</csymbol>
```

</td><td>

```xml
<subset/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">proper-subset</csymbol>
```

</td><td>

```xml
<prsubset/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">not-subset</csymbol>
```

</td><td>

```xml
<notsubset/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">not-proper-subset</csymbol>
```

</td><td>

```xml
<notprsubset/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">set-difference</csymbol>
```

</td><td>

```xml
<setdiff/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">cardinality</csymbol>
```

</td><td>

```xml
<card/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">cartesian-product</csymbol>
```

</td><td>

```xml
<cartesianproduct/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">sum</csymbol>
```

</td><td>

```xml
<sum/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">product</csymbol>
```

</td><td>

```xml
<product/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">limits</csymbol>
```

</td><td>

```xml
<limit/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">tends-to</csymbol>
```

</td><td>

```xml
<tendsto/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">sine</csymbol>
```

</td><td>

```xml
<sin/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">cosine</csymbol>
```

</td><td>

```xml
<cos/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">tangent</csymbol>
```

</td><td>

```xml
<tan/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">secant</csymbol>
```

</td><td>

```xml
<sec/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">cosecant</csymbol>
```

</td><td>

```xml
<csc/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">catangent</csymbol>
```

</td><td>

```xml
<cot/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">arcsine</csymbol>
```

</td><td>

```xml
<arcsin/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">arccosine</csymbol>
```

</td><td>

```xml
<arccos/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">arctangent</csymbol>
```

</td><td>

```xml
<arctan/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">arcsecant</csymbol>
```

</td><td>

```xml
<arcsec/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">arccosecant</csymbol>
```

</td><td>

```xml
<arccsc/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">arccotangent</csymbol>
```

</td><td>

```xml
<arccot/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">hyperbolic-sine</csymbol>
```

</td><td>

```xml
<sinh/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">hyerpbolic-cosine</csymbol>
```

</td><td>

```xml
<cosh/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">hyperbolic-tangent</csymbol>
```

</td><td>

```xml
<tanh/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">hyperbolic-secant</csymbol>
```

</td><td>

```xml
<sech/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">hyperbolic-cosecant</csymbol>
```

</td><td>

```xml
<csch/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">hyperbolic-catangent</csymbol>
```

</td><td>

```xml
<coth/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">hyperbolic-arcsine</csymbol>
```

</td><td>

```xml
<arcsinh/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">hyperbolic-arccosine</csymbol>
```

</td><td>

```xml
<arccosh/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">hyperbolic-arctangent</csymbol>
```

</td><td>

```xml
<arctanh/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">hyperbolic-arcsecant</csymbol>
```

</td><td>

```xml
<arcsech/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">hyperbolic-arccosecant</csymbol>
```

</td><td>

```xml
<arccsch/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">hyperbolic-arccotangent</csymbol>
```

</td><td>

```xml
<arccoth/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">exponential-function</csymbol>
```

</td><td>

```xml
<exp/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">natural-logarithm</csymbol>
```

</td><td>

```xml
<ln/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">logarithm</csymbol>
```

</td><td>

```xml
<log/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">mean</csymbol>
```

</td><td>

```xml
<mean/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">standard-deviation</csymbol>
```

</td><td>

```xml
<sdev/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">variance</csymbol>
```

</td><td>

```xml
<variance/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">median</csymbol>
```

</td><td>

```xml
<median/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">mode</csymbol>
```

</td><td>

```xml
<mode/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">moment</csymbol>
```

</td><td>

```xml
<moment/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">vector</csymbol>
```

</td><td>

```xml
<vector>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">matrix</csymbol>
```

</td><td>

```xml
<matrix>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">determinant</csymbol>
```

</td><td>

```xml
<determinant/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">transpose</csymbol>
```

</td><td>

```xml
<transpose/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">selector</csymbol>
```

</td><td>

```xml
<selector/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">vector-product</csymbol>
```

</td><td>

```xml
<vectorproduct/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">scalar-product</csymbol>
```

</td><td>

```xml
<scalarproduct/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">outer-product</csymbol>
```

</td><td>

```xml
<outerproduct/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">integer-numbers</csymbol>
```

</td><td>

```xml
<integers/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">real-numbers</csymbol>
```

</td><td>

```xml
<reals/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">rational-numbers</csymbol>
```

</td><td>

```xml
<rationals/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">natural-numbers</csymbol>
```

</td><td>

```xml
<naturalnumbers/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">complex-numbers</csymbol>
```

</td><td>

```xml
<complexes/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">prime-numbers</csymbol>
```

</td><td>

```xml
<primes/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">euler-constant</csymbol>
```

</td><td>

```xml
<exponentiale/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">imaginary-number</csymbol>
```

</td><td>

```xml
<imaginaryi/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">not-a-number</csymbol>
```

</td><td>

```xml
<notanumber/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">true</csymbol>
```

</td><td>

```xml
<true/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">false</csymbol>
```

</td><td>

```xml
<false/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">empty-set</csymbol>
```

</td><td>

```xml
<emptyset/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">pi</csymbol>
```

</td><td>

```xml
<pi/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">euler-gamma</csymbol>
```

</td><td>

```xml
<eulergamma/>
```

</td></tr>
<tr><td>

```xml
<csymbol cd="intent">infinity</csymbol>
```

</td><td>

```xml
<infinity/>
```

</td></tr></tbody></table>