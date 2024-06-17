---
layout: post
title: Generating music with an RNN
---
<!--<img src="/images/fulls/01.jpg" class="fit image">-->
This was the final project of the Growth Tribe course I finished in October 2020. I wanted to see if I could get and LSTM network to create music ideas on piano with midi, based only on music from the Beatles.

## Introduction

The objective was to show that we were able to utilize some machine learning code with our own dataset and/or tweak the hyper parameters. We were not expected to build a neural network from ground up.

As a musician/composer my self, I was interested in having an AI help me compose songs more efficiently. However, it seemed no one was really focusing on this in any articles I could find. The aim of all projects I could find, was to try to make a finished piece of original music that sounded good in it self, which we are still fairly far from achieving.

I wanted to make an AI that could make simple, but unique pieces of music on the most basic level on piano. Only chords and melody showcasing a verse and a chorus. That way it could be very useful for composers as an idea generation tool. Kind of like having a friend playing one original idea after the other on the piano, and if you liked one of the ideas you could use that as a basis for a new song.

I found [this Medium article](https://towardsdatascience.com/how-to-generate-music-using-a-lstm-neural-network-in-keras-68786834d4c5) by Sigurður Skúli, who used an LSTM network to create original peices of music, where the network was trained on a repertoire of midi music from the computer game Final Fantasy.

Here is an example of a song in the training data set. In the picture you can see the notes are not seperated into chords and melody the same way as if it was a more standard pop song where the melody was supposed to be sung by a singer. It is a bit more like classical music where everything is mixed together.
![png](/images/Music-with-LSTM/FF_example.png)

It sounds like this:
<iframe width="500" height="100" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/1050924700%3Fsecret_token%3Ds-dqCJ0fHCNMN&color=%23333333&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe><div style="font-size: 10px; color: #cccccc;line-break: anywhere;word-break: normal;overflow: hidden;white-space: nowrap;text-overflow: ellipsis; font-family: Interstate,Lucida Grande,Lucida Sans Unicode,Lucida Sans,Garuda,Verdana,Tahoma,sans-serif;font-weight: 100;"><a href="https://soundcloud.com/kasperbirk" title="Kasper Birk" target="_blank" style="color: #cccccc; text-decoration: none;">Kasper Birk</a> · <a href="https://soundcloud.com/kasperbirk/original/s-dqCJ0fHCNMN" title="Original" target="_blank" style="color: #cccccc; text-decoration: none;">Original</a></div>

My idea was to train the network only on Beatles songs, which I would record my self in my music program in a very specific way where the chords were in the lower notes and the melody was by it self in the higher notes. No rythm and breaking up the chords were added.

The midi files all look more like this example which is the song Imagine:
![png](/images/Music-with-LSTM/Pop_example.png)

You can clearly see the melody is seperated from the chords.

It sounds like this:
<iframe width="500" height="100" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/1050924739%3Fsecret_token%3Ds-WpmqBwRvkcW&color=%23ff5500&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe><div style="font-size: 10px; color: #cccccc;line-break: anywhere;word-break: normal;overflow: hidden;white-space: nowrap;text-overflow: ellipsis; font-family: Interstate,Lucida Grande,Lucida Sans Unicode,Lucida Sans,Garuda,Verdana,Tahoma,sans-serif;font-weight: 100;"><a href="https://soundcloud.com/kasperbirk" title="Kasper Birk" target="_blank" style="color: #cccccc; text-decoration: none;">Kasper Birk</a> · <a href="https://soundcloud.com/kasperbirk/basic/s-WpmqBwRvkcW" title="Basic" target="_blank" style="color: #cccccc; text-decoration: none;">Basic</a></div>

In it self this sounds very boring, but if multiple original pieces of music like this could be generated in a few minutes, it could be very useful for composing.
That way the composer could use it to quickly get ideas for new compositions. Perhaps by trying to play the chords him self on the piano or guitar and sing the melody with some lyrics.

## Google Colab and TPU computing power

For running the algorithm, I used Google colab to make use of Google's free TPU computing power, which is even faster than using a GPU.

## Training the LSTM network

Below is the code from the Medium article. I will breifly explain what each part does.
As you can see the network is built with keras/tensorflow.

```python
import glob
import pickle
import numpy
from music21 import converter, instrument, note, chord
from keras.models import Sequential
from keras.layers import Dense
from keras.layers import Dropout
from keras.layers import LSTM
from keras.layers import Activation
from keras.layers import BatchNormalization as BatchNorm
from keras.utils import np_utils
from keras.callbacks import ModelCheckpoint
```

The first function simply collects and runs all of the different functions together.
You can see there are 4 parts: Getting the data (notes) from the midi files, prepare the sequences, create the network and train the network.


```python
def train_network():
    """ Train a Neural Network to generate music """
    notes = get_notes()
    # get amount of pitch names
    n_vocab = len(set(notes))

    network_input, network_output = prepare_sequences(notes, n_vocab)

    model = create_network(network_input, n_vocab)

    train(model, network_input, network_output)
```

The get_notes function basically arranges the midi data into an array so we can easier work with it.
A special library (music21) is used to interpret the midi files and extract the notes. Midi files contain much more information like velocity of the notes, duration, sustain, and more. But only the pitch of the notes are extracted in this example. If more of these features were extracted better music could probably be generated.

```python
def get_notes():
    """ Get all the notes and chords from the midi files in the ./midi_songs directory """
    notes = []

    for file in glob.glob("/content/drive/My Drive/lstm_music_generator/midi_songs_whole2/*.mid"):
        midi = converter.parse(file)

        print("Parsing %s" % file)

        notes_to_parse = None

        try: # file has instrument parts
            s2 = instrument.partitionByInstrument(midi)
            notes_to_parse = s2.parts[0].recurse() 
        except: # file has notes in a flat structure
            notes_to_parse = midi.flat.notes

        for element in notes_to_parse:
            if isinstance(element, note.Note):
                notes.append(str(element.pitch))
            elif isinstance(element, chord.Chord):
                notes.append('.'.join(str(n) for n in element.normalOrder))

    with open('/content/drive/My Drive/lstm_music_generator/data/notes', 'wb') as filepath:
        pickle.dump(notes, filepath)

    return notes
```

Next sequences of the music is prepared with a certain length (sequence_length). This determines how far back the LSTM network will look to try and predict the next note to play.

```python
def prepare_sequences(notes, n_vocab):
    """ Prepare the sequences used by the Neural Network """
    sequence_length = 80

    # get all pitch names
    pitchnames = sorted(set(item for item in notes))

    # create a dictionary to map pitches to integers
    note_to_int = dict((note, number) for number, note in enumerate(pitchnames))

    network_input = []
    network_output = []

    # create input sequences and the corresponding outputs
    for i in range(0, len(notes) - sequence_length, 1):
        sequence_in = notes[i:i + sequence_length]
        sequence_out = notes[i + sequence_length]
        network_input.append([note_to_int[char] for char in sequence_in])
        network_output.append(note_to_int[sequence_out])

    n_patterns = len(network_input)

    # reshape the input into a format compatible with LSTM layers
    network_input = numpy.reshape(network_input, (n_patterns, sequence_length, 1))
    # normalize input
    network_input = network_input / float(n_vocab)

    network_output = np_utils.to_categorical(network_output)

    return (network_input, network_output)
```

In create_network the model is simply created and returned in a variable.
We can se the model is sequential and has 3 LSTM layers with an output dimentionality of 512 units.
Afterwards there are 2 sets of batch normalization, dropout, dense and activation layers. The batch nomrmalization and dropout layers aim to speed up the learning and add some regularization to prevent over fitting. The dense layers are neural networks with 256 neurons using relu activation function in the first one and n_vocab neurons in the last one using softmax.
n_vocab is the number of unique notes in the training set, which makes sense to have this amount to choose from when the network attempts to predict the next note.

```python
def create_network(network_input, n_vocab):
    """ create the structure of the neural network """
    model = Sequential()
    model.add(LSTM(
        512,
        input_shape=(network_input.shape[1], network_input.shape[2]),
        recurrent_dropout=0.3,
        return_sequences=True
    ))
    model.add(LSTM(512, return_sequences=True, recurrent_dropout=0.3,))
    model.add(LSTM(512))
    model.add(BatchNorm())
    model.add(Dropout(0.3))
    model.add(Dense(256))
    model.add(Activation('relu'))
    model.add(BatchNorm())
    model.add(Dropout(0.3))
    model.add(Dense(n_vocab))
    model.add(Activation('softmax'))
    model.compile(loss='categorical_crossentropy', optimizer='rmsprop')

    return model
```

The train function below starts the actual training with a certain number of epochs and batch size that can be adjusted.
Note that this function generates a file with weights after each epoch. This can be used to generate music with afterwards when using the trained network.


```python
def train(model, network_input, network_output):
    """ train the neural network """
    filepath = "/content/drive/My Drive/lstm_music_generator/weights/weights-improvement-{epoch:02d}-{loss:.4f}-bigger.hdf5"
    checkpoint = ModelCheckpoint(
        filepath,
        monitor='loss',
        verbose=0,
        save_best_only=True,
        mode='min'
    )
    callbacks_list = [checkpoint]

    model.fit(network_input, network_output, epochs=200, batch_size=256, callbacks=callbacks_list)

if __name__ == '__main__':
    train_network()
```

## Generating music with the trained network

My apporach was to get the code working and generate a song using the training data the author has been using from Final Fantasy. The I wanted to record a number of Beatles songs myself in the special basic format I wanted on the piano, to use as the training data. Then I wanted to see if I was able to generate an original composition that sounded a bit like a Beatles song.
It turned out to be quite time consuming to record all these songs, but I was able to record around 30 which was ok the prove the concept and to show that I was able to understand how to navigate and use the code.

The first audio clip is just the result from training the network and generating a piano peice on the same data as the author used.

Test with original training data:
<iframe width="500" height="100" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/1050924694%3Fsecret_token%3Ds-J4cg2pvZZyR&color=%23333333&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe><div style="font-size: 10px; color: #cccccc;line-break: anywhere;word-break: normal;overflow: hidden;white-space: nowrap;text-overflow: ellipsis; font-family: Interstate,Lucida Grande,Lucida Sans Unicode,Lucida Sans,Garuda,Verdana,Tahoma,sans-serif;font-weight: 100;"><a href="https://soundcloud.com/kasperbirk" title="Kasper Birk" target="_blank" style="color: #cccccc; text-decoration: none;">Kasper Birk</a> · <a href="https://soundcloud.com/kasperbirk/original-test/s-J4cg2pvZZyR" title="Original Test" target="_blank" style="color: #cccccc; text-decoration: none;">Original Test</a></div>

When using my own data I had some trouble getting the network to train properly due to the smaller data set. As you can hear the first result did not amount to much.

First try:
<iframe width="500" height="100" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/1050924712%3Fsecret_token%3Ds-9wgSNAXuuoC&color=%23333333&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe><div style="font-size: 10px; color: #cccccc;line-break: anywhere;word-break: normal;overflow: hidden;white-space: nowrap;text-overflow: ellipsis; font-family: Interstate,Lucida Grande,Lucida Sans Unicode,Lucida Sans,Garuda,Verdana,Tahoma,sans-serif;font-weight: 100;"><a href="https://soundcloud.com/kasperbirk" title="Kasper Birk" target="_blank" style="color: #cccccc; text-decoration: none;">Kasper Birk</a> · <a href="https://soundcloud.com/kasperbirk/first-try/s-9wgSNAXuuoC" title="First Try" target="_blank" style="color: #cccccc; text-decoration: none;">First Try</a></div>

After fiddeling with the hyper parameters especially sequence length and batch size I was able to generate the piece below.

Best try:
<iframe width="500" height="100" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/1050924721%3Fsecret_token%3Ds-wDE93NO2Cfk&color=%23333333&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe><div style="font-size: 10px; color: #cccccc;line-break: anywhere;word-break: normal;overflow: hidden;white-space: nowrap;text-overflow: ellipsis; font-family: Interstate,Lucida Grande,Lucida Sans Unicode,Lucida Sans,Garuda,Verdana,Tahoma,sans-serif;font-weight: 100;"><a href="https://soundcloud.com/kasperbirk" title="Kasper Birk" target="_blank" style="color: #cccccc; text-decoration: none;">Kasper Birk</a> · <a href="https://soundcloud.com/kasperbirk/best-try/s-wDE93NO2Cfk" title="Best Try" target="_blank" style="color: #cccccc; text-decoration: none;">Best Try</a></div>

I was happy to get it working and to hear that my own data was being usied. However it is pretty obvious that the network is over fitting the data, and it sounds more or less like a copy of the Beatles song "Something".

## Future work

To test out my idea properly I would need way more data. My guess would be that I would need 3-500 songs recorded this way.

Also, as the author suggests him self it would be very interesting to try and extract more information from the midi files like duration and velocity of the notes and pauses.

Lastly, LSTM network has been out matched by transformer network in recent times. When the data set is comprehensive enough, it would be nice to try and upgrade the model to this new standard.