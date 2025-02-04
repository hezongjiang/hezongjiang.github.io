---
title: 基于 BERT 的影评文本分类
catalog: true
date: 2025-01-04 19:44:36
subtitle:
header-img:
tags:
- 机器学习
---
BERT 和其他 Transformer 编码器体系结构在 NLP（自然语言处理）的各种任务上取得了巨大成功。他们适合在深度学习模型中使用的自然语言的矢量空间表示。

如果还不清楚 Transformer 模型，可以先看看 [万字详解 ChatGPT 基本原理](https://www.jianshu.com/p/fad652ffa1a2)，了解 Transformer  模型基本原理。

本次将根据电影评论的文本训练情感分析模型，以将电影评论分为正面或负面。

### 1、依赖包

```
pip install "tensorflow-text==2.13.0"
pip install "tf-models-official==2.13.0"
```
需要注意安装的版本需要一致。可以使用 `pip show <package>` 查看已安装包的详细信息。

安装完成后，在代码中导入相关依赖。

```
import os
import shutil

import tensorflow as tf
import tensorflow_hub as hub
import tensorflow_text as text
from official.nlp import optimization  # to create AdamW optimizer

import matplotlib.pyplot as plt
```

### 2、IMDB 影评数据集
下载数据集
```
url = 'https://ai.stanford.edu/~amaas/data/sentiment/aclImdb_v1.tar.gz'

dataset = tf.keras.utils.get_file('aclImdb_v1.tar.gz', url,
                                  untar=True, cache_dir='.',
                                  cache_subdir='')

dataset_dir = os.path.join(os.path.dirname(dataset), 'aclImdb')

train_dir = os.path.join(dataset_dir, 'train')

# remove unused folders to make it easier to load the data
remove_dir = os.path.join(train_dir, 'unsup')
shutil.rmtree(remove_dir)
```

IMDB 数据集已经分为训练集和测试集，但缺乏验证集。使用`validation_split` 将数据分割为 80:20 比例数据的验证集。
```
AUTOTUNE = tf.data.AUTOTUNE
batch_size = 32
seed = 42

raw_train_ds = tf.keras.utils.text_dataset_from_directory(
    'aclImdb/train',
    batch_size=batch_size,
    validation_split=0.2,
    subset='training',
    seed=seed)

class_names = raw_train_ds.class_names
train_ds = raw_train_ds.cache().prefetch(buffer_size=AUTOTUNE)

val_ds = tf.keras.utils.text_dataset_from_directory(
    'aclImdb/train',
    batch_size=batch_size,
    validation_split=0.2,
    subset='validation',
    seed=seed)

val_ds = val_ds.cache().prefetch(buffer_size=AUTOTUNE)

test_ds = tf.keras.utils.text_dataset_from_directory(
    'aclImdb/test',
    batch_size=batch_size)

test_ds = test_ds.cache().prefetch(buffer_size=AUTOTUNE)
```
让我们看一些评论，这里打印前几条评论。
```
for text_batch, label_batch in train_ds.take(1):
  for i in range(3):
    print(f'Review: {text_batch.numpy()[i]}')
    label = label_batch.numpy()[i]
    print(f'Label : {label} ({class_names[label]})')

# Review: b'"Pandemonium" is a horror movie spoof that comes off more stupid than funny. Believe me when I tell you, I love comedies. Especially comedy spoofs. "Airplane", "The Naked Gun" trilogy, "Blazing Saddles", "High Anxiety", and "Spaceballs" are some of my favorite comedies that spoof a particular genre. "Pandemonium" is not up there with those films. Most of the scenes in this movie had me sitting there in stunned silence because the movie wasn\'t all that funny. There are a few laughs in the film, but when you watch a comedy, you expect to laugh a lot more than a few times and that\'s all this film has going for it. Geez, "Scream" had more laughs than this film and that was more of a horror film. How bizarre is that?<br /><br />*1/2 (out of four)'
# Label : 0 (neg)
# Review: b"David Mamet is a very interesting and a very un-equal director. His first movie 'House of Games' was the one I liked best, and it set a series of films with characters whose perspective of life changes as they get into complicated situations, and so does the perspective of the viewer.<br /><br />So is 'Homicide' which from the title tries to set the mind of the viewer to the usual crime drama. The principal characters are two cops, one Jewish and one Irish who deal with a racially charged area. The murder of an old Jewish shop owner who proves to be an ancient veteran of the Israeli Independence war triggers the Jewish identity in the mind and heart of the Jewish detective.<br /><br />This is were the flaws of the film are the more obvious. The process of awakening is theatrical and hard to believe, the group of Jewish militants is operatic, and the way the detective eventually walks to the final violent confrontation is pathetic. The end of the film itself is Mamet-like smart, but disappoints from a human emotional perspective.<br /><br />Joe Mantegna and William Macy give strong performances, but the flaws of the story are too evident to be easily compensated."
# Label : 0 (neg)
# Review: b'Great documentary about the lives of NY firefighters during the worst terrorist attack of all time.. That reason alone is why this should be a must see collectors item.. What shocked me was not only the attacks, but the"High Fat Diet" and physical appearance of some of these firefighters. I think a lot of Doctors would agree with me that,in the physical shape they were in, some of these firefighters would NOT of made it to the 79th floor carrying over 60 lbs of gear. Having said that i now have a greater respect for firefighters and i realize becoming a firefighter is a life altering job. The French have a history of making great documentary\'s and that is what this is, a Great Documentary.....'
# Label : 1 (pos)
```

### 3、使用 TensorFlow Hub的加载模型

可以从 TensorFlow Hub 中选择使用哪种 BERT 模型。有多种 BERT 模型可用。

*   [BERT-Base](https://tfhub.dev/tensorflow/bert_en_uncased_L-12_H-768_A-12/3)：由 BERT 原始作者训练并发布。
*   [Small BERTs](https://tfhub.dev/google/collections/bert/1)：具有相同的体系结构，但包含的参数较少，可以提高训练速度。
*   [ALBERT](https://tfhub.dev/google/collections/albert/1)：通过在层之间共享参数来降低模型大小。
*   [BERT Experts](https://tfhub.dev/google/collections/experts/bert/1)：八个具有Bert-Base体系结构的模型，但在不同的预训练域之间提供了选择，以更加与目标任务保持一致。

这里我们使用 Small BERT，因为训练速度更快，如果需要更高的准确率，可以使用 BERT Experts。

```
bert_model_name = 'small_bert/bert_en_uncased_L-4_H-512_A-8' 

map_name_to_handle = {
    'bert_en_uncased_L-12_H-768_A-12': 'https://tfhub.dev/tensorflow/bert_en_uncased_L-12_H-768_A-12/3',
    'bert_en_cased_L-12_H-768_A-12': 'https://tfhub.dev/tensorflow/bert_en_cased_L-12_H-768_A-12/3',
    'bert_multi_cased_L-12_H-768_A-12': 'https://tfhub.dev/tensorflow/bert_multi_cased_L-12_H-768_A-12/3',
    'small_bert/bert_en_uncased_L-2_H-128_A-2': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-2_H-128_A-2/1',
    'small_bert/bert_en_uncased_L-2_H-256_A-4': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-2_H-256_A-4/1',
    'small_bert/bert_en_uncased_L-2_H-512_A-8': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-2_H-512_A-8/1',
    'small_bert/bert_en_uncased_L-2_H-768_A-12': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-2_H-768_A-12/1',
    'small_bert/bert_en_uncased_L-4_H-128_A-2': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-4_H-128_A-2/1',
    'small_bert/bert_en_uncased_L-4_H-256_A-4': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-4_H-256_A-4/1',
    'small_bert/bert_en_uncased_L-4_H-512_A-8': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-4_H-512_A-8/1',
    'small_bert/bert_en_uncased_L-4_H-768_A-12': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-4_H-768_A-12/1',
    'small_bert/bert_en_uncased_L-6_H-128_A-2': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-6_H-128_A-2/1',
    'small_bert/bert_en_uncased_L-6_H-256_A-4': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-6_H-256_A-4/1',
    'small_bert/bert_en_uncased_L-6_H-512_A-8': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-6_H-512_A-8/1',
    'small_bert/bert_en_uncased_L-6_H-768_A-12': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-6_H-768_A-12/1',
    'small_bert/bert_en_uncased_L-8_H-128_A-2': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-8_H-128_A-2/1',
    'small_bert/bert_en_uncased_L-8_H-256_A-4': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-8_H-256_A-4/1',
    'small_bert/bert_en_uncased_L-8_H-512_A-8': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-8_H-512_A-8/1',
    'small_bert/bert_en_uncased_L-8_H-768_A-12': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-8_H-768_A-12/1',
    'small_bert/bert_en_uncased_L-10_H-128_A-2': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-10_H-128_A-2/1',
    'small_bert/bert_en_uncased_L-10_H-256_A-4': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-10_H-256_A-4/1',
    'small_bert/bert_en_uncased_L-10_H-512_A-8': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-10_H-512_A-8/1',
    'small_bert/bert_en_uncased_L-10_H-768_A-12': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-10_H-768_A-12/1',
    'small_bert/bert_en_uncased_L-12_H-128_A-2': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-12_H-128_A-2/1',
    'small_bert/bert_en_uncased_L-12_H-256_A-4': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-12_H-256_A-4/1',
    'small_bert/bert_en_uncased_L-12_H-512_A-8': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-12_H-512_A-8/1',
    'small_bert/bert_en_uncased_L-12_H-768_A-12': 'https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-12_H-768_A-12/1',
    'albert_en_base': 'https://tfhub.dev/tensorflow/albert_en_base/2',
    'electra_small': 'https://tfhub.dev/google/electra_small/2',
    'electra_base': 'https://tfhub.dev/google/electra_base/2',
    'experts_pubmed': 'https://tfhub.dev/google/experts/bert/pubmed/2',
    'experts_wiki_books': 'https://tfhub.dev/google/experts/bert/wiki_books/2',
    'talking-heads_base': 'https://tfhub.dev/tensorflow/talkheads_ggelu_bert_en_base/1',
}

map_model_to_preprocess = {
    'bert_en_uncased_L-12_H-768_A-12': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'bert_en_cased_L-12_H-768_A-12': 'https://tfhub.dev/tensorflow/bert_en_cased_preprocess/3',
    'small_bert/bert_en_uncased_L-2_H-128_A-2': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-2_H-256_A-4': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-2_H-512_A-8': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-2_H-768_A-12': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-4_H-128_A-2': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-4_H-256_A-4': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-4_H-512_A-8': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-4_H-768_A-12': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-6_H-128_A-2': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-6_H-256_A-4': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-6_H-512_A-8': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-6_H-768_A-12': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-8_H-128_A-2': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-8_H-256_A-4': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-8_H-512_A-8': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-8_H-768_A-12': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-10_H-128_A-2': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-10_H-256_A-4': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-10_H-512_A-8': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-10_H-768_A-12': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-12_H-128_A-2': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-12_H-256_A-4': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-12_H-512_A-8': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'small_bert/bert_en_uncased_L-12_H-768_A-12': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'bert_multi_cased_L-12_H-768_A-12': 'https://tfhub.dev/tensorflow/bert_multi_cased_preprocess/3',
    'albert_en_base': 'https://tfhub.dev/tensorflow/albert_en_preprocess/3',
    'electra_small': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'electra_base': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'experts_pubmed': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'experts_wiki_books': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
    'talking-heads_base': 'https://tfhub.dev/tensorflow/bert_en_uncased_preprocess/3',
}

tfhub_handle_encoder = map_name_to_handle[bert_model_name]
tfhub_handle_preprocess = map_model_to_preprocess[bert_model_name]

print(f'BERT model selected           : {tfhub_handle_encoder}')
print(f'Preprocess model auto-selected: {tfhub_handle_preprocess}')
```

### 4、模型预处理
```
bert_preprocess_model = hub.KerasLayer(tfhub_handle_preprocess)
text_test = ['this is such an amazing movie!']
text_preprocessed = bert_preprocess_model(text_test)
```
创建了一个 Keras 层，用于将文本数据预处理为适合 BERT 模型的格式。打印模型信息。

```
print(f'Keys       : {list(text_preprocessed.keys())}')
print(f'Shape      : {text_preprocessed["input_word_ids"].shape}')
print(f'Word Ids   : {text_preprocessed["input_word_ids"][0, :12]}')
print(f'Input Mask : {text_preprocessed["input_mask"][0, :12]}')
print(f'Type Ids   : {text_preprocessed["input_type_ids"][0, :12]}')

# Keys       : ['input_word_ids', 'input_type_ids', 'input_mask']

# Shape      : (1, 128) 
# 1：表示有一个样本。128：表示每个样本的长度（填充后的长度）。

# Word Ids   : [ 101 2023 2003 2107 2019 6429 3185  999  102    0    0    0]  
# 第一个样本的前 12 个单词 ID。
# 101：BERT 的起始标记。
# 2023：单词 this 的 ID。
# 2003：单词 is 的 ID。
# ……
# 102：BERT 的结束标记 。
 
# Input Mask : [1 1 1 1 1 1 1 1 1 0 0 0] 
# 第一个样本的前 12 个掩码值。1：表示实际的单词，0：表示填充的位置

# Type Ids   : [0 0 0 0 0 0 0 0 0 0 0 0]
```

### 5、使用BERT模型
创建了一个 Keras 层，用于将预处理后的数据传递给 BERT 模型进行编码。
```
bert_model = hub.KerasLayer(tfhub_handle_encoder)
bert_results = bert_model(text_preprocessed)
```

```
print(f'Loaded BERT: {tfhub_handle_encoder}')
print(f'Pooled Outputs Shape:{bert_results["pooled_output"].shape}')
print(f'Pooled Outputs Values:{bert_results["pooled_output"][0, :12]}')
print(f'Sequence Outputs Shape:{bert_results["sequence_output"].shape}')
print(f'Sequence Outputs Values:{bert_results["sequence_output"][0, :12]}')

# Loaded BERT: https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-4_H-512_A-8/1
# 这表示加载的 BERT 模型是 small_bert/bert_en_uncased_L-4_H-512_A-8，这是一个较小的 BERT 模型，具有 4 层、512 个隐藏单元和 8 个注意力头。

# Pooled Outputs Shape:(1, 512)
# pooled_output 是 BERT 模型对整个序列的聚合表示，通常用于分类任务。1：表示有一个样本。512：表示每个样本的聚合表示的维度（隐藏单元数）。

# Pooled Outputs Values:[ 0.762629    0.99280983 -0.18611868  0.36673862  0.15233733  0.6550447 0.9681154  -0.9486271   0.00216128 -0.9877732   0.06842692 -0.97630584]
# BERT 模型对整个序列的聚合表示的前 12 个维度的值。

# Sequence Outputs Shape:(1, 128, 512)
# sequence_output 是 BERT 模型对每个位置的隐藏状态的表示，通常用于序列标注任务。1：表示有一个样本。128：表示每个样本的长度（填充后的长度）。512：表示每个位置的隐藏状态的维度（隐藏单元数）。

# Sequence Outputs Values:[
# [-0.28946346  0.3432128   0.33231518 ...  0.21300825  0.7102068 -0.05771117]
# [-0.28742072  0.31981036 -0.23018576 ...  0.58455    -0.21329743  0.72692114]
# [-0.66157067  0.68876773 -0.8743301  ...  0.1087725  -0.26173177  0.47855407]
# ...
# [-0.2256118  -0.2892561  -0.0706445  ...  0.47566038  0.83277136  0.40025333]
# [-0.2982428  -0.27473134 -0.05450517 ...  0.48849747  1.0955354   0.18163396]
# [-0.44378242  0.00930811  0.07223688 ...  0.1729009   1.1833243   0.07898017]]
# BERT 模型对每个位置的隐藏状态的表示的前 12 个位置的值。每个位置的表示是一个 512 维的向量
```

### 6、定义模型
```
def build_classifier_model():
  text_input = tf.keras.layers.Input(shape=(), dtype=tf.string, name='text')
  preprocessing_layer = hub.KerasLayer(tfhub_handle_preprocess, name='preprocessing')
  encoder_inputs = preprocessing_layer(text_input)
  encoder = hub.KerasLayer(tfhub_handle_encoder, trainable=True, name='BERT_encoder')
  outputs = encoder(encoder_inputs)
  net = outputs['pooled_output']
  net = tf.keras.layers.Dropout(0.1)(net)
  net = tf.keras.layers.Dense(1, activation=None, name='classifier')(net)
  return tf.keras.Model(text_input, net)

classifier_model = build_classifier_model()
bert_raw_result = classifier_model(tf.constant(text_test))
```
这里总共定义了 5 层：
1. 输入层：输入字符串形式的文本数据。
2. 预处理层：加载预处理模型，将文本数据预处理为 BERT 模型所需的格式。
3. BERT 编码层：加载 BERT 模型，并把上一步预处理后的数据传递给 BERT 模型。
4. Dropout 层：防止过拟合。Dropout 比例为 0.1。
5. 全连接层（Dense 层）：输出一个未激活的分数（logits），用于分类任务。

![](https://upload-images.jianshu.io/upload_images/2708793-a80a637368ef105f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 7、训练模型
##### 7.1、损失函数
由于这是一个二分类问题，并且模型的输出是一个概率，因此损失函数使用 `BinaryCrossentRopy`。
```
loss = tf.keras.losses.BinaryCrossentropy(from_logits=True)
metrics = tf.metrics.BinaryAccuracy()
```
##### 7.2、优化器

创建优化器。这里使用了 `optimization.create_optimizer` 函数，用于创建一个优化器实例。优化器类型为 'adamw'，即 AdamW 优化器，它是 Adam 优化器的一个变体，适用于权重衰减。

```
# 将训练 5 轮
epochs = 5
steps_per_epoch = tf.data.experimental.cardinality(train_ds).numpy()
num_train_steps = steps_per_epoch * epochs
num_warmup_steps = int(0.1*num_train_steps)
# 学习率
init_lr = 3e-5
optimizer = optimization.create_optimizer(init_lr=init_lr,
                                          num_train_steps=num_train_steps,
                                          num_warmup_steps=num_warmup_steps,
                                          optimizer_type='adamw')
```

##### 7.3、编译并训练模型
```
classifier_model.compile(optimizer=optimizer,
                         loss=loss,
                         metrics=metrics)

history = classifier_model.fit(x=train_ds,
                               validation_data=val_ds,
                               epochs=epochs)
```
训练过程打印。
```
Training model with https://tfhub.dev/tensorflow/small_bert/bert_en_uncased_L-4_H-512_A-8/1
Epoch 1/5
625/625 [==============================] - 695s 1s/step - loss: 0.4986 - binary_accuracy: 0.7390 - val_loss: 0.3835 - val_binary_accuracy: 0.8370
Epoch 2/5
625/625 [==============================] - 677s 1s/step - loss: 0.3318 - binary_accuracy: 0.8508 - val_loss: 0.3758 - val_binary_accuracy: 0.8456
Epoch 3/5
625/625 [==============================] - 674s 1s/step - loss: 0.2568 - binary_accuracy: 0.8929 - val_loss: 0.3913 - val_binary_accuracy: 0.8440
Epoch 4/5
625/625 [==============================] - 676s 1s/step - loss: 0.1964 - binary_accuracy: 0.9209 - val_loss: 0.4450 - val_binary_accuracy: 0.8528
Epoch 5/5
625/625 [==============================] - 676s 1s/step - loss: 0.1593 - binary_accuracy: 0.9392 - val_loss: 0.4698 - val_binary_accuracy: 0.8504
```
训练结果分析：
* 「训练集」的损失值（loss）在每个 epoch 中持续下降，从 0.4986 降低到 0.1593，表明模型在训练集上的拟合效果逐渐变好。
* 「训练集」的二分类准确率（binary_accuracy）在每个 epoch 中持续提高，从 73.90% 提高到 93.92%，表明模型在训练集上的分类性能逐渐变好。
* 「验证集」的损失值（val_loss）在前两个 epoch 中略有下降，但在第 3 个 epoch 后开始上升，表明模型在验证集上的拟合效果在后期有所下降，可能存在过拟合现象。
* 「验证集」的二分类准确率（val_binary_accuracy）保持在 85.04%，表明模型在验证集上的分类性能较为稳定。

##### 7.4、评估模型
```
loss, accuracy = classifier_model.evaluate(test_ds)

print(f'Loss: {loss}')
print(f'Accuracy: {accuracy}')

# 782/782 [==============================] - 222s 284ms/step - loss: 0.4493 - binary_accuracy: 0.8542
# Loss: 0.4492934048175812
# Accuracy: 0.8541600108146667
```

##### 7.5、绘制准确率和损失值
```
history_dict = history.history
print(history_dict.keys())

acc = history_dict['binary_accuracy']
val_acc = history_dict['val_binary_accuracy']
loss = history_dict['loss']
val_loss = history_dict['val_loss']

epochs = range(1, len(acc) + 1)
fig = plt.figure(figsize=(10, 6))
fig.tight_layout()

plt.subplot(2, 1, 1)
# r is for "solid red line"
plt.plot(epochs, loss, 'r', label='Training loss')
# b is for "solid blue line"
plt.plot(epochs, val_loss, 'b', label='Validation loss')
plt.title('Training and validation loss')
# plt.xlabel('Epochs')
plt.ylabel('Loss')
plt.legend()
plt.show()

plt.subplot(2, 1, 2)
plt.plot(epochs, acc, 'r', label='Training acc')
plt.plot(epochs, val_acc, 'b', label='Validation acc')
plt.title('Training and validation accuracy')
plt.xlabel('Epochs')
plt.ylabel('Accuracy')
plt.legend(loc='lower right')
plt.show()
```

![](https://upload-images.jianshu.io/upload_images/2708793-50353618be4df90a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 8、保存模型
```
dataset_name = 'imdb'
saved_model_path = './{}_bert'.format(dataset_name.replace('/', '_'))

classifier_model.save(saved_model_path, include_optimizer=False)
```
重新加载保存的模型，并进行验证。
```
reloaded_model = tf.saved_model.load(saved_model_path)

def print_my_examples(inputs, results):
  result_for_printing = \
    [f'input: {inputs[i]:<30} : score: {results[i][0]:.6f}'
                         for i in range(len(inputs))]
  print(*result_for_printing, sep='\n')
  print()


examples = [
    'this is such an amazing movie!',  # this is the same sentence tried earlier
    'The movie was great!',
    'The movie was meh.',
    'The movie was okish.',
    'The movie was terrible...'
]

reloaded_results = tf.sigmoid(reloaded_model(tf.constant(examples)))
original_results = tf.sigmoid(classifier_model(tf.constant(examples)))

print('Results from the saved model:')
print_my_examples(examples, reloaded_results)
print('Results from the model in memory:')
print_my_examples(examples, original_results)
```

```
Results from the saved model:
input: this is such an amazing movie! : score: 0.999361
input: The movie was great!           : score: 0.989547
input: The movie was meh.             : score: 0.856317
input: The movie was okish.           : score: 0.022705
input: The movie was terrible...      : score: 0.001032

Results from the model in memory:
input: this is such an amazing movie! : score: 0.999361
input: The movie was great!           : score: 0.989547
input: The movie was meh.             : score: 0.856317
input: The movie was okish.           : score: 0.022705
input: The movie was terrible...      : score: 0.001032
```
使用内存中的模型，与保存的模型，进行验证，结果相同，都对影评进行了正确分类。