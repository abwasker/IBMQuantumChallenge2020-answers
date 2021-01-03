```python
from IPython.display import Image, display
Image("ryoko.png", width="70")
```




![png](output_0_0.png)



# Message from Dr. Ryoko
"*Hi! I hope you now understand how the Lights Out puzzle works.<br/>The floor of the room I must cross has 3 x 3 tiles. Each tile is made of a single qubit.<br/>
Some of these qubits fluctuate between the ground state and the excited state. <br/> I have been observing their behavior and noticed that there is a pattern - the floor can only be in either one of the four patterns as shown in each of the examples below. <br/>
Due to decoherence, there is not enough time to play around with the switches.<br/> You need to find out which board can be cleared with three switch operations. Good luck!*"<br/>

# Week2-B: Four-Lights Out
In this problem, we are dealing with multiple binary data at the same time. 
We have to determine if each of the given four Lights Out boards are solvable under the given constraints, so let's devise a quantum circuit to solve them all at the same time.

As an example, let's consider how to find a board that can be cleared with just a single switch operation from the 4 boards given below. The initial state of the 4 boards is given in the following two-dimensional array, where "0" and "1" represent "off" and "on" respectively similar to the previous learning problem:

lightsout4_ex=\[\[Board 0\],\[Board 1\],\[Board 2\],\[Board 3\]\]


```python
from IPython.display import Image, display
Image('4lightsout_ex.png')
```




![png](output_3_0.png)



## Answer Strategy
If only one board is given, this is a decision problem.
Using the algorithm from the first Lights Out puzzle (2A), you can solve this problem by counting the "1"s in the output.
 
If we are given multiple boards, there will be several approaches.
1. Iterate the same "one board algorithm" for each board.
2. Hold information for multiple boards at the same time and solve the problems in a single run (execute the algorithm once). 
- For the rest of this document, we discuss how to use the latter approach to solve this type of problem.

First, how do we keep data for all the boards at the same time?
1. Naive data structures:　　9 Qubits/board * 4 boards > 32 qubits (Upper limit of ibm_qasm_simulator).
2. Prepare the  superposition state:   $\vert Board 0\rangle + \vert Board 1\rangle + \vert Board 2\rangle + \vert Board 3\rangle$.
    - The circuit configuration used for state generation is non-trivial.
3. *qRAM* is known as one solution. 
    - **Pros**: Intuitive implementation. 
    - **Cons**: Computationally expensive. 

Of course you can devise and adopt other smart ways to do this.

Here, we will focus on *qRAM* and describe its configuration and implementation.


```python
from qiskit import *
from qiskit import QuantumRegister, ClassicalRegister, QuantumCircuit
from qiskit import IBMQ, Aer, execute, transpile
provider = IBMQ.load_account()
```

    ibmqfactory.load_account:WARNING:2020-11-25 17:20:33,504: Credentials are already in use. The existing account in the session will be replaced.


## qRAM: Quantum Random Access Memory

In classical computers, RAM (Random Access Memory) is a type of volatile memory that has memory addresses $j$ and stores binary data corresponding to each address $D_j$.

