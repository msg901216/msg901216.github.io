---
layout:     post
title:      "How to Create an LSTM Recurrent Neural Network Using DL4J"
subtitle:   "用deeplearning4j创建lstm生成一定风格的句子"
date:       2018-09-21
author:     "msg"
header-img: "img/posts/01.jpg"
header-mask: 0.3
catalog:    true

tags:
    - java
    - 学习
    - 自然语言处理
    - LSTM
    - deeplearning4j
    - 转载
---

Long short-term memory recurrent neural networks, or LSTM RNNs for short, are neural networks that can memorize and regurgitate sequential data. They’ve become very popular these days, primarly because they can be used to create bots that can generate articles, stories, music, poems, screenplays - you name it! How? Well, its because a lot of things humans do involve sequences.

To make things clearer, let me give you a few examples. An LSTM can be trained on a novel to generate sentences that look very similar to those present in the novel. If you train it with multiple novels written by the same author, the LSTM will start sounding like the author. Similarly, an LSTM trained using a collection of songs that belong to a specific genre of music will be able to generate “songs” that belong to the same genre. I hope you get the idea.

In this tutorial, I’ll show you how to use Deeplearning4J to create an LSTM that can generate sentences that are similar to those written by the prolific 19th century author Emma Leslie. We’ll be using her novel Hayslope Grange as our training data, so, before we begin, I suggest you download the novel as plain text and store it somewhere on your computer.

### Read the Novel

The first thing you need to do is, of course, convert the text of the novel into a string you can use in your program. The easiest way to do so is to use the ***IOUtils.toString()*** method, which is available in the Apache Commons IO library.

```java
String inputData = IOUtils.toString(new FileInputStream("/tmp/temp.txt"), "UTF-8");
```

You are free to use all the text that’s available in the novel. I, however, in order to speed things up, will be using only the first 50000 characters.

```java
inputData = inputData.substring(0, 50000);
```

### Design the LSTM Neural Network

Our neural network shall have an input LSTM layer, a hidden layer, and an output RNN layer. How many neurons should be present in these layers? Well, that depends on how many different characters your network can handle.

For now, let’s say our network can handle all the letters of the English alphabet, in both uppercase and lowercase, all the digits from 0-9, and a few special characters.

The following string has all the characters I will be using:

```java
String validCharacters = 
        "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz1234567890\"\n',.?;()[]{}:!- ";
```

Our input and output layers must have one neuron for each character that’s present in the above string. As for the hidden layer, I’ll use 30 neurons. You are free to change that number.

To create an input LSTM layer with DL4J, you must use the GravesLSTM class. Similarly, to create an output RNN layer, you must use the RnnOutputLayer class. While creating these layers, you must remember to specify the activation functions they should use. For best results, using TANH for the input layer and SOFTMAX for the output layer is recommended.

Accordingly, add the following code:

```java
GravesLSTM.Builder lstmBuilder = new GravesLSTM.Builder();
lstmBuilder.activation(Activation.TANH);
lstmBuilder.nIn(validCharacters.length());
lstmBuilder.nOut(30); // Hidden
GravesLSTM inputLayer = lstmBuilder.build();

RnnOutputLayer.Builder outputBuilder = new RnnOutputLayer.Builder();
outputBuilder.lossFunction(LossFunctions.LossFunction.MSE);
outputBuilder.activation(Activation.SOFTMAX);
outputBuilder.nIn(30); // Hidden
outputBuilder.nOut(validCharacters.length());
RnnOutputLayer outputLayer = outputBuilder.build();
```

Well, now that our neuron layers are ready, we can use them to create a MultiLayerNetwork object. You must, however, also create a NeuralNetConfiguration.Builder in order to configure the network. As always, during the configuration, you must specify details such as which optimization algorithm and updater to use, how to initialize the weights, and what the learning rate should be.

Here’s the configuration I used:

```java
NeuralNetConfiguration.Builder nnBuilder = new NeuralNetConfiguration.Builder();
nnBuilder.optimizationAlgo(OptimizationAlgorithm.STOCHASTIC_GRADIENT_DESCENT);
nnBuilder.updater(Updater.ADAM);
nnBuilder.weightInit(WeightInit.XAVIER);
nnBuilder.learningRate(0.01);
nnBuilder.miniBatch(true);

MultiLayerNetwork network = new MultiLayerNetwork(
            nnBuilder.list().layer(0, inputLayer)
                    .layer(1, outputLayer)
                    .backprop(true).pretrain(false)
                    .build());

network.init();
```

### Create Training Data

As you might already know, DL4J expects you to place all your training data inside INDArray objects. That means, we must now create two INDArray objects: one for the input values and one for the expected output values, or labels.

