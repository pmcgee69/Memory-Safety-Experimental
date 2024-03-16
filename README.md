# Memory-Safety-Experimental
Experimenting with ways to increase memory safety in Delphi.

As of March 2024, memory safety of programming languages is in the news.
C and C++ have been excoriated for decades ... and Delphi was listed among a suggested 10 'memory safe' languages.

This came as a surprise to some  

<a href="https://forums.adug.org.au/t/memory-safety-and-delphi-in-the-news/60252">Aust Delphi Usergroup Forums</a>

---

As an example of the ability to access memory past its proper lifetime, I suggested :
(This inspired by @GrahameDGrieve’s presentation semi-recently)
```pascal
{$APPTYPE CONSOLE}
program Project1;
uses
  system.SysUtils;

type
  TMyClass = class
      field : integer;
      procedure   read;
      procedure   incr;
      constructor create;
  end;

  procedure   TMyClass.read;      begin   writeln(field);    end;
  procedure   TMyClass.incr;      begin   inc(field);        end;
  constructor TMyClass.create;    begin   field := 42;       end;

begin
  var c   := TMyClass.create;
  var fn1 := c.incr;
  var fn2 := c.read;

      c.incr;   c.read;
      c.incr;   c.read;
      c.free;
      //freeandnil(c);
      fn1();
      fn2();

      readln;
end.
```
<ul> 
Output :
<li> 43 <\li>
<li> 44 <\li>
<li> 45 <\li>
<\ul>
  
FreeAndNil works “better” than free, but only at runtime.

---

(Should be ) famous author Jarrod Hollingworth contributed here : https://forums.adug.org.au/t/memory-safety-and-delphi-in-the-news/60252/31

To help catch access in freed memory immediately, at least where the memory is not reallocated, and to catch reads, you could use something to trigger AV’s like the following instead of calling .Free:
```delphi
procedure FreeAndInvalidate(var Obj);
var
  Temp: TObject;
begin
  Temp := TObject(Obj);
  Pointer(Obj) := $C0FFEE;
  Temp.Free;
end;

  var o := TThing.Create;
  ...
  FreeAndInvalidate(o);
  ...
  // Access member field or call a method that does = AV: read of address 00C0FFEE
  if o.Something = 42 then ...
```
Calling FreeAndNil() also works but that would get mixed in with ‘access before allocated’ or other uninitialised ref code.

---

And highly experienced ADUG member Grahame Grieve had already given a talk on preventing Double Frees :

https://www.youtube.com/watch?v=zwXlkmnCGX0

---

I wanted to explore ideas about ways to prevent runtime issues :
- access to freed memory
- double frees
- other similar issues

       <li>
          <a href="Non-owning-references.html">Non-owning References</a>
       </li>
