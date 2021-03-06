## AudioDataset

A Pytorch dataset class for raw audio data. Overrides torch.utils.data.Dataset.  
Loads audio files from a specified directory or a csv file containing file paths, together with the corresponding parameter file.

Originally written for an audio generative task: predicting the sample at time _t+1_ given a sample at time _t_.

**Directory structure**  
The dataloader works with two types of popular directory structures for data:
1. Each class in separate folder under root data directory  
(Note the dataloader does not take the class label from the folder name nor the directory structure into consideration. To use class labels e.g. as a target for classification, grab it from a param file instead. Therefore, please prepare param files beforehand for best use of the AudioDataset class - see Parameters section below) 
2. All data files in root data directory

The dataloader does not between distinguish the above two directory structures.

**_getitem_ method**  
The dataloader ___getitem___ will iterate through the dataset. It returns each data sequence as a pytorch tensor consisting of the audio data and the various parameters in the order (sequence,features).  
e.g. for a data sequence 5 samples long, every row is a timestep, columns are (audio samples,param1,param2,param3):
```bash
input = 
tensor([[  0.5020,   0.1437,   0.0000,  70.0000],
        [  0.5020,   0.1437,   0.0000,  70.0000],
        [  0.5020,   0.1437,   0.0000,  70.0000],
        [  0.5059,   0.1437,   0.0000,  70.0000],
        [  0.5098,   0.1437,   0.0000,  70.0000]])
```
The target sequence unaltered will be the input audio sequence above shifted 1 sample down. It has a single value per timestep which are encoded in mu-law:
```bash
target = 
tensor([[ 128],
        [ 128],
        [ 129],
        [ 130],
        [ 130]])
```		
**Parameters**  
The philosophy behind this dataloader is that everything extra to the audio data itself, e.g. class labels, audio features, additional meta-data etc. should go inside a parameter file. Each audio clip has a corresponding parameter file that can be loaded if specified, including the specific parameters that are needed using the AudioDataset parameter 'prop' and the parameter dicionary key. 

The parameters files consist of json objects structed as nested python dicts. They are saved, loaded and modified via the [paramManager library](https://github.com/lonce/paramManager). 
To learn how to use ParamManager, try out the instructional ProcessFiles.ipynb in the paramManager repo. Also see an example of an actual param file in this [repo](https://github.com/muhdhuz/audioDataloader/blob/master/dataparam/brass_acoustic_018-070-127.params). 

**Running the AudioDataset class**
```bash
from transforms import mulawnEncode,mulaw,array2tensor,dic2tensor	

sr = 16000
seqLen = 5
stride = 1

adataset = AudioDataset(sr,seqLen,stride,
		datadir="dataset",extension=".wav",
		paramdir="dataparam",prop=['rmse','instID','midiPitch'],  #parameters used for training can be specified here
		transform=transform.Compose([mulawnEncode(256,0,1),array2tensor(torch.FloatTensor)]),
		param_transform=dic2tensor(torch.FloatTensor),
		target_transform=transform.Compose([mulaw(256),array2tensor(torch.LongTensor)]))

for i in range(len(adataset)): #to see what the input looks like
    inp,target = adataset[i]
    print(inp)
    print(target)
    
    if i == 2:
        break 
```
**Dependencies**  
* pytorch 0.4.0+ 
* PySoundFile 0.9.0  

**Authors**  
* Muhammad Huzaifah






