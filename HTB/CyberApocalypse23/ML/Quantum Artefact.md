solution was to use the script (see the folder), get the counts and then convert the bytes to hexadecimal and then decrypt this


script.py
```
#!/usr/bin/env python

from qat.interop.openqasm import OqasmParser
from qiskit import QuantumCircuit, Aer, execute
from qiskit.visualization import plot_histogram

##### Printing all the logic gates and that #####
"""
with open("quantum_artifact.qasm", "r") as f:
        all_data = f.read()

parser = OqasmParser()
circuit = parser.compile(all_data)
print("the gates: ", list(circuit.iterate_simple()))
#"""



##### Aer - "running" it #####
sim = Aer.get_backend('qasm_simulator')
job = execute(qc, sim, shots=1000)
result = job.result()
#result = sim.run(qc).result()
counts = result.get_counts()
print("Counts: ", counts) --> THIS I THE KEY:)
plot_histogram(counts)
qc.draw()

```