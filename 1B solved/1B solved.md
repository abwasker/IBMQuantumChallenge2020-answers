```python
from IPython.display import Image, display
Image("ryoko.png", width="70")
```

# Message from Dr. Ryoko
"*Hi! Here's another exercise you can work on this week. <br/>
It focuses on a famous quantum algorithm known as Grover's algorithm. <br/>
This should give you the basic knowledge necessary to tackle more advanced problems in the coming weeks. Good luck!*"

# Week1-B: Grover's Algorithm

Let us learn about a well-known quantum algorithm called Grover's algorithm. <br/> Also, **there is a learning exercise for you to tackle on your own at the end of this tutorial.** <br/>

You may have heard that one of the advantages a quantum computer has over a classical computer is its superior speed searching databases.<br/>
Grover's algorithm demonstrates this capability. This algorithm can speed up an unstructured search problem quadratically (a classical computation requires on the order of $N$ steps to search $N$ entries problem, while a quantum computer requires just $\sqrt{N}$), but its uses extend beyond that; it can serve as a general trick or subroutine to obtain quadratic run time improvements for a variety of other algorithms. This is called the amplitude amplification trick.

This page will walk you through the description of the search problem, building the oracle - the circuit representation of our search problem, and implementing the complete Grover algorithm in Qiskit.

# Unstructured search
Suppose you are given a large list of $N$ items. Among these items there is one item with a unique property that we wish to locate; we will call this one the winner, ${w}$. Think of each item in the list as a box of a particular item. Say all items in the list have grey colored items except the winning item.


```python
from IPython.display import Image, display
Image('unstructured_search.png', width="700")
```




![png](output_5_0.png)



To find the winning item -- the marked item -- using classical computation, one would have to check on average $N/2$ of these boxes, and in the worst case, all $N$ of them. On a quantum computer, however, we can find the marked item in roughly $\sqrt{N}$ steps with Grover's amplitude amplification trick. A quadratic speedup is indeed a substantial time-saver for finding marked items in long lists. Additionally, the algorithm does not use the list's internal structure, which makes it generic; this is why it immediately provides a quadratic quantum speed-up for many classical problems.

# Creating an oracle that marks the winning item
How will the list items be provided to the quantum computer? A common way to encode such a list is in terms of a function $f$ that returns $f(x)=0$ for all unmarked items $x$, and $f(w)=1$ for the winner. To use a quantum computer for this problem, we must provide the items in superposition to this function, so we encode the function into a unitary matrix called an **oracle**. First, we choose a binary encoding of the items $x,w \in \{0,1\}^n$ so that $N=2^n$. This way, we can represent it in terms of qubits on a quantum computer. We then define the oracle matrix $U_w$ to act on any of the simple, standard basis states $|x\rangle$ by $U_w |x\rangle = (-1)^{f(x)}|x\rangle$

We see that if $x$ is an unmarked item, the oracle does nothing to the state. However, when we apply the oracle to the basis state $|w\rangle$, it maps $U_w |w\rangle = -|w\rangle$. Geometrically, this unitary matrix corresponds to a reflection about the origin for the marked item in an $N=2^n$-dimensional vector space.


```python
Image("oracle.png", width="700")
```




![png](output_8_0.png)



# Amplitude amplification
So how does the algorithm work? Before looking at the list of items, we have no idea where the marked item is. Therefore, any guess of its location is as good as any other. You may have heard the term superposition. This can be expressed in terms of a uniform superposition: <br/>
$$|s\rangle = \frac{1}{\sqrt N}\sum_{x=0}^{N-1} |x\rangle $$

If at this point we were to measure in the standard basis $|x\rangle$, this superposition would collapse to any one of the basis states with the same probability of $\frac{1}{N} = \frac{1}{2^{n}} $. Our chances of guessing the right value $|w\rangle$ is therefore $\frac{1}{2^{n}}$, as could be expected. Hence, on average we would need to try about $N=2^{n}$ times to guess the correct item.

Now, let's enter the procedure called amplitude amplification, which is how a quantum computer significantly enhances the probability of finding the correct item. This procedure stretches out (amplifies) the amplitude of the marked item, which shrinks the other items' amplitudes, so that measuring the final state will return the right item with near-certainty.

This algorithm has a nice geometrical interpretation in terms of two reflections, which generate a rotation in a two-dimensional plane. The only two special states we need to consider are the winner $|w\rangle$ and the uniform superposition $|s\rangle$ . These two vectors span a two-dimensional plane in the vector space $\mathbb C^{N}$ . They are not quite perpendicular because $|w\rangle$ occurs in the superposition with amplitude $N^{-1/2}$ as well.

