# Sincronização

- Coordenação da execução de threads que de alguma forma cooperam.
  Uma forma de sincronização que pode à primeira vista ser diferente
  do que enunciei é a sincronização com as operadores de I/O (cenário
  onde a cooperação entre thread é involuntária)

- Duas operações básicas para implementar a sincronização: **Acquire** e
**Release**

-**Acquire**: operação potencialmente bloqueante e que permite à *thread*
  invocante sincronizar-se com a ocorrência de um "evento".
  
-**Release**: operação não bloqueante que reporta a ocorrência de "eventos"
  que são aguardados pela(s) thread(s) blouqeadas nas operaçoes **Acquire**.

  Bounded-Buffer com capacidade limitada
  Operações básicas: put(T) e T take()

# Anatomia de um Sincronizador Genérico
``` C#
class SynchState {}	// representa o estado do sincronizador
```

#### Exemplos:
- Semáforo: contador com o número de autorizações sob custódio do semáforo
- *Manual-reset event* : booleano que indica se o evento está ou não sinalizado
- *Unbounded-message queue*: lista das mensagens disponíveis para recepção

``` C#
public class InitializationArgs {}	// parâmetros do constructor

class AcquireArgs {}	// tipo do argumento da operação acquire

class AcquireResult {}	// tipo do resultado da operação acquire

class ReleaseArgs {}	// tipo do argumento da operação release
```

```C#
class GenericSynchronizerMonitorStylePseudoCode {
	// the lock
	private Lock _lock = new Lock();
	
	// the waiting queue
	private WaitQueue waitQueue = new WaitQueue();
	
	// the synchronization state
	private SynchState synchState;
	
	// initialize
	public GenericSynchronizerMonitorStylePseudoCode(Initialization initialState) {
		initialize "synchState" according to information specified by "initialState";
	}
	
	// check if synchronization state allows immediate acquire
	private bool CanAcquire(AcquireArgs acquireArgs) {
		returns true if "syncState" statisfies an immediate acquire according to "acquireArgs";
	}
	
	// executes the side effect processing related to successful acquire
	private AcquireResult AcquireSideEffect(AcquireArgs acquireArgs) {
		update "synchState" according to "acquireArgs" after a successful acquire;
		return "the-proper-acquire-result";
	}
	
	// update synchronization state due to a release operation
	private void UpdateStateOnRelease(ReleaseArgs releaseArgs) {
		update "synchState" according to "releaseArgs";
	}
	
	// do the Acquire operation
	public AcquireResult Acquire(AcquireArgs acquireArgs) {
		_lock.Acquire();
		try {
			while (!CanAcquire(acquireArgs)) {
				enqueue the current thread on the "waitQueue" sensible to posterior wakeups;
				int depth = _lock.ReleaseAll();
				block the current until it is waked by a releaser thread;
				_lock.ReAcquire(depth);
			}
			return AcquireSideEffect(acquireArgs);
		} finally {
			_lock.Release();
		}
	}
	
	// do the release operation
	public void Release(ReleaseArgs releaseArgs) {
		_lock.Acquire();
		try {
			UpdateStateOnRelease(releaseArgs);
			wakeup all the blocked threads that can have its acquire satisfied;
			with the new state of "syncState";
		} finally {
			_lock.Release();
		}
	}
}
```





