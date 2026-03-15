I've been coming back to something I read in "F# in Action" about sequences and how they're lazily evaluated.  [Lazy evaluation](https://en.wikipedia.org/wiki/Lazy_evaluation) is not something I ever encountered or thought about previously as a hobbyist programmer so I had to spend more time looking into this.  To see this in action, it helps to run through some examples in FSI, the interactive REPL.

If we create a list, the REPL immediately reports back what was created:
```
> let myList = [ 1 .. 10 ];;
val myList: int list = [1; 2; 3; 4; 5; 6; 7; 8; 9; 10]
```
The same is true if we created an array:
```
> let myArray = [| 1 .. 10 |];;
val myArray: int array = [|1; 2; 3; 4; 5; 6; 7; 8; 9; 10|]
```
But look what happens if we define a sequence:
```
> let mySequence = seq { 1 .. 10 };;
val mySequence: int seq
```
Unlike lists and arrays, which are evaluated upon creation (eager evaluation), sequences are evaluated lazily, or on demand.  Where this comes in handy is illustrated by an example in the book asking us to find the dates of all Mondays between February and May 2020, inclusive.  In the example, we start by creating a sequence to contain all representable dates(!):
```
let allDates = seq {
    let mutable theDate = DateTime.MinValue
    while theDate <= DateTime.MaxValue do
        theDate
        theDate <- theDate.AddDays 1.
}
```
Using standard library functions, we can then filter this sequence to obtain only the dates we are interested in[^*]:
```
let mondays =
    allDates

    // skip all dates while function evaluates to true
    |> Seq.skipWhile (fun d -> d.Year < 2020 || d.Month < 2)

    // return only those dates where function evaluates to true
    |> Seq.filter (fun d -> d.DayOfWeek = DayOfWeek.Monday)

    // take all dates while function evaluates as true
    |> Seq.takeWhile (fun d -> d.Year = 2020 && d.Month < 6)

    // return result as array
    |> Seq.toArray
```
F# returns the result of this query almost instantaenously despite the fact that there could be over 3.6 million possible dates represented by `allDates`:
```
> DateTime.MinValue;;
val it: DateTime = 1/1/0001 12:00:00 AM ...

> DateTime.MaxValue;;
val it: DateTime = 12/31/9999 11:59:59 PM ...
```
I say "could be" because F# only evalutes sequences when a result is needed, improving the performance of our code.

To be fair, if we used lists or arrays on modern hardware, we would likely get the same result in a reasonable amount of time.  But the point is that sequences allow us to work with potentially infinite data structures without running into memory issues.  This is a powerful concept that can be applied in many different contexts, such as working with streams of data or generating large datasets on the fly. (In "F# in Action", the author does demonstrate an example of generating an array from a sequence of 100 million elements which does take a noticeable amount of time to execute; however, when we skip the `Seq.toArray` call, the results are returned almost immediately.)

A more practical example might be reading in lines from a large file in a memory-constrained environment.  Using a sequence, we can read the file line-by-line without loading the entire file into memory at once.  This allows us to process large files efficiently and avoid out-of-memory errors:
```
> let lines = System.IO.File.ReadLines("journalctl.log")
- 
- lines
- |> Seq.filter (fun line -> line.Contains("Network Time"))
- |> Seq.take 10
- |> Seq.iter (printfn "%s" )
- ;;
Dec 24 10:34:33 nixos systemd[1]: Starting Network Time Synchronization...
Dec 24 10:34:33 nixos systemd[1]: Started Network Time Synchronization.
Dec 24 20:53:37 nixos systemd[1]: Starting Network Time Synchronization...
Dec 24 20:53:37 nixos systemd[1]: Started Network Time Synchronization.
Dec 26 16:42:34 nixos systemd[1]: Starting Network Time Synchronization...
Dec 26 16:42:34 nixos systemd[1]: Started Network Time Synchronization.
Dec 26 16:58:35 nixos systemd[1]: Stopping Network Time Synchronization...
Dec 26 16:58:35 nixos systemd[1]: Stopped Network Time Synchronization.
Dec 26 16:59:27 nixos systemd[1]: Starting Network Time Synchronization...
Dec 26 16:59:27 nixos systemd[1]: Started Network Time Synchronization.
val lines: string seq
val it: unit = ()
```

[^*]: The `Seq.skipWhile` and `Seq.takeWhile` functions are used to limit the range of dates we are working with, while `Seq.filter` is used to select only the dates that fall on a Monday.  If we try to just run a filter on `allDates` without `skipWhile` and `takeWhile`, we will run into `System.ArgumentOutOfRangeException` errors, at least on my machine:
    ```
    > let mondays =                                                                              
    -     allDates                                                                               
    -     |> Seq.filter (fun d ->                                                                
    -         (d.DayOfWeek = DayOfWeek.Monday) && (d.Year = 2020) && (d.Month > 1 && d.Month < 6)
    -     );;                                                                                    
    val mondays: DateTime seq
    
    > mondays |> Seq.toArray;;                                                                   
    System.ArgumentOutOfRangeException: The added or subtracted value results in an un-representable DateTime. (Parameter 'value')
       at System.DateTime.ThrowDateArithmetic(Int32 param)
       at FSI_0005.allDates@16.GenerateNext(IEnumerable`1& next) in /home/samir/Documents/programming/fsharp/stdin:line 19
       at Microsoft.FSharp.Core.CompilerServices.GeneratedSequenceBase`1.MoveNextImpl() in /build/dotnet-10.0.103/src/fsharp/src/FSharp.Core/seqcore.fs:line 488
       at Microsoft.FSharp.Collections.Internal.IEnumerator.next@277[T](FSharpFunc`2 f, IEnumerator`1 e, FSharpRef`1 started, Unit unitVar0) in /build/dotnet-10.0.103/src/fsharp/src/FSharp.Core/seq.fs:line 278
       at Microsoft.FSharp.Collections.Internal.IEnumerator.filter@267.System.Collections.IEnumerator.MoveNext() in /build/dotnet-10.0.103/src/fsharp/src/FSharp.Core/seq.fs:line 283
       at Microsoft.FSharp.Collections.SeqModule.toArray$cont@1028[T](IEnumerator`1 e, Unit unitVar) in /build/dotnet-10.0.103/src/fsharp/src/FSharp.Core/seq.fs:line 1032
       at Microsoft.FSharp.Collections.SeqModule.ToArray[T](IEnumerable`1 source) in /build/dotnet-10.0.103/src/fsharp/src/FSharp.Core/seq.fs:line 1027
       at <StartupCode$FSI_0012>.$FSI_0012.main@() in /home/samir/Documents/programming/fsharp/stdin:line 43
       at System.RuntimeMethodHandle.InvokeMethod(ObjectHandleOnStack target, Void** arguments, ObjectHandleOnStack sig, BOOL isConstructor, ObjectHandleOnStack result)
       at System.RuntimeMethodHandle.InvokeMethod(ObjectHandleOnStack target, Void** arguments, ObjectHandleOnStack sig, BOOL isConstructor, ObjectHandleOnStack result)
       at System.Reflection.MethodBaseInvoker.InterpretedInvoke_Method(Object obj, IntPtr* args)
    at System.Reflection.RuntimeMethodInfo.Invoke(Object obj, BindingFlags invokeAttr, Binder binder, Object[] parameters, CultureInfo culture)
    Stopped due to error
    ```