```java
INDArray inputArray = Nd4j.zeros(1, inputLayer.getNIn(), inputData.length());
INDArray inputLabels = Nd4j.zeros(1, outputLayer.getNOut(), inputData.length());
```

While using an LSTM, the label for an input value is nothing but the next input value. For example, if your training data is the string “abc”, for the input value “a”, the label will be “b”. Similarly, for the input value “b”, the label will be “c”.

Keeping that in mind, you can populate the INDArray objects using the following code:

```java
for(int i=0;i<inputData.length() - 1;i++) {
    int positionInValidCharacters1 = validCharacters.indexOf(inputData.charAt(i));
    inputArray.putScalar(new int[]{0, positionInValidCharacters1, i}, 1);

    int positionInValidCharacters2 = validCharacters.indexOf(inputData.charAt(i+1));
    inputLabels.putScalar(new int[]{0, positionInValidCharacters2, i}, 1);
}
```

Using the INDArray objects, you can now create a DataSet that can be directly used by your neural network.

```java
DataSet dataSet = new DataSet(inputArray, inputLabels);
```

### Training the Neural Network

At this point, all you need to do is call the fit() method and pass the data set to the neural network. However, trying to fit the data set just once is usually not enough. You must fit it several times before the neural network becomes accurate. I suggest you do it a thousand times at least.

Obviously, fitting the data a thousand times is going to take quite a long time. To be able to see the intermediate results the network generates for every iteration, you can pass test data to the network after each call to the fit() method.

The test data, of course, will be another INDArray object whose size is equal to the size of the network’s input layer. It will contain the index of just one character, which will also be the first character of the network’s generated text.

Using that character, our LSTM will generate a new character. By passing that generated character back to the LSTM as the next test data, you can generate another character. By repeating these steps again and again, you can generate strings that are arbitrarily long.

The following code shows you how to generate a string that’s 200 characters long for every iteration:

```java
for(int z=0;z<1000;z++) {
    network.fit(dataSet);

    INDArray testInputArray = Nd4j.zeros(inputLayer.getNIn());
    testInputArray.putScalar(0, 1);

    network.rnnClearPreviousState();
    String output = "";
    for (int k = 0; k < 200; k++) {
        INDArray outputArray = network.rnnTimeStep(testInputArray);
        double maxPrediction = Double.MIN_VALUE;
        int maxPredictionIndex = -1;
        for (int i = 0; i < validCharacters.length(); i++) {
            if (maxPrediction < outputArray.getDouble(i)) {
                maxPrediction = outputArray.getDouble(i);
                maxPredictionIndex = i;
            }
        }
        // Concatenate generated character
        output += validCharacters.charAt(maxPredictionIndex);
        testInputArray = Nd4j.zeros(inputLayer.getNIn());
        testInputArray.putScalar(maxPredictionIndex, 1);
    }
    System.out.println(z + " > A" + output + "\n----------\n");
}
```

As you can see in the above code, we are finding the output neuron with the highest value and using it to determine the next character. Also note that you must always remember to call the rnnClearPreviousState() method in every iteration.

And that’s all there is to creating an LSTM. Go ahead and run the program to see it generate sentences and paragraphs that are eerily similar to valid English prose.

Here are some paragraphs my LSTM generated after being trained for nearly an hour:

```
# At iteration 400
At as tebeer spws sowingg yeay ans tulm uulmr, and ane smel-nolng one loomyeang one loo loon one loor and the Harlond mayeld hat could make
the far-lnold years cfer
he s had make
so furils of the reang

# At iteration 600
At arss the sof mangsming dayd anys civme dard any of
the darmyeand coumy on ofeendmeeand fol of
the dere deing wmilendewe of
ftw ongling going of Haknf
the farm-loket could have wondeere thamen-folk f

# At iteration 800
At arslas going gayeads sfmiry as after s fmeras and anys names and song and anlmaso off
the ands of
thad soingl, soing amelas of
the formesars civil were of
England, and noul Haaldeha have wondered it

# At iteration 1000
As was sweet spring yaays summer, and any one of Hayslope any one sumirer all seee lone loo long, now so full of the promise of earl that the farm could have wondered smiling England of farm-labourers
As you can see, the paragraphs still contain quite a lot of jibberish. By increasing the number of iterations, and by using more of the novel, you can get better results.
```

### Conclusion

You now know how to create a simple LSTM using DeepLearning4J. Although we created a character-based LSTM, it is possible to create LSTMs that are word based, which will generate sentences that are more natural. Doing so, however, would definitely be slightly more complex.
