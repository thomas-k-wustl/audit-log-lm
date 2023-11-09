# Autoregressive Language Models For Estimating the Entropy of Epic EHR Audit Logs

This repo contains code for training and evaluating transformer-based tabular language models for Epic EHR audit logs. 
You can use several of our pretrained models for entropy estimation or tabular generation, or train your own model 
from scratch. 

## Installation
Use `pip install -r requirements.txt` to install the required packages. If updated ` pipreqs . --savepath
requirements.txt --ignore Sophia` to update. Use `git submodule update --init --recursive` to get Sophia for training.

This project uses pre-commit hooks for `black` if you would like to contribute. To install run `pre-commit install`.

## Pretrained Model Usage

Our pretrained models can be downloaded from Hugging Face. Here's a rough example of how you can use them to get the cross-entropy loss:

```
from transformers import AutoTokenizer, AutoModelForCausalLM
from model import EHRAuditGPT2
from vocab import EHRVocab, EHRTokenizer
from data import timestamp_space_calculation

vocab = EHRVocab(vocab_path="vocab.pkl")
tk = EHRAuditTokenizer(vocab, timestamp_spaces_cal=timestamp_space_calculation([
                    config["timestamp_bins"]["spacing"],
                    config["timestamp_bins"]["min"],
                    config["timestamp_bins"]["max"],
                    config["timestamp_bins"]["bins"],
                ]))
model = EHRAuditGPT2.from_pretrained("bcwarner/[TBA]")

df = pd.read_csv("your_audit_log_data.csv")

input_ids = tk.encode(df)
output = model(input_ids, labels=input_ids, return_dict=True)
loss = output.loss
```

Since they're built with `transformers` you can also use these models for generative tasks in nearly the same way as any other language model. See `gen.py` for an example of how to do this.

## Training from Scratch

Our models were trained on ICU clinicians from a single hospital system. The data is not publicly available, but the code can be reused for any set of clinicians for the Epic EHR audit log system (and potentially other EHR systems).

### Training Usage
The training code assumes that clinicians are split up into different folders and each folder has a CSV file with each audit log event as a row, with an action column, patient id column, and event time column (defaults to METRIC_NAME, PAT_ID, and ACCESS_TIME + ACCESS_INSTANT).

Before the model can be used the configuration needs to be updated. A skeleton YAML file is provided in
[`config_template.yaml`](config_template.yaml) and should be copied to `config.yaml` and updated.

The vocab can be generated using `python vocab.py` inside the `model` folder given a list of METRIC_NAMEs and PAT_ID/ACCESS_TIME counts, and can be run once `config.yaml` is updated.

The model can be run using `python train.py`. The dataset will be cached for a ~25x reduction in the amount of data used which will enable it to load faster. If needed `--reset_cache` can be used to overwrite the old data.

The entropy over the test or validation set can be evaluated using `python entropy.py`. You'll be prompted with the list of models you've trained. Entropy as it relates to other measures can be evaluated using `python entropy.py --exp "exp1,exp2,...,expn"`. In addition, the entropy values for each row can be cached with `python entropy_cache.py` to save them for later analysis.

The generation capabilities can be evaluated using `python gen.py` which will run a selected generation metric for a given amount of samples.

The `model/modules.py` file contains the PyTorch Lightning data and model modules for the project. The `model/data.py` file contains the data loading and preprocessing code, and we assume clinician data is split up into different folders. 

###  Results evaluation

Some brief statistics were run using `analysis/dataset_characteristics.py`, we have not included them in the paper as they are beyond scope. 

Not all METRIC_NAMEs in the dataset were in our initial METRIC_NAME dictionary, so we made `analysis/missing_metric_names.py` to find them among the dataset. The output .xlsx can be used in vocab.py to add them to the METRIC_NAME dictionary.

Because our training was interrupted and restarted several times, we wrote `analysis/tb_join_plot.py` to join the Tensorboard output.


## Citation

Please cite our paper if you use this code in your own work:

```

```
