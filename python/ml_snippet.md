### kaggle中导入其他文件
```python
### kaggle中导入其他文件

# import module we'11 need to import our custom module 
from shutil import copyfile
# copy our file into the working directory (make sure it has py suffix) 
copyfile(src "../input/function/network/gan.py",dst ="../working/gan.py")
# import all our functions 
from gan import
# we can now use this function!

```

### 使用sklearn绘制混淆矩阵

```python  
### 使用sklearn绘制混淆矩阵

from sklearn.metrics import ConfusionMatrixDisplay, confusion_matrix
def plot_confusion_matrix(y_preds, y_true, labels):
  cm = confusion_matrix(y_true, y_preds, normalize="true")
  fig, ax = plt.subplots(figsize=(6, 6))
  disp = ConfusionMatrixDisplay(confusion_matrix=cm, display_labels=labels)
  disp.plot(cmap="Blues", values_format=".2f", ax=ax, colorbar=False)
  plt.title("Normalized confusion matrix")
  plt.show()
y_preds = lr_clf.predict(X_valid)
plot_confusion_matrix(y_preds, y_valid, labels)

```
### weibo 100k微调实例
```python
# 加载微博100k数据集
weibo_dataset = load_dataset('csv', data_files='./weibo_senti_100k.csv')
# 加载话题评论
tokenizer = AutoTokenizer.from_pretrained("bert-base-chinese")

def tokenize_function(examples):
    return tokenizer(examples["review"], padding="max_length", truncation=True, max_length=128)


def tokenize_function_2(examples):
    return tokenizer(examples["content"] or "空", padding="max_length", truncation=True, max_length=128)


tokenized_datasets = weibo_dataset.map(tokenize_function, batched=True)
tokenized_datasets = tokenized_datasets["train"].shuffle(seed=42).train_test_split(0.05)
small_train_dataset = tokenized_datasets['train'].select(range(50))
small_eval_dataset = tokenized_datasets['test']

comment_dataset = load_dataset('csv', data_files='./uesd_comment.csv')
comment_dataset = comment_dataset.remove_columns(['mid', 'parent_id', 'weibo_id', 'comment_level', 'topic', 'faces'])
comment_dataset = comment_dataset.map(tokenize_function_2, batched=False)
comment_dataset = comment_dataset["train"]
comment_dataset = comment_dataset.add_column('label', np.zeros(len(comment_dataset), dtype=np.int))
# comment_dataset.rename

dataset = load_dataset("codyburker/yelp_review_sampled")
tokenizer = AutoTokenizer.from_pretrained("bert-base-cased")


def tokenize_function(examples):
    return tokenizer(examples["text"], padding="max_length", truncation=True)


tokenized_datasets = dataset.map(tokenize_function, batched=True)
small_train_dataset = tokenized_datasets["train"].shuffle(seed=42).select(range(1000))
small_eval_dataset = tokenized_datasets["test"].shuffle(seed=42).select(range(1000))
```

### imdb finetune实例
```python
from datasets import load_dataset

imdb = load_dataset("imdb")
from transformers import AutoTokenizer
tokenizer = AutoTokenizer.from_pretrained("bert-base-cased")

def preprocess_function(examples):
    return tokenizer(examples["text"], truncation=True)

tokenized_imdb = imdb.map(preprocess_function, batched=True)

from transformers import DataCollatorWithPadding
data_collator = DataCollatorWithPadding(tokenizer=tokenizer)

from transformers import AutoModelForSequenceClassification
model = AutoModelForSequenceClassification.from_pretrained("bert-base-cased", num_labels=2)


from transformers import TrainingArguments, Trainer


trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_imdb["train"],
    eval_dataset=tokenized_imdb["test"],
    tokenizer=tokenizer,
    data_collator=data_collator,
)

trainer.train()

```

