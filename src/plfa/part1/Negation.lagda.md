---
title     : "Negation: Negation, with intuitionistic and classical logic"
layout    : page
prev      : /Connectives/
permalink : /Negation/
next      : /Quantifiers/
---

```
module plfa.part1.Negation where
```

This chapter introduces negation, and discusses intuitionistic
and classical logic.

## Imports

```
open import Relation.Binary.PropositionalEquality using (_≡_; refl)
open import Data.Nat using (ℕ; zero; suc)
open import Data.Empty using (⊥; ⊥-elim)
open import Data.Sum using (_⊎_; inj₁; inj₂)
open import Data.Product using (_×_)
open import plfa.part1.Isomorphism using (_≃_; extensionality)
```


## Negation

Given a proposition `A`, the negation `¬ A` holds if `A` cannot hold.
We formalise this idea by declaring negation to be the same
as implication of false:
```
¬_ : Set → Set
¬ A = A → ⊥
```
This is a form of _reductio ad absurdum_: if assuming `A` leads
to the conclusion `⊥` (an absurdity), then we must have `¬ A`.

Evidence that `¬ A` holds is of the form

    λ{ x → N }

where `N` is a term of type `⊥` containing as a free variable `x` of type `A`.
In other words, evidence that `¬ A` holds is a function that converts evidence
that `A` holds into evidence that `⊥` holds.

Given evidence that both `¬ A` and `A` hold, we can conclude that `⊥` holds.
In other words, if both `¬ A` and `A` hold, then we have a contradiction:
```
¬-elim : ∀ {A : Set}
  → ¬ A
  → A
    ---
  → ⊥
¬-elim ¬x x = ¬x x
```
Here we write `¬x` for evidence of `¬ A` and `x` for evidence of `A`.  This
means that `¬x` must be a function of type `A → ⊥`, and hence the application
`¬x x` must be of type `⊥`.  Note that this rule is just a special case of `→-elim`.

We set the precedence of negation so that it binds more tightly
than disjunction and conjunction, but less tightly than anything else:
```
infix 3 ¬_
```
Thus, `¬ A × ¬ B` parses as `(¬ A) × (¬ B)` and `¬ m ≡ n` as `¬ (m ≡ n)`.

In _classical_ logic, we have that `A` is equivalent to `¬ ¬ A`.
As we discuss below, in Agda we use _intuitionistic_ logic, where
we have only half of this equivalence, namely that `A` implies `¬ ¬ A`:
```
¬¬-intro : ∀ {A : Set}
  → A
    -----
  → ¬ ¬ A
¬¬-intro x  =  λ{¬x → ¬x x}
```
Let `x` be evidence of `A`. We show that assuming
`¬ A` leads to a contradiction, and hence `¬ ¬ A` must hold.
Let `¬x` be evidence of `¬ A`.  Then from `A` and `¬ A`
we have a contradiction, evidenced by `¬x x`.  Hence, we have
shown `¬ ¬ A`.

An equivalent way to write the above is as follows:
```
¬¬-intro′ : ∀ {A : Set}
  → A
    -----
  → ¬ ¬ A
¬¬-intro′ x ¬x = ¬x x
```
Here we have simply converted the argument of the lambda term
to an additional argument of the function.  We will usually
use this latter style, as it is more compact.

We cannot show that `¬ ¬ A` implies `A`, but we can show that
`¬ ¬ ¬ A` implies `¬ A`:
```
¬¬¬-elim : ∀ {A : Set}
  → ¬ ¬ ¬ A
    -------
  → ¬ A
¬¬¬-elim ¬¬¬x  =  λ x → ¬¬¬x (¬¬-intro x)
```
Let `¬¬¬x` be evidence of `¬ ¬ ¬ A`. We will show that assuming
`A` leads to a contradiction, and hence `¬ A` must hold.
Let `x` be evidence of `A`. Then by the previous result, we
can conclude `¬ ¬ A`, evidenced by `¬¬-intro x`.  Then from
`¬ ¬ ¬ A` and `¬ ¬ A` we have a contradiction, evidenced by
`¬¬¬x (¬¬-intro x)`.  Hence we have shown `¬ A`.

