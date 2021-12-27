# Hardware-Notes

## firrtl & Chisel-tester2 reading

### Emitter.scala
```
-E smt2:
Seq(RunFirrtlTransformAnnotation(Dependency(SMTLibEmitter)), EmitCircuitAnnotation(SMTLibEmitter.getClass))
```
### Firrtl.backends.experimental.smt.*
* ***Btor2Serializer.scala***: SMTEmitter调用，输入TransitionSystem
* ***SMTExprSerializer.scala***：SMTExpr调用，
 
BVExpr, line35, 
```
override def toString: String = SMTExprSerializer.serialize(this)
```
ArrayExpr, line192, 
```
override def toString: String = SMTExprSerializer.serialize(this)
```
* ***SMTLibSerializer.scala***: 输入TransitionSystem，转为SMT

SMTEmitter调用，

SMTTransitionSystemEncoder调用，line102,
```
private def id(s: String): String = SMTLibSerializer.escapeIdentifier(s)
```

* ***SMTTransitionSystemEncoder.scala***:
```
def encode(sys: TransitionSystem): Iterable[SMTCommand] = {...}
```

* ***FirrtlToTransitionSystem.scala***:
```
/** Contains code to convert a flat firrtl module into a functional transition system which
  * can then be exported as SMTLib or Btor2 file.
  */
  ```
输入之前需要进行firrtl module的inline，但是目前还没清楚具体再那一步完成了inline
```
// since this pass only runs on the main module, inlining needs to happen before
  override def optionalPrerequisites: Seq[TransformDependency] = Seq(Dependency[firrtl.passes.InlineInstances])
```

execute中会new一个ModuleToTransitionSystem类，调用其中的run方法
```
override protected def execute(state: CircuitState): CircuitState = {
 ...
 // convert the main module
   ...
      case m: ir.Module =>
        new ModuleToTransitionSystem(presetRegs = presetRegs, memInit = memInit, uninterpreted = uninterpreted).run(m)
    }
}
```

run方法
```
// first pass over the module to convert expressions; discover state and I/O
    m.foreachPort(onPort)
    m.foreachStmt(onStatement)
```
onPort中遍历IO类型，得到clocks和inputs，inputs中append的是BVSymbol类型
onStatement根据不同statement类型得到inputs, signals, states等

返回一个TransitionSystem
```
TransitionSystem(m.name, inputs.toList, states.values.toList, signals.toList, comments.toMap, header)
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
* ***firrtl.stage.phases.ConvertCompilerAnnotations.scala***中定义class ConvertCompilerAnnotations, 其中有：
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
定义在***firrtl.backends.experimental.smt.FirrtlToTransitionSystem.scala***中，根据注释，在FirrtlToTransitionSystem方法中进行转换：
```
/** Contains code to convert a flat firrtl module into a functional transition system which
  * can then be exported as SMTLib or Btor2 file.
  */
object FirrtlToTransitionSystem extends Transform with DependencyAPIMigration {
}
```


