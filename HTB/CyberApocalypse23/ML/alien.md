
## Groups
list(f['optimizer_weights']) //dset1
['Adam', 'iteration:0']

list(f['model_weights'])  //dset2
['conv2d_3', 'conv2d_4', 'conv2d_5', 'dense_2', 'dense_3', 'flatten_1', 'lambda_1', 'max_pooling2d_2', 'max_pooling2d_3', 'top_level_model_weights', 'uZDNyc3Q0bmR9']

## Attributes
### dset1
list(dset1.attrs)
['backend', 'keras_version', 'layer_names']

dset1.attrs['backend']
'tensorflow'
dset1.attrs['keras_version']
'2.11.0'
dset1.attrs['layer_names']
array(['conv2d_3', 'max_pooling2d_2', 'conv2d_4', 'max_pooling2d_3',
       'conv2d_5', 'uZDNyc3Q0bmR9', 'flatten_1', 'dense_2', 'dense_3',
       'lambda_1'], dtype=object)

### dset2
list(dset2.attrs)
['weight_names']

dset2.attrs['weight_names']
array(['iteration:0', 'Adam/m/conv2d_3/kernel:0',
       'Adam/v/conv2d_3/kernel:0', 'Adam/m/conv2d_3/bias:0',
       'Adam/v/conv2d_3/bias:0', 'Adam/m/conv2d_4/kernel:0',
       'Adam/v/conv2d_4/kernel:0', 'Adam/m/conv2d_4/bias:0',
       'Adam/v/conv2d_4/bias:0', 'Adam/m/conv2d_5/kernel:0',
       'Adam/v/conv2d_5/kernel:0', 'Adam/m/conv2d_5/bias:0',
       'Adam/v/conv2d_5/bias:0', 'Adam/m/dense_2/kernel:0',
       'Adam/v/dense_2/kernel:0', 'Adam/m/dense_2/bias:0',
       'Adam/v/dense_2/bias:0', 'Adam/m/dense_3/kernel:0',
       'Adam/v/dense_3/kernel:0', 'Adam/m/dense_3/bias:0',
       'Adam/v/dense_3/bias:0'], dtype=object)


The flag was hidden as a split up base64 encoded string. The first bit wa available using strings, and the ret through reading the file and putting them all togeter, before ismply decrypting in CyberChef.
