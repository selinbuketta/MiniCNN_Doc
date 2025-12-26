# MiniCNN Repository 

## AI Use Disclosure Statement
ChatGPT by OpenAI was used as a supportive tool to clarify theoretical concepts, assist with code comprehension, and help structure explanations. All implementations, experiments, performance measurements, and conclusions presented in this work were developed and validated independently by the author.

## 1. Big Picture
This MiniCNN repo focused on inference which is designed to behave like a deployment-time inference engine. It is a forwars pass only neural network runtime written in pure C++.

- The network topology is fixed at compile time. 
- The weights are loaded at runtime.
- Only forward propagation exists.

Lenet.raw file is a pretrained snapshot with weights.

## 2. Execution Flow 

Execution starts in actual LeNet inference driver, lenet.cpp.

```bash
$ make
$ ./lenet lenet.raw t10k-images-idx3-ubyte
```

"t10k-images-idx3-ubyte" is the binary MNIST image file. The interpretation of the file is handled by mnist.hpp. mnist.hpp file reads one image file and creates a tensor of shape [1, 1, 28, 28] from bytes. This is the input tensor for CNN.

### lenet.cpp

1. main(...)
2. resolve_path() 
3. MNIST mnist() → enters mnist.hpp

### mnist.hpp

4. MNIST::MNIST(std::string path)
5. MNIST::load()
6. MNIST::is_little_endian()
7. MNIST::load() continues: imgs_ = Tensor(num_imgs, 1, 28 + 2*pad_, 28 + 2*pad_) → calls Tensor constructor in tensor.hpp

### lenet.cpp → mnist.hpp

8. input = mnist.at(0);
9. MNIST::at(size_t idx)
10. MNIST::slice(size_t idx, size_t num)

At this point, input tensor is ready. 

### lenet.cpp → network.hpp

11. NeuralNetwork net = build_lenet(debug);
12. build_lenet(bool debug). Constructs NeuralNetwork net(debug) → NeuralNetwork::NeuralNetwork in network.hpp

LeNet-5 Variant for MNIST Dataset:

        Conv2d(1,6,5,1,2)
        ReLu()
        MaxPool2d(2,2,0)
        Conv2d(6,16,5,1,0)
        ReLu()
        MaxPool2d(2,2,0)
        Flatten()
        Linear(400,120)
        ReLu()
        Linear(120,84)
        ReLu()
        Linear(84,10)
        SoftMax()

13. NeuralNetwork::add(Layer* layer). Stores the layer in layers_ as a std::unique_ptr<Layer>
14. resolve_path(model_path). Load weights from lenet.raw.
15. net.load(model.string()) → into network.hpp

### network.hpp

16. NeuralNetwork::load(std::string file)

At this point, architecture is built and the parameters are loaded.

### lenet.cpp

17. Tensor output = net.predict(input); → into network.hpp
### network.hpp
18. NeuralNetwork::predict(Tensor input)

19. Predict returns. Output is shown.

## Example Output for Task 1
```bash
$ make
g++ -std=c++17 -O2 -Wall lenet.cpp -o lenet
$ ./lenet ../lenet.raw ../t10k-images-idx3-ubyte
Loading first MNIST image from '../t10k-images-idx3-ubyte'...
Loading model from '../lenet.raw'...
MNIST image shape: N=1 C=1 H=28 W=28
LeNet prediction probabilities:
  class 0: 0.00474183
  class 1: 0.00660528
  class 2: 0.0243872
  class 3: 0.0248431
  class 4: 0.00122579
  class 5: 0.00316218
  class 6: 3.97861e-05
  class 7: 0.925183
  class 8: 0.00331346
  class 9: 0.00649874
```
