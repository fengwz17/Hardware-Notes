# Hardware-Notes

## firrtl & Chisel-tester2 reading

### Emitter.scala
```
-E smt2:
Seq(RunFirrtlTransformAnnotation(Dependency(SMTLibEmitter)), EmitCircuitAnnotation(SMTLibEmitter.getClass))
```
### Firrtl.backends.experimental.smt.*
* Btor2Serializer: SMTEmitter调用，输入TransitionSystem
* SMTExprSerializer：SMTExpr调用，
 
BVExpr, line35, 
```
override def toString: String = SMTExprSerializer.serialize(this)
```
ArrayExpr, line192, 
```
override def toString: String = SMTExprSerializer.serialize(this)
```
* SMTLibSerializer: 输入TransitionSystem，转为SMT

SMTEmitter调用，

SMTTransitionSystemEncoder调用，line102,
```
private def id(s: String): String = SMTLibSerializer.escapeIdentifier(s)
```

* SMTTransitionSystemEncoder
```
def encode(sys: TransitionSystem): Iterable[SMTCommand] = {...}
```

### Maltese.scala
* bmc调用toTransitionSystem()：
```
val sysInfo = toTransitionSystem(circuit, sysAnnos)
```

* toTransitionSystem(circuit: ir.Circuit, annos: AnnotationSeq):
```
val res = firrtlPhase.transform(
  Seq(
    RunFirrtlTransformAnnotation(Dependency(SMTLibEmitter)),
    new CurrentFirrtlStateAnnotation(Forms.LowForm),
    FirrtlCircuitAnnotation(circuit)
  ) ++: logLevel ++: annos ++: LoweringAnnos ++: opts
)
```
class FirrtlPhase定义在firrtl.FirrtlStage中:
```
class FirrtlPhase
    extends PhaseManager(
      targets = Seq(
        Dependency[firrtl.stage.phases.Compiler],
        Dependency[firrtl.stage.phases.ConvertCompilerAnnotations]
      )
    )
```
* firrtl.stage.phases.ConvertCompilerAnnotations中定义class ConvertCompilerAnnotations, 其中有：
```
override def transform(annotations: AnnotationSeq): AnnotationSeq = {
 ...
 RunFirrtlTransformAnnotation(a.emitter)
}
```
val res = firrtlPhase.transform()可能调用的是这里的transform

* 从toTransitionSystem()中依然没办法直观看出来如何进行firrtl到transition system的转换，但是在定义res之后有一行代码：
```
val sys = res.collectFirst { case TransitionSystemAnnotation(s) => s }.get
```
其中
```
case class TransitionSystemAnnotation(sys: TransitionSystem) 
```
定义在***firrtl.backends.experimental.smt.FirrtlToTransitionSystem***中，根据注释，在FirrtlToTransitionSystem方法中进行转换：
```
/** Contains code to convert a flat firrtl module into a functional transition system which
  * can then be exported as SMTLib or Btor2 file.
  */
object FirrtlToTransitionSystem extends Transform with DependencyAPIMigration {
}
```


