## P1B1: Autoencoder Compressed Representation for Gene Expression

### Model description:
P1B1, an autoencoder that can collapse high dimensional expression profiles into low dimensional vectors without much loss of information.
The proposed network architecture includes encoding layers, dropout layers, bottleneck layer, and decoding layers. At least three hidden layers are required, including one encoding layer, one bottleneck layer, and one decoding layer.
The model is trained and cross-validated on RNA-seq data from GDC. The sample size is 4,000 (3000 training + 1000 test).
The full set of expression features contains 60,483 float columns transformed from RNA-seq FPKM-UQ values.
It is useful for compressing the high dimensional expression profiles into low dimensional vectors.

### Description of the Data:
* Data source: RNA-seq data from GDC 
* Input dimensions: 60,484 floats; log(1+x) transformed FPKM-UQ values
* Output dimensions: Same as input
* Latent representation dimension: 1000
* Sample size: 4,000 (3000 training + 1000 test)
* Notes on data balance and other issues: unlabeled data draw from a diverse set of cancer types

### Setup:
To setup the python environment needed to train and run this model, first make sure you install [conda](https://docs.conda.io/en/latest/) package manager, clone this repository, then create the environment as shown below.

```bash
   conda env create -f environment.yml -n P1B1
   conda activate P1B1
   ```
### Running the baseline implementation:

The script [p1b1_baseline_keras2.py](p1b1_baseline_keras2.py) does the following:
* Reads the model configuration parameters from [p1b1_default_model.txt](p1b1_default_model.txt)
* Downloads the training data and splits it to training/validation sets
* Creates and trains the keras model
* Saves the best trained model based on the model performance on the validation dataset
* Evaluates the best model on the test dataset
* Creates the 2D latent representation of encoded data

```cd Pilot1/P1B1
   python p1b1_baseline_keras2.py --cp TRUE --save_path p1b1 --model vae
   ```
The baseline implementation supports three types of autoencoders controlled by the `--model` parameter: regular autoencoder (`ae`), variational autoencoder (`vae`), and conditional variational autoencoder (`cvae`).

#### Example output
```
Using TensorFlow backend.

Shape x_train: (2700, 60483)
Shape x_val:   (300, 60483)
Shape x_test:  (1000, 60483)
Range x_train: [0, 1]
Range x_val:   [0, 1]
Range x_test:  [0, 1]
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
input_1 (InputLayer)         (None, 60483)             0         
_________________________________________________________________
dense_1 (Dense)              (None, 2000)              120968000 
_________________________________________________________________
dense_2 (Dense)              (None, 600)               1200600   
_________________________________________________________________
dense_3 (Dense)              (None, 2)                 1202      
_________________________________________________________________
model_2 (Model)              (None, 60483)             122230283 
=================================================================
Total params: 244,400,085
Trainable params: 244,400,085
Non-trainable params: 0
_________________________________________________________________
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
input_3 (InputLayer)         (None, 2)                 0         
_________________________________________________________________
dense_4 (Dense)              (None, 600)               1800      
_________________________________________________________________
dense_5 (Dense)              (None, 2000)              1202000   
_________________________________________________________________
dense_6 (Dense)              (None, 60483)             121026483 
=================================================================
Total params: 122,230,283
Trainable params: 122,230,283
Non-trainable params: 0
_________________________________________________________________
None
Train on 2700 samples, validate on 300 samples

Epoch 99/100
 - 13s - loss: 0.0382 - xent: 0.4579 - corr: 0.7867 - val_loss: 0.0466 - val_xent: 0.4815 - val_corr: 0.7418
Current time ....1283.445
Epoch 100/100
 - 13s - loss: 0.0382 - xent: 0.4579 - corr: 0.7867 - val_loss: 0.0466 - val_xent: 0.4812 - val_corr: 0.7420
Current time ....1296.327
```

### Preliminary performance:

The current best performance in terms of validation correlation for the three types of autoencoders are as follows:

* AE: 0.74
* VAE: 0.77
* CVAE: 0.77

Here is a visual example of the 2D latent representation from VAE.

![VAE latent representation](https://github.com/Hokiee/NCI-DOE-Collab-Pilot1-Gene_Expression_Autoencoder/blob/master/Pilot1/P1B1/images/p1b1.keras.vae.D1%3D2000.D2%3D600.A%3Drelu.B%3D100.E%3D100.L%3D2.LR%3DNone.S%3Dminmax.latent.png)

### Inference: 

To test the trained model in inference, the script [p1b1_infer.py](p1b1_infer.py) does the following:
* Loads the trained model
* Downloads the processed test dataset
* Learns a representation for the test dataset
* Reports the performance of the model on the test dataset, including mse, r2_score, and correlation


```bash
   python p1b1_infer.py --model vae --model_name p1b1
   ```
#### Example output
```
Evaluation on test data: {'mse': 0.03219663, 'r2_score': 0.18068748776161042, 'correlation': 0.8252006683040468}
```
