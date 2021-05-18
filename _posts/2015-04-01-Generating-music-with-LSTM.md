---
layout: post
title: Generating music with an LSTM algorithm
---
<!--<img src="/images/fulls/01.jpg" class="fit image">-->
This was the final project of the Growth Tribe course I finished in October 2020. I wanted to see if I could create  music ideas based on only music from the Beatles with an LSTM network.

The objective was to show that we were able to find and utilize some machine learning code with our own dataset and/or tweak the hyper parameters. We were not expected to build a neural network from ground up.

As a musician/composer my self I was interested in having an AI help me compose songs more efficiently. However, it seemed no one really was focusing on this in any articles I could find. The aim of all projects I could find was to try to make a finished peice of music that sounds good in it self and original, which we are still fairly far from achieving.

I wanted to make an AI that could make simple, but unique pieces of music on the most basic level on piano. Only chords and melody showcasing a verse and a chorus. That way it could be very useful for composers as an idea generation tool. Kinda of like having a friend playing one original idea after the other on the piano, and if you liked one of the ideas you could use that as a basis for a new song.

I found [this Medium article](https://towardsdatascience.com/how-to-generate-music-using-a-lstm-neural-network-in-keras-68786834d4c5) by Sigurður Skúli, who used an LSTM network to create original peices of music where the network was trained on a repertoire of midi music from the computer game Final Fantasy.

![png](/images/Music-with-LSTM/FF_example.png)
![](/images/Music-with-LSTM/Original.mp3)

My idea was to train the network on all time pop songs, which I recorded my self in my music program in a very specific wasy where the chords where in the lower notes and the melody was by it self in the higher notes. No rythm and breaking up the chords were added:

![png](/images/Music-with-LSTM/Pop_example.png)
<iframe width="100%" height="50" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/1050924739%3Fsecret_token%3Ds-WpmqBwRvkcW&color=%23ff5500&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe><div style="font-size: 10px; color: #cccccc;line-break: anywhere;word-break: normal;overflow: hidden;white-space: nowrap;text-overflow: ellipsis; font-family: Interstate,Lucida Grande,Lucida Sans Unicode,Lucida Sans,Garuda,Verdana,Tahoma,sans-serif;font-weight: 100;"><a href="https://soundcloud.com/kasperbirk" title="Kasper Birk" target="_blank" style="color: #cccccc; text-decoration: none;">Kasper Birk</a> · <a href="https://soundcloud.com/kasperbirk/basic/s-WpmqBwRvkcW" title="Basic" target="_blank" style="color: #cccccc; text-decoration: none;">Basic</a></div>

In it self this sounds very boring, but if original pieces of music in this format could be generated on command with an AI, it could be very useful for composing.


## Google Colab and TPU computing power

For running the algorithm I used Google colab to make use of Google's free TPU computing power, which is even faster than using a GPU.

## Training the LSTM network

Below is the code from the Medium article. I will breifly explain what each part does.
As yuo can see the network is built with keras.

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

The get_notes function basically prepares the midi data, so it has a format that can be used in the network.
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



Test with original repertoire:
![](/images/Music-with-LSTM/Original_test.mp3)


First try:
![](/images/Music-with-LSTM/First_try.mp3)


Best try:
![](/images/Music-with-LSTM/Best_try.mp3)


## Future work





[[embed url=http://www.youtube.com/watch?v=6YbBmqUnoQM]]