Another law of logic is _contraposition_,
stating that if `A` implies `B`, then `¬ B` implies `¬ A`:
```
contraposition : ∀ {A B : Set}
  → (A → B)
    -----------
  → (¬ B → ¬ A)
contraposition f ¬y x = ¬y (f x)
```
Let `f` be evidence of `A → B` and let `¬y` be evidence of `¬ B`.  We
will show that assuming `A` leads to a contradiction, and hence `¬ A`
must hold. Let `x` be evidence of `A`.  Then from `A → B` and `A` we
may conclude `B`, evidenced by `f x`, and from `B` and `¬ B` we may
conclude `⊥`, evidenced by `¬y (f x)`.  Hence, we have shown `¬ A`.

Using negation, it is straightforward to define inequality:
```
_≢_ : ∀ {A : Set} → A → A → Set
x ≢ y  =  ¬ (x ≡ y)
```
It is trivial to show distinct numbers are not equal:
```
_ : 1 ≢ 2
_ = λ()
```
This is our first use of an absurd pattern in a lambda expression.
The type `M ≡ N` is occupied exactly when `M` and `N` simplify to
identical terms. Since `1` and `2` simplify to distinct normal forms,
Agda determines that there is no possible evidence that `1 ≡ 2`.
As a second example, it is also easy to validate
Peano's postulate that zero is not the successor of any number:
```
peano : ∀ {m : ℕ} → zero ≢ suc m
peano = λ()
```
The evidence is essentially the same, as the absurd pattern matches
all possible evidence of type `zero ≡ suc m`.

Given the correspondence of implication to exponentiation and
false to the type with no members, we can view negation as
raising to the zero power.  This indeed corresponds to what
we know for arithmetic, where

    0 ^ n  ≡  1,  if n ≡ 0
           ≡  0,  if n ≢ 0

Indeed, there is exactly one proof of `⊥ → ⊥`.  We can write
this proof two different ways:
```
id : ⊥ → ⊥
id x = x

id′ : ⊥ → ⊥
id′ ()
```
But, using extensionality, we can prove these equal:
```
id≡id′ : id ≡ id′
id≡id′ = extensionality (λ())
```
By extensionality, `id ≡ id′` holds if for every
`x` in their domain we have `id x ≡ id′ x`. But there
is no `x` in their domain, so the equality holds trivially.

Indeed, we can show any two proofs of a negation are equal:
```
assimilation : ∀ {A : Set} (¬x ¬x′ : ¬ A) → ¬x ≡ ¬x′
assimilation ¬x ¬x′ = extensionality (λ x → ⊥-elim (¬x′ x))
```
Evidence for `¬ A` implies that any evidence of `A`
immediately leads to a contradiction.  But extensionality
quantifies over all `x` such that `A` holds, hence any
such `x` immediately leads to a contradiction,
again causing the equality to hold trivially.


#### Exercise `<-irreflexive` (recommended)

