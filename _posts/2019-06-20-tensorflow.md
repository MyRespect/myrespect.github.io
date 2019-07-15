---
layout: post
title:  "Notes for TensorFlow2.0"
categories: Deep_Learning
tags: TensorFlow
--- 

* content
{:toc}


TensorFlow is an open source software library for high performance numerical computation, and it is developed into TF2.0. Here I add some new notes about tensorflow new features, for more details, you can refer to the official website.




#### **Basic changes**

1.	super(student, self).__init__() 是对继承自父类的属性进行初始化，而且是用父类的初始化方法来初始化继承的属性。当然， 如果初始化的逻辑与父类不同，不使用父类的方法自己重新初始化也可以.

2.	在tf2.0中，如果需要构建计算图，我们只需要给python函数加上\@tf.function的装饰器。TF2.0中默认使用eager execution动态图，计算会被直接执行。TF2的一个重要改变是去除tf.Session(). 计算图越复杂，使用静态图的执行效率和加速效果越明显.

3.	将python代码转换成图表示代码的工具叫做autoGraph, 如果一个函数被\@tf.function装饰了，那么autoGraph将会被自动调用，从而讲python函数转换成可执行的图表示.

4.	在第一次调用被\@tf.function装饰的函数时，该函数被执行并跟踪，eager会在这个函数中被禁用，因此每个tf.API只会定义一个生成tf.Tensor输出的节点.

5.	使用tf.function时需要注意，将一个在动态图中可行的函数转换成静态图需要用静态图的方式思考该函数是否可行，在动态图中, tf.variable 时一个普通的python变量, 超出了其作用域范围就会被销毁. 而在静态图中, tf.varuable 则是计算图中一个持续存在的节点, 不受python的作用域的影响.  我们可以将tf.variable作为函数的参数传入，将tf.variable作为类属性来调用.

```
(1) Every tf.Session.run call should be replaced by a Python function.
    The feed_dict and tf.placeholders become function arguments.
    The fetches become the function's return value.
(2) Use tf.Variable instead of tf.get_variable.
(3) Prefer tf.keras.Model.fit over building your own training loops.
(4) Use tf.data datasets for data input.
```
```
import tensorflow_datasets as tfds
datasets, info = tfds.load(name='mnist', with_info=True, as_supervised=True)
mnist_train, mnist_test = datasets['train'], datasets['test']
def scale(image, label):
    image = tf.cast(image, tf.float32)
    image /= 255
    return image, label
train_data = mnist_train.map(scale).shuffle(BUFFER_SIZE).batch(BATCH_SIZE).take(5)
test_data = mnist_test.map(scale).batch(BATCH_SIZE).take(5)
```

#### **Using GPU**
You can specify the running device by tf.device(), for example, "/cpu:0" is the cpu of your machine, "/device:GPU:0":The first GPU of your machine, "/device:GPU:1":The second GPU of your machine.
```
# specify in code, or you can set when you run the python file
import os
os.environ["CUDA_VISIBLE_DEVICES"] = "2"

# allocate gpu according to its need
config = tf.ConfigProto()
config.gpu_options.allow_growth = True

# specifiy GPU memory fraction
config.gpu_options.per_process_gpu_memory_fraction = 0.4

# TF2.0
gpus = tf.config.experimental.list_physical_devices('GPU')
if gpus:
    for gpu in gpus:
      tf.config.experimental.set_memory_growth(gpu, True)

gpus = tf.config.experimental.list_physical_devices('GPU')
if gpus:
    tf.config.experimental.set_visible_devices(gpus[0], 'GPU')
```

#### **Customize the training step**
If you need more flexibility and control, you can have it by implementing your own training loop. There are three steps:

(1) Iterate over a Python generator or tf.data.Dataset to get batches of examples.

(2) Use tf.GradientTape to collect gradients.

(3) Use a tf.keras.optimizer to apply weight updates to the model's variables.

Calling minimize() takes care of both computing the gradients and applying them to the variables. If you want to process the gradients before applying them you can instead use the optimizer in three steps:

(1) Compute the gradients with tape.gradient().

(2) Process the gradients as you wish.

(3) Apply the processed gradients with apply_gradients().

```
model = tf.keras.Sequential([
    tf.keras.layers.Conv2D(32, 3, activation='relu',
                           kernel_regularizer=tf.keras.regularizers.l2(0.02),
                           input_shape=(28, 28, 1)),
    tf.keras.layers.MaxPooling2D(),
    tf.keras.layers.Flatten(),
    tf.keras.layers.Dropout(0.1),
    tf.keras.layers.Dense(64, activation='relu'),
    tf.keras.layers.BatchNormalization(),
    tf.keras.layers.Dense(10, activation='softmax')
])

optimizer = tf.keras.optimizers.Adam(0.001)
loss_fn = tf.keras.losses.SparseCategoricalCrossentropy()

# Create the metrics
loss_metric = tf.keras.metrics.Mean(name='train_loss')
accuracy_metric = tf.keras.metrics.SparseCategoricalAccuracy(name='train_accuracy')

@tf.function
def train_step(inputs, labels):
    with tf.GradientTape() as tape:
      predictions = model(inputs, training=True)
      regularization_loss = tf.math.add_n(model.losses)
      pred_loss = loss_fn(labels, predictions)
      total_loss = pred_loss + regularization_loss

    gradients = tape.gradient(total_loss, model.trainable_variables)
    optimizer.apply_gradients(zip(gradients, model.trainable_variables))
    # Update the metrics
    loss_metric.update_state(total_loss)
    accuracy_metric.update_state(labels, predictions)


for epoch in range(NUM_EPOCHS):
    # Reset the metrics
    loss_metric.reset_states()
    accuracy_metric.reset_states()

    for inputs, labels in train_data:
      train_step(inputs, labels)
    # Get the metric results
    mean_loss = loss_metric.result()
    mean_accuracy = accuracy_metric.result()

    print('Epoch: ', epoch)
    print('  loss:     {:.3f}'.format(mean_loss))
    print('  accuracy: {:.3f}'.format(mean_accuracy))
```

#### **Implementing custom layers**
```
class MyDenseLayer(tf.keras.layers.Layer):
    def __init__(self, num_outputs):
      super(MyDenseLayer, self).__init__()
      self.num_outputs = num_outputs
    
    def build(self, input_shape):
      self.kernel = self.add_variable("kernel", shape=[int(input_shape[-1]), self.num_outputs])
    
    def call(self, input):
      return tf.matmul(input, self.kernel)
  
layer = MyDenseLayer(10)
print(layer(tf.zeros([10, 5])))
print(layer.trainable_variables)
```

#### **Implementing custom model**
```
class MNISTModel(Model):
    def __init__(self):
        super(MNISTModel, self).__init__()
        self.conv1 = Conv2D(32, 3, activation='relu')
        self.flatten = Flatten()
        self.d1 = Dense(128, activation='relu')
        self.d2 = Dense(10, activation='softmax')

    def call(self, x):
        x = self.conv1(x)
        x = self.flatten(x)
        x = self.d1(x)
        return self.d2(x)
```
