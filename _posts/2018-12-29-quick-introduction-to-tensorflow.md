---
layout: post
title:  "A quick introduction to TensorFlow"
date:   2018-12-29 18:55:20 +0200
author: Andrea
comments: true
---

[TensorFlow][tensorflow] has become one of the most widely used tools for training deep neural networks, especially with [Keras][keras] on top which makes it very easy to use. Though if you want to create an architecture that deviates from the standard, you run into the limits of Keras pretty quickly, so it's worth getting at least a basic understanding of how TensorFlow works. This is a quick introduction to the session and graph based programming style used by TensorFlow, geared toward those who are already familiar with neural networks in general. So let's look at a basic example of an MNIST classifier written in pure TensorFlow to get started!

The thing about TensorFlow that feels unnatural to most people coming from an object oriented coding background is the graph based program flow. When executing a regular Python script, the commands in each line immediately get executed and produce the appropriate values. In TensorFlow on the other hand, you first construct a *graph* of your program. This graph is like a map showing the computer which dependencies exist between different elements of a program, and in which order to execute commands. In the case of a neural network, these elements are layers and their units, and the mathematical operations needed for learning. Let's create the graph of layers needed for our classifier:

{% highlight python %}
# placeholder objects for our training data
input_pl = tf.placeholder(shape=(None, 28, 28), dtype=tf.float32)
label_pl = tf.placeholder(dtype=tf.int32)

# neural network consisting of one input and two fully connected layers
input_layer = tf.layers.flatten(input_pl)
first_layer_output = tf.layers.dense(inputs=input_layer, units=128, activation=tf.nn.relu)
output_layer = tf.layers.Dense(units=10, activation=tf.nn.softmax)
output = output_layer(first_layer_output)
{% endhighlight %}

In lines 2 and 3, we create placeholders for our training data. These serve as a means for us to insert our data into the computational graph. *input_pl* represents the MNIST images and therefore has a shape of `(None, 28, 28)` (`None` being the batch size), and *label_pl* represents the labels. As you see, it is also possible to define a placeholder without specifying its size beforehand. Then we add an input layer that flattens the images to a vector of length 784, and feed this to the first dense layer. Notice that in lines 7 an 8, we add two dense layers, but the first dense layer is specified as `tf.layers.dense`, while the second one is `tf.layers.Dense` (with a capital D). These are actually two different ways to add a layer: `dense` is the actual function call executing the layer, and returns a tensor which contains the output of that layer and nothing else. When using `Dense`, you are creating an actual object of the `Dense` class. So instead of directly feeding the input of that layer into the function call as with the first dense layer, you have to call your created `Dense` object to receive an output, as seen in line 9. This is useful when you want to access certain attributes of the layer class, for example the weights. Now lets see what we get when we print *output*:

{% highlight python %}
Tensor("dense_1/Softmax:0", shape=(?, 10), dtype=float32)
{% endhighlight %}

We now have created our computational graph, and the print-out above returns the placeholder for the tensor that will later hold our network's output when the graph is executed. Now that we have our layers set up, we need to take care of adding the actual training routine to the graph:

{% highlight python %}
loss = tf.losses.sparse_softmax_cross_entropy(labels=label_pl, logits=output)
optimizer = tf.train.AdamOptimizer(learning_rate=0.01)
train_op = optimizer.minimize(loss)
{% endhighlight %}

We define a loss function, in this case *sparse_softmax_cross_entropy*, and the *AdamOptimizer* for gradient descent. train_op is the operation that is executed at each training step, which here means that the optimizer is minimizing the loss. Now our computational graph contains everything we need to train our neural network. The last element that is missing is the *global_variables_initializer*:

{% highlight python %}
initializer = tf.global_variables_initializer()
{% endhighlight %}

This takes care of initializing any variables in our architecture that need random initial values, for example the weights of the network. This is a crucial step at the beginning of every training procedure, and will lead to errors when omitted because the network will start training without having its weights set. At this point all we need to run the graph and train the network is a session. Creating a session and training the classifier is as simple as this:

{% highlight python %}
# load the training data
(x_train, y_train), (x_test, y_test) = tf.keras.datasets.mnist.load_data()
x_train = x_train / 255.0

with tf.Session() as sess:
  sess.run(initializer)

  for i in range(5):
    _, loss_val= sess.run(
    (train_op, loss),
    feed_dict={input_pl: x_train, label_pl: y_train})

    print("Iteration", i, "| Loss", loss_val)
{% endhighlight %}

The session object is created by calling `tf.Session()` (doing this in a `with` block only means that the session exists solely in the scope of that block, but not outside of it). Now we can run arbitrary parts of our graph using `sess.run()` - akin to what we would do when running a regular Python script. The *initializer* should be run before anything else, because it fills the elements in our graph with the necessary initial values. Then we can finally run our *train_op* - we do this in a for-loop that specifies the number of iterations we'd like to train for, 5 in this example. The first argument of `sess.run()` specifies which elements of the graph sould be run. All other operations that are necessary to run those elements will also be run, so you don't have to add every layer here to run the whole network, adding *train_op* is enough. Now is also the time to pass the training data to the session object, so it can use it when the graph is executed. We do this by using the so-called *feed_dict*: this is a dictionary that contains all values we'd like to change in the graph. Theoretically we could change arbitrary elements in the graph in this way, like the weights of a certain layer. In this example, we want to replace *input_pl* and *label_pl* with the real data, and do so by passing the placeholder variables as the keys, and the real data as the corresponding values of the feed_dict. The values returned by `sess.run()` are the actual values being created when the graph is executed, for example the actual loss values after each iteration. And that's it! Once you get the hang of how to access certain elements in the graph and replace them with the values you want and execute the whole thing, you can build arbitrarily complex architectures in pure TensorFlow. Getting used to building a graph first, and then executing it to see the actual calculations happen, might see cumbersome at first, but becomes routine pretty quickly if you play around a little. Being able to directly manipulate the computational graph to fit your needs then gives you a lot of control over your neural network architectures, so it's definitely worth getting into!


[tensorflow]: https://www.tensorflow.org/
[keras]: https://keras.io/