Using negation, show that
[strict inequality](/Relations/#strict-inequality)
is irreflexive, that is, `n < n` holds for no `n`.

```
open import plfa.part1.Relations using (_<_; s<s; z<s)

<-irreflexive : ∀ {n : ℕ} → ¬ n < n
-- you just cannot construct z<z (z<s takes zero and suc n)
<-irreflexive (s<s n<n) = <-irreflexive n<n
```


#### Exercise `trichotomy` (practice)

Show that strict inequality satisfies
[trichotomy](/Relations/#trichotomy),
that is, for any naturals `m` and `n` exactly one of the following holds:

* `m < n`
* `m ≡ n`
* `m > n`

Here "exactly one" means that not only one of the three must hold,
but that when one holds the negation of the other two must also hold.

```
open import Relation.Binary.PropositionalEquality using (cong)

data Trichotomy (m n : ℕ) : Set where
  forward :
      m < n
    → ¬ n < m
    → m ≢ n
      ---------
    → Trichotomy m n

  flipped :
      n < m
    → ¬ m < n
    → m ≢ n
      ---------
    → Trichotomy m n

  equal :
      m ≡ n
    → ¬ m < n
    → ¬ n < m
      ---------
    → Trichotomy m n

<-trichotomy : ∀ (m n : ℕ) → Trichotomy m n
<-trichotomy zero zero = equal refl (λ ()) λ ()
<-trichotomy zero (suc n) = forward z<s (λ ()) (λ ())
<-trichotomy (suc m) zero = flipped z<s (λ ()) (λ ())
<-trichotomy (suc m) (suc n) with <-trichotomy m n
-- TODO: why is the type of m≢n m ≢ m?
... | forward m<n ¬n<m m≢n =
              forward (s<s m<n)
                      (λ {(s<s n<m) → ¬n<m n<m})
                      λ {refl → m≢n refl}
... | flipped n<m ¬m<n m≢n =
              flipped (s<s n<m)
                      (λ {(s<s m<n) → ¬m<n m<n})
                      λ {refl → m≢n refl}
... | equal m≡n ¬m<n ¬n<m  =
              equal (cong suc m≡n)
                    (λ {(s<s m<n) → ¬m<n m<n})
                    λ {(s<s n<m) → ¬n<m n<m}
```

#### Exercise `⊎-dual-×` (recommended)

Show that conjunction, disjunction, and negation are related by a
version of De Morgan's Law.

    ¬ (A ⊎ B) ≃ (¬ A) × (¬ B)

This result is an easy consequence of something we've proved previously.

```
open import Data.Product using (_,_; proj₁; proj₂)

⊎-dual-× : ∀ {A B : Set} → ¬ (A ⊎ B) ≃ (¬ A) × (¬ B)
⊎-dual-× =
  record
    { to = λ ¬a⊎b → (λ a → ¬a⊎b (inj₁ a)) , λ b → ¬a⊎b (inj₂ b)
    ; from = λ { ¬a×¬b (inj₁ a) → proj₁ ¬a×¬b a
               ; ¬a×¬b (inj₂ b) → proj₂ ¬a×¬b b
               }
    ; from∘to = λ { ¬a⊎b → extensionality λ { (inj₁ x) → refl
                                            ; (inj₂ y) → refl
                                            }
                  }
    ; to∘from = λ ¬a×¬b → refl
    }

import plfa.part1.Connectives as C

⊎-dual-×′ : ∀ {A B : Set} → (A C.⊎ B → ⊥) ≃ (A → ⊥) C.× (B → ⊥)
⊎-dual-×′ = C.→-distrib-⊎
```


Do we also have the following?

    ¬ (A × B) ≃ (¬ A) ⊎ (¬ B)

If so, prove; if not, can you give a relation weaker than
isomorphism that relates the two sides?

Answer: Only the `from` part of the isomorphism holds. It's also
interesting to note that there's a progression in strength from
implication to equivalence to embedding to isomorphism.

```
⊎-x : ∀ {A B : Set} → ¬ A ⊎ ¬ B → ¬ (A × B)
⊎-x = λ { (inj₁ ¬a) (a , _) → ¬a a ; (inj₂ ¬b) (_ , b) → ¬b b}
```
## Intuitive and Classical logic

In Gilbert and Sullivan's _The Gondoliers_, Casilda is told that
as an infant she was married to the heir of the King of Batavia, but
that due to a mix-up no one knows which of two individuals, Marco or
Giuseppe, is the heir.  Alarmed, she wails "Then do you mean to say
that I am married to one of two gondoliers, but it is impossible to
say which?"  To which the response is "Without any doubt of any kind
whatever."

Logic comes in many varieties, and one distinction is between
_classical_ and _intuitionistic_. Intuitionists, concerned
by assumptions made by some logicians about the nature of
infinity, insist upon a constructionist notion of truth.  In
particular, they insist that a proof of `A ⊎ B` must show
_which_ of `A` or `B` holds, and hence they would reject the
claim that Casilda is married to Marco or Giuseppe until one of the
two was identified as her husband.  Perhaps Gilbert and Sullivan
anticipated intuitionism, for their story's outcome is that the heir
turns out to be a third individual, Luiz, with whom Casilda is,
conveniently, already in love.

Intuitionists also reject the law of the excluded middle, which
asserts `A ⊎ ¬ A` for every `A`, since the law gives no clue as to
_which_ of `A` or `¬ A` holds. Heyting formalised a variant of
Hilbert's classical logic that captures the intuitionistic notion of
provability. In particular, the law of the excluded middle is provable
in Hilbert's logic, but not in Heyting's.  Further, if the law of the
excluded middle is added as an axiom to Heyting's logic, then it
becomes equivalent to Hilbert's.  Kolmogorov showed the two logics
were closely related: he gave a double-negation translation, such that
a formula is provable in classical logic if and only if its
translation is provable in intuitionistic logic.

Propositions as Types was first formulated for intuitionistic logic.
It is a perfect fit, because in the intuitionist interpretation the
formula `A ⊎ B` is provable exactly when one exhibits either a proof
of `A` or a proof of `B`, so the type corresponding to disjunction is
a disjoint sum.

(Parts of the above are adopted from "Propositions as Types", Philip Wadler,
_Communications of the ACM_, December 2015.)

## Excluded middle is irrefutable

The law of the excluded middle can be formulated as follows:
```
postulate
  em : ∀ {A : Set} → A ⊎ ¬ A
```
As we noted, the law of the excluded middle does not hold in
intuitionistic logic.  However, we can show that it is _irrefutable_,
meaning that the negation of its negation is provable (and hence that
its negation is never provable):
```
em-irrefutable : ∀ {A : Set} → ¬ ¬ (A ⊎ ¬ A)
em-irrefutable = λ k → k (inj₂ (λ x → k (inj₁ x)))
```
The best way to explain this code is to develop it interactively:

    em-irrefutable k = ?

Given evidence `k` that `¬ (A ⊎ ¬ A)`, that is, a function that given a
value of type `A ⊎ ¬ A` returns a value of the empty type, we must fill
in `?` with a term that returns a value of the empty type.  The only way
we can get a value of the empty type is by applying `k` itself, so let's
expand the hole accordingly:

    em-irrefutable k = k ?

We need to fill the new hole with a value of type `A ⊎ ¬ A`. We don't have
a value of type `A` to hand, so let's pick the second disjunct:

    em-irrefutable k = k (inj₂ λ{ x → ? })

The second disjunct accepts evidence of `¬ A`, that is, a function
that given a value of type `A` returns a value of the empty type.  We
bind `x` to the value of type `A`, and now we need to fill in the hole
with a value of the empty type.  Once again, the only way we can get a
value of the empty type is by applying `k` itself, so let's expand the
hole accordingly:

    em-irrefutable k = k (inj₂ λ{ x → k ? })

This time we do have a value of type `A` to hand, namely `x`, so we can
pick the first disjunct:

    em-irrefutable k = k (inj₂ λ{ x → k (inj₁ x) })

There are no holes left! This completes the proof.

The following story illustrates the behaviour of the term we have created.
(With apologies to Peter Selinger, who tells a similar story
about a king, a wizard, and the Philosopher's stone.)

Once upon a time, the devil approached a man and made an offer:
"Either (a) I will give you one billion dollars, or (b) I will grant
you any wish if you pay me one billion dollars.
Of course, I get to choose whether I offer (a) or (b)."

The man was wary.  Did he need to sign over his soul?
No, said the devil, all the man need do is accept the offer.

The man pondered.  If he was offered (b) it was unlikely that he would
ever be able to buy the wish, but what was the harm in having the
opportunity available?

"I accept," said the man at last.  "Do I get (a) or (b)?"

The devil paused.  "I choose (b)."

The man was disappointed but not surprised.  That was that, he thought.
But the offer gnawed at him.  Imagine what he could do with his wish!
Many years passed, and the man began to accumulate money.  To get the
money he sometimes did bad things, and dimly he realised that
this must be what the devil had in mind.
Eventually he had his billion dollars, and the devil appeared again.

"Here is a billion dollars," said the man, handing over a valise
containing the money.  "Grant me my wish!"

The devil took possession of the valise.  Then he said, "Oh, did I say
(b) before?  I'm so sorry.  I meant (a).  It is my great pleasure to
give you one billion dollars."

And the devil handed back to the man the same valise that the man had
just handed to him.

(Parts of the above are adopted from "Call-by-Value is Dual to Call-by-Name",
Philip Wadler, _International Conference on Functional Programming_, 2003.)


#### Exercise `Classical` (stretch)

Consider the following principles:

  * Excluded Middle: `A ⊎ ¬ A`, for all `A`
  * Double Negation Elimination: `¬ ¬ A → A`, for all `A`
  * Peirce's Law: `((A → B) → A) → A`, for all `A` and `B`.
  * Implication as disjunction: `(A → B) → ¬ A ⊎ B`, for all `A` and `B`.
  * De Morgan: `¬ (¬ A × ¬ B) → A ⊎ B`, for all `A` and `B`.

Show that each of these implies all the others.

```
em→dne : (∀ {A : Set} → A ⊎ ¬ A) → ∀ {A : Set} → ¬ ¬ A → A
em→dne em ¬¬a with em
em→dne em ¬¬a | inj₁ a = a
em→dne em ¬¬a | inj₂ ¬a = ⊥-elim (¬¬a ¬a)

em→pierce : (∀ {A : Set} → A ⊎ ¬ A) → ∀ {A B : Set} → ((A → B) → A) → A
em→pierce em aba with em
em→pierce em aba | inj₁ a = a
em→pierce em aba | inj₂ ¬a = aba (λ a → ⊥-elim (¬a a))

em→impl-disj : (∀ {A : Set} → A ⊎ ¬ A) → ∀ {A B : Set} → (A → B) → ¬ A ⊎ B
em→impl-disj em ab with em
em→impl-disj em ab | inj₁ a = inj₂ (ab a)
em→impl-disj em ab | inj₂ ¬a = inj₁ ¬a

em→demorgan : (∀ {A : Set} → A ⊎ ¬ A) → ∀ {A B : Set} → ¬ (¬ A × ¬ B) → A ⊎ B
em→demorgan em ¬¬a×¬b with em
em→demorgan em ¬¬a×¬b | inj₁ a = inj₁ a
em→demorgan em ¬¬a×¬b | inj₂ ¬a with em
em→demorgan em ¬¬a×¬b | inj₂ ¬a | inj₁ b = inj₂ b
em→demorgan em ¬¬a×¬b | inj₂ ¬a | inj₂ ¬b = ⊥-elim (¬¬a×¬b (¬a , ¬b))

dne→em : (∀ {A : Set} → ¬ ¬ A → A) → ∀ {A : Set} → A ⊎ ¬ A
dne→em dne = dne em-irrefutable
-- dne→em dne = dne λ z → z (inj₁ (dne (λ z₁ → z (inj₂ z₁))))

dne→pierce : (∀ {A : Set} → ¬ ¬ A → A) → ∀ {A B : Set} → ((A → B) → A) → A
dne→pierce dne = em→pierce (dne→em dne)

dne→impl-disj : (∀ {A : Set} → ¬ ¬ A → A) → ∀ {A B : Set} → (A → B) → ¬ A ⊎ B
dne→impl-disj dne = em→impl-disj (dne→em dne)

dne→demorgan : (∀ {A : Set} → ¬ ¬ A → A) → ∀ {A B : Set} → ¬ (¬ A × ¬ B) → A ⊎ B
dne→demorgan dne = em→demorgan (dne→em dne)

pierce→em : (∀ {A B : Set} → ((A → B) → A) → A) → ∀ {A : Set} → A ⊎ ¬ A
pierce→em pierce = pierce (λ z → inj₂ (λ x → z (inj₁ x)))

pierce→dne : (∀ {A B : Set} → ((A → B) → A) → A) → ∀ {A : Set} → ¬ ¬ A → A
pierce→dne pierce = em→dne (pierce→em pierce)

pierce→impl-disj : (∀ {A B : Set} → ((A → B) → A) → A) → ∀ {A B : Set} → (A → B) → ¬ A ⊎ B
pierce→impl-disj pierce = em→impl-disj (pierce→em pierce)

pierce→demorgan : (∀ {A B : Set} → ((A → B) → A) → A) → ∀ {A B : Set} → ¬ (¬ A × ¬ B) → A ⊎ B
pierce→demorgan pierce = em→demorgan (pierce→em pierce)

⊍-symmetry : ∀ {A : Set} → ¬ A ⊎ A → A ⊎ ¬ A
⊍-symmetry (inj₁ ¬x) = inj₂ ¬x
⊍-symmetry (inj₂ x) = inj₁ x

impl-disj→em : (∀ {A B : Set} → (A → B) → ¬ A ⊎ B) → ∀ {A : Set} → A ⊎ ¬ A
impl-disj→em impl-disj = ⊍-symmetry (impl-disj (λ a → a))

impl-disj→dne : (∀ {A B : Set} → (A → B) → ¬ A ⊎ B) → ∀ {A : Set} → ¬ ¬ A → A
impl-disj→dne impl-disj = em→dne (impl-disj→em impl-disj)

impl-disj→pierce : (∀ {A B : Set} → (A → B) → ¬ A ⊎ B) → ∀ {A B : Set} → ((A → B) → A) → A
impl-disj→pierce impl-disj = em→pierce (impl-disj→em impl-disj)

impl-disj→demorgan : (∀ {A B : Set} → (A → B) → ¬ A ⊎ B) → ∀ {A B : Set} → ¬ (¬ A × ¬ B) → A ⊎ B
impl-disj→demorgan impl-disj  = em→demorgan (impl-disj→em impl-disj)

demorgan→em : (∀ {A B : Set} → ¬ (¬ A × ¬ B) → A ⊎ B) → {A : Set} → A ⊎ ¬ A
demorgan→em demorgan = demorgan (λ ¬a×¬¬a → proj₂ ¬a×¬¬a (proj₁ ¬a×¬¬a))

demorgan→dne : (∀ {A B : Set} → ¬ (¬ A × ¬ B) → A ⊎ B) → {A : Set} → ¬ ¬ A → A
demorgan→dne demorgan = em→dne (demorgan→em demorgan)

demorgan→pierce : (∀ {A B : Set} → ¬ (¬ A × ¬ B) → A ⊎ B) → {A B : Set} → ((A → B) → A) → A
demorgan→pierce demorgan = em→pierce (demorgan→em demorgan)

demorgan→impl-disj : (∀ {A B : Set} → ¬ (¬ A × ¬ B) → A ⊎ B) → {A B : Set} → (A → B) → ¬ A ⊎ B
demorgan→impl-disj demorgan = em→impl-disj (demorgan→em demorgan)
```


#### Exercise `Stable` (stretch)

Say that a formula is _stable_ if double negation elimination holds for it:
```
Stable : Set → Set
Stable A = ¬ ¬ A → A
```
Show that any negated formula is stable, and that the conjunction
of two stable formulas is stable.

```
neg-stable : ∀ {A : Set} → Stable (¬ A)
-- neg-stable = λ ¬¬¬a a → (¬¬¬-elim ¬¬¬a) a
neg-stable = ¬¬¬-elim

×-stable : ∀ {A B : Set}
  → Stable A
  → Stable B
    --------------
  → Stable (A × B)
×-stable sa sb = λ ¬¬a×b → (sa (λ ¬a → ¬¬a×b (λ a×b → ¬a (proj₁ a×b)))) , sb (λ ¬b → ¬¬a×b (λ a×b → ¬b (proj₂ a×b)))

```

## Standard Prelude

Definitions similar to those in this chapter can be found in the standard library:
```
import Relation.Nullary using (¬_)
import Relation.Nullary.Negation using (contraposition)
```

## Unicode

This chapter uses the following unicode:

    ¬  U+00AC  NOT SIGN (\neg)
    ≢  U+2262  NOT IDENTICAL TO (\==n)
