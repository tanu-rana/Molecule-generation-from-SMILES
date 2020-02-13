## Molecule-generation-from-SMILES

This is an attempt to use a recurrent neural network (with LSTM cells) to learn the patterns in a dataset of 200k molecules (SMILES pattern), and consequentially generate, molecules in the form of SMILES strings (SMILES is a string representation of a molecule, based on its structure and components.). I'm describing the process briefly below.

Step 1 - MAPPING
The first step, as is in most language processing RNNs, is to create a mapping of characters to integers (to allow the neural network to process the data) and vice versa (to translate results back to characters).The easiest way to do this is by creating a set of unique characters and enumerate each item. In a natural language like English, there are 26 letters (double that for capital letters), along with a whole host of grammatical symbols and syntax characters. In SMILES strings, there are two types of characters: Special characters: “/”, “(“, “)”, “=”, etc. ; Elemental symbols: “C”, “O”, “Si”, “Co”, etc.

This list of unique characters is enumerated and conveniently placed into a dictionary.
It’s worth noting that the dictionary does not consider elemental symbols consisting of 2 characters as a single element. For example, the “S” and “i” of the elemental symbol for Silicon counts as 2 independent characters. This means that the model will have to learn the difference between “Si” and “S”, which are both entirely different elements. It’s totally possible to hard code these 2-character symbols into the dictionary, but just for fun, I left them separate to see how well the model would perform.

Step 2 — Data preprocessing:
Once we have our unique character mapping, we can go about translating every character in the SMILES string dataset into integers. A simple call of the dictionary we built in Step 1 should do the trick. At the same time, we can normalize all the integers by simply dividing each by the total number of unique characters in the dataset. Finally, reshape the resulting integerized and normalized dataset into a format that fits the neural network model.

Step 3 — Training:
I used categorical cross entropy as the loss function with an Adam optimizer (A mix of RMS-prop with ADAgrad and a built in momentum). To take advantage of as much of the dataset as possible, the model was given 19 epochs of 512 batch size to learn from. Generally, more epochs paired with smaller batch sizes allow the network to learn from the data much better, but at the cost of a much longer training time.

Step 4 — Generating:
Generating is relatively straight forward. First we import the checkpoints that we saved from training (so that we don’t have to retrain the model every time we want to generate a new molecule). The next step is to select a random SMILES string from the dataset as reference, and finally generate a specified number of characters.
Results will vary according to the choice of dataset size, learning rate, and other custom adjustments that can be made to the model, but ideally, you should get an output that resembles something like a valid molecule. If you plug in a checkpoint file with a lower accuracy, you could actually see how the neural network learns and gets better over time. Usually, it starts with a sequence of just once character:
CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
But it gets better over time to include alternating characters
C1C1C1C1C1C1C1C1C1C1C1C1C1C1C1C1C1C1C1C1C1C1C1C1C1C1C1
Eventually the output will be even more diverse as it learns substructures
C1C1C1C1C1C1((((((((((/////////cccccccccHcHcHcHcHcHcHcHc)))))))
The goal is to get is as close to
O1C=C[C@H]([C@H]1O2)c3c2cc(OC)c4c3OC(=O)C5=C4CCC(=O)5
as possible.

