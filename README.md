# README

# TabNet : Attentive Interpretable Tabular Learning

This is a pyTorch implementation of Tabnet (Arik, S. O., & Pfister, T. (2019). TabNet: Attentive Interpretable Tabular Learning. arXiv preprint arXiv:1908.07442.) https://arxiv.org/pdf/1908.07442.pdf.

[![CircleCI](https://circleci.com/gh/dreamquark-ai/tabnet.svg?style=svg)](https://circleci.com/gh/dreamquark-ai/tabnet)

[![PyPI version](https://badge.fury.io/py/pytorch-tabnet.svg)](https://badge.fury.io/py/pytorch-tabnet)

![PyPI - Downloads](https://img.shields.io/pypi/dm/pytorch-tabnet)

Any questions ? Want to contribute ? To talk with us ? You can join us on [Slack](https://join.slack.com/t/mltooling/shared_invite/zt-fxaj0qk7-SWy2_~EWyhj4x9SD6gbRvg)

# Installation

## Easy installation
You can install using pip by running:
`pip install pytorch-tabnet`

## Source code
If you wan to use it locally within a docker container:

- `git clone git@github.com:dreamquark-ai/tabnet.git`

- `cd tabnet` to get inside the repository

-----------------
#### CPU only
- `make start` to build and get inside the container

#### GPU
- `make start-gpu` to build and get inside the GPU container

-----------------
- `poetry install` to install all the dependencies, including jupyter

- `make notebook` inside the same terminal. You can then follow the link to a jupyter notebook with tabnet installed.

# What problems does pytorch-tabnet handles?

- TabNetClassifier : binary classification and multi-class classification problems
- TabNetRegressor : simple and multi-task regression problems
- TabNetMultiTaskClassifier:  multi-task multi-classification problems

# How to use it?

TabNet is now scikit-compatible, training a TabNetClassifier or TabNetRegressor is really easy.

```
from pytorch_tabnet.tab_model import TabNetClassifier, TabNetRegressor

clf = TabNetClassifier()  #TabNetRegressor()
clf.fit(
  X_train, Y_train,
  eval_set=[(X_valid, y_valid)]
)
preds = clf.predict(X_test)
```

or for TabNetMultiTaskClassifier :

```
from pytorch_tabnet.multitask import TabNetMultiTaskClassifier
clf = TabNetMultiTaskClassifier()
clf.fit(
  X_train, Y_train,
  eval_set=[(X_valid, y_valid)]
)
preds = clf.predict(X_test)
```

### Custom early_stopping_metrics

```
from pytorch_tabnet.metrics import Metric
from sklearn.metrics import roc_auc_score

class Gini(Metric):
    def __init__(self):
        self._name = "gini"
        self._maximize = True

    def __call__(self, y_true, y_score):
        auc = roc_auc_score(y_true, y_score[:, 1])
        return max(2*auc - 1, 0.)

clf = TabNetClassifier()
clf.fit(
  X_train, Y_train,
  eval_set=[(X_valid, y_valid)],
  eval_metric=[Gini]
)

```

A specific customization example notebook is available here : https://github.com/dreamquark-ai/tabnet/blob/develop/customizing_example.ipynb

# Useful links

- explanatory video : https://youtu.be/ysBaZO8YmX8
- binary classification examples : https://github.com/dreamquark-ai/tabnet/blob/develop/census_example.ipynb
- multi-class classification examples : https://github.com/dreamquark-ai/tabnet/blob/develop/forest_example.ipynb
- regression examples : https://github.com/dreamquark-ai/tabnet/blob/develop/regression_example.ipynb
- multi-task regression examples : https://github.com/dreamquark-ai/tabnet/blob/develop/multi_regression_example.ipynb
- multi-task multi-class classification examples : https://www.kaggle.com/optimo/tabnetmultitaskclassifier

## Model parameters

- n_d : int (default=8)

    Width of the decision prediction layer. Bigger values gives more capacity to the model with the risk of overfitting.
    Values typically range from 8 to 64.

- n_a : int (default=8)

    Width of the attention embedding for each mask.
    According to the paper n_d=n_a is usually a good choice. (default=8)

- n_steps : int (default=3)

    Number of steps in the architecture (usually between 3 and 10)  

- gamma : float  (default=1.3)

    This is the coefficient for feature reusage in the masks.
    A value close to 1 will make mask selection least correlated between layers.
    Values range from 1.0 to 2.0.

- cat_idxs : list of int (default =[])

    List of categorical features indices.

- cat_emb_dim : list of int

    List of embeddings size for each categorical features. (default =1)

- n_independent : int  (default=2)

    Number of independent Gated Linear Units layers at each step.
    Usual values range from 1 to 5.

- n_shared : int (default=2)

    Number of shared Gated Linear Units at each step
    Usual values range from 1 to 5
- epsilon : float  (default 1e-15)

    Should be left untouched.

- seed : int (default=0)

    Random seed for reproducibility

- momentum : float

    Momentum for batch normalization, typically ranges from 0.01 to 0.4 (default=0.02)

- clip_value : float (default None)

    If a float is given this will clip the gradient at clip_value.
- lambda_sparse : float (default = 1e-3)

    This is the extra sparsity loss coefficient as proposed in the original paper. The bigger this coefficient is, the sparser your model will be in terms of feature selection. Depending on the difficulty of your problem, reducing this value could help.

- optimizer_fn : torch.optim (default=torch.optim.Adam)

    Pytorch optimizer function

- optimizer_params: dict (default=dict(lr=2e-2))

    Parameters compatible with optimizer_fn used initialize the optimizer. Since we have Adam as our default optimizer, we use this to define the initial learning rate used for training. As mentionned in the original paper, a large initial learning of ```0.02 ```  with decay is a good option.

- scheduler_fn : torch.optim.lr_scheduler (default=None)

    Pytorch Scheduler to change learning rates during training.

- scheduler_params : dict

    Dictionnary of parameters to apply to the scheduler_fn. Ex : {"gamma": 0.95, "step_size": 10}

- model_name : str (default = 'DreamQuarkTabNet')

    Name of the model used for saving in disk, you can customize this to easily retrieve and reuse your trained models.

- saving_path : str (default = './')

    Path defining where to save models.

- verbose : int (default=1)

    Verbosity for notebooks plots, set to 1 to see every epoch, 0 to get None.

- device_name : str (default='auto')
    'cpu' for cpu training, 'gpu' for gpu training, 'auto' to automatically detect gpu.

- mask_type: str (default='sparsemax')
    Either "sparsemax" or "entmax" : this is the masking function to use for selecting features

## Fit parameters

- X_train : np.array

    Training features

- y_train : np.array

    Training targets

- eval_set: list of tuple  

    List of eval tuple set (X, y).  
    The last one is used for early stopping  

- eval_name: list of str  
              List of eval set names.  

- eval_metric : list of str  
              List of evaluation metrics.  
              The last metric is used for early stopping.

- max_epochs : int (default = 200)

    Maximum number of epochs for trainng.
- patience : int (default = 15)

    Number of consecutive epochs without improvement before performing early stopping.

- weights : int or dict (default=0)

    /!\ Only for TabNetClassifier
    Sampling parameter
    0 : no sampling
    1 : automated sampling with inverse class occurences
    dict : keys are classes, values are weights for each class

- loss_fn : torch.loss or list of torch.loss

    Loss function for training (default to mse for regression and cross entropy for classification)
    When using TabNetMultiTaskClassifier you can set a list of same length as number of tasks,
    each task will be assigned its own loss function

- batch_size : int (default=1024)

    Number of examples per batch, large batch sizes are recommended.

- virtual_batch_size : int (default=128)

    Size of the mini batches used for "Ghost Batch Normalization"

- num_workers : int (default=0)

    Number or workers used in torch.utils.data.Dataloader

- drop_last : bool (default=False)

    Whether to drop last batch if not complete during training

- callbacks : list of callback function  
        List of custom callbacks
