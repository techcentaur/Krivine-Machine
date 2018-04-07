# Krivine-Machine

Following is the implementation of an abstract machine in OCaml namely **krivine machine** which implements **call-by-name** semantics. There exists another abstract machine namely **secd machine** which implements **call-by-value** semantics, whose code can be found [here]().


## What are abstract machines?

Abstract machines got the name abstract because they permit step-by-step execution of programs and also because they omit the many details of real(hardware) machines. They bridge the gap between the high level of a programming language and the low level of a real machine. The instructions of an abstract machine are tailored to the particular operations required to implement operations of a specific source language or class of source languages.

More about it can be found [here](https://www.sciencedirect.com/science/article/pii/S0167739X99000886).

## Krivine Machine

The Krivine machine is a simple and natural implementation of the weak-head call-by-name reduction strategy for pure λ-terms. More specifically it aims to define rigorously head normal form reduction of a lambda term using call-by-name reduction. On the other hand Krivine machine implements call-by-name because it evaluates the body of a β-redex before it evaluates its parameter

### What is a call-by-name startegy?

Call by name is an evaluation strategy where the arguments to a function are not evaluated before the function is called—rather, they are substituted directly into the function body(using the closures) and then left to be evaluated whenever they appear in the function. If an argument is not used in the function body, the argument is never evaluated; if it is used several times, it is re-evaluated each time it appears, which can be optimized more to be evaluated just once and put it over some stack from which it can called everytime there is a need.
More about it can be found [here](https://en.wikipedia.org/wiki/Evaluation_strategy#Call_by_name)


### Language of expressions

For this implementation of krivine abstract machine, we consider a tiny language consisting of expressions, having the key operations of Lambda abstraction (lamba.x e), application of the abstraction (e1(e2)) and also some base operations on integer.
```ocaml

type exp = Int of int | Var of string | Abs of string * exp | App of exp * exp;;

```

Which can be extended to include operations like addition, subtraction, multiplication, division, absolution of a number on integers, and and, or, not operation on boolean types, and also some comparsion operations like greater than, less than, equal, greater than or equal, less than or equal, and also some conditional statements like if-then-else. The OCaml representations of which on expressions can be defined as - 
```ocaml

type exp = Nil | Int of int | Bool of bool | Var of string | Abs of string * exp | App of exp * exp | Absolute of exp| Not of exp
	| Add of exp*exp| Sub of exp*exp| Div of exp*exp| Mul of exp*exp| Mod of exp*exp| Exp of exp*exp
	| And of exp*exp| Or of exp*exp| Imp of exp*exp
	| Equ of exp*exp| GTEqu of exp*exp| LTEqu of exp*exp| Grt of exp*exp| Lst of exp*exp
	| Ifthenelse of (exp*exp*exp);;

```

### Krivine evaluation

Whenever we have a list of expressions (say program), we evaluate its each expression by defining it as an closure of the tuple of the expression and the environment closure. For the k-machine we also need an stack of closures.

#### Types 

The type of the above syntax can be defined in OCaml as 

```ocaml
type stackCLOS = closure list
	and closure = CLtype of exp*environmentCLOS
		and environmentCLOS = (exp*closure) list;;
```
#### Lookup of variable

On facing a base closure type, where the expression is a Integer, or boolean, or Nil; we return the some closure type. Whereas, when facing a closure type with a variable in it, we look up the variable in the attached enviornment and if found, we then insert that closure whose expression is the same as we looked up, into its environment and return the closure type. Implementation of such `lookup` function can be defined as

```ocaml
let rec lookup (e, env) = match (e, env) with
	| (e, (e1,cl)::c') -> if e1<>e then lookup (e, c') else
					(match cl with
					| CLtype (Abs (x,x1), env) -> CLtype (Abs (x, x1), (e1,cl)::env)
					| _ -> cl)
	| (e, []) -> raise Variable_not_intialized;;

```

#### Abstraction

Now, for abstraction on lambda-calculi, we define this function as 

```ocaml
(* val absApplied : closure * closure list -> closure * closure list = <fun> *)
```

Whenver we face a closure with the expression as an abstraction, we put the 'binding variable' with the closure on the top of the stack, meaning the closure of 'bounded variable', and we return this new closure type. This `absApplied` function is performing the same functionality as stated above.

```ocaml
let absApplied (cl, s) = match (cl, s) with
	| (CLtype(Abs(x ,e), env), c::c') -> (CLtype(e, (Var x ,c)::env), c')
	| (_,[]) -> raise InvalidOperation
	| _ -> raise InvalidOperation;;
```

#### Applied function.

The application of function i.e. e1(e2) can be easily defined as 
```ocaml
CLtype (App(e1, e2), env) -> krivinemachine (CLtype(e1, env)) (CLtype(e2, env)::s)
```
where CLtype is the closure.

#### The tiny krivine machine

All the functionality stated above can be combined to this tiny krivine machine evaluation function.

```ocaml

let rec krivinemachine cl (s:stackCLOS) = match cl with
	| CLtype (Int i, env) -> CLtype (Int i, env)
	| CLtype (Var v, env) -> krivinemachine (lookup (Var(v), env)) s
	| CLtype (Abs(x, e), env) -> 
					let (cl', s') = absApplied (cl, s) in
						krivinemachine cl' s'
	| CLtype (App(e1, e2), env) -> krivinemachine (CLtype(e1, env)) (CLtype(e2, env)::s)

```

The OCaml code of the krivine machine which various features can be found [here]().

## Contributing
Found a bug or have a suggestion? Feel free to create an issue or make a pull request!

## Attention
If this code is one of your assignments, I recommend you to try it out first by yourself and then come and have a look if needed. Otherwise, feel free to use it if you want to expand the language or for any other purpose.
 

## Documents for further information

- This is the official research paper of call-by-name semantic implementing machine - [A call-by-name lambda-calculus machine](https://www.irif.fr/~krivine/articles/lazymach.pdf)

- Extension of K-Machine and some new features - [A Certified Extension of the Krivine Machine for a
Call-by-Name Higher-Order Imperative Language](http://drops.dagstuhl.de/opus/volltexte/2014/4634/pdf/13.pdf)