We can, however, introduce an additional state $|s'\rangle$ that is in the span of these two vectors, is perpendicular to $|w\rangle$, and is obtained from $|s\rangle$ by removing $|w\rangle$ and rescaling.


**Step 0** :
The amplitude amplification procedure starts out in the uniform superposition $|s\rangle$ .  The uniform superposition is easily constructed from $|s\rangle = H^{\otimes n}|0\rangle^{n}$. At $t=0$, the initial state is $|\psi_{0}\rangle = |s\rangle$.


```python
Image("step0.png", width="700")
```




![png](output_12_0.png)



**Step 1** :
We apply the oracle reflection $U_{w}$ to the state $U_{w}|\psi_{t}\rangle = |\psi_{t'}\rangle$.


```python
Image("step1.png", width="700")
```




![png](output_14_0.png)



Geometrically, this corresponds to a reflection of the state $|\psi_{t}\rangle$ about  $|s'\rangle$ . This transformation means that the amplitude in front of the $|w\rangle$ state becomes negative, which in turn means that the average amplitude has been lowered. (Note how the dotted line in the right graph is decreasing).

**Step 2**:
We now apply an additional reflection $U_{s}$ about the state $|s\rangle$: $U_{s} = 2|s\rangle \langle s| - 1 $ . This transformation maps the state to $U_{s}|\psi_{t'}\rangle$ and completes the transformation $|\psi_{t+1}\rangle = U_{s}U_{w}|\psi_{t}\rangle$ .（Note how the amplitude at $|w\rangle$ is amplified in the right graph).


```python
Image("step2.png", width="700")
```




![png](output_16_0.png)



Two reflections always correspond to a rotation. The transformation $ U_{s}U_{w}$ rotates the initial state $|s\rangle$ closer toward the winner $|w\rangle$ . Notice the left graph in Step 2. The action of the reflection $U_{s}$ in the amplitude bar diagram can be understood as a reflection about the average amplitude. Since the average amplitude has been lowered by the first reflection, this transformation boosts the negative amplitude of $|w\rangle$ to roughly three times its original value, while it decreases the other amplitudes. We then go to **Step １** to repeat the application. This procedure will be repeated several times to zero in on the winner. 

After $t$ steps, the state will have transformed to $|\psi_{t}\rangle = (U_{s}U_{w})^{t}|\psi_{0}\rangle$.

How many times do we need to apply the rotation? It turns out that roughly $\sqrt N$ rotations suffice. This becomes clear when looking at the amplitudes of the state $|\psi_{t}\rangle$ . We can see that the amplitude of $|w\rangle$ grows linearly with the number of applications（$ \sim tN^{1/2}$）. However, since we are dealing with amplitudes and not probabilities, the vector space's dimension enters as a square root. Therefore it is the amplitude, and not just the probability, that is being amplified in this procedure.

If there are multiple solutions, $M$, it can be shown that roughly $\sqrt{(N/M)}$ rotations will suffice.


```python
Image("grover_algorithm.png", width="700")
```




![png](output_19_0.png)



### Qiskit implementation: Grover's algorithm using two qubits
Now, let's impement Grover's algorithm using Qiskit. In this example, we will use two qubits to find the state $|11\rangle$.

First we prepare our environment.


```python
#initialization
import matplotlib.pyplot as plt
%matplotlib inline
import numpy as np

# importing Qiskit
from qiskit import IBMQ, BasicAer
from qiskit.providers.ibmq import least_busy
from qiskit import QuantumCircuit, ClassicalRegister, QuantumRegister, execute

# import basic plot tools
from qiskit.tools.visualization import plot_histogram
```

As we have seen in Step1, let us create a phase oracle to mark the state $|11\rangle$.


```python
def phase_oracle(circuit, register):
    circuit.cz(register[0], register[1])

qr = QuantumRegister(2)
oracleCircuit = QuantumCircuit(qr)
phase_oracle(oracleCircuit, qr)
oracleCircuit.draw(output="mpl")
```




![png](output_23_0.png)



Next, we set up the circuit for inversion about the average as we saw in Step 2. This circuit is sometimes called an amplitude ampification module or a diffusion circuit.


```python
def inversion_about_average(circuit, register):
    """Apply inversion about the average step of Grover's algorithm."""
    circuit.h(register)
    circuit.x(register)
    circuit.h(register[1])
    circuit.cx(register[0], register[1])
    circuit.h(register[1])
    circuit.x(register)
    circuit.h(register)
```


```python
qAverage = QuantumCircuit(qr)
inversion_about_average(qAverage, qr)
qAverage.draw(output='mpl')
```




![png](output_26_0.png)



Now we put the pieces together, with the creation of a uniform superposition at the start of the circuit and a measurement at the end. Note that since there is one solution and four possibilities, we will only need to run one iteration.


```python
qr = QuantumRegister(2)
cr = ClassicalRegister(2)

groverCircuit = QuantumCircuit(qr,cr)
groverCircuit.h(qr)

phase_oracle(groverCircuit, qr)
inversion_about_average(groverCircuit, qr)

groverCircuit.measure(qr,cr)
groverCircuit.draw(output="mpl")
```




![png](output_28_0.png)



### Experiment with simulators
We now run the above circuit on the simulator.


```python
backend = BasicAer.get_backend('qasm_simulator')
shots = 1024
results = execute(groverCircuit, backend=backend, shots=shots).result()
answer = results.get_counts()
plot_histogram(answer)
```




![png](output_30_0.png)



As we can see, the algorithm discovers our marked states.

### Experiment with real devices
We can run the circuit on the real device as shown below.


```python
# Load our saved IBMQ accounts and get the least busy backend device
from qiskit import IBMQ
IBMQ.load_account()
provider = IBMQ.get_provider(hub='ibm-q')
backend_lb = least_busy(provider.backends(filters=lambda b: b.configuration().n_qubits >= 3 and
                                   not b.configuration().simulator and b.status().operational==True))
print("Least busy backend: ", backend_lb)
```

    ibmqfactory.load_account:WARNING:2020-11-16 11:38:33,332: Credentials are already in use. The existing account in the session will be replaced.


    Least busy backend:  ibmq_athens



```python
# Run our circuit on the least busy backend. Monitor the execution of the job in the queue
from qiskit.tools.monitor import job_monitor

backend = backend_lb
shots = 1024
job_exp = execute(groverCircuit, backend=backend, shots=shots)

job_monitor(job_exp, interval = 2)
```

    Job Status: job is queued (23)    


    ---------------------------------------------------------------------------

    KeyboardInterrupt                         Traceback (most recent call last)

    <ipython-input-11-37043d7fec73> in <module>
          6 job_exp = execute(groverCircuit, backend=backend, shots=shots)
          7 
    ----> 8 job_monitor(job_exp, interval = 2)
    

    /usr/local/lib/python3.8/site-packages/qiskit/tools/monitor/job_monitor.py in job_monitor(job, interval, quiet, output)
         83         _interval_set = True
         84 
    ---> 85     _text_checker(job, interval, _interval_set,
         86                   quiet=quiet, output=output)


    /usr/local/lib/python3.8/site-packages/qiskit/tools/monitor/job_monitor.py in _text_checker(job, interval, _interval_set, quiet, output)
         41     while status.name not in ['DONE', 'CANCELLED', 'ERROR']:
         42         time.sleep(interval)
    ---> 43         status = job.status()
         44         msg = status.value
         45 


    /usr/local/lib/python3.8/site-packages/qiskit/providers/ibmq/job/ibmqjob.py in status(self)
        480 
        481         with api_to_job_error():
    --> 482             api_response = self._api.job_status(self.job_id())
        483             self._status, self._queue_info = self._get_status_position(
        484                 api_response['status'], api_response.get('info_queue', None))


    /usr/local/lib/python3.8/site-packages/qiskit/providers/ibmq/api/clients/account.py in job_status(self, job_id)
        300             ApiIBMQProtocolError: If unexpected data is received from the server.
        301         """
    --> 302         return self.client_api.job(job_id).status()
        303 
        304     def job_final_status(


    /usr/local/lib/python3.8/site-packages/qiskit/providers/ibmq/api/rest/job.py in status(self)
        157         """
        158         url = self.get_url('status')
    --> 159         raw_response = self.session.get(url)
        160         try:
        161             api_response = raw_response.json()


    /usr/local/lib/python3.8/site-packages/requests/sessions.py in get(self, url, **kwargs)
        541 
        542         kwargs.setdefault('allow_redirects', True)
    --> 543         return self.request('GET', url, **kwargs)
        544 
        545     def options(self, url, **kwargs):


    /usr/local/lib/python3.8/site-packages/qiskit/providers/ibmq/api/session.py in request(self, method, url, bare, **kwargs)
        247         try:
        248             self._log_request_info(url, method, kwargs)
    --> 249             response = super().request(method, final_url, **kwargs)
        250             response.raise_for_status()
        251         except RequestException as ex:


    /usr/local/lib/python3.8/site-packages/requests/sessions.py in request(self, method, url, params, data, headers, cookies, files, auth, timeout, allow_redirects, proxies, hooks, stream, verify, cert, json)
        528         }
        529         send_kwargs.update(settings)
    --> 530         resp = self.send(prep, **send_kwargs)
        531 
        532         return resp


    /usr/local/lib/python3.8/site-packages/requests/sessions.py in send(self, request, **kwargs)
        641 
        642         # Send the request
    --> 643         r = adapter.send(request, **kwargs)
        644 
        645         # Total elapsed time of the request (approximately)


    /usr/local/lib/python3.8/site-packages/requests/adapters.py in send(self, request, stream, timeout, verify, cert, proxies)
        437         try:
        438             if not chunked:
    --> 439                 resp = conn.urlopen(
        440                     method=request.method,
        441                     url=url,


    /usr/local/lib/python3.8/site-packages/urllib3/connectionpool.py in urlopen(self, method, url, body, headers, retries, redirect, assert_same_host, timeout, pool_timeout, release_conn, chunked, body_pos, **response_kw)
        668 
        669             # Make the request on the httplib connection object.
    --> 670             httplib_response = self._make_request(
        671                 conn,
        672                 method,


    /usr/local/lib/python3.8/site-packages/urllib3/connectionpool.py in _make_request(self, conn, method, url, timeout, chunked, **httplib_request_kw)
        424                     # Python 3 (including for exceptions like SystemExit).
        425                     # Otherwise it looks like a bug in the code.
    --> 426                     six.raise_from(e, None)
        427         except (SocketTimeout, BaseSSLError, SocketError) as e:
        428             self._raise_timeout(err=e, url=url, timeout_value=read_timeout)


    /usr/local/lib/python3.8/site-packages/urllib3/packages/six.py in raise_from(value, from_value)


    /usr/local/lib/python3.8/site-packages/urllib3/connectionpool.py in _make_request(self, conn, method, url, timeout, chunked, **httplib_request_kw)
        419                 # Python 3
        420                 try:
    --> 421                     httplib_response = conn.getresponse()
        422                 except BaseException as e:
        423                     # Remove the TypeError from the exception chain in


    /usr/local/Cellar/python@3.8/3.8.5/Frameworks/Python.framework/Versions/3.8/lib/python3.8/http/client.py in getresponse(self)
       1345         try:
       1346             try:
    -> 1347                 response.begin()
       1348             except ConnectionError:
       1349                 self.close()


    /usr/local/Cellar/python@3.8/3.8.5/Frameworks/Python.framework/Versions/3.8/lib/python3.8/http/client.py in begin(self)
        305         # read until we get a non-100 response
        306         while True:
    --> 307             version, status, reason = self._read_status()
        308             if status != CONTINUE:
        309                 break


    /usr/local/Cellar/python@3.8/3.8.5/Frameworks/Python.framework/Versions/3.8/lib/python3.8/http/client.py in _read_status(self)
        266 
        267     def _read_status(self):
    --> 268         line = str(self.fp.readline(_MAXLINE + 1), "iso-8859-1")
        269         if len(line) > _MAXLINE:
        270             raise LineTooLong("status line")


    /usr/local/Cellar/python@3.8/3.8.5/Frameworks/Python.framework/Versions/3.8/lib/python3.8/socket.py in readinto(self, b)
        667         while True:
        668             try:
    --> 669                 return self._sock.recv_into(b)
        670             except timeout:
        671                 self._timeout_occurred = True


    /usr/local/Cellar/python@3.8/3.8.5/Frameworks/Python.framework/Versions/3.8/lib/python3.8/ssl.py in recv_into(self, buffer, nbytes, flags)
       1239                   "non-zero flags not allowed in calls to recv_into() on %s" %
       1240                   self.__class__)
    -> 1241             return self.read(nbytes, buffer)
       1242         else:
       1243             return super().recv_into(buffer, nbytes, flags)


    /usr/local/Cellar/python@3.8/3.8.5/Frameworks/Python.framework/Versions/3.8/lib/python3.8/ssl.py in read(self, len, buffer)
       1097         try:
       1098             if buffer is not None:
    -> 1099                 return self._sslobj.read(len, buffer)
       1100             else:
       1101                 return self._sslobj.read(len)


    KeyboardInterrupt: 



```python
# get the results from the computation
results = job_exp.result()
answer = results.get_counts(groverCircuit)
plot_histogram(answer)
```

### Qiskit implementation: 2 qubit Grover's algorithm using ancilla bits
We are going to find the state $|11\rangle$ just like in the previous example, but this time we will use an ancilla bit. Ancilla bbits let you work with more qubits or implement more complex oracles.

Let us prepare the environment first.


```python
# Initialization
import matplotlib.pyplot as plt
%matplotlib inline
import numpy as np

# Importing Qiskit
from qiskit import IBMQ, BasicAer
from qiskit.providers.ibmq import least_busy
from qiskit import QuantumCircuit, ClassicalRegister, QuantumRegister, execute

# Import basic plot tools
from qiskit.tools.visualization import plot_histogram
```

We will create an oracle that will flip the phase of the answer we are looking for ( in this case $|11\rangle$ ). This time, using the ancilla bit to make the target bit's phase flip when the input state is $|11\rangle$ . Note that in order to make this phase flip work, you need to prepare the ancilla bit to be $|1\rangle$ by using an x gate.


```python
def phase_oracle(circuit, register,oracle_register):
    circuit.h(oracle_register)
    circuit.ccx(register[0], register[1],oracle_register)
    circuit.h(oracle_register)
    
qr = QuantumRegister(3)
oracleCircuit = QuantumCircuit(qr)
oracleCircuit.x(qr[2])
phase_oracle(oracleCircuit, qr,qr[2])
oracleCircuit.draw(output="mpl")
```




![png](output_39_0.png)



Next, we prepare the amplitude amplification module/diffusion circuit. Make sure that the circuit does not act on the ancilla bit.


```python
def inversion_about_average(circuit, register):
    """Apply inversion about the average step of Grover's algorithm."""
    circuit.h(register)
    circuit.x(register)
    circuit.h(register[1])
    circuit.cx(register[0], register[1])
    circuit.h(register[1])
    circuit.x(register)
    circuit.h(register)
```


```python
qAverage = QuantumCircuit(qr)
inversion_about_average(qAverage, qr[0:2])
qAverage.draw(output='mpl')
```




![png](output_42_0.png)



Just like we did in the previous example without using the ancilla bit, we first create a uniform superposition by using the Hadamard (H gate) , incorporate the transformation and then take measurement. Again, make sure that you do not apply the H gate to your ancilla bit.


```python
qr = QuantumRegister(3)
cr = ClassicalRegister(3)

groverCircuit = QuantumCircuit(qr,cr)
groverCircuit.h(qr[0:2])
groverCircuit.x(qr[2])

phase_oracle(groverCircuit, qr,qr[2])
inversion_about_average(groverCircuit, qr[0:2])

groverCircuit.measure(qr,cr)
groverCircuit.draw(output="mpl")
```




![png](output_44_0.png)



### Experiment with simulators
We now run this circuit on a simulator. Make sure to specify 'qasm_simulator' as your backend. 


```python
backend = BasicAer.get_backend('qasm_simulator')
shots = 1024
results = execute(groverCircuit, backend=backend, shots=shots).result()
answer = results.get_counts()
plot_histogram(answer)
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-3-361c694b852b> in <module>
          1 backend = BasicAer.get_backend('qasm_simulator')
          2 shots = 1024
    ----> 3 results = execute(groverCircuit, backend=backend, shots=shots).result()
          4 answer = results.get_counts()
          5 plot_histogram(answer)


    NameError: name 'groverCircuit' is not defined


We can see how the state $|11\rangle$ is being amplified just like we saw previously without using the ancilla bit. You can ignore the 1 in the highest order, as that comes from the ancilla bit. 

## Tips: Number of iterations
I mentioned that the number of Grover algorithm iterations to be performed before the solution is fully amplified is approximately $\sqrt{N}$. Let's go further and think about the number of times the solution is amplified the most.

For example, when running Grover's algorithm on a database with $N = 2^4$, the probabilities obtained by changing the number of iterations are as follows.


```python
backend = BasicAer.get_backend('qasm_simulator')
prob_of_ans = []

iterations= 15
for x in range(iterations):
    database = QuantumRegister(7)
    oracle = QuantumRegister(1)
    ancilla = QuantumRegister(5) 
    cr = ClassicalRegister(7)
    qc = QuantumCircuit(database, oracle, ancilla, cr)
    qc.h(database[:])
    qc.x(oracle[0])
    qc.h(oracle[0])


    for j in range(x):
    # oracle_4q
        # search 7: 0111 
        qc.x(database[0])
        qc.mct(database[:], oracle[0], ancilla[:], mode='basic') 
        qc.x(database[0])

    # diffusion_4q
        qc.h(database[:])
        qc.x(database[:])
        qc.h(database[6])
        qc.mct(database[0:6], database[6], ancilla[:], mode='basic')
        qc.h(database[6])
        qc.x(database[:])
        qc.h(database[:])


    qc.h(oracle[0])
    qc.x(oracle[0])
    qc.measure(database,cr)
    # Change the endian 
    qc = qc.reverse_bits() 
    
    job = execute(qc, backend=backend, shots=1024, seed_simulator=12345, backend_options={"fusion_enable":True})
    result = job.result()
    count = result.get_counts()
    answer = count['0111111']
    prob_of_ans.append(answer)
    qc.draw(output='mpl')
```


```python
# import numpy as np
import matplotlib.pyplot as plt
iteration = [i for i in range(iterations)]
correct = prob_of_ans
plt.bar(iteration, correct)
plt.xlabel('# of iteration')
plt.ylabel('# of times the solution was obtained')
```




    Text(0, 0.5, '# of times the solution was obtained')




![png](output_50_1.png)



```python
qc.draw(output='mpl')
```




![png](output_51_0.png)



# Learning Exercise I-B
Find the number of iterations with the largest amplitude when you run the Grover's algorithm with one solution in a database with $N = 2 ^ 7$. As shown above, change the number of iterations and check the amplification. The answer must be an integer.

Hint: Fewer than 15 times. 


```python
# Change ans of following code and check your answer.
# ans must be an interger.

from qc_grader import grade_ex1b
grade_ex1b(8)
```

    Grading your answer. Please wait...
    
    Congratulations 🎉! Your answer is correct.
    Feel free to submit your answer.



```python
# Change ans of following code and submit it.
# ans must be interger.

from qc_grader import submit_ex1b
submit_ex1b(8)
```

    Submitting your answer. Please wait...
    
    Success 🎉! Your answer has been submitted.
    Have you ever wonder about the quantum realm? Ask Dr. Ryoko about it.



```python
backend = BasicAer.get_backend('qasm_simulator')
prob_of_ans = []

iterations=12
for x in range(iterations):
    database = QuantumRegister(7)
    oracle = QuantumRegister(2)
    ancilla = QuantumRegister(5) 
    cr = ClassicalRegister(7)
    qc = QuantumCircuit(database, oracle, ancilla, cr)
    qc.h(database[:])
    qc.x(oracle[0])
    qc.h(oracle[0])


    for j in range(x):
    # oracle_4q
        # search 7: 0111 
        qc.x(database[0])
        qc.mct(database[:], oracle[0], ancilla[:], mode='basic') 
        qc.x(database[0])

    # diffusion_4q
        qc.h(database[:])
        qc.x(database[:])
        qc.h(database[6])
        qc.mct(database[0:6], database[6], ancilla[:], mode='basic')
        qc.h(database[6])
        qc.x(database[:])
        qc.h(database[:])


    qc.h(oracle[0])
    qc.x(oracle[0])
    qc.measure(database,cr)
    qc = qc.reverse_bits() 
    
    job = execute(qc, backend=backend, shots=1000, seed_simulator=12345, backend_options={"fusion_enable":True})
    result = job.result()
    count = result.get_counts()
    answer = count['0111111']
    prob_of_ans.append(answer)
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-1-fa70875094c9> in <module>
    ----> 1 backend = BasicAer.get_backend('qasm_simulator')
          2 prob_of_ans = []
          3 
          4 iterations=12
          5 for x in range(iterations):


    NameError: name 'BasicAer' is not defined



```python
# import numpy as np
import matplotlib.pyplot as plt
iteration = [i for i in range(iterations)]
correct = prob_of_ans
plt.bar(iteration, correct)
plt.xlabel('# of iteration')
plt.ylabel('# of times the solution was obtained')
```




    Text(0, 0.5, '# of times the solution was obtained')




![png](output_56_1.png)



```python

```


```python

```


```python

```


```python

```
