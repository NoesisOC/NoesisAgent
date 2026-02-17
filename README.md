# Noesis

The Chain of Continuous Thought (NOESIS) paper that implements a novel latent reasoning paradigm. The main idea is to generate thoughts in latent space by utilizing the hidden states during prefilling before we start decoding response. 

Built on the public dataset from the paper for math.

[NOESIS gsm8k_synthetic_cot](https://huggingface.co/datasets/casperhansen/gsm8k_synthetic_cot)

The code relies on [wandb](https://wandb.ai/site/) for logging.

## Data

The data for training and evaluation is presented as a json file like this:

```python
[
  {
    "question": "...",
    "answer": "...",
    "steps": ["...", "...", ...]
  },
  ...
]
```

This file contains a list of data points. Each data point is composed of a question (str), an answer (str), and a list of steps (str), where each of them is a string.

## Arguments

The configuration of a run is specified in a yaml file (an example can be found [here](args/gsm_noesis.yaml)).

- **General settings**

  - **project**: Project name for wandb
  - **save_path**: path to store the checkpoints
  - **only_eval**: If true, only load a model and test on the data from `val_path` (must used along with `load_model_path`). Otherwise, train the model on `train_path` and test on `val_path` after every epoch.

- **Method**
  - **coconut**: Train Noesis model
  - **cot**: Train cot model
  - **no_thoughts**: Train Noesis (w/o thought) model
  - **no_cot**: Train no-cot model

- **Training settings**

  - **c_thought**: Number of continuous thoughts for each reasoning step
  - **epochs_per_stage**: Number of epochs for every training stage
  - **max_latent_stage**: The maximum number of training stages (in addition to the initial stage)
  - **pad_latent_to_max**: If the number of reasoning steps is fewer than the index of current training stage, pad the number of continuous thoughts.
  - **save_only_improve**: Save the model only when there the best validation accuracy is updated. Recommended to set `False` for Noesis model training, because otherwise the checkpoints in the last stage might now get saved.
  - **uniform_prob**: The probability to mix data from other stages. 0 for standard experiment, 0.3 for analysis experiment.
  - **model_id**: Huggingface model id to load as the initialization, e.g., `openai-community/gpt2`
  - **load_model_path**: The path to a checkpoint to load. Used in two cases: (1) for evaluation (2) to initialize Noesis from a CoT-tuned model.
  - **seed**: Random seed.
  - **resume**: The epoch to resume. Can be used when we want to skip the initial training stages.
  - **bf16**: Whether to use bf16 training.
  - **train_path**: Path to the training set.
  - **val_path**: Path to the validation or test set (depending on `only_eval`)
  - **reset_optimizer**: Whether to reset the optimizer when swtiching training stages.
  - **batch_size_training**: Batch size to train the model per GPU.
  - **debug**: If true, there is no wandb and model saving. A subset of data will be used.
  - **gradient_accumulation_steps**: Gradient accumulation steps
  - **num_epochs**: Maximum training epoches.
  - **lr**: Learning rate
  - **weight_decay**: Weight decay


## Training

Run the following commands (replacing `N_GPUS` and `PATH_TO_ARGS`):

```
torchrun --nnodes 1 --nproc_per_node N_GPUS run.py PATH_TO_ARGS
```


### GSM8K

Preprocessing data:

```bash
bash preprocessing/gsm_noesis.bash
```

Trained the model with CoT (as the stage 0 training)

```bash
torchrun --nnodes 1 --nproc_per_node 4 run.py args/gsm_noesis.yaml
```

Select a checkpoint as the initialization of Noesis (the validation accuracy is expected to be around 40%). Replace the `load_model_path` in the [args/gsm_noesis.yaml](args/gsm_noesis.yaml) with your selected checkpoint, and run:

```bash
torchrun --nnodes 1 --nproc_per_node 4 run.py args/gsm_noesis.yaml
```

The checkpoint with best validation accuracy, and put the path as `load_model_path` in [args/gsm_noesis_eval.yaml](args/gsm_noesis_eval.yaml). To evaluate:

```bash
torchrun --nnodes 1 --nproc_per_node 4 run.py args/gsm_noesis_eval.yaml
```

![noesis](https://media.discordapp.net/attachments/1174494134552240249/1410718791851180243/Add_a_heading_3.png?ex=68b209c1&is=68b0b841&hm=e9f575bc014719377f7a0846208b8fe83920a3d075067e18c49c8c67aac011de&=&format=webp&quality=lossless&width=1232&height=411)


git commit -m "Refactor usability tests.

Co-authored-by: pkxro <solponomarev@gmail.com>"