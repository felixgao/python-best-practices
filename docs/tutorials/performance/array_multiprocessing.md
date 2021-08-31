!!! Summary

    :white_check_mark: Partition data into independent batches/chunks.
    
    :white_check_mark: Use shared memory for large data.
 
    :x: Avoid naive element-wise processing.

    :x: Avoid Using multithreading for CPU-bound tasks.


# Array Multi-Processing

Due to the infamous python GIL, when you need more CPU power to crunch some data, multi-processing is the way to go.   Python has a built-in multiprocessing library that make this feature avaiable out of the box. 

## Multiprocessing library

Let's take a look of the basic example of using multiprocessing in python.

```python
import multiprocessing
def cpu_bound_processing(data):
    pass

process = multiprocessing.Process(target=cpu_bound_processing, args=(x, y, z))
process.start()
process.join() # any process finishes but not been joined becomes zombie process.
```
with this example you can now utitlize more CPU cores to do CPU bound tasks.


### Shared Memory
when the input size is large, it is in-efficient to make copies of the object to be passed to each sub-process.  In this case, we should create one shared memory block of the original large object. 

Python have some capabilities build-in for this.  You can use [shared_memory] module to do that.

> As mentioned above, when doing concurrent programming it is usually best to avoid using shared state as far as possible. This is particularly true when using multiple processes.
>
> However, if you really do need to use some shared data then multiprocessing provides a couple of ways of doing so.


### Numpy multiprocessing

Often we uses [NumPy] arrays to represent image or video data.  This is a great candidate for multiprocessing.  Everything can be done with the build-in [shared_memory] module above, but the [SharedArray] library it is a wrapper around `shm_open`, `mmap` and friends function in C with python binding to makes this more user friendly when working with numpy. 

SharedArray has a few key functions:

- `SharedArray.create(name, shape, dtype=float)` creates a shared memory array
- `SharedArray.attach(name)` attaches a previously created shared memory array to a variable
- `SharedArray.delete(name)` deletes a shared memory array, however, existing attachments remain valid


### Example

With SharedArray. 

```python

import SharedArray

def multiprocess_load_images(num_workers = 12):
    files = os.listdir("train_images")

    # this assumes each file with 1400x1200 resolution with 3 (RBG) channels
    number_of_files = len(files)
    data = SharedArray.create('data', (number_of_files, 1400, 2100, 3))

    worker_amount = int(number_of_files/num_workers)
    residual = number_of_files - (num_works * worker_amount)

    def load_images(i, n):
        # Chucking the potential inputs. this could be optmized
        to_load = files[i: i+n]
        for j, file in enumerate(to_load):
            data[i + j] = cv2.imread("train_images/" + file)
        
    processes = []
    for worker_num in range(num_workers):
        run_worker_amount = worker_amount
        if worker_num == num_workers - 1 and residual != 0:
            run_worker_amount += residual
        process = multiprocessing.Process(target=load_images, args=(worker_amount*worker_num, run_worker_amount))
        processes.append(process)
        process.start()
    
    for process in processes:
        process.join()
    
    return data

```

An example with SharedMemory directly

```python
from multiprocessing.shared_memory import SharedMemory
from multiprocessing.managers import SharedMemoryManager
from concurrent.futures import ProcessPoolExecutor, as_completed
from multiprocessing import current_process, cpu_count, Process
import numpy as np

def work_with_shared_memory(shm_name, shape, dtype):
    print(f'With SharedMemory: {current_process()=}')
    # Locate the shared memory by its name
    shm = SharedMemory(shm_name)
    # Create the np.recarray from the buffer of the shared memory
    np_array = np.recarray(shape=shape, dtype=dtype, buf=shm.buf)
    return np.nansum(np_array.val)

# Some way to create that numpy array.
np_array = ...
shape, dtype = np_array.shape, np_array.dtype
with SharedMemoryManager() as smm:
        # Create a shared memory of size np_arry.nbytes
        shm = smm.SharedMemory(np_array.nbytes)
        # Create a np.recarray using the buffer of shm
        shm_np_array = np.recarray(shape=shape, dtype=dtype, buf=shm.buf)
        # Copy the data into the shared memory
        np.copyto(shm_np_array, np_array)
        # Spawn some processes to do some work
        with ProcessPoolExecutor(cpu_count()) as exe:
            fs = [exe.submit(work_with_shared_memory, shm.name, shape, dtype)
                  for _ in range(cpu_count())]
            for _ in as_completed(fs):
                pass
```



[shared_memory]: https://docs.python.org/3/library/multiprocessing.shared_memory.html#module-multiprocessing.shared_memory

[NumPy]: https://numpy.org/
[SharedArray]: https://pypi.org/project/SharedArray/

[ray]: https://docs.ray.io/en/latest/index.html
[dask]: https://docs.dask.org/
