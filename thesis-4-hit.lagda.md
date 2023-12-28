Import
```agda
{-#  OPTIONS --rewriting --without-K  #-}

module thesis-4-hit where

open import thesis-4-ua public
```

We defined inductive types which are 'freely generated by some points'
Now we declare the existence of some types which is
'freely generated by some points and higher paths'.
Such a type is called a higher inductive type.
HoTT in Agda doesn't support automated application of inference rules of higher inductive types,
hence we have to directly handle elimination rules by our hands
and manually append some judgemental equalities corresponding to computation rules.

First declare the following command in Agda,
which will allow us to add some judgemental equalities.
```agda
{-# BUILTIN REWRITE _＝_ #-}
```



Pushout
```agda
postulate
  pushout : {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''}
          → (f₁ : X → A₁) (f₂ : X → A₂) → type (ℓ ⊔ ℓ' ⊔ ℓ'')

postulate
  in₁ : {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''} {f₁ : X → A₁} {f₂ : X → A₂}
      → A₁ → pushout f₁ f₂

  in₂ : {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''} {f₁ : X → A₁} {f₂ : X → A₂}
      → A₂ → pushout f₁ f₂

  glue : {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''} {f₁ : X → A₁} {f₂ : X → A₂}
      → (x : X) → in₁ {f₁ = f₁} {f₂ = f₂} (f₁ x) ＝ in₂ (f₂ x)

postulate
  pushout-elim : {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''}
          → (f₁ : X → A₁) (f₂ : X → A₂)
          → (𝓟 : pushout f₁ f₂ → type ℓ''')
          → (φ₁ : Π (𝓟 ∘ in₁)) → (φ₂ : Π (𝓟 ∘ in₂))
          → (I : (x : X) → (φ₁ (f₁ x)) ＝↑ φ₂ (f₂ x) [ glue x ]over 𝓟 )
          → Π 𝓟
```
Above code formulate formation and elimination rules of pushout.
Now we use REWRITE command to append corresponding computation rules:
```agda
postulate
  pushout-comp-1 : {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''}
          → (f₁ : X → A₁) (f₂ : X → A₂)
          → (𝓟 : pushout f₁ f₂ → type ℓ''')
          → (φ₁ : Π (𝓟 ∘ in₁)) → (φ₂ : Π (𝓟 ∘ in₂))
          → (I : (x : X) → (φ₁ (f₁ x)) ＝↑ φ₂ (f₂ x) [ glue x ]over 𝓟 )
          → (a1 : A₁)
          → (pushout-elim f₁ f₂ 𝓟 φ₁ φ₂ I) (in₁ a1) ＝ (φ₁ a1)

  pushout-comp-2 : {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''}
          → (f₁ : X → A₁) (f₂ : X → A₂)
          → (𝓟 : pushout f₁ f₂ → type ℓ''')
          → (φ₁ : Π (𝓟 ∘ in₁)) → (φ₂ : Π (𝓟 ∘ in₂))
          → (I : (x : X) → (φ₁ (f₁ x)) ＝↑ φ₂ (f₂ x) [ glue x ]over 𝓟 )
          → (a2 : A₂)
          → (pushout-elim f₁ f₂ 𝓟 φ₁ φ₂ I) (in₂ a2) ＝ (φ₂ a2)

{-# REWRITE pushout-comp-1 #-}
{-# REWRITE pushout-comp-2 #-}
```

