---
lang: zh
title: 手搓一个文本分类器
layout: post
latex: true
github_repo: https://github.com/AzusaTsang/simple_text_classify
---

## 数据

来自Rotten Tomato的影评文本数据。其中训练集`data_rt.train`和测试集`data_rt.test`均包含了3554条影评，每条影评包含了文本和情感标签。示例如下：

```
+1 visually , 'santa clause 2' is wondrously creative .
```

其中，`+1` 表示这条影评蕴涵了正面感情，后面是影评的具体内容。

## 数据特征提取

特征模型采用词袋模型、$n$-gram 模型和词向量模型。

```python
def extractFeatures(
    x: str, 
    model_type: Literal['bow', 'ngram', 'wv'] = 'bow', 
    n: int = 3, 
    wv: dict[str, list[float]] = None
) -> Counter:
    """
    从字符串x中提取特征
    `x`: 输入字符串
    `model_type`: 模型类型，`'bow'`词袋模型，`'ngram'`n-gram模型，`'wv'`词向量模型。
    `n`: `model_type == 'ngram'` 时的 `n`。
    `wv`: `model_type == 'wv'` 时储存词向量的字典。
    """
    ls = x.strip().split()
    if model_type == 'bow':
        c = collections.Counter(ls)
    if model_type == 'ngram':
        c = collections.Counter()
        for i in range(n):
            for j in range(i, len(ls)):
                c[' '.join(ls[j - i: j + 1])] += 1
    if model_type == 'wv':
        c = collections.Counter()
        for word in ls:
            if word in wv:
                for i, v in enumerate(wv[word]):
                    c[i] += v
    return c
```

## 分类器

这里采用线性分类器来对句子 $x$ 提取出的特征 $\phi(x)$ 来进行分类判断，类别记为 $y\in\{-1,+1\}$。预测公式 $\hat y = \mathbf{w}\cdot\phi(x) + b$ 来进行计算，其中 $\hat y$ 为预测值。目标为使 Hinge 损失函数 $\mathcal{L}(x,y; \mathbf{w},b)=\max(0,1-\hat y y)$ 最小。利用梯度下降的方法来求 $\mathbf{w}$ 和 $b$ 使 $\mathcal{L}$ 最小化。

$$
\mathcal{L}(x,y; \mathbf{w},b) = \begin{cases}
            1- (\mathbf{w}\cdot\phi(x) + b) y  & , 1- \hat y y > 0 \\
            0 & , 1-\hat y y < 0
        \end{cases} 
$$

对 $\mathbf{w}$ 和 $b$ 进行求导


$$
    \nabla_{\mathbf{w}}\mathcal{L} = \begin{cases}
        -\phi(x)y & ,  1- \hat y y > 0 \\
        0 &,  1-\hat y y < 0
    \end{cases} 
$$


$$
    \nabla_{b}\mathcal{L} = \begin{cases}
        -y & ,  1- \hat y y > 0 \\
        0 &,  1-\hat y y < 0
    \end{cases}
$$

再根据给定的学习率 $\eta$ 得到相应的梯度下降公式

$$
\mathbf{w} \leftarrow \mathbf{w}-\eta\nabla_{\mathbf{w}}\mathcal{L} = 
\begin{cases}
    \mathbf{w}+\eta y\phi(x) \ & ,  1-\hat y y > 0 \\
    \mathbf{w} & ,  1-\hat y y < 0
\end{cases}
$$

$$
b \leftarrow b-\eta\nabla_{b}\mathcal{L} = 
\begin{cases}
    b+\eta y \ & ,  1-\hat y y > 0 \\
    b & ,  1-\hat y y < 0
\end{cases}
$$

```python
def dotProduct(d1: Counter, d2: Counter) -> float:
    if len(d1) < len(d2):
        return dotProduct(d2, d1)
    else:
        return sum(d1[f] * v for f, v in d2.items())

def increment(d1: Counter, scale: float, d2: Counter):
    for f, v in d2.items():
        d1[f] = d1[f] + v * scale

def learnPredictor(
    trainExamples: list[tuple[str, int]], 
    testExamples: list[tuple[str, int]], 
    featureExtractor: Callable[..., Counter], 
    numIters: int, 
    eta: float
) -> tuple[Counter, float]:
    '''
    给定训练数据和测试数据，特征提取器`featureExtractor`、训练轮数`numIters`和学习率`eta`，
    返回学习后的weights与bias
    '''
    weights = Counter()
    bias = 0.
    for epoch in range(numIters):
        X, y = random.choice(trainExamples)
        X = featureExtractor(X)
        y_pred = dotProduct(weights, X) + bias
        loss = 1 - y_pred * y
        if loss > 0:
            increment(weights, eta*y, X)
            bias += eta * y
    return weights, bias
```

## 训练

```python
# 词袋模型
featureExtractor = extractFeatures 

# n-gram 模型
featureExtractor = lambda x: extractFeatures(x, 'ngram') 

# 词向量模型
wv = dict[str, list[float]]() # 从网上下载或自己训练的词向量
featureExtractor = lambda x: extractFeatures(x, 'wv', wv = wv)

def readExamples(path:str, return_X_y: bool = False) -> Union[list[tuple[str, int]], tuple[list[str], list[int]]]:
    if return_X_y:
        X, y = [], []
    else:
        examples = []
    for line in open(path, encoding = 'utf8'):
        _y, _x = line.split(' ', 1)
        if return_X_y:
            X.append(_x.strip())
            y.append(int(_y))
        else:
            examples.append((_x.strip(), int(_y)))
    print(f'Read {len(X if return_X_y else examples)} examples from {path}')
    return (X, y) if return_X_y else examples

trainExamples = readExamples('data/data_rt.train')
testExamples = readExamples('data/data_rt.test')

weights, bias = learnPredictor(trainExamples, testExamples, featureExtractor, numIters=numIters, eta=eta)
```

## 检验 

```python
def evaluatePredictor(examples: list[tuple], predictor: Callable) -> float:
    '''
    在`examples`上测试`predictor`的性能，返回错误率
    '''
    error = 0
    with open('errors.txt', 'w+', encoding='utf8') as f:
        for x, y in examples:
            if predictor(x) != y:
                error += 1
                f.write(f'{y:+d} {x}\n')
    return error / len(examples)

trainError = evaluatePredictor(trainExamples, lambda x: (1 if dotProduct(featureExtractor(x), weights) + bias >= 0 else -1))
testError = evaluatePredictor(testExamples, lambda x: (1 if dotProduct(featureExtractor(x), weights) + bias >= 0 else -1))

print (f"train error = {trainError}, test error = {testError}")
```

## 效果

| 模型         | 训练集错误率 | 测试集错误率 |
| ------------ | ------------ | ------------ |
| 词袋模型     | 1.69%        | 26.13%       |
| Trigram 模型 | 0.00%        | 25.86%       |
| 词向量模型   | 19.0%       | 23.83%       |

错误判断的样本句子主要含有转折关系（如 it's fun, but a psychological mess, ...）或隐晦表达（如 it tells more than it shows.）。这些都明显无法通过仅使用简单的基于统计的特征提取与线性分类器来解决，只能通过更高级的算法来理解当中的含义来做出准确的判断。基于统计的特征与线性分类器确实可以在一定程度上实现文本分类功能，但仅限于直白表述、结构简单的语句。