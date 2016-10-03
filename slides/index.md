- title : Session#
- description : Session types for .Net
- author : Fahd Abdeljallal
- theme : sky
- transition : default



<img width="250" src="images/logo.png">

## Session types for F#

> **By** Fahd Abdeljallal

***

### Table of Contents

* [Motivation](#motivation)
* [What are Session Types?](#session)
* [Scribble: A language to describe application-level protocols](#scribble)
* [Session#: A Generative Type Provider](#session2)
* [Demo](#demo)
* [In Depth](#depth)

***

### Motivation <a name="motivation"></a>

* Codify the structure of communication
* Avoid communication errors such as deadlocks, communication mismatches 
* Provide type-safe way of communicating regarding a network protocol.

***

<div style="float: right;" width ="50%">
    <ul>
    <li>The adoption of session types in
         mainstream programming languages is challenging </li>     
    <li> Disadvantages of code generation </li>
        <ul>
        <li> Maintanability </li>     
        <li> Increases the overall project size and complexity </li>     
        <li> The generated types can be easily tempered </li>     
        </ul>     
    </ul>

</div>

---

### Background -- Type Provider 

    type Test = JsonProvider<""" { "name" : "Fahd" ,"age" : "22" } """>
    let test = Test.Parse(""" { "name" : "Khalissa" , "age" : "1" } """)
    let age = test.Age // -> 1
    let name = test.Name // -> Khalissa

* Compiler extension
* Generate types into the compiler instead of providing them in the source code
* Provides types ***On-Demand***
* Using Meta-information as a schema

***
### Example 

* Fibonacci 
    * Client - Server interaction/conversation
    * Client 
        * sends 2 Values = **U**_n_ and **U**_n+1_
        * finishes _conversation_
    * Server 
        * receives 2 Values and adds them ***OR*** the conversation 
        * sends the result


<img width="400" src="images/example.png">

***

### Session types <a name="session"></a>

* Formalism to describe protocol
* 2 Kinds: 
    * Binary
    * Multiparty
* Global Type

<blockquote>
<p>1 : C &#x2192; S :  c{&mu;<strong>t</strong>.fib &#60; int,int &#62; 
: S &#x2192; C : fib &#60; int &#62;.<strong>t</strong> <br>
bye &#60; &#62; .end }</p>
</blockquote>


---

### Session types 

<img width="400" src="images/operations.png">

***

### Session types

<ul>
<li>
Local Type = projection over a local role
<li><p> &#8660; <strong> CFSM </strong> </p> </li>
</li>
<li> <strong> Duality </strong> </li>
</ul>

<img width="400" src="images/st.gif">

***

### Scribble <a name="scribble"></a>

* Language to describe application-level protocols
* Based on session types Formalism
* Verify (Well-formedness)
* Project (Global-type *to* Local-type ) 


    [lang=yaml]
    module demo;

    type <fsharp> "System.Int32" from "Location" as Int;

    global protocol Fibonacci(role A, role B){
        rec Fib {
            choice at A {
                fib(Int,Int) from A to B;
                fib(Int) from B to A;
                continue Fib;
            } or {
                bye() from A to B;
            }
        }
    }

***

### Local Type via Projection

    [lang=yaml]
    local protocol Fibonacci_A at A(A, B) {
        rec Fib {
            choice at A{
                fib(Int, Int) to B;
                fib(Int) from B;
                continue Fib;
            } or {
                bye() to B;
            }
        }
    }

---
    [lang=yaml]
    local protocol Fibonacci_B at B(A, B) {
        rec Fib {
            choice at A{
                fib(Int, Int) from A;
                fib(Int) to A;
                continue Fib;
            } or {
                bye() from A;
            }
        }
    }

***

<img width="500" src="images/fibo.png">

***

### Session types primitives - Send and Receive <a name="session2"></a>
    


    [lang=yaml]
    fib(Int) from B to A;

<div align = "left">
     <ul>
        <li> Implementation for B </li>
    </ul>
</div>

    let c = new Provided.TypeProviderFile<"Scribble.scr",protocol,"B">() 
    
    let n = 5                            
    c.sendfib(A,n)

<div align = "left">
    <ul>
        <li> Implementation for A </li>
    </ul>
</div>

    
    let c = new Provided.TypeProviderFile<"Scribble.scr",protocol,"A">()
    
    let buf = new DomainModel.Buf<int>()       
    c.receivefib(B,buf)

***

### Session types primitives -- Selection

    [lang=yaml]
    choice at A {
        fib(Int,Int) from A to B;
        ...
    } or {
         bye() from A to B;
    }

Implementation for A

    let n = 5 , let m = 8
    match value with
        |0 -> c.sendbye(B)
        |n -> c.sendfib(B,n,m)

***

### Session types primitives -- Branching


    [lang=yaml]
    choice at A {
        fib(Int,Int) from A to B;
        ...
    } or {
         bye() from A to B;
    }

Implementation for B

    let buf1,buf2 = new DomainModel.Buf<int>()

    match c.branch() with
        | :? ByeChoice as bye -> bye.receive(A)
        | :? FibChoice as fib -> fib.receive(A,buf1,buf2)

***


### IntelliSense

<img width="1000" src="images/intellisense.jpg">

***

### General Architecture 

<img width="1000" src="images/Design.png">

***

### Demo-time <a name="demo"></a>

![alt text](images/logo.png "Session#")

***

### Generative Type Provider <a name="depth"></a>

<img width="1000" src="images/done.png">

***

### Communication Architecture 

<img width="600" src="images/Actor.png">

***

### Communication Architecture 

    type MessageAction =
        |SendMessage of byte [] * string 
        |ReceiveMessage of byte[] list * string * string list * AsyncReplyChannel<byte [] list> 
        |ReceiveMessageAsync of byte[] list * string * string list * AsyncReplyChannel<byte [] list>
    
---
### Multi-Stage Programming

<img width="560" src="images/msp.png">

***

### Conclusion 

* An F# Library for :
    * Distributed programming
    * On-demand static generation types
    * Supporting Asynchronous + Synchronous operations
    * Basic session types operators
    * ***.NET*** Types

***

### Future Work 

* Linearity at Compile-time


    let x = new A_Channel()
    let a = x.doSomething()
    //let b = x.doSomething()


* Add support for Multi-thread programming
* Improve support for branching with lambdas or Discriminated Unions

***

### Question

    type Question = TypeProvider<""" "Ask": "A question" """>
    let question = Question.Parse(""" "Ask" : "Session types rock" """)
    let response = question.Ask
***