The computation rule for path is given as follows:
(We DON'T assume the equality holds judgementally for path cases)
```agda
postulate
  pushout-comp-glue : {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''}
          → (f₁ : X → A₁) (f₂ : X → A₂)
          → (𝓟 : pushout f₁ f₂ → type ℓ''')
          → (φ₁ : Π (𝓟 ∘ in₁)) → (φ₂ : Π (𝓟 ∘ in₂))
          → (I : (x : X) → (φ₁ (f₁ x)) ＝↑ φ₂ (f₂ x) [ glue x ]over 𝓟 )
          → (x : X) → apd (pushout-elim f₁ f₂ 𝓟 φ₁ φ₂ I) (glue x) ＝ I x
```


For the non-dependent case, we induce the following recursion rule:
```agda
pushout-rec : {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''}
          → (f₁ : X → A₁) (f₂ : X → A₂)
          → (Y : type ℓ''')
          → (φ₁ : A₁ → Y) → (φ₂ : A₂ → Y)
          → (I : (x : X) → (φ₁ (f₁ x)) ＝ φ₂ (f₂ x) )
          → (pushout f₁ f₂) → Y
pushout-rec {X = X} f₁ f₂ Y φ₁ φ₂ I
  = pushout-elim f₁ f₂ (λ _ → Y) φ₁ φ₂ ((λ x → tr-const (glue x) (φ₁ (f₁ x))) ∙ₕ I)

pushout-rec-comp-1 : {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''}
          → (f₁ : X → A₁) (f₂ : X → A₂)
          → (Y : type ℓ''')
          → (φ₁ : A₁ → Y) → (φ₂ : A₂ → Y)
          → (I : (x : X) → (φ₁ (f₁ x)) ＝ φ₂ (f₂ x) )
          → (a1 : A₁) → (pushout-rec f₁ f₂ Y φ₁ φ₂ I) (in₁ a1) ＝ φ₁ a1
pushout-rec-comp-1 f₁ f₂ Y φ₁ φ₂ I a1 = refl _

pushout-rec-comp-2 : {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''}
          → (f₁ : X → A₁) (f₂ : X → A₂)
          → (Y : type ℓ''')
          → (φ₁ : A₁ → Y) → (φ₂ : A₂ → Y)
          → (I : (x : X) → (φ₁ (f₁ x)) ＝ φ₂ (f₂ x) )
          → (a2 : A₂) → (pushout-rec f₁ f₂ Y φ₁ φ₂ I) (in₂ a2) ＝ φ₂ a2
pushout-rec-comp-2 f₁ f₂ Y φ₁ φ₂ I a2 = refl _

pushout-rec-comp-glue : {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''}
          → (f₁ : X → A₁) (f₂ : X → A₂)
          → (Y : type ℓ''')
          → (φ₁ : A₁ → Y) → (φ₂ : A₂ → Y)
          → (I : (x : X) → (φ₁ (f₁ x)) ＝ φ₂ (f₂ x) )
          → (x : X) → ap (pushout-rec f₁ f₂ Y φ₁ φ₂ I) (glue x) ＝ I x
-- type checks by the above judgemental equality (refl) !
pushout-rec-comp-glue {X = X} f₁ f₂ Y φ₁ φ₂ I x
  = ap s (glue x)                ＝⟨ ap (λ - → - ∙ ap s (glue x)) (sym (∙-sym-l q)) ⟩
    q ⁻¹ ∙ q ∙ ap s (glue x)     ＝⟨ ∙-assoc (q ⁻¹) q (ap s (glue x)) ⟩
    q ⁻¹ ∙ (q ∙ ap s (glue x)) ＝⟨ ap (λ - → q ⁻¹ ∙ -) (sym (apd-const s (glue x))) ⟩
    q ⁻¹ ∙ apd s (glue x) ＝⟨ ap (λ - → q ⁻¹ ∙ -) (P x) ⟩
    q ⁻¹ ∙ (q ∙ (I x)) ＝⟨ sym (∙-assoc (q ⁻¹) q (I x)) ⟩
    q ⁻¹ ∙ q ∙ I x ＝⟨ ap (λ - → - ∙ I x) (∙-sym-l q) ⟩
    I x ∎
  where
  s = pushout-rec f₁ f₂ Y φ₁ φ₂ I
  P :  (x₁ : X) →
        apd s (glue x₁) ＝ ((λ t → tr-const (glue t) (φ₁ (f₁ t))) ∙ₕ I) x₁
  P = pushout-comp-glue f₁ f₂ (λ - → Y) φ₁ φ₂ ((λ x → tr-const (glue x) (φ₁ (f₁ x))) ∙ₕ I)
  q = tr-const (glue x) (s (in₁ (f₁ x)))
  -- apd-const s (glue x) : apd s (glue x) ＝ q ∙ ap s (glue x)
  -- P x : apd s (glue x) ＝ tr-const (glue x) (φ₁ (f₁ x)) ∙ I x
  -- 그런데 s ∘ in₁ ≡ φ₁  by computation rule
  -- P x : apd s (glue x) ＝ q ∙ I x

    -- pushout-rec 을 정의할 때 넣은 input 그대로임.
```

Now we show that the defined type becomes
the colimit of the prepushout diagram A ← X → B
"up to homotopy"

First we define a square diagram commute up to homotopy:
```agda
square :  {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''} {Y : type ℓ'''}
          → (f₁ : X → A₁) (f₂ : X → A₂) (φ₁ : A₁ → Y) (φ₂ : A₂ → Y)
          → type (ℓ ⊔ ℓ''')
square f₁ f₂ φ₁ φ₂ = φ₁ ∘ f₁ ∼ φ₂ ∘ f₂
```

Pushout with f₁ and f₂ constitutes a square with in₁ and in₂ by glue:
```agda
pushout-square :  {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''}
          → (f₁ : X → A₁) (f₂ : X → A₂)
          → square f₁ f₂ (in₁ {f₁ = f₁} {f₂ = f₂}) in₂
pushout-square f₁ f₂ = glue
```

One can show that it is "up to homotopy" initial among (square f₁ f₂).
That is, if we have a square "H : square f₁ f₂ g h" with Y as codomain of g,
there exists (up to homotopy) unique function k : (pushout f g) → Y
such that suitable commutances hold.
From this one can easily prove that pushout is up to type equivalence unique,
resulting in pushouts with the same base are equivalent.

```agda
pushout-∃ : {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''} {Y : type ℓ'''}
          → (f₁ : X → A₁) (f₂ : X → A₂) (φ₁ : A₁ → Y) (φ₂ : A₂ → Y)
          → square f₁ f₂ φ₁ φ₂
          → (pushout f₁ f₂) → Y
pushout-∃ {Y = Y} f₁ f₂ φ₁ φ₂ H = pushout-rec f₁ f₂ Y φ₁ φ₂ H

-- The above-constructed function s satisfies the homotopy
-- apₛ ∘ glue ≡ s ∘ₗ glue ∼ H
-- by pushout-comp-path.

ASSOC' :  {X : type ℓ} {x1 x2 x3 x4 x5 x6 : X}
      → (p1 : x1 ＝ x2) (p2 : x2 ＝ x3) (p3 : x3 ＝ x4) (p4 : x4 ＝ x5) (p5 : x5 ＝ x6)
      → p1 ∙ p2 ∙ (p3 ∙ p4 ∙ p5) ＝ p1 ∙ (p2 ∙ p3 ∙ p4) ∙ p5
ASSOC' (refl _) (refl _) p3 p4 p5 = refl _


pushout-! : {X : type ℓ} {A₁ : type ℓ'} {A₂ : type ℓ''} {Y : type ℓ'''}
          → (f₁ : X → A₁) (f₂ : X → A₂) (φ₁ : A₁ → Y) (φ₂ : A₂ → Y)
          → (H : square f₁ f₂ φ₁ φ₂)
          → (α : (pushout f₁ f₂) → Y)
          → (K₁ : φ₁ ∼ α ∘ in₁) → (K₂ : α ∘ in₂ ∼ φ₂)
          → (K₁ ∘ᵣ f₁) ∙ₕ (α ∘ₗ (glue {f₁ = f₁} {f₂ = f₂})) ∙ₕ (K₂ ∘ᵣ f₂) ∼ H
          → α ∼ (pushout-∃ f₁ f₂ φ₁ φ₂ H)
pushout-! {X = X} {Y = Y} f₁ f₂ φ₁ φ₂ H α K₁ K₂ COH
  = pushout-elim f₁ f₂ 𝓟
                 (ht-sym K₁)
                 K₂
                 G
  where
  φ = pushout-∃ f₁ f₂ φ₁ φ₂ H
  𝓟 = (λ z → α z ＝ φ z)
  G : (x : X) → ((ht-sym K₁) (f₁ x)) ＝↑ (K₂ (f₂ x)) [ glue x ]over 𝓟
  G x = tr (λ z → α z ＝ φ z) (glue x) ((K₁ (f₁ x)) ⁻¹) ＝⟨ tr-path-btwmaps α φ (glue x) ((K₁ (f₁ x)) ⁻¹) ⟩
        ap α (glue x) ⁻¹ ∙ K₁ (f₁ x) ⁻¹ ∙ ap φ (glue x) ＝⟨ ap (λ - → ap α (glue x) ⁻¹ ∙ K₁ (f₁ x) ⁻¹ ∙ -) φ-comp ⟩
        ap α (glue x) ⁻¹ ∙ K₁ (f₁ x) ⁻¹ ∙ H x ＝⟨ ap (λ - → ap α (glue x) ⁻¹ ∙ K₁ (f₁ x) ⁻¹ ∙ -) (sym (COH x)) ⟩
        ap α (glue x) ⁻¹ ∙ K₁ (f₁ x) ⁻¹ ∙ (K₁ (f₁ x) ∙ ap α (glue x) ∙ K₂ (f₂ x))
             ＝⟨ ASSOC' (ap α (glue x) ⁻¹) (K₁ (f₁ x) ⁻¹) (K₁ (f₁ x)) (ap α (glue x)) (K₂ (f₂ x)) ⟩
        ap α (glue x) ⁻¹ ∙ (K₁ (f₁ x) ⁻¹ ∙ K₁ (f₁ x) ∙ ap α (glue x)) ∙ K₂ (f₂ x)
             ＝⟨ ap (λ - →  ap α (glue x) ⁻¹ ∙ ( - ∙ ap α (glue x)) ∙ K₂ (f₂ x)) (∙-sym-l (K₁ (f₁ x))) ⟩
        ap α (glue x) ⁻¹ ∙ ap α (glue x) ∙ K₂ (f₂ x) ＝⟨ ap (λ - → - ∙ K₂ (f₂ x)) (∙-sym-l (ap α (glue x))) ⟩
        K₂ (f₂ x) ∎
    where
    φ-comp : ap φ (glue x) ＝ H x
    φ-comp = pushout-rec-comp-glue f₁ f₂ Y φ₁ φ₂ H x
```
COH x : K₁ (f₁ x) ∙ ap α (glue x) ∙ K₂ (f₂ x) ＝ H x

∴ The HIT pushout defines the initial cocone up to natural isomorphism.
We call the related object as hotomopy colimit. For the prepushout diagram, we call it homotopy pushout.




Suspension, spheres and the circle

Suspension and inductively defined spheres
```agda
𝚺 : (X : type ℓ) → type ℓ
𝚺 X = pushout {X = X} const⋆ const⋆  --BGS

S^_ : ℕ → 𝓤
S^ 0 = 𝟚
S^ suc n = 𝚺 (S^ n)


-- notation for S^ 1
north south : S^ 1
north = in₁ ⋆
south = in₂ ⋆

west east : north ＝ south
west = glue ₁
east = glue ₂
```



Another formulation of the circle- S¹
```agda
postulate
  S¹ : 𝓤
  base : S¹
  loop : base ＝ base

postulate
  S¹elim : (𝓟 : S¹ → type ℓ) (x : 𝓟 base) (ℓ : x ＝↑ x  [ loop ]over 𝓟 )
        → (z : S¹) → 𝓟 z

postulate
  S¹comp-base :  (𝓟 : S¹ → type ℓ) (x : 𝓟 base) (ℓ : x ＝↑ x  [ loop ]over 𝓟)
             →  ((S¹elim 𝓟 x ℓ) base) ＝ x
{-#  REWRITE S¹comp-base  #-}

postulate
  S¹comp-loop :  (𝓟 : S¹ → type ℓ) (x : 𝓟 base) (ℓ : x ＝↑ x [ loop ]over 𝓟)
             → apd (S¹elim 𝓟 x ℓ) loop ＝ ℓ
```

non-dependent case follows from the above axioms as:
```agda
S¹rec : (A : type ℓ) (a : A) (p : a ＝ a)
      → S¹ → A
S¹rec A a p = S¹elim (\ - → A) a (tr-const loop a ∙ p)


S¹rec-comp-base : (A : type ℓ) (a : A) (p : a ＝ a)
           → ((S¹rec A a p) base) ＝ a
S¹rec-comp-base  A a p  = refl a   --judgementally holds by S¹comp-base


S¹rec-comp-loop : (A : type ℓ) (a : A) (p : a ＝ a)
           → ap (S¹rec A a p) loop ＝ p
S¹rec-comp-loop  A a p
  = ap f loop   ＝⟨ ap (λ - → - ∙ (ap f loop)) (sym (∙-sym-l q)) ⟩
    (q ⁻¹ ∙ q) ∙ (ap f loop)   ＝⟨ ∙-assoc (q ⁻¹) q (ap f loop) ⟩
    q ⁻¹ ∙ (q ∙ ap f loop)   ＝⟨ ap (λ - → q ⁻¹ ∙ -) (sym (apd-const f loop)) ⟩
    q ⁻¹ ∙ (apd f loop)  ＝⟨ ap (λ - → q ⁻¹ ∙ -) (S¹comp-loop (λ - → A) a (q ∙ p)) ⟩
    q ⁻¹ ∙ (q ∙ p)   ＝⟨ sym (∙-assoc (q ⁻¹) q p) ⟩
    q ⁻¹ ∙ q ∙ p   ＝⟨ ap (λ - → - ∙ p) (∙-sym-l q) ⟩
    p ∎
  where
  f = S¹rec A a p
  q = tr-const loop a

--apd-const f loop : (apd f loop) ＝ ℓ ∙ (ap f loop) (≡ a ＝ a [loop]over (constfamily A))
--S¹comp-loop (λ - → A) a (ℓ ∙ p) : (apd f loop) ＝ ℓ ∙ p

S¹rec-loop-sym :  (A : type ℓ) (a : A) (p : a ＝ a)
      → ap (S¹rec A a p) (loop ⁻¹) ＝ p ⁻¹
S¹rec-loop-sym A a p
  = ap f (sym loop)  ＝⟨ ap-sym f loop ⟩
    sym (ap f loop)  ＝⟨ ap (\ - → sym -) (S¹rec-comp-loop A a p) ⟩
    sym p   ∎
  where
  f = S¹rec A a p
```



The circle defined by pushout
(the full subcategory of the topological circle's fundamental groupoid
generated by the north pole and the south pole)
and the circle defined by another higher inductive type
(the loop space of the topological circle) is isomorphic
```agda

𝟚toloopspace : 𝟚 → base ＝ base
𝟚toloopspace ₁ = loop
𝟚toloopspace ₂ = refl base

toS¹ : (S^ 1) → S¹
toS¹ = pushout-rec   const⋆ const⋆
        S¹
        (λ - → base)
        (λ - → base)
        𝟚toloopspace

fromS¹ : S¹ → (S^ 1)
fromS¹ = S¹rec
        (S^ 1)
        north
        (west ∙ (east ⁻¹))



app-fromS¹-loop :  ap (fromS¹) loop ＝ west ∙ east ⁻¹
app-fromS¹-loop = S¹rec-comp-loop (S^ 1) north (west ∙ (east ⁻¹))

app-toS¹ : (x : 𝟚) → ap toS¹ (glue x) ＝ 𝟚toloopspace x
app-toS¹ = pushout-rec-comp-glue const⋆ const⋆ S¹ (λ - → base) (λ - → base) 𝟚toloopspace
app-toS¹-west : ap toS¹ west ＝ loop
app-toS¹-west = app-toS¹ ₁
app-toS¹-east : ap toS¹ east ＝ refl base
app-toS¹-east = app-toS¹ ₂


tofrom : toS¹ ∘ fromS¹ ∼ id
tofrom = S¹elim
           𝓟
           loop
           looploop-over-loop
    where
    𝓟 = λ z → (toS¹ ∘ fromS¹) z ＝ id z
    looploop-over-loop : loop ＝↑ loop [ loop ]over 𝓟
    looploop-over-loop
      = tr 𝓟 loop loop ＝⟨ tr-path-btwmaps (toS¹ ∘ fromS¹) id loop loop ⟩
       (ap (toS¹ ∘ fromS¹) loop) ⁻¹ ∙ loop ∙ ap id loop ＝⟨ ap (λ - → ap (toS¹ ∘ fromS¹) loop ⁻¹ ∙ loop ∙ -) (ap-id loop) ⟩
       (ap (toS¹ ∘ fromS¹) loop) ⁻¹ ∙ loop ∙ loop ＝⟨ ap (λ - → (-) ⁻¹ ∙ loop ∙ loop) (ap-∘ toS¹ fromS¹ loop) ⟩
       (ap toS¹ (ap fromS¹ loop)) ⁻¹ ∙ loop ∙ loop ＝⟨ ap (λ - → (ap toS¹ -) ⁻¹ ∙ loop ∙ loop) app-fromS¹-loop ⟩
       (ap toS¹ (west ∙ east ⁻¹)) ⁻¹ ∙ loop ∙ loop ＝⟨ ap (λ - → - ∙ loop ∙ loop) (sym (ap-sym toS¹ (west ∙ east ⁻¹))) ⟩
       (ap toS¹ ((west ∙ east ⁻¹) ⁻¹)) ∙ loop ∙ loop ＝⟨ ap (λ - → ap toS¹ - ∙ loop ∙ loop) (sym∙ west (sym east)) ⟩
       ap toS¹ (sym (sym east) ∙ west ⁻¹) ∙ loop ∙ loop ＝⟨ ap (λ - → ap toS¹ (- ∙ west ⁻¹) ∙ loop ∙ loop) (sym²~id east) ⟩
       ap toS¹ (east ∙ west ⁻¹) ∙ loop ∙ loop ＝⟨ ap (λ - → - ∙ loop ∙ loop) (ap-∙ toS¹ east (west ⁻¹)) ⟩
       (ap toS¹ east) ∙ (ap toS¹ (west ⁻¹)) ∙ loop ∙ loop
                ＝⟨ ap (λ - → - ∙ loop ∙ loop) (ap₂ (λ - ~ → - ∙ ~) app-toS¹-east (ap-sym toS¹ west)) ⟩
       (ap toS¹ west) ⁻¹ ∙ loop ∙ loop ＝⟨ ap (λ - → (-) ⁻¹ ∙ loop ∙ loop) app-toS¹-west ⟩
       loop ⁻¹ ∙ loop ∙ loop ＝⟨ ap (λ - → - ∙ loop) (∙-sym-l loop) ⟩
       loop ∎



fromto : fromS¹ ∘ toS¹ ∼ id
fromto = pushout-elim  const⋆ const⋆
           𝓟
           (λ {⋆ → refl north})
           (λ {⋆ → east})
           refltoeast-over-glue
  where
  𝓟 = (λ z → (fromS¹ ∘ toS¹) z ＝ id z)
  refltoeast-over-glue : (x : 𝟚) → (refl north) ＝↑ east [ glue x ]over 𝓟
  refltoeast-over-glue ₁
    = tr 𝓟 west (refl north) ＝⟨ tr-path-btwmaps (fromS¹ ∘ toS¹) id west (refl north)  ⟩
      (ap (fromS¹ ∘ toS¹) west)⁻¹ ∙ (refl north) ∙ (ap id west) ＝⟨ ∙-assoc ((ap (fromS¹ ∘ toS¹) west)⁻¹) (refl north) (ap id west) ⟩
      (ap (fromS¹ ∘ toS¹) west)⁻¹ ∙ ap id west ＝⟨ ap (λ - → (ap (fromS¹ ∘ toS¹) west)⁻¹ ∙ -) (ap-id west) ⟩
      (ap (fromS¹ ∘ toS¹) west)⁻¹ ∙ west ＝⟨ ap (λ - → (-)⁻¹ ∙ west) (ap-∘ fromS¹ toS¹ west) ⟩
      (ap fromS¹ (ap toS¹ west))⁻¹ ∙ west ＝⟨ ap (λ - → (ap fromS¹ -) ⁻¹ ∙ west) app-toS¹-west ⟩
      (ap fromS¹ loop)⁻¹ ∙ west ＝⟨ ap (λ - → - ⁻¹ ∙ west) app-fromS¹-loop ⟩
      (west ∙ (east)⁻¹)⁻¹ ∙ west ＝⟨ ap (λ - → - ∙ west) (sym∙ west (east ⁻¹)) ⟩
      (east ⁻¹)⁻¹ ∙ west ⁻¹ ∙ west ＝⟨ ap (λ - → - ∙ west ⁻¹ ∙ west) (sym²~id east) ∙ (∙-assoc east (west ⁻¹) west) ⟩
      east ∙ (west ⁻¹ ∙ west) ＝⟨ ap (λ - → east ∙ -) (∙-sym-l west) ∙ (∙-refl-r east) ⟩
      east ∎

  refltoeast-over-glue ₂
    = tr 𝓟 east (refl north) ＝⟨ tr-path-btwmaps (fromS¹ ∘ toS¹) id east (refl north) ⟩
      (ap (fromS¹ ∘ toS¹) east)⁻¹ ∙ refl north ∙ ap id east ＝⟨ ∙-assoc ((ap (fromS¹ ∘ toS¹) east)⁻¹) (refl north) (ap id east) ⟩
      (ap (fromS¹ ∘ toS¹) east)⁻¹ ∙ ap id east ＝⟨ ap (λ - → (ap (fromS¹ ∘ toS¹) east)⁻¹ ∙ -) (ap-id east) ⟩
      (ap (fromS¹ ∘ toS¹) east)⁻¹ ∙ east ＝⟨ ap (λ - → (-)⁻¹ ∙ east) (ap-∘ fromS¹ toS¹ east) ⟩
      (ap fromS¹ (ap toS¹ east))⁻¹ ∙ east ＝⟨ ap (λ - → (ap fromS¹ -)⁻¹ ∙ east) app-toS¹-east ⟩
      east ∎


S^1≅S¹ : S^ 1 ≅ S¹
S^1≅S¹ = ≅pf toS¹ (Ivtbl fromS¹ tofrom fromto)
```

Now we show that (base ＝ base) ≅ ℤ