---
layout: post
title:  "Notes for TensorFlow1.0"
categories: Deep_Learning
tags: TensorFlow
--- 

* content
{:toc}

TensorFlow is an open source software library for high performance numerical computation. The original post when I worte was in Tensorflow 1.0, and now Tensorflow is developed into TF2.0. Here I add some new notes about tensorflow.





#### **Basic**

It is still possible to run 1.X code, unmodified (except for contrib), in TensorFlow 2.0:

```
import tensorflow.compat.v1 as tf
tf.disable_v2_behavior()

The tf.layers module is used to contain layer-functions that relied on tf.variable_scope to define and reuse variables.
If you were using regularizers of initializers from tf.contrib, these have more argument changes than others.
# replace tf.contrib.layers.
tf.keras.layers.Conv2D(32, 3, activation='relu', kernel_regularizer=tf.keras.regularizers.l2(0.04),                input_shape=(28, 28, 1))
```

```
import tensorflow as tf

x=tf.placeholder(tf.float32, shape=...)
y=tf.variable(0, name='x', dtype=tf.float32)
z=tf.constant(2.0, shape=..., dtype=...)
w=tf.get_variable('w', [1,2,3], initializer=tf.contrib.layers.xavier_initializer(seed=0))

tf.nn.conv2d(input, filter, strides, padding, use_cudnn_on_gpu=True, data_format='NHWC', dilations=[1, 1, 1, 1], name=None)

tf.nn.max_pool(value, ksize=[1,f,f,1], strides=[1,s,s,1], padding, data_format='NHWC', name=None) #ksize: kernel size, a window of size(f,f), stride of size(s,s)

tf.contrib.layers.flatten(inputs, outputs_collections=None, scope=None) # input is a tensor of size[batch_size,...], return a flattened tensor with shape [batch_size, k].

tf.contrib.layers.fully_connected(
    inputs,
    num_outputs,
    activation_fn=tf.nn.relu,
    normalizer_fn=None,
    normalizer_params=None,
    weights_initializer=initializers.xavier_initializer(),
    weights_regularizer=None,
    biases_initializer=tf.zeros_initializer(),
    biases_regularizer=None,
    reuse=None,
    variables_collections=None,
    outputs_collections=None,
    trainable=True,
    scope=None
) # fully_connected creates a variable called weights, representing a fully connected weight matrix, which is multiplied by the inputs to produce a Tensor of hidden units, returns the tensor variable representing the result of the series of operations. The input is a tensor of at least rank 2 and static value for the last dimension; i.e. [batch_size, depth], [None, None, None, channels]


tf.nn.softmax_cross_entropy_with_logits(_sentinel=None, labels=None, logits=None, dim=-1, name=None)

tf.math.reduce_mean(input_tensor, axis=None, keepdims=None, name=None, reduction_indices=None, keep_dims=None) #Reduces input_tensor along the dimensions given in axis, if keepdims is true, the reduced dimensions are retained with length 1.

tf.reshape(x, [-1, 28, 28, 1])
tf.split(x,n,0) #ｘ is the Tensor to split, n is the number of splits, 0 is the dimension along which to split.

tf.nn.depthwise_conv2d(input, filter, strides, padding, rate=None, name=None, data_format=None) # Must have strides = [1, stride, stride, 1], Given a 4D input tensor and a filter tensor of shape [filter_height, filter_width, in_channels, channel_multiplier], containing in_channels convolutional filters of depth 1, depthwise_conv2d applies a different filter to each input channel (expanding from 1 channel to channel_multiplier channels for each), then concatenates the results together. The output has in_channels * channel_multiplier channels.
```
#### **Optimization Methods**

Gradient Descent: taking gradient step with respect to all m examples on each step

Stochastic GD: using only 1 training examples before updating the gradients

Minibatch GD: using a small part of examples

Momentum: taking into account the past gradients to smooth out the update, and storing the direction of the previous gadients.This will be the exponentially weighted average of gradient on previous steps.

Adam: combing ideas from RMSProp and Momentum.

#### **Save Model**

checkpoints is a binary file，which mapping variables into corresponding tensors．

```
saver=tf.train.Saver(...variables...)
sess = tf.Session()
for step in xrange(1000000):
    sess.run(..training_op..)
    if step % 1000 == 0:
        # Append the step number to the checkpoint name:
        saver.save(sess, 'my-model', global_step=step)   #  ==> filename: 'my-model-[step]'
```

tf.app.flags用于支持接受命令行传递参数；
```
flags = tf.app.flags
FLAGS = flags.FLAGS
flags.DEFINE_float('learning_rate', 0.01, 'Initial learning rate.')
```
From [stackoverflow](https://stackoverflow.com/questions/33932901/whats-the-purpose-of-tf-app-flags-in-tensorflow), note that this module is currently packaged as a convenience for writing demo apps, and is not technically part of the public API, so it may change in future. We recommend that you implement your own flag parsing using argparse or whatever library you prefer.

#### **Load Model**
```
# freeze model

pickle.dump(predictions, open("predictions.p", "wb"))
pickle.dump(history, open("history.p", "wb"))
tf.train.write_graph(sess.graph_def, '.', 'har.pbtxt')
saver.save(sess, save_path="./har.ckpt")
sess.close()

from tensorflow.python.tools import freeze_graph

MODEL_NAME = 'har'

input_graph_path = MODEL_NAME + '.pbtxt'
checkpoint_path = MODEL_NAME + '.ckpt'
restore_op_name = "save/restore_all"
filename_tensor_name = "save/Const:0"
output_frozen_graph_name = 'frozen_' + MODEL_NAME + '.pb'

freeze_graph.freeze_graph(input_graph_path, input_saver="",
                          input_binary=False, input_checkpoint=checkpoint_path,
                          output_node_names="y_", restore_op_name="save/restore_all",
                          filename_tensor_name="save/Const:0",
                          output_graph=output_frozen_graph_name, clear_devices=True, initializer_nodes="")

#Load Model

with tf.Graph().as_default():
    output_graph_def = tf.GraphDef()

    with open(pb_file_path, "rb") as f:
        output_graph_def.ParseFromString(f.read())
        _ = tf.import_graph_def(output_graph_def, name="")

    with tf.Session() as sess:
        init = tf.global_variables_initializer()
        sess.run(init)
        input_x = tf.get_default_graph().get_tensor_by_name("input:0")
        out_softmax = sess.graph.get_tensor_by_name("prob:0")
        out_result = sess.run(out_softmax, feed_dict={input_x: reshaped_segments})
```