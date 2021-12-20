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
