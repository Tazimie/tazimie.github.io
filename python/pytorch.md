# pytorch

## torch multiply operation

> torch.Tensor有4种常见的乘法：*, torch.mul, torch.mm, torch.matmul. 

- **二维矩阵乘法 torch.mm()**

```python
torch.mm(mat1, mat2, out=None) # 不支持广播
```

- **三维带batch的矩阵乘法 torch.bmm()**

```python
torch.bmm(bmat1, bmat2, out=None) # 不支持广播
```

- **多维矩阵乘法 torch.matmul()**

```python
torch.matmul(input, other, out=None) #支持广播
```

- **矩阵逐元素(Element-wise)乘法 torch.mul()**

```python
torch.mul(mat1, other, out=None) # 支持广播 
```
参考
https://zhuanlan.zhihu.com/p/146707216

https://pytorch.org/docs/stable/torch.html





`masked_fill` fills elements of self tensor with value where mask is True

`masked_scatter` copy elements from source into self tensor at positions where the mask is True

`masked_select` Returns a new 1-D tensor which indexes the `input` tensor according to the boolean mask `mask` which is a BoolTensor.



## 为什么加载预训练模型是依然需要init_weights

`self.__init_weights`

`self.init_weights`

Have a look at the code for [`.from_pretrained()`](https://github.com/huggingface/transformers/blob/a9aa7456ac824c9027385b149f405e4f5649273f/src/transformers/modeling_utils.py#L490). What actually happens is something like this:

- find the correct base model class to initialise
- initialise that class with pseudo-random initialisation (by using the `_init_weights` function that you mention)
- find the file with the pretrained weights
- overwrite the weights of the model that we just created with the pretrained weightswhere applicable

This ensure that layers were not pretrained (e.g. in some cases the final classification layer) *do* get initialised in `_init_weights` but don't get overridden.

> Example __init__weights

```python
    def _init_weights(self, module):
        """Initialize the weights"""
        if isinstance(module, nn.Linear):
            # Slightly different from the TF version which uses truncated_normal for initialization
            # cf https://github.com/pytorch/pytorch/pull/5617
            module.weight.data.normal_(mean=0.0, std=self.config.initializer_range)
            if module.bias is not None:
                module.bias.data.zero_()
        elif isinstance(module, nn.Embedding):
            module.weight.data.normal_(mean=0.0, std=self.config.initializer_range)
            if module.padding_idx is not None:
                module.weight.data[module.padding_idx].zero_()
        elif isinstance(module, nn.LayerNorm):
            module.bias.data.zero_()
            module.weight.data.fill_(1.0)
```



`_keys_to_ignore_on_load_missing`

## dynamic padding

```python
from transformers import DataCollatorWithPadding
data_collator = DataCollatorWithPadding(tokenizer=tokenizer)
```

注意，我们需要使用tokenizer来初始化这个`DataCollatorWithPadding`，因为需要tokenizer来告知具体的padding token是啥，以及padding的方式是在左边还是右边（不同的预训练模型，使用的padding token以及方式可能不同）。

```python
batch = data_collator(samples)  # samples中必须包含 input_ids 字段，因为这就是collator要处理的对象
batch.keys()
# >>> dict_keys(['attention_mask', 'input_ids', 'token_type_ids', 'labels'])

# 再打印长度：
[len(x) for x in batch['input_ids']]
tokenizer = BertTokenizer.from_pretrained('bert-base-chinese')
data_collator = DataCollatorWithPadding(tokenizer=tokenizer, padding='longest')
raw_datasets = load_dataset('json', data_files={"test": "./fine_tuning_train.json"})
samples = raw_datasets['test'][:5]
batch = data_collator(samples)  # samples中必须包含 input_ids 字段，因为这就是collator要处理的对象
batch.keys()
```

## split train and test

```python
load_dataset('csv', data_files=config.train_data_path).rename_column('review', 'text').shuffle(seed)['train'].train_test_split(test_size=0.2)
```

## tokenizer

```python
raw_datasets.map(
    lambda examples: tokenizer(examples['text'], truncation=True, padding="max_length", max_length=128))

inputs = tokenizer("一个小小的测试 这是第二句", "这是第二句", return_tensors="pt",truncation=True)
```



## Dataset

```python
class TestDataset(Dataset):
    def __init__(self):
        self.data = [
        ]

    def __len__(self):
        return len(self.data)

    def __getitem__(self, idx):
        return self.data[idx]
```

## BertForPreTraining相关

- **Masked Language Model（MLM）**：在句子中随机用`[MASK]`替换一部分单词，然后将句子传入 BERT 中得到每一个单词的embedding，最终用`[MASK]`的embedding预测该位置的正确单词，这一任务旨在训练模型**根据上下文理解单词的意思**；
- **Next Sentence Prediction（NSP）**：将句子对 A 和 B 输入 BERT，使用`[CLS]`的embedding进行预测 B 是否 A 的下一句，这一任务**旨在训练模型理解预测句子间的关系。**

HuggingFace的transformers库提供了对**两个目标都进行预训练**、以及只对其中一个任务进行预训练的Bert预训练模型：

1. BertForPreTraining：进行MLM和NSP两个任务的预训练；
2. BertForMaskedLM：只进行 MLM 任务的预训练；
3. BertLMHeadModel：这个和上一个的区别在于，这一模型是**作为 decoder 运行的版本**；
4. BertForNextSentencePrediction：只进行 NSP 任务的预训练。

![img](https://ifwind.github.io/2021/08/24/BERT相关——（8）BERT-based Model代码分析/BertPreTraining相关.png)

```python
from transformers import BertTokenizer, BertForPreTraining
import torch
tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
model = BertForPreTraining.from_pretrained('bert-base-uncased')
inputs = tokenizer("Hello, my dog is cute", return_tensors="pt")
outputs = model(**inputs)
prediction_logits = outputs.prediction_logits
seq_relationship_logits = outputs.seq_relationship_logits
```

### `_keys_to_ignore_on_load_unexpected` `_keys_to_ignore_on_load_missing`



when you load you model。you can ignore the keys whether more or less by specfy those two variable





![img](https://ifwind.github.io/2021/08/24/BERT相关——（8）BERT-based Model代码分析/下游任务.png)



## torch.narrow torch.unbind

### torch.narrow()

PyTorch 中的narrow()函数起到了筛选一定维度上的数据作用。个人感觉与x[begin:end] 相同！

用法：`torch.narrow(input, dim, start, length) → Tensor`

返回输入张量的切片操作结果。 输入tensor和返回的tensor共享内存。



### torch.unbind()

`torch.unbind()`移除指定维后，返回一个元组，包含了沿着指定维切片后的各个切片。

参考官网：[torch.unbind()](https://pytorch.org/docs/master/generated/torch.unbind.html)

用法：`torch.unbind(input, dim=0) → seq`

返回指定维度切片后的元组。



### fune-tune with training api

```python
from datasets import load_dataset
from transformers import AutoTokenizer
from transformers import AutoModelForSequenceClassification
from transformers import TrainingArguments
from transformers import Trainer

import numpy as np
from datasets import load_metric

tokenizer = AutoTokenizer.from_pretrained("bert-base-cased")
model = AutoModelForSequenceClassification.from_pretrained("bert-base-cased", num_labels=2)

metric = load_metric("accuracy")
raw_datasets = load_dataset("imdb")

def tokenize_function(examples):
    return tokenizer(examples["text"], padding="max_length", truncation=True)

tokenized_datasets = raw_datasets.map(tokenize_function, batched=True)
small_train_dataset = tokenized_datasets["train"].shuffle(seed=42).select(range(1000))
small_eval_dataset = tokenized_datasets["test"].shuffle(seed=42).select(range(1000))

def compute_metrics(eval_pred):
    logits, labels = eval_pred
    predictions = np.argmax(logits, axis=-1)
    return metric.compute(predictions=predictions, references=labels)

# training
trainer = Trainer(
  model=model,
  args=training_args,
  train_dataset=small_train_dataset,
  eval_dataset=small_eval_dataset,
)

trainer.train()
trainer.evaluate()

```

### fine-tune with native pytorch

```python
from datasets import load_dataset
from transformers import AutoTokenizer
from transformers import AutoModelForSequenceClassification
from transformers import TrainingArguments
from transformers import Trainer
from torch.utils.data import DataLoader
from transformers import AutoModelForSequenceClassification
from transformers import AdamW
from transformers import get_scheduler

tokenizer = AutoTokenizer.from_pretrained("bert-base-cased")
model = AutoModelForSequenceClassification.from_pretrained("bert-base-cased", num_labels=2)
raw_datasets = load_dataset("imdb")

tokenized_datasets = tokenized_datasets.remove_columns(["text"])
tokenized_datasets = tokenized_datasets.rename_column("label", "labels")
tokenized_datasets.set_format("torch")

small_train_dataset = tokenized_datasets["train"].shuffle(seed=42).select(range(1000))
small_eval_dataset = tokenized_datasets["test"].shuffle(seed=42).select(range(1000))

train_dataloader = DataLoader(small_train_dataset, shuffle=True, batch_size=8)
eval_dataloader = DataLoader(small_eval_dataset, batch_size=8)
optimizer = AdamW(model.parameters(), lr=5e-5)


num_epochs = 3
num_training_steps = num_epochs * len(train_dataloader)
lr_scheduler = get_scheduler(
    "linear",
    optimizer=optimizer,
    num_warmup_steps=0,
    num_training_steps=num_training_steps
)

import torch

device = torch.device("cuda") if torch.cuda.is_available() else torch.device("cpu")
model.to(device)

from tqdm.auto import tqdm

progress_bar = tqdm(range(num_training_steps))

model.train()
for epoch in range(num_epochs):
    for batch in train_dataloader:
        batch = {k: v.to(device) for k, v in batch.items()}
        outputs = model(**batch)
        loss = outputs.loss
        loss.backward()

        optimizer.step()
        lr_scheduler.step()
        optimizer.zero_grad()
        progress_bar.update(1)


metric= load_metric("accuracy")
model.eval()
for batch in eval_dataloader:
    batch = {k: v.to(device) for k, v in batch.items()}
    with torch.no_grad():
        outputs = model(**batch)

    logits = outputs.logits
    predictions = torch.argmax(logits, dim=-1)
    metric.add_batch(predictions=predictions, references=batch["labels"])

metric.compute()
```