In the case of [qRAM](https://arxiv.org/abs/0708.1879) in a quantum computer, **address qubits $a$** have the $N$-addresses as superposition and the corresponding binary data is stored in **data qubits $d$** as a state vector.
\\[
\sum_{j}\frac{1}{\sqrt{N}}\vert j \rangle_{a}\vert 0 \rangle_{d}\xrightarrow{qRAM}\sum_{j}\frac{1}{\sqrt{N}}\vert j \rangle_{a}\vert D_{j} \rangle_{d}
\\]　　
We call the right-hand side state "qRAM" and the corresponding gate operation "qRAM operation".

Although qRAM operation requires $\mathcal{O}(N\log N)$ gates, it can be used to create superposition states of binary data intuitively.  

qRAM has previously been applied to various quantum machine learning algorithms such as the HHL algorithm. For this problem, let's apply qRAM to Grover's algorithm.

## Example: Find the data from qRAM
Prepare a qRAM of $n$-addresses in which the numbers $k_0, k_1, .. , k_{n-1}$ are stored in this order.  
Find the address in which the number $m$ is stored using Grover's algorithm.  
- $n = 4$
- $k = [1,2,5,7]$
- $m = 7$

### qRAM operation.
Here we show a circuit example of qRAM.


```python
address = QuantumRegister(2)
data = QuantumRegister(3)
c = ClassicalRegister(5)
qc = QuantumCircuit(address,data,c)

# address preparation
qc.h([address[0],address[1]])
qc.barrier()
# address 0 -> data = 1
qc.x([address[0],address[1]])
qc.ccx(address[0],address[1],data[2])
qc.x([address[0],address[1]])
qc.barrier()
# address 1 -> data = 2
qc.x(address[0])
qc.ccx(address[0],address[1],data[1])
qc.x(address[0])
qc.barrier()
# address 2 -> data = 5
qc.x(address[1])
qc.ccx(address[0],address[1],data[2])
qc.ccx(address[0],address[1],data[0])
qc.x(address[1])
qc.barrier()
# address 3 -> data = 7
qc.ccx(address[0],address[1],data[2])
qc.ccx(address[0],address[1],data[1])
qc.ccx(address[0],address[1],data[0])
qc.barrier()


#Check the qRAM　status
qc.measure(address[0:2], c[0:2])
qc.measure(data[0:3], c[2:5])
 
# Reverse the output string.
#qc = qc.reverse_bits()

#backend = provider.get_backend('ibmq_qasm_simulator')
backend = Aer.get_backend('qasm_simulator')
job = execute(qc, backend=backend, shots=8000, seed_simulator=12345, backend_options={"fusion_enable":True})
#job = execute(qc, backend=backend, shots=8192)
result = job.result()
count =result.get_counts()
print(count)

qc.draw(output='mpl')
```

    /opt/conda/lib/python3.7/site-packages/ipykernel_launcher.py:41: DeprecationWarning: Using `backend_options` kwarg has been deprecated as of qiskit-aer 0.7.0 and will be removed no earlier than 3 months from that release date. Runtime backend options should now be added directly using kwargs for each option.


    {'10000': 1989, '10101': 2039, '11111': 1964, '01010': 2008}





![png](output_8_2.png)



### qRAM Data Search
To perform Grover's algorithm, we invert the sign of the **address qubit** containing $m$. We also need to initialize the **data qubit** by another qRAM operation before the Diffusion operation,

\begin{align*}
\vert j \rangle_{a}\vert D_{j} \rangle_{d} \vert - \rangle_{f}
\xrightarrow{oracle}  
\left \{
 \begin{array}{l}
-\vert j \rangle_{a}\vert D_{j} \rangle_{d} \vert - \rangle_{f},  D_{j} = m\\
\vert j \rangle_{a}\vert D_{j} \rangle_{d} \vert - \rangle_{f},  D_{j}  \neq m
 \end{array}
 \right.
 \xrightarrow{qRAM}
\left \{
 \begin{array}{l}
-\vert j \rangle_{a}\vert 0 \rangle_{d}\vert - \rangle_{f},  D_{j} = m \\
\vert j \rangle_{a}\vert 0 \rangle_{d}\vert - \rangle_{f},　D_{j}\neq m
 \end{array}
 \right.
 \end{align*}
 
where $f$ denotes the flag qubit.  

In this case, we can configure an oracle operation using the [C3X gate](https://qiskit.org/documentation/stubs/qiskit.circuit.library.C3XGate.html#qiskit.circuit.library.C3XGate) . 

Here, we show the whole circuit for our [qRAM example](#qRAM-Example:-Find-the-data-from-qRAM).


```python
Image('circuit_ex.png')
```




![png](output_10_0.png)



### Considerations for qRAM implementation
In the above description we have introduced a naive *qRAM operation* circuit.
Depending on the data structure, we can simplify the circuit by using **gate synthesis** (equivalence transformation) techniques.
Also, some simplified gates, e.g. [RCCX](https://qiskit.org/documentation/stubs/qiskit.circuit.library.RCCXGate.html#qiskit.circuit.library.RCCXGate), may help improve your *CNOT*-saving implementation.

An example of gate synthesis is shown below.


```python
Image('gatesynthesis_ex.png')
```




![png](output_12_0.png)



## Learning Exercise II-B
Let's solve a 4-Lights Out problem with qRAM.  

When the initial board state lightsout4=\[\[Board 0\],\[Board 1\],\[Board 2\],\[Board 3\]\] is described by the following data, 
determine the _binary_ number of the solvable boards in $3$ switch operations.  (ex. Board 0 → 00, 1 → 01, 2 → 10, 3 → 11)

Answer by creating a quantum circuit to solve the puzzle shown in the figure below. In the quantum circuit to be submitted, measure only the `solution` (2bit) that solves the puzzle.

To submit your solution, create a function which takes "lightsout4" as an input and returns `QuantumCircuit`.  You can choose a function name you like. Make sure it works even if you input another dataset to "lightsout4".

 **In addition, please implement the quantum circuit within 28 qubits.**

Please note that you can get the answer with the same endian as the one used in the description. You can also use the following function.
```python
qc = qc.reverse_bits()
```


```python
Image('4lightsout_pr.png')
```




![png](output_14_0.png)




```python
lightsout4=[[1, 1, 1, 0, 0, 0, 1, 0, 0],[1, 0, 1, 0, 0, 0, 1, 1, 0],[1, 0, 1, 1, 1, 1, 0, 0, 1],[1, 0, 0, 0, 0, 0, 1, 0, 0]]
```

### Hints
- Change the oracle of [qRAM data search](#qRAM-Data-search) to an appropriate one.
- Data storing/writing in *QRAM operation* can be performed in any order. We can reduce the number of gates by taking into account the _hamming distance_ of the address and input data.


```python
n = len(lightsout4[0])
def week2b_ans_func(lightout4):
    ##### Build your cirucuit here
    ####  In addition, please make sure your function can
    #### solve the problem with different inputs (lightout4).
    #### We will cross validate with different inputs.
    grps = [[0, 1, 3], [0, 1, 2, 4], [1, 2, 5], [0, 3, 4, 6], [1, 3, 4, 5, 7], [2, 4, 5, 8], [3, 6, 7],
            [4, 6, 7, 8], [5, 7, 8]]
    flip = QuantumRegister(n)     # variable qubits (superposition / database)
    address = QuantumRegister(2)  # qRAM addresses and also output sum
    tile = QuantumRegister(n)     # lights input as qubits (data in our qRAM addresses)
    oracle = QuantumRegister(1)   # result from oracle
    cbits = ClassicalRegister(2)  # address location and solution at that address
    ancilla = QuantumRegister(7, name='ancilla') 
    qc = QuantumCircuit(flip, address, tile, ancilla, oracle, cbits)
    # address preparation
    qc.h(address)
    qc.h(flip)
    qc.x(oracle)
    qc.h(oracle)    
    for j in range(4):
        ###########################################################################
        #qRAM
        # address 0 -> board 0
        qc.x([address[0],address[1]])
        for i in range(n):
            if lightsout4[0][i] == 1:
                qc.ccx(address[0],address[1],tile[i]) # addr|00> sets l|00...i.....>
        qc.x([address[0], address[1]])
        # address 1 -> board 1
        qc.x(address[0])
        for i in range(n):
            if lightsout4[1][i] == 1:
                qc.ccx(address[0],address[1], tile[i]) # addr|00> sets l|01....i...>
        qc.x(address[0])
        # address 2 -> board 2
        qc.x(tile[1])
        for i in range(n):
            if lightsout4[2][i] == 1:
                qc.ccx(address[0],address[1], tile[i]) # addr|00> sets l|10....i...>
        qc.x(address[1])
        # address 3 -> board 3
        for i in range(n):
            if lightsout4[3][i] == 1:
                qc.ccx(address[0],address[1],tile[i]) # addr |11> sets l|11....i...>
        #############################################################################        #Compute u2a
        for i, ls in enumerate(grps):
            for j in ls:
                qc.cx(flip[i], tile[j])
        # Flip 'output' bit if all patterns are satisfied
        qc.x(tile)
        qc.mct(tile, oracle, ancilla, mode='basic')
        qc.x(tile)
        #############################################################################
        # count the number of flips.... since 'tiles' are all zeroes we can put
        # the result there for now .....
        for i in range(len(flip)):
            qc.mct([flip[i], tile[0], tile[1], tile[2]], tile[3])
            qc.mct([flip[i], tile[0], tile[1]], tile[2])
            qc.mct([flip[i], tile[0]], tile[1])
            qc.cx(flip[i], tile[0])
        #for i in range(9):
        #    qc.mct([flip[i],ancilla[0],ancilla[1],ancilla[2]], ancilla[3], mode = 'noancilla')
        #    qc.mct([flip[i],ancilla[0],ancilla[1]],ancilla[2], mode = 'noancilla')
        #    qc.ccx(flip[i],ancilla[0],ancilla[1])
        #    qc.cx(flip[i],ancilla[0])        #############################################################################
        # tile[0:3] contains the count ... where anything greater than 3 will have
        # bits in tile[2:4]... so if we want only solutions with less than 3 flips
        # we select those with tile[2:4] all zeroes... we forec that here:
        qc.x(tile[0:2])
        qc.mct(tile[0:2],oracle, ancilla, mode='basic') # so, if x(l) is all ones, we'll have our result
        qc.x(tile[0:2])
        ##############################################################################
        # reverse the counter... after the for loop, tile[0:3] is all ones
        for i in range(len(flip)):
                    qc.cx(flip[i], tile[0])
                    qc.mct([flip[i], tile[0]], tile[1])
                    qc.mct([flip[i], tile[0], tile[1]], tile[2])
                    qc.mct([flip[i], tile[0], tile[1], tile[2]], tile[3])
        # all ones... flip back to zeroes...
        #for i in range(9):
        #    qc.cx(flip[i],ancilla[0])
        #    qc.ccx(flip[i],ancilla[0],ancilla[1])
        #    qc.mct([flip[i],ancilla[0],ancilla[1]],ancilla[2], mode = 'noancilla')
        #    qc.mct([flip[i],ancilla[0],ancilla[1],ancilla[2]], ancilla[3], mode = 'noancilla')
        qc.x(tile[0:4])
        ###############################################################################
        # un-compute u2a
        qc.x(tile)
        qc.mct(tile, oracle, ancilla, mode='basic')
        qc.x(tile)
        # Flip 'output' bit if all pattern are satisfied
        for i, ls in enumerate(grps):
            for j in ls:
                qc.cx(flip[i], tile[j])
        ################################################################################
        # un-compute all the qRAM stuff...
        # address 0 -> board 0
        qc.x([address[0],address[1]])
        for i in range(n):
            if lightsout4[0][i] == 1:
                qc.ccx(address[0],address[1],tile[i]) # addr |00> sets l|00...i...>
        qc.x([address[0],address[1]])
        qc.barrier()
        # address 1 -> board 1
        qc.x(address[0])
        for i in range(n):
            if lightsout4[1][i] == 1:
                qc.ccx(address[0],address[1],tile[i]) # addr |00> sets l|01...i...>
        qc.x(address[0])
        qc.barrier()
        # address 2 -> board 2
        qc.x(tile[1])
        for i in range(n):
            if lightsout4[2][i] == 1:
                qc.ccx(address[0],address[1],tile[i]) # addr |00> sets l|10...i...>
        qc.x(address[1])
        qc.barrier()
        # address 3 -> board 3
        for i in range(n):
            if lightsout4[3][i] == 1:
                qc.ccx(address[0],address[1],tile[i]) # addr |11> sets l|01...i...>
        qc.barrier()       
        qc.h(address)
        qc.x(address)
        qc.h(address[1])
        qc.mct(address[0:1], address[1], ancilla, mode='basic')
        qc.h(address[1])
        qc.x(address)
        qc.h(address)
    qc.measure(address,cbits)
    qc = qc.reverse_bits()
    qc = transpile(qc, optimization_level = 3)
    return qc
```


```python
%%time
backend = provider.get_backend('ibmq_qasm_simulator')
cir = week2b_ans_func(lightsout4)
job = execute(cir, backend=backend, shots=1024, seed_simulator=12345, backend_options={"fusion_enable":True})
result = job.result()
count = result.get_counts()
print(count)
```

    {'00': 247, '01': 263, '10': 262, '11': 252}
    CPU times: user 21.6 s, sys: 942 ms, total: 22.6 s
    Wall time: 6min 55s



```python
# Submission code
from qc_grader import prepare_ex2b, grade_ex2b, submit_ex2b

# Execute your circuit with following prepare_ex2b() function.
# The prepare_ex2b() function works like the execute() function with only QuantumCircuit as an argument.
job  =  prepare_ex2b(week2b_ans_func)

result = job.result()
count = result.get_counts()
original_problem_set_counts = count[0]

original_problem_set_counts
# The bit string with the highest number of observations is treated as the solution.
```

    Running week2b_ans_func...
    Computing cost...
    Starting experiments. Please wait...
    You may monitor the job (id: 5fbe9e173645ce001367477c) status and proceed to grading when it successfully completes.





    {'00': 1951, '01': 2058, '10': 2045, '11': 1946}




```python
# Check your answer by executing following code.
# The quantum cost of the QuantumCircuit is obtained as the score. The quantum cost is related to rank only in the third week.
grade_ex2b(job)
```

    Grading your answer. Please wait...
    
    Congratulations 🎉! Your answer is correct.
    Your score is 72399.
    Feel free to submit your answer.



```python
# Submit your results by executing following code. You can submit as many times as you like during the period. 
submit_ex2b(job)
```

    Submitting your answer. Please wait...
    
    Success 🎉! Your answer has been submitted.
    There seems to be huge "noise clusters" interfering with Dr. Ryoko’s device. Can you please help?



```python

```
