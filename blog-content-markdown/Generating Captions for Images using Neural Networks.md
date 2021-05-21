We all know that Neural Networks can do so much in terms of replicating a human brain while performing certain tasks. The applications of Neural Networks in Computer Vision and Natural Language Generation has been truly remarkable.

This article will introduce the readers to one such application of Neural Networks and let the readers understand how we can actually generate Captions (descriptions) for Images using a Hybrid Network of CNNs and RNNs (LSTM). The dataset that we are using for this task is the popular _flickr 8k Image Dataset_ which is the benchmark data for this task and can be accessed by the below link.

Kaggle — [https://www.kaggle.com/adityajn105/flickr8k](https://www.kaggle.com/adityajn105/flickr8k)

> _Note : We will be splitting our dataset as 7k for training and 1k for testing purpose._

We will first discuss what are the different components (layers) and their function in our Hybrid Neural Network. Along with this, we will also look at the practical implementation of developing our Hybrid Neural Network using Tensorflow, Keras, and Python.

## Overall Architecture of Neural Network

Let's look at the overall architecture of our Neural Network that we will be using for generating captions.

![](https://miro.medium.com/max/700/1*D3JoyOelmtyowMuoSIkadg.png)

In simple terms, the above Neural Network has 3 main components (Sub Networks), each assigned with a specific task namely —The Convolutional Network (for extracting features from Images), The RNN (LSTM) (for text generation) and A Decoder (for combining both networks).

Now let's talk about each component in detail and understand their working.

## Image Feature Extractor

For generating features from an Image we will be using a Convolutional Neural Network with just a bit of modification. Let's look at a ConvNet that is used for Image Recognition.

![](https://miro.medium.com/max/700/1*YXP94kNxfDlyqCaTQg9pEg.jpeg)

As you can see, our CNN has two sub-networks —

1. **Feature Learning Network** — Network responsible for generating feature maps from the Images (Network of multiple Convolution & Pooling Layers).

2. **Classification Network** — Fully connected Deep Neural Network responsible for Image classification (Network of multiple Dense Layers and a single Output Layer).

Since we are only interested in extracting features from Images and not their classification, we are only interested in the Feature Learning Part of the CNN and that is how we can extract features from the Images.

Below code can be used to extract features from any set of images-

```python
import tensorflow as tf
from keras.preprocessing import image

import numpy as np

# function to extract features from image
def extract_image_features():

    model = tf.keras.models.Sequential()

    # adding first layers of convolution and pooling layers to network
    model.add(tf.keras.layers.Conv2D(filters=64, kernel_size=(3,3), input_shape=(90,90,3), padding="VALID", activation="relu"))
    model.add(tf.keras.layers.Conv2D(filters=64, kernel_size=(3,3), activation="relu"))
    model.add(tf.keras.layers.MaxPool2D(pool_size=2, strides=2))

    # adding second layers of convolution and pooling layers to network
    model.add(tf.keras.layers.Conv2D(filters=32, kernel_size=(3,3), padding="VALID", activation="relu"))
    model.add(tf.keras.layers.Conv2D(filters=32, kernel_size=(3,3), activation="relu"))
    model.add(tf.keras.layers.AveragePooling2D(pool_size=2, strides=1))

    # flattening the output using flatten layer, since the input to neural net has to be flat
    model.add(tf.keras.layers.Flatten())

    # model summary
    model.summary()

    return model

for file in os.listdir(image_path):
    path = image_path + "//" + file
    img = image.load_img(path, target_size=(90, 90))
    img_data = image.img_to_array(img)
    img_data = np.expand_dims(img_data, axis=0)
    img_data = preprocess_input(img_data)

    feature = extract_image_features.predict(img_data)
    feature = np.reshape(feature, feature.shape[1])

```

Yes, anyone can build their own Image Feature Extractor using the above code, but there is catch…

**_The above model is too simple to extract each and every important detail from our set of images and hence will impact the performance of our overall model. Also, making model too complex (multiple Dense Layers with high number of Neurons) is also challenging due to the non-availability of high performing GPUs and systems._**

To solve this problem, we have very popular pre-trained CNN models (VGG-16, ResNet50 etc. developed by scientists of different universities and organizations) that are available in Tensorflow and can be used to extract features from images. **Just remember to remove the output layer from the model before using it for Feature Extraction.**

The below code will let you understand how to use these pre-trained models in Tensorflow for extracting features from images.

```python
import tensorflow as tf
from keras.preprocessing import image
from keras.applications.resnet50 import ResNet50
from keras.applications.resnet50 import preprocess_input
from keras.models import Model

# load the ResNet50 Model
feature_extractor = ResNet50(weights='imagenet', include_top=False)
feature_extractor_new = Model(feature_extractor.input, feature_extractor.layers[-2].output)
feature_extractor_new.summary()

for file in os.listdir(image_path):
    path = image_path + "//" + file
    img = image.load_img(path, target_size=(90, 90))
    img_data = image.img_to_array(img)
    img_data = np.expand_dims(img_data, axis=0)
    img_data = preprocess_input(img_data)

    feature = feature_extractor_new.predict(img_data)
    feature_reshaped = np.array(feature).flatten()
```

As you can see below, if you execute the above code you will see that our image feature is nothing but a numpy array of shape — (18432,)

```
image_feature_dictionary[list(image_feature_dictionary. Keys())[0]].shape
(18432,)
```

Next we will develop our LSTM Network (RNN) for generating captions for images.

## LSTM for generating Captions

Text generation is one of the most popular applications of LSTM Networks. LSTM cells (the basic building block of LSTM Networks) have the capability to generate output basis on the output of previous layers i.e. it retains the output of previous layers (the memory) and uses that memory to generate (predict) the next output in the sequence.

For our dataset, we have 5 captions for each images i.e. 40k captions in total.

Let's look at our dataset-

![](https://miro.medium.com/max/375/1*LO2YGullmHu-iFUtVWL_EA.jpeg)

**Captions for Image**

1. **_A child in a pink dress is climbing up a set of stairs in an entry way._**

2. **_A girl going into a wooden building._**

3. **_A little girl climbing into a wooden playhouse._**

4. **_A little girl climbing the stairs to her playhouse._**

5. **_A little girl in a pink dress going into a wooden cabin._**

As seen, all the captions describe the Image quite well. Our task now is to design an RNN that could replicate this task for any similar set of images.

Coming back to the original task, we first have to look at how an LSTM Network generates text. For an LSTM network caption is nothing but a long sequence of individual words (encoded as digits) placed together. Using this information, it tries to predict the next word in the sequence based on the previous words (Memory).

In our case, since the captions can be of varying length, we first need to specify the starting and ending of every caption. Let us see what this actually means-

![](https://miro.medium.com/max/3482/1*ccAxD4C8u86gE4Z_-shFPw.jpeg)

First we will be adding `<start>` and `<end>` to each and every caption in our dataset. After that we will tokenize each & every caption in our training dataset before creating our final vocabulary. For training our model, we will be dropping tokes (words) from our vocabulary that has frequency less than or equal to 10. This step is added to improve the generic performance of our model and to prevent it from overfitting the training dataset.

The following code can be used to implement this-

```python
# loading captions from captions file
import pandas as pd

# loading captions.txt
captions = pd.read_csv('/kaggle/input/flickr8k/captions.txt', sep=",")
captions = captions.rename(columns=lambda x: x.strip().lower())
captions['image'] = captions['image'].apply(lambda x: x.split(".")[0])
captions = captions[['image', 'caption']]
# adding <start> and <end> to every caption
captions['caption'] = "<start> " + captions['caption'] + " <end>"

# in case we have any missing caption/blank caption drop it
print(captions.shape)
captions = captions.dropna()
print(captions.shape)

# training and testing image captions split
train_image_captions = {}
test_image_captions = {}

# list for storing every caption
all_captions = []

# storing training data
for image in train_data_images:
    tempDf = captions[captions['image'] == image]
    list_of_captions = tempDf['caption'].tolist()
    train_image_captions[image] = list_of_captions
    all_captions.append(list_of_captions)

# store testing data
for image in test_data_images:
    tempDf = captions[captions['image'] == image]
    list_of_captions = tempDf['caption'].tolist()
    test_image_captions[image] = list_of_captions
    all_captions.append(list_of_captions)

print("Data Statistics")
print(f"Training Images Captions {len(train_image_captions.keys())}")
print(f"Testing Images Captions {len(test_image_captions.keys())}")
```

The above code will generate the below output —

```
train_image_captions[list(train_image_captions. Keys())[150]]
['<start> A brown dog chases a tattered ball around the yard . <end>',
 '<start> A brown dog is chasing a tattered soccer ball across a low cut field . <end>',
 '<start> Large brown dog playing with a white soccer ball in the grass . <end>',
 '<start> Tan dog chasing a ball . <end>',
 '<start> The tan dog is chasing a ball . <end>']
```

Once we have loaded in the captions, we will first tokenize everything using spacy and Tokenizer (from _tensorflow.preprocessing.text_ class).

Tokenization is nothing but breaking down a sentence into different words while removing special characters, lowercasing everything. Result is we have a corpus of meaningful words (tokens) in the sentence that we can further encode before using it as an input to our model.

```python
import spacy
nlp = spacy.load('en', disable=['tagger', 'parser', 'ner'])

# tokenize evry captions, remove punctuations, lowercase everything
for key, value in train_image_captions.items():
    ls = []
    for v in value:
        doc = nlp(v)
        new_v = " "
        for token in doc:
            if not token.is_punct:
                if token.text not in [" ", "\n", "\n\n"]:
                    new_v = new_v + " " + token.text.lower()

        new_v = new_v.strip()
        ls.append(new_v)
    train_image_captions[key] = ls


# create a vocabulary of all the unique words present in captions
# flatten the list
all_captions = [caption for list_of_captions in all_captions for caption in list_of_captions]

# use spacy to convert to lowercase and reject any special characters
tokens = []
for captions in all_captions:
    doc = nlp(captions)
    for token in doc:
        if not token.is_punct:
            if token.text not in [" ", "\n", "\n\n"]:
                tokens.append(token.text.lower())

# get tokens with frequency less than 10
import collections
word_count_dict = collections.Counter(tokens)
reject_words = []
for key, value in word_count_dict.items():
    if value < 10:
        reject_words.append(key)

reject_words.append("<")
reject_words.append(">")

 # remove tokens that are in reject words
tokens = [x for x in tokens if x not in reject_words]

# convert the token to equivalent index using Tokenizer class of Keras
from keras.preprocessing.text import Tokenizer
tokenizer = Tokenizer()
tokenizer.fit_on_texts(tokens)
```

The above code will generate a dictionary with every token encoded as an integer and vice versa. The sample output looks like below —

```
tokenizer.word_index
{'a': 1,
 'end': 2,
 'start': 3,
 'in': 4,
 'the': 5,
 'on': 6,
 'is': 7,
 'and': 8,
 'dog': 9,
 'with': 10,
 'man': 11,
 'of': 12,
 'two': 13,
 'black': 14,
 'white': 15,
 'boy': 16,
 'woman': 17,
 'girl': 18,
 'wearing': 19,
 'are': 20,
 'brown': 21.....}
```

After this, we need to find the **length of our vocabulary and length of longest caption. Let's see the significance of both of these measures in creating our model.**

1. **Vocabulary Length** → Length of Vocabulary is basically the count of unique words in our corpus. Also, the neurons in our output layers would be equal to **Vocabulary Length + 1** (+ 1 is for extra whitespace due to padding sequence) since at every iteration we need our model to generate a new word from our corpus of words.

2. **Maximum Caption Length** → Since in our dataset, we have captions of varying length even for the same image. Let's try to understand this in a bit detail-

![](https://miro.medium.com/max/700/1*i19g_Cl5k03OT-iKMwZmxA.png)

As you can see every caption is of different length and hence we cannot use these as an input to our LSTM model. For fixing this problem, we fill pad every caption to the length of maximum caption.

![](https://miro.medium.com/max/603/1*Lv4dkrjtVMw-jXsRBfTdnQ.png)

Notice every sequence has a set of extra 0s to increase its length to the maximum sequence.

```python
# compute length of vocabulary and maximum length of a caption (for padding)
vocab_len = len(tokenizer.word_counts) + 1
print(f"Vocabulary length - {vocab_len}")

max_caption_len = max([len(x.split(" ")) for x in all_captions])
print(f"Maximum length of caption - {max_caption_len}")
```

Next we need to create training dataset for our model specifying the input and output. For our problem we have 2 inputs and 1 output. Let's see this in a bit detail for understanding-

![](https://miro.medium.com/max/700/1*MhI3macM9r6x6xKlbosppQ.png)

For every image we have-

1. **Input Image Feature (X1)** → Numpy array of shape (18432, ) extracted using ResNet50 Model

2. **Input Sequence (X2)** → This requires a bit more explanation. Every caption is nothing but a list of sequence and our model is trying to predict to next best element in the sequence. Hence for every caption, we will first start of with the first element in the sequence and the corresponding output to that element would be the very next element. In the next iteration, the output of previous iteration would become the new input along with the input of previous iteration (the memory) and this goes on until we have reached the end of sequence.

3. **Output (y)** → The next word in the sequence.

The below code can be used to implement the above logic for creating training dataset-

```python
from keras.preprocessing.sequence import pad_sequences
from keras.utils import to_categorical

# generator function to generate inputs for model
def create_trianing_data(captions, images, tokenizer, max_caption_length, vocab_len, photos_per_batch):

    X1, X2, y = list(), list(), list()
    n=0

    # loop through every image
    while 1:
        for key, cap in captions.items():
            n+=1
            # retrieve the photo feature
            image = images[key]

            for c in cap:
                # encode the sequence
                sequnece = [tokenizer.word_index[word] for word in c.split(' ') if word in list(tokenizer.word_index.keys())]

                # split one sequence into multiple X, y pairs

                for i in range(1, len(sequence)):
                    # creating input, output
                    inp, out = sequence[:i], sequence[i]
                    # padding input
                    input_seq = pad_sequences([inp], maxlen=max_caption_length)[0]
                    # encode output sequence
                    output_seq = to_categorical([out], num_classes=vocab_len)[0]
                    # store
                    X1.append(image)
                    X2.append(input_seq)
                    y.append(output_seq)

            # yield the batch data
            if n==photos_per_batch:
                yield ([np.array(X1), np.array(X2)], np.array(y))
                X1, X2, y = list(), list(), list()
                n=0
```

## Combining the two Sub Networks

Now that we have developed our two sub networks (Image Feature Extractor & LSTM for generating captions), lets combine these two to create our final model.

![](https://miro.medium.com/max/700/1*9t0uK6vJ28m9LYL4DyqGkQ.png)

For any new image (must be similar to ones used during training) our model will generate the caption based on the knowledge it has acquired while training over similar set of images and captions.

> _Note: For good results, the testing images should be similar to the ones used during training._

The following code creates our final model

```python
import keras

def create_model(max_caption_length, vocab_length):

    # sub network for handling the image feature part
    input_layer1 = keras.Input(shape=(18432))
    feature1 = keras.layers.Dropout(0.2)(input_layer1)
    feature2 = keras.layers.Dense(max_caption_length*4, activation='relu')(feature1)
    feature3 = keras.layers.Dense(max_caption_length*4, activation='relu')(feature2)
    feature4 = keras.layers.Dense(max_caption_length*4, activation='relu')(feature3)
    feature5 = keras.layers.Dense(max_caption_length*4, activation='relu')(feature4)

    # sub network for handling the text generation part
    input_layer2 = keras.Input(shape=(max_caption_length,))
    cap_layer1 = keras.layers.Embedding(vocab_length, 300, input_length=max_caption_length)(input_layer2)
    cap_layer2 = keras.layers.Dropout(0.2)(cap_layer1)
    cap_layer3 = keras.layers.LSTM(max_caption_length*4, activation='relu', return_sequences=True)(cap_layer2)
    cap_layer4 = keras.layers.LSTM(max_caption_length*4, activation='relu', return_sequences=True)(cap_layer3)
    cap_layer5 = keras.layers.LSTM(max_caption_length*4, activation='relu', return_sequences=True)(cap_layer4)
    cap_layer6 = keras.layers.LSTM(max_caption_length*4, activation='relu')(cap_layer5)

    # merging the two sub network
    decoder1 = keras.layers.merge.add([feature5, cap_layer6])
    decoder2 = keras.layers.Dense(256, activation='relu')(decoder1)
    decoder3 = keras.layers.Dense(256, activation='relu')(decoder2)

    # output is the next word in sequence
    output_layer = keras.layers.Dense(vocab_length, activation='softmax')(decoder3)
    model = keras.models.Model(inputs=[input_layer1, input_layer2], outputs=output_layer)

    model.summary()

    return model
```

Before compiling the model, we need to add weights to our embedding layer. This is done by creating word embeddings (representation of token in a high dimensional vector space) for every token present in our corpus (vocabulary). There are some very popular word embedding models that can be used for this purpose (GloVe, Gensim Word Embedding Models etc.).

We will be using Spacy’s inbuilt “en_core_web_lg” model for creating vector representation of our token (i.e. every token would be represented as a (300,) numpy array).

The following code can be used to create word embeddings and add it to our model embedding layer.

```python
# create word embeddings
import spacy
nlp = spacy.load('en_core_web_lg')

# create word embeddings
embedding_dimension = 300
embedding_matrix = np.zeros((vocab_len, embedding_dimension))

# travel through every word in vocabulary and get its corresponding vector
for word, index in tokenizer.word_index.items():
    doc = nlp(word)
    embedding_vector = np.array(doc.vector)
    embedding_matrix[index] = embedding_vector

# adding embeddings to model
predictive_model.layers[2]
predictive_model.layers[2].set_weights([embedding_matrix])
predictive_model.layers[2].trainable = False
```

Now that we have created everything, we just have to compile and train our model.

> **_Note : Training time of this network would be very high (with high number of epochs) due to complex nature of our task_**

```python
# get training data
train_data = create_trianing_data(train_image_captions, train_image_features, tokenizer, max_caption_len, vocab_length, 32)

# initialize model
model = create_model(max_caption_len, vocab_len)

steps_per_epochs = len(train_image_captions)//32

# compile model
model.compile(optimizer='adam', loss='categorical_crossentropy')
model.fit_generator(train_data, epochs=100, steps_per_epoch=steps_per_epochs)
```

For generating new captions, we first need to convert an image into a numpy array of same dimension as images of training dataset (18432,) and using `<start>` as the input to model .

During sequence generation, we would terminate the process as soon as we have encountered a `<end>` in our output.

```python
import matplotlib.pyplot as plt
import seaborn as sns
from PIL import Image
%matplotlib inline

# method for generating captions
def generate_captions(model, image, tokenizer.word_index, max_caption_length, tokenizer.index_word):

    # input is <start>
    input_text = '<start>'

    # keep generating words till we have encountered <end>
    for i in range(max_caption_length):
        seq = [tokenizer.word_index[w] for w in in_text.split() if w in list(tokenizer.word_index.keys())]
        seq = pad_sequences([sequence], maxlen=max_caption_length)
        prediction = model.predict([photo,sequence], verbose=0)
        prediction = np.argmax(prediction)
        word = tokenizer.index_word[prediction]
        input_text += ' ' + word
        if word == '<end>':
            break

    # remove <start> and <end> from output and return string
    output = in_text.split()
    output = output[1:-1]
    output = ' '.join(output)
    return output

# traverse through testing images to generate captions
count = 0
for key, value in test_image_features.items():
    test_image = test_image_features[key]
    test_image = np.expand_dims(test_image, axis=0)
    final_caption = generate_captions(predictive_model, test_image, tokenizer.word_index, max_caption_len, tokenizer.index_word)

    plt.figure(figsize=(7,7))
    image = Image.open(image_path + "//" + key + ".jpg")
    plt.imshow(image)
    plt.title(final_caption)

    count = count + 1
    if count == 3:
        break
```

Now lets check the output of our model

![](https://miro.medium.com/max/587/1*ulg85KwidtQb8WSp8BLmuA.png)

![](https://miro.medium.com/max/592/1*OoLsEIIWzXJmH7qEsOGgNg.png)

![](https://miro.medium.com/max/584/1*9KUnZcGwlbv2_X7YvbiNkA.png)

![](https://miro.medium.com/max/590/1*tpaX1wqUO0BxnidQwnAZrQ.png)

![](https://miro.medium.com/max/595/1*jSahQnd1oWWO6toKcEvYtw.png)

As you can see our model generates good enough captions for some images but for some the captions are not explanatory.

This can be improved by increasing the epochs, training data, adding layers to our final model but all this requires high end machines (GPUs) for processing.

So this is how we can generate captions for images using our own deep learning model.
