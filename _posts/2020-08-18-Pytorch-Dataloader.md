---
layout: post
title:  "Custom dataloading with pytorch"
date:   2020-08-18
categories: tutorial,python,pytorch
---

Design requirement: 
 - Not all members of the dataset are sampled per epoch.
 - We want the entire dataset to be sampled evenly across the different epochs.

Test description:
 - dataset has 4 elements
 - each batch has 1 element
 - we want to define number of batches per epoch = 2. 
 
```python
class Custom_Dataset():
    def __init__(self,data=['a','b','c','d']):
        super(torch.utils.data.Dataset)
        self.dat = data
    def __getitem__(self,idx):
        return self.dat[idx]
    def __len__(self):
        return len(self.dat)
    
#Instantiate an object of the class
dataset_obj = Custom_Dataset()
```

 - We will create a pytorch `Dataloader` object, which is an iterable.
 - `iter` function is defined for any iterable in python.
 - `iter(dataloader_obj)` creates an iterator, on which which `next()` can be applied to 'pop' elements.
 - `RandomSampler` object randomly generate valid indices over the dataset
 - when a new iterator is created for the same `dataloader_obj`, we want to make sure that the randomness is not reset. 

```python
 #This sampler allows the randomness to not to be reset for different iterators over the iterable 
random_sampler = torch.utils.data.RandomSampler(dataset_obj, replacement=False)

#Defined this way, a dataloader is an iterable
dataloader_obj = DataLoader(dataset, batch_size=1, shuffle=False, sampler=random_sampler)
```

We now test this scheme:
```python
n_batches_per_epoch = 2
n_epochs = 5

#Increment epoch count
for _ in range(n_epochs):
    dat_iterator = iter(dataloader_obj)
    #Increment epoch count
    for _ in range(n_batches_per_epoch):
        print(next(dat_iterator))
    print('----')
```
The output:

```python
['c']
['a']
----
['b']
['c']
----
['a']
['b']
----
['d']
['a']
----
['b']
['d']
----
```

We see that across epochs, the elements picked from the dataset are not the same and thus satisfies the design requirement. 