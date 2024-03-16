I asked on our ADUG forum :
"Someone please show me a nicer way than this to create non-owning method references …"
```pascal
{$APPTYPE CONSOLE}
program Project_Non_owning_ref;
uses
  system.SysUtils;

function make_non_owning(const [ref] c:TObject; const m:Tproc):TProc;
  begin
     var _c := @c;
     result := procedure begin
                            var x := TObject(Pointer(_c)^);
                            if  x <> nil then m()
                                         else writeln('dead call')
                         end
  end;

type
  TMyClass = class
      field : integer;
      procedure   read;
      procedure   incr;
      constructor create;
  end;

  procedure   TMyClass.read;    begin   writeln(field);    end;
  procedure   TMyClass.incr;    begin   inc(field);        end;
  constructor TMyClass.create;  begin   field := 42;       end;

begin
  var c   := TMyClass.create;
  var fn1 := make_non_owning(c,c.incr);
  var fn2 := make_non_owning(c,c.read);

      c.incr;   c.read;
      c.incr;   c.read;
      fn1();
      fn2();
      //c.free;
      freeandnil(c);
      fn1();
      fn2();

      readln;
end.
```
<h6><dl>
<dt>Output :</dt>
<dd>43</dd>
<dd>44</dd>
<dd>45</dd>
<dd>dead call</dd>
<dd>dead call</dd>
</dl></h6